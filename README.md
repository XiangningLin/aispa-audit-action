# AISPA Audit Action

A GitHub Action for auditing AI system prompts against the **AISPA (AI System Prompt Auditing)** standard — like Snyk for security, but for LLM system prompt ethics.

## How It Works

1. Reads your `.aispa.yml` config to know where your prompts are
2. Extracts prompts from source code files (Python constants, JSON fields, etc.) or standalone prompt files
3. Calls the AISPA auditing API for each prompt
4. Posts results as a PR comment and fails the check if any prompt is non-compliant

**Two scan modes:**
- **Full scan** (`workflow_dispatch`): Audits ALL prompts — use for initial baseline or on-demand checks
- **Incremental scan** (`pull_request`): Only audits prompts in files modified by the PR. Skips entirely if no prompt files were changed

## Quick Start

### Step 1: Find your system prompts

System prompts in most projects are stored as:
- Python string constants (e.g. `SYSTEM_PROMPT = """..."""`)
- JSON/YAML config files with prompt fields
- Standalone `.txt`, `.md`, or `.prompt` files

Quick discovery command:
```bash
grep -rn "PROMPT\|TEMPLATE\|INSTRUCTION\|system_prompt\|You are" \
  --include="*.py" --include="*.ts" --include="*.json" \
  --include="*.yaml" --include="*.yml" .
```

### Step 2: Create `.aispa.yml`

Create a `.aispa.yml` file in your repo root that tells the action where your prompts are:

```yaml
# .aispa.yml
sources:
  # Standalone prompt files in a directory
  - path: "./prompts"
    type: directory

  # Python files with prompt string constants
  - path: "src/llm/prompts.py"
    type: python-constants

  # JSON files with prompt fields
  - path: "config/prompts/*.json"
    type: json-field
    fields: ["system_prompt", "context_prompt"]

  # YAML files with prompt fields
  - path: "config/agents/*.yaml"
    type: yaml-field
    fields: ["system_prompt"]
```

**Source types:**

| Type | Description | Options |
|------|-------------|---------|
| `directory` | Scans for `.txt`, `.md`, `.prompt`, `.system` files | — |
| `python-constants` | Extracts string constants matching a pattern from `.py` files | `pattern`: regex for variable names (default: `PROMPT\|TEMPLATE\|INSTRUCTION\|SYSTEM`) |
| `json-field` | Extracts specific fields from JSON files | `fields`: list of field names to extract |
| `yaml-field` | Extracts specific fields from YAML files | `fields`: list of field names to extract |

### Step 3: Add the workflow

```yaml
name: AISPA Prompt Audit

on:
  pull_request:         # Incremental: only audit changed prompt files
  workflow_dispatch:     # Manual trigger: full scan of all prompts

jobs:
  audit:
    runs-on: ubuntu-latest
    permissions:
      pull-requests: write

    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0    # Needed for git diff in PR mode

      - uses: XiangningLin/aispa-audit-action@v1
        with:
          threshold: 70
```

## Inputs

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `config-file` | No | `.aispa.yml` | Path to config file |
| `prompts-dir` | No | — | Fallback: directory with prompt files (if no `.aispa.yml`) |
| `threshold` | No | `0` | Minimum compliance rate (0-100) |
| `api-url` | No | Production URL | AISPA service base URL |
| `api-key` | No | — | API key for authentication |
| `model` | No | — | LLM model for auditing |
| `comment-on-pr` | No | `true` | Post audit results as a PR comment |

## Outputs

| Output | Description |
|--------|-------------|
| `status` | `PASSED`, `FAILED`, or `SKIPPED` |

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

## Examples

### Dify (prompts in Python files)

```yaml
# .aispa.yml
sources:
  - path: "api/core/llm_generator/prompts.py"
    type: python-constants
  - path: "api/core/agent/prompt/template.py"
    type: python-constants
  - path: "api/core/prompt/prompt_templates/*.json"
    type: json-field
    fields: ["context_prompt", "query_prompt"]
```

### Project with standalone prompt files

```yaml
# .aispa.yml
sources:
  - path: "./prompts"
    type: directory
```

### Mixed sources

```yaml
# .aispa.yml
sources:
  - path: "./prompts"
    type: directory
  - path: "src/**/*.py"
    type: python-constants
    pattern: "SYSTEM_PROMPT|AGENT_PROMPT"
  - path: "config/agents.json"
    type: json-field
    fields: ["system_message"]
```

## License

MIT
