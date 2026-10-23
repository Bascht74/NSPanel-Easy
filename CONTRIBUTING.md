# Contributing to NSPanel Easy

Thank you for your interest in contributing to NSPanel Easy.
This document covers everything you need to know before opening a pull request.

---

## Table of Contents

- [PR Title Format](#pr-title-format)
- [Commit Messages](#commit-messages)
- [Code Style](#code-style)
- [Branching](#branching)
- [How AI Is Used in This Project](#how-ai-is-used-in-this-project)

---

## PR Title Format

Every pull request title **must** follow this format:

```text
<prefix>: <Description starting with a capital letter>
```

- The prefix is **always lowercase**.
- A colon and a single space separate the prefix from the description.
- The description starts with a **capital letter**.
- **No trailing period.**

### Allowed Prefixes

| Prefix | When to use |
| --- | --- |
| `fix:` | Something was broken and is now corrected |
| `feat:` | A new user-facing feature or capability |
| `improve:` | An enhancement to existing behavior that is neither a bug fix nor a new feature |
| `ci:` | Changes limited to GitHub Actions workflows or CI scripts |
| `docs:` | Documentation-only changes |
| `style:` | Cosmetic or formatting changes with no logic impact |
| `build:` | Build system, dependencies, or packaging |
| `refactor:` | Internal restructuring with no behavior change |
| `test:` | Test additions or corrections only |
| `chore:` | Maintenance tasks that do not fit any other category |

### Examples

```text
fix: Correct baud rate negotiation timeout on EU model
feat: Add per-page brightness control
improve: Reduce boot time by deferring TFT version check
ci: Add PR title validation workflow
docs: Document release prefix conventions
```

### Why This Matters

PR titles become GitHub Release titles verbatim. Users reading the release history
use the prefix to understand at a glance whether an update is a bug fix, a new
feature, or an internal change. Automated pipelines built by community members
also parse these prefixes to classify releases — a non-conforming title will
break those integrations.

A required CI check enforces this format on every PR. The check runs on
`.github/workflows/validate_pr_title.yml`.

---

## Commit Messages

Individual commit messages within a PR follow the same prefix convention and
format as PR titles. Squash commits inherit the PR title, so keeping them
consistent is the most important thing.

---

## Code Style

- **C++**: 2-space indentation; formatted with `clang-format` (config at `.clang-format`).
  The CI workflow applies and auto-commits fixes.
- **YAML**: Validated with `yamllint` (config at `.rules/yamllint.yml`).
- **Markdown**: Validated with `markdownlint-cli2` (config at `.rules/.markdownlint.jsonc`).
  The CI workflow applies and auto-commits fixes.

All linters run automatically on pull requests. You do not need to run them locally,
but doing so before pushing will save you a round trip.

---

## Branching

Work on a feature branch and open a PR targeting `main`. There is no separate
`dev` branch in the current workflow. Keep branches short-lived and scoped to
a single logical change.

---

## How AI Is Used in This Project

AI tools (mainly Claude by Anthropic) assist with maintaining this project.
This section describes how, so you know what to expect.

### What Is Automated

- **Pull request reviews:** [Sourcery](https://sourcery.ai) and [CodeRabbit](https://coderabbit.ai)
  automatically review pull requests. Their comments are suggestions; they do not approve or merge anything.

Nothing else is triggered automatically. No bot replies to issues or discussions on its own.

### What Is AI-Assisted

- Drafting answers to issues and discussions
- Writing or refactoring code, documentation, and tests
- Investigating bugs and reviewing logs

In all cases, the work is started manually by the maintainer. Each answer is reviewed, and often edited,
before it is posted. Each code change is reviewed and tested before it is merged.

Posts made through the AI integration always appear under the
[panel-easy-assistant](https://github.com/apps/panel-easy-assistant) bot account
(shown as `panel-easy-assistant[bot]`), never under my personal account ([@edwardtfn](https://github.com/edwardtfn)).
The bot is a GitHub App owned and operated by me, and it only acts when I trigger it.

Comments under my personal account are my own, though I sometimes use AI to help draft them before posting.

I still personally read every issue, discussion, and pull request, and I remain responsible for everything
posted or merged here.

### Your Data

Content you post publicly here (text, logs, configuration) may be shared with an AI provider when I ask for help
analyzing it. Never post secrets such as API keys, passwords, Wi-Fi credentials, or tokens.
Redact them before sharing logs or YAML.

### Contributions Using AI

AI-assisted contributions are welcome. You are expected to understand, test, and stand behind the code you submit.
