---
name: ambrogio
description: Run AI-powered reliability scenarios from your terminal or coding agent. Use when building, testing, or verifying features against a live app with Ambrogio.
allowed-tools: Bash(ambrogio:*)
---

# Ambrogio

Run AI-powered reliability scenarios from your terminal or coding agent.

[Ambrogio](https://ambrogio.tech) is the product reliability layer: define scenarios in natural language, and Ambrogio runs them against your app using real browsers in the cloud. This skill lets you trigger and verify scenarios directly from your coding agent.

## Install the CLI

```bash
npm install -g ambrogio
```

Or run directly with `npx ambrogio`.

## Setup

```bash
ambrogio set-api-key <your-api-key>    # get from app.ambrogio.tech > CI/CD
ambrogio set-product <product-id>       # your product ID from the dashboard
```

Both values can also be set via environment variables:

```bash
export AMBROGIO_API_KEY=ambra_xxx
export AMBROGIO_PRODUCT_ID=42
```

Priority: flags > env vars > config file (`~/.ambrogio/config.json`).

## Core commands

### Run existing scenarios (regression)

```bash
# Against a deployed URL
ambrogio run --url https://staging.example.com

# Against localhost (auto-creates a Cloudflare tunnel)
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

After making changes, run the regression suite and verify new features:

```bash
# 1. Run existing regression suite against your local dev server
ambrogio run --local 3000

# 2. If you built a new feature, verify it
ambrogio verify --intent "User can add item to cart and checkout" --local 3000
```

`run` executes existing scenarios (regression). `verify` creates a new scenario from intent and runs it (verification). Together they form the full loop: build, verify, review, regress.

The JSON output and exit code give you a clear signal to continue or fix.

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
