---
name: kaggle
description: "Unified Kaggle skill. Use when the user mentions kaggle, kaggle.com, Kaggle competitions, datasets, models, notebooks, GPUs, TPUs, badges, or anything Kaggle-related. Handles account setup, competition reports, dataset/model downloads, notebook execution, competition submissions, badge collection, and general Kaggle questions."
license: MIT
compatibility: "Python 3.9+, pip packages kagglehub, kaggle, requests, python-dotenv. Optional: playwright for browser badges. Playwright MCP tools for competition reports."
homepage: https://github.com/shepsci/kaggle-skill
metadata: {"author": "shepsci", "version": "1.0.1", "primaryEnv": "KAGGLE_KEY", "openclaw": {"requires": {"bins": ["python3", "pip3"], "env": ["KAGGLE_USERNAME", "KAGGLE_KEY", "KAGGLE_API_TOKEN"]}}}
allowed-tools: Bash Read WebFetch
---

# Kaggle — Unified Skill

Complete Kaggle integration for any LLM or agentic coding system (Claude Code,
gemini-cli, Cursor, etc.): account setup, competition reports, dataset/model
downloads, notebook execution, competition submissions, badge collection, and
general Kaggle questions. Four integrated modules working together.

> **Overlap guard:** For hackathon grading evaluation and alignment analysis,
> use the **kaggle-hackathon-grading** skill instead.

**Network requirements:** outbound HTTPS to `api.kaggle.com`, `www.kaggle.com`,
and `storage.googleapis.com`.

## Modules

| Module | Purpose |
|--------|---------|
| **registration** | Account creation, API key generation, credential storage |
| **comp-report** | Competition landscape reports with Playwright scraping |
| **kllm** | Core Kaggle interaction (kagglehub, CLI, MCP, UI) |
| **badge-collector** | Systematic badge earning across 5 phases |

## Credential Setup

**Always run the credential checker first:**

```bash
python3 skills/kaggle/shared/check_all_credentials.py
```

Three credential types are needed for full compatibility:

| Variable | Format | Purpose |
|----------|--------|---------|
| `KAGGLE_USERNAME` | Kaggle handle | Identity for all tools |
| `KAGGLE_KEY` | 32-char hex | Legacy key (CLI, kagglehub, most MCP) |
| `KAGGLE_API_TOKEN` | `KGAT_`-prefixed | Scoped token (some MCP endpoints) |

If any are missing, follow the registration walkthrough:
`Read modules/registration/README.md` for the full step-by-step guide.

**Security:** Never echo, log, or commit actual credential values.

## Module: Registration

Walks users through creating a Kaggle account and generating all three API
credentials. Saves to `.env` and `~/.kaggle/kaggle.json`.

Key commands:
```bash
python3 skills/kaggle/modules/registration/scripts/check_registration.py
bash skills/kaggle/modules/registration/scripts/setup_env.sh
```

`Read modules/registration/README.md` for the complete walkthrough.

## Module: Competition Reports

Generates comprehensive landscape reports of recent Kaggle competition activity.
Uses Python API for metadata + Playwright MCP tools for SPA content.

6-step workflow:
1. Verify credentials
2. Gather competition list across all categories
3. Get structured details per competition (files, leaderboard, kernels)
4. Scrape problem statements, evaluation metrics, writeups via Playwright
5. Compose markdown report with Methods & Insights analysis
6. Present inline

```bash
python3 skills/kaggle/modules/comp-report/scripts/list_competitions.py --lookback-days 30 --output json
python3 skills/kaggle/modules/comp-report/scripts/competition_details.py --slug SLUG
```

`Read modules/comp-report/README.md` for full details including hackathon handling.

## Module: Kaggle Interaction (kllm)

Four methods to interact with kaggle.com:

| Method | Best For |
|--------|----------|
| **kagglehub** | Quick dataset/model download in Python |
| **kaggle-cli** | Full workflow scripting |
| **MCP Server** | AI agent integration |
| **Kaggle UI** | Account setup, verification |

Capability matrix:

| Task | kagglehub | kaggle-cli | MCP | UI |
|------|-----------|------------|-----|-----|
| Download dataset | `dataset_download()` | `datasets download` | Yes | Yes |
| Download model | `model_download()` | `models instances versions download` | Yes | Yes |
| Execute notebook | — | `kernels push/status/output` | Yes | Yes |
| Submit to competition | — | `competitions submit` | Yes | Yes |
| Publish dataset | `dataset_upload()` | `datasets create` | Yes | Yes |
| Publish model | `model_upload()` | `models create` | Yes | Yes |

**Known issues:**
- `dataset_load()` broken in kagglehub v0.4.3 — use `dataset_download()` + `pd.read_csv()`
- `competitions download` has no `--unzip` in CLI >= 1.8
- Competition-linked datasets return 403 — use standalone copies

`Read modules/kllm/README.md` for full details and all task workflows.

## Module: Badge Collector

Systematically earns ~38 automatable Kaggle badges across 5 phases:

| Phase | Name | Badges | Time |
|-------|------|--------|------|
| 1 | Instant API | ~16 | 5-10 min |
| 2 | Competition | ~7 | 10-15 min |
| 3 | Pipeline | ~3 | 15-30 min |
| 4 | Browser | ~8 | 5-10 min |
| 5 | Streaks | ~4 | Setup only |

```bash
python3 skills/kaggle/modules/badge-collector/scripts/orchestrator.py --dry-run
python3 skills/kaggle/modules/badge-collector/scripts/orchestrator.py --phase 1
python3 skills/kaggle/modules/badge-collector/scripts/orchestrator.py --status
```

`Read modules/badge-collector/README.md` for full details.

## Orchestration Workflow

This skill is primarily a **reference** — use the modules and scripts as needed
based on the user's request. When explicitly asked to run the **full Kaggle
workflow**, follow these steps:

### Step 1: Check Credentials

```bash
python3 skills/kaggle/shared/check_all_credentials.py
```

If any credentials are missing, walk through the registration module. **Never
echo or log actual credential values.**

### Step 2: Generate Competition Landscape Report

Run the comp-report workflow: list competitions, get details, scrape with
Playwright, compose report. Output inline.

### Step 3: Summarize Kaggle Interaction Methods

Present a concise summary of the four ways to interact with Kaggle (kagglehub,
kaggle-cli, MCP Server, UI) with the capability matrix from the kllm module.

### Step 4: Present Interactive Menu

Ask the user what they'd like to do next:

- **Earn Kaggle badges** — Run the badge collector (5 phases, ~38 automatable badges)
- **Explore recent competitions** — Dive deeper into specific competitions from the report
- **Enter a Kaggle competition** — Register, download data, build a submission, submit
- **Download a Kaggle dataset** — Search for and download any public dataset
- **Download a Kaggle model** — Download pre-trained models (LLMs, CV, etc.)
- **Run a notebook on Kaggle** — Push and execute a notebook on KKB with free GPU/TPU
- **Publish to Kaggle** — Upload a dataset, model, or notebook
- **Learn about Kaggle progression** — Tiers, medals, how to rank up
- **Something else** — Free-form Kaggle help

### Step 5: Execute and Continue

Handle the user's choice using the appropriate module, then loop back to offer
more options.

## Security

- **Never** commit `.env`, `kaggle.json`, or any credential files
- **Never** echo or log actual credential values in terminal output
- The `.gitignore` excludes `.env`, `kaggle.json`, and related files
- Set file permissions: `chmod 600 .env ~/.kaggle/kaggle.json`
- If credentials are accidentally exposed, rotate them immediately at
  [https://www.kaggle.com/settings](https://www.kaggle.com/settings)

## Scope of Operations

This skill performs both read-only and write operations on kaggle.com.

**Read-only operations** (no account side-effects):
- List/search competitions, datasets, models, notebooks
- Download datasets, models, competition data
- View leaderboards, competition details, badge progress
- Generate competition landscape reports

**Write operations** (create or modify resources on your account):
- Create/publish datasets, notebooks, models (always private by default)
- Submit predictions to competitions
- Push and execute notebooks on Kaggle Kernel Backend (KKB)
- Earn badges through API activity (profile-visible)

**Phase 5 (Streaks)** generates a local shell script for daily execution but
does **not** auto-install cron jobs or launchd plists. Users must manually
configure scheduling if desired.

## Scripts Index

**Shared:**
- `shared/check_all_credentials.py` — Unified credential checker (all 3 types)

**Registration:**
- `modules/registration/scripts/check_registration.py` — Check all 3 credentials
- `modules/registration/scripts/setup_env.sh` — Auto-configure credentials from env/dotenv

**Competition Reports:**
- `modules/comp-report/scripts/utils.py` — Credential check, API init, rate limiting
- `modules/comp-report/scripts/list_competitions.py` — Fetch competitions across categories
- `modules/comp-report/scripts/competition_details.py` — Files, leaderboard, kernels per competition

**Kaggle Interaction (kllm):**
- `modules/kllm/scripts/setup_env.sh` — Auto-configure credentials (with .env loading)
- `modules/kllm/scripts/check_credentials.py` — Verify and auto-map credentials
- `modules/kllm/scripts/network_check.sh` — Check Kaggle API reachability
- `modules/kllm/scripts/cli_download.sh` — Download datasets/models via CLI
- `modules/kllm/scripts/cli_execute.sh` — Execute notebook on KKB
- `modules/kllm/scripts/cli_competition.sh` — Competition workflow (list/download/submit)
- `modules/kllm/scripts/cli_publish.sh` — Publish datasets/notebooks/models
- `modules/kllm/scripts/poll_kernel.sh` — Poll kernel status and download output
- `modules/kllm/scripts/kagglehub_download.py` — Download via kagglehub
- `modules/kllm/scripts/kagglehub_publish.py` — Publish via kagglehub

**Badge Collector:**
- `modules/badge-collector/scripts/orchestrator.py` — Main entry point
- `modules/badge-collector/scripts/badge_registry.py` — 59 badge definitions
- `modules/badge-collector/scripts/badge_tracker.py` — Progress persistence
- `modules/badge-collector/scripts/utils.py` — Shared utilities
- `modules/badge-collector/scripts/phase_1_instant_api.py` — Instant API badges
- `modules/badge-collector/scripts/phase_2_competition.py` — Competition badges
- `modules/badge-collector/scripts/phase_3_pipeline.py` — Pipeline badges
- `modules/badge-collector/scripts/phase_4_browser.py` — Browser badges
- `modules/badge-collector/scripts/phase_5_streaks.py` — Streak automation

## References Index

- `modules/registration/references/kaggle-setup.md` — Full credential setup guide with troubleshooting
- `modules/comp-report/references/competition-categories.md` — Competition types and API mapping
- `modules/kllm/references/kaggle-knowledge.md` — Comprehensive Kaggle platform knowledge
- `modules/kllm/references/kagglehub-reference.md` — Full kagglehub Python API reference
- `modules/kllm/references/cli-reference.md` — Complete kaggle-cli command reference
- `modules/kllm/references/mcp-reference.md` — Kaggle MCP server reference
- `modules/badge-collector/references/badge-catalog.md` — Complete 59-badge catalog
