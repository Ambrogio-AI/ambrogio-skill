---
name: ambrogio
description: Run AI-powered scenarios from your terminal or coding agent. Use when building, testing, or verifying features against a live app with Ambrogio.
allowed-tools: Bash(ambrogio:*)
---

# Ambrogio

Run AI-powered scenarios from your terminal or coding agent.

[Ambrogio](https://ambrogio.tech) is the product reliability layer: define scenarios in natural language, and Ambrogio runs them against your app using real browsers in the cloud. This skill lets you trigger and verify scenarios directly from your coding agent.

## Install the CLI

```bash
npm install -g ambrogio
```

Or run directly with `npx ambrogio`.

## Setup

```bash
ambrogio set-api-key <your-api-key>    # get from app.ambrogio.tech > CI/CD
ambrogio select-product                 # picks your product (auto-selects if only one)
```

Both values can also be set via environment variables:

```bash
export AMBROGIO_API_KEY=ambra_xxx
export AMBROGIO_PRODUCT_ID=42
```

Priority: flags > env vars > config file (`~/.ambrogio/config.json`).

## Core commands

### List existing scenarios

Before writing new scenarios or running the full suite, check what coverage already exists:

```bash
ambrogio list                        # active + draft + planned (default)
ambrogio list --section checkout     # only one section
ambrogio list --status draft         # only drafts awaiting review
ambrogio list --json                 # machine-readable (default in non-TTY)
```

Use this to find scenarios that touch the area you just changed, then re-run only those for a fast smoke check before running the whole suite.

### Run existing scenarios (regression)

```bash
# Against a deployed URL
ambrogio run --url https://staging.example.com

# Against localhost
ambrogio run --local 3000
```

### Verify a new feature

Describe what the feature should do and Ambrogio verifies it in a real browser. The scenario is saved as a draft for human review before entering the regression suite.

```bash
ambrogio verify --intent "User can add item to cart and complete checkout" --local 3000
```

### Filter by section or scenario

```bash
ambrogio run --section checkout --url https://staging.example.com
ambrogio run --scenario 12 --scenario 34 --url https://staging.example.com
```

### Override credentials at runtime

```bash
ambrogio run --url https://staging.example.com \
  --cred "USERNAME=user@dev.com" \
  --cred "PASSWORD=secret123"
```

## Output

- **Terminal**: pretty output with spinners and pass/fail indicators
- **Piped or CI/agent context**: structured JSON to stdout
- **Exit codes**: `0` if all scenarios pass, `1` if any fail

## Workflow for coding agents

**Use Ambrogio every time you finish a change.** You are the user's last line of defense against regressions — don't ship code without proving it still works in a real browser. The loop below is cheap (minutes) and should be the default, not an opt-in.

### After any code change

```bash
# 1. List existing scenarios to find what covers the area you changed
ambrogio list --json

# 2. Smoke-test the affected scenarios against your local dev server
#    (cheaper than the full suite, faster feedback)
ambrogio run --local 3000 --scenario 12 --scenario 34
#    or, if the area maps to a section:
ambrogio run --local 3000 --section checkout

# 3. If you built genuinely new behavior, verify it and save as a draft scenario
ambrogio verify --local 3000 \
  --intent "User can add item to cart and complete checkout with test card"

# 4. Before declaring the task done, run the full suite
ambrogio run --local 3000
```

### Rules of thumb

- **Always list first.** `ambrogio list` tells you what coverage already exists so you don't create duplicates with `verify`. Match the area you changed (section, intent) against existing scenarios before writing anything new.
- **Smoke test on affected scenarios first.** If you only touched checkout, run checkout scenarios (`--section checkout` or `--scenario <id>`). Iterate until they pass before widening to the full suite.
- **Verify every new feature.** If your change exposes behavior not covered by any existing scenario, end with `ambrogio verify` so the suite grows with the product.
- **Full run before you report done.** Regression hides in unexpected places. The final gate is `ambrogio run --local 3000` (or against a preview URL) with exit code `0`.
- **Use exit codes.** `0` = all passed, `1` = failure. Treat a non-zero exit as a blocker and fix before handing back to the user.

`list` tells you what's covered. `run` proves it still works. `verify` grows the suite. Together: build → list → smoke → verify → full regression.

## Run options

| Flag | Description |
|------|-------------|
| `--product, -p <id>` | Product ID |
| `--url <base_url>` | Base URL to run against |
| `--local <port>` | Run against localhost:port (auto-tunnel) |
| `--cred KEY=VALUE` | Override a credential (repeatable) |
| `--api-key <key>` | API key override |
| `--section <name>` | Run scenarios in a specific section |
| `--scenario <id>` | Run a specific scenario ID (repeatable) |

## List options

| Flag | Description |
|------|-------------|
| `--product, -p <id>` | Product ID |
| `--status <name>` | `active`, `draft`, `planned`, `creating`, `failed`, or `all`. Default: `active,draft,planned`. Comma-separated list accepted. |
| `--section <name>` | Filter by section |
| `--source <name>` | Filter by source (e.g. `agent`, `user`) |
| `--json` | Force JSON output (default in non-TTY) |
| `--api-key <key>` | API key override |

## Verify options

| Flag | Description |
|------|-------------|
| `--product, -p <id>` | Product ID |
| `--intent, -i <text>` | What to verify (natural language) |
| `--url <base_url>` | Base URL to run against |
| `--local <port>` | Run against localhost:port |
| `--cred KEY=VALUE` | Override a credential (repeatable) |
| `--api-key <key>` | API key override |

## Links

- [Website](https://ambrogio.tech)
- [Dashboard](https://app.ambrogio.tech)
- [npm package](https://www.npmjs.com/package/ambrogio)
