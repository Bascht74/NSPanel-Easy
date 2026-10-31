# Network and Internet Access

This page explains which network connections an NSPanel running NSPanel Easy makes, why the panel may need internet
access, what is exchanged, what happens if that access is blocked, and which firewall rules allow it.

## Summary

- The panel needs internet access for a single purpose: downloading the display (TFT) file from GitHub.
- No telemetry, analytics, cloud service, or "phone home" exists. No information about your home, Home Assistant, or
  entities is sent anywhere outside your network.
- Everything else (Home Assistant communication, time, firmware updates over the air) stays in your local network.
- Blocking internet access is supported, as long as TFT files are served from a local server or the panel is allowed
  out while a TFT update is running (see [Running without internet access](#running-without-internet-access)).

## Why the panel accesses the internet

The NSPanel has two processors: the ESP32, running the ESPHome firmware, and the Nextion display, running the TFT file.
Both must be on compatible versions. The firmware is installed from your ESPHome Device Builder over the local network,
but the TFT file is downloaded by the panel itself and transferred to the display through the
[Upload TFT add-on](addon_upload_tft.md), which is part of the default installation.

By default, the TFT file is downloaded from the project repository on GitHub:

```text
https://raw.githubusercontent.com/edwardtfn/NSPanel-Easy/v<version>/hmi/nspanel_<model>.tft
```

Where `<version>` is the firmware version and `<model>` is `landscape`, `portrait` or `blank`, according to the model
selected in the **Update TFT - Display Model** setting.

The download happens only when one of the following occurs:

- **Automatic update:** about 65 seconds after boot, when the TFT on the display is older than the version required by
  the firmware. This typically happens once, right after a firmware update. It can be disabled with
  `upload_tft_automatically: false`.
- **Manual update:** when the **Update TFT display** button is pressed in Home Assistant.
- **Action call:** when the `upload_tft` action is called (only available when `include_action_upload_tft: true`).

Outside of a TFT update, the panel does not connect to the internet.

## What is exchanged

During a TFT update, the panel sends standard HTTPS requests (`HEAD` and `GET`) to the URL above, downloading the file in
chunks with HTTP `Range` headers. The requests contain only the file URL and generic HTTP headers. As with any web
request, the server can see your public IP address and, from the URL, the firmware version and display model.

No credentials, device identifiers, Home Assistant data, or user settings are sent. Nothing is uploaded.

Each download is about 0.7 MB for `blank` and 13.5 MB for `landscape` and `portrait`. Failed attempts are downloaded
again.

## Local network traffic

These connections stay in your local network and are not affected by blocking internet access:

| Purpose | Direction | Protocol / Port |
| :- | :- | :- |
| Home Assistant API (entities, actions, Blueprint, time) | Home Assistant to panel | TCP 6053 |
| Firmware updates over the air (OTA) | ESPHome Device Builder to panel | TCP 3232 |
| Device discovery (mDNS) | Both directions | UDP 5353 |
| Network setup (DHCP) and name resolution (DNS) | Panel to your router / DNS server | UDP 67/68, UDP/TCP 53 |

The panel gets its time from Home Assistant through the API, so no NTP server on the internet is required.

## Firewall rules

To allow TFT downloads from GitHub, allow the panel to reach:

| Destination | Protocol / Port | Purpose |
| :- | :- | :- |
| Your DNS server | UDP/TCP 53 | Resolve `raw.githubusercontent.com` |
| `raw.githubusercontent.com` | TCP 443 (HTTPS) | Download the TFT file |

> [!TIP]
> Create the rule based on the host name (FQDN) whenever your firewall supports it. `raw.githubusercontent.com` is
> served by a content delivery network (CDN) whose IP addresses are shared with other services and may change without
> notice, so rules based on IP addresses can break at any time.

If you change `nextion_update_base_url` or `nextion_update_url`, allow the host and port of that URL instead. When the
TFT files are served from a local server, no internet access is required at all.

## Consequences of blocking internet access

Blocking internet access does not affect the daily use of the panel: the Home Assistant connection, pages, buttons,
relays, the embedded thermostat, and firmware updates keep working.

The only affected feature is the TFT download. Whenever a TFT update is required (usually after a firmware update) and
the download fails, the panel:

1. Shows the TFT upload screen.
2. Retries the download several times, restarting the display halfway through.
3. Restarts the panel when all attempts fail.

As the TFT is still outdated after the restart, the automatic update starts again on the next boot, so the cycle
repeats until the download succeeds. In the meantime, the panel is mostly unusable.

When automatic updates are disabled (`upload_tft_automatically: false`), the panel does not try to download the file,
but it keeps running with an outdated TFT, which may cause missing or broken features until the TFT is updated.

## Running without internet access

Choose one of the options below:

- **Allow access only during updates:** keep the panel blocked and open the firewall rules above only while a firmware
  update (and the following TFT update) is running.
- **Serve the TFT files locally:** download the TFT files matching your firmware version from the
  [`hmi`](https://github.com/edwardtfn/NSPanel-Easy/tree/latest/hmi) folder, copy them to a local web server (for example, the `www` folder of Home Assistant, served
  as `/local/`), and set `nextion_update_base_url` to that folder. The model and version selection keep working, but
  the files must be updated on every release. See
  [Using a local base URL](addon_upload_tft.md#example-using-a-local-base-url-with-automatic-model-selection).

## Connections outside the panel

Installing and updating NSPanel Easy also requires internet access on other devices. These are not made by the panel
and are not covered by the rules above:

- **ESPHome Device Builder:** downloads the firmware packages from `github.com` (the `remote_package` in your panel YAML,
  refreshed every 5 minutes by default) and the build tools and libraries used to compile the firmware.
- **Home Assistant:** downloads the Blueprint from `raw.githubusercontent.com` when it is imported or re-imported.
