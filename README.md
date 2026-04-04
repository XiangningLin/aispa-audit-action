# AISPA Audit Action

A GitHub Action for auditing AI system prompts against the **AISPA (AI System Prompt Auditing)** standard — like Snyk for security, but for LLM system prompt ethics.

## Usage

```yaml
name: AISPA Prompt Audit

on:
  push:
    branches: [main]
    paths: ["prompts/**"]
  pull_request:
    paths: ["prompts/**"]

jobs:
  audit:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: XiangningLin/aispa-audit-action@v1
        with:
          prompts-dir: ./prompts
          threshold: 70
```

## Inputs

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `prompts-dir` | Yes | `./prompts` | Path to directory containing prompt files |
| `threshold` | No | `0` | Minimum compliance rate (0-100) |
| `api-url` | No | Production URL | AISPA service base URL |
| `api-key` | No | - | API key for authentication |
| `model` | No | - | LLM model for auditing |
| `output-format` | No | `table` | Output format: `table` or `json` |
| `version` | No | `latest` | aispa-audit CLI version |

## Outputs

| Output | Description |
|--------|-------------|
| `result` | JSON audit result (when output-format is json) |
| `status` | Overall status: `PASSED` or `FAILED` |

## How It Works

1. Sets up Python 3.12
2. Installs `aispa-audit` CLI from PyPI
3. Scans all prompt files (`.txt`, `.md`, `.prompt`, `.system`) in the specified directory
4. Calls the AISPA auditing API for each prompt
5. Evaluates results against the threshold
6. Fails the CI check if any prompt doesn't meet the standard

## AISPA Dimensions

Each prompt is evaluated across 8 ethical dimensions:

| ID | Dimension |
|----|-----------|
| D1 | Identity Transparency |
| D2 | Truthfulness & Information Integrity |
| D3 | Privacy & Data Protection |
| D4 | Tool/Action Safety |
| D5 | User Agency & Manipulation Prevention |
| D6 | Unsafe Request Handling |
| D7 | Harm Prevention & User Safety |
| D8 | Fairness, Inclusion & Neutrality |

## License

MIT
