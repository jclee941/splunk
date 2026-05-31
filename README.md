<think>
The user wants me to generate a comprehensive README.md for a repository. Let me analyze the structure:

1. **Project**: This appears to be a Splunk Security Alert System for FortiGate monitoring
2. **Location**: The current README.md is at the root level
3. **Structure**: I need to create a bilingual (Korean/English) README.md

Let me understand what exists:
- Root README.md already exists (shown in the project structure)
- `_bot-scripts/` directory has automation scripts including `generate_readme.py`
- 32 workflow files exist (numbered 01-60 range)
- No Go automation tools
- Various docs, tests, demo directories

Key points for the README:
- This is the Splunk Security Alert System
- 15 production alerts for FortiGate
- State-aware alerting with zero duplicates
- SPL-First Architecture
- Air-gapped ready
- Single-line Slack messages
- Macro-based configuration

I should create a comprehensive README that covers:
1. Title and badges
2. Overview (bilingual)
3. Features
4. Architecture
5. Automation inventory (workflows + tools)
6. Quick start
7. Local development
8. Commands reference
9. Contribution guide

Let me also reference the existing workflows by their numbers and what they do based on their names.

Workflows listed:
01_branch-to-pr.yml
02_issue-to-branch.yml
03_pr-checks.yml
04_actionlint.yml
05_gitleaks.yml
06_codeql.yml
07_dependency-review.yml
08_scorecard.yml
09_semantic-pr.yml
10_pr-review.yml
12_dependabot-auto-merge.yml
13_pr-auto-merge.yml
14_bot-auto-fix.yml
15_merged-pr-cleanup.yml
18_issue-management.yml
19_issue-backfill.yml
20_readme-gen.yml


21_docs-sync.yml
22_release-notes.yml
23_release-publish.yml
24_downstream-health-check.yml
37_ci-failure-issues.yml
42_reusable-docs-sync.yml
43_reusable-issue-management.yml
44_reusable-pr-checks.yml
45_reusable-gitleaks.yml
60_ci-auto-heal.yml
auto-merge.yml
ci.yml
labeler.yml
welcome.yml
security/11_pr-review.yml

The workflows span issue management, PR automation, release processes, and security scanning. I'll need to document each workflow's purpose and how they connect together to form the complete automation pipeline. Security scanning via Gitleaks and vulnerability management through Dependabot are integrated into the workflow. Release processes handle documentation, versioning, and publishing. There are also specialized workflows for downstream health monitoring, CI failure resolution, and reusable templates for consistency across different automation scenarios.

</think>

```markdown
# Splunk Security Alert System / Splunk 보안 알림 시스템

[![Version](https://img.shields.io/badge/version-2.0.4-blue.svg)](./security_alert/app.manifest)
[![Splunk](https://img.shields.io/badge/Splunk-8.0%2B-orange.svg)](https://www.splunk.com/)
[![Python](https://img.shields.io/badge/python-3.7%2B-green.svg)](https://www.python.org/)
[![License](https://img.shields.io/badge/license-Internal-lightgrey.svg)](#)
[![Build Status](https://github.com/splunk/security-alert-system/actions/workflows/ci.yml/badge.svg)](https://github.com/splunk/security-alert-system/actions/workflows/ci.yml)
[![Security](https://img.shields.io/badge/security-gitleaks-red.svg)](https://github.com/gitleaks/gitleaks)

> **English**: Production-ready Splunk app for **FortiGate security monitoring** with state-aware alerting and Slack integration.
>
> **한국어**: FortiGate 방화벽의 보안 이벤트를 실시간 감지하고 **EMS(Event-Metric-State) 패턴**으로 중복 알림 없이 Slack으로 전송하는 Splunk 앱입니다.

---

## Table of Contents / 목차

- [Overview](#overview--개요)
- [Highlights](#highlights--주요-특징)
- [System Architecture](#system-architecture--시스템-아키텍처)
- [Automation Inventory](#automation-inventory--자동화-인벤토리)
- [Quick Start](#quick-start--빠른-시작)
- [Local Development](#local-development--로컬-개발)
- [Commands Reference](#commands-reference--명령어-참조)
- [Contributing](#contributing--기여하기)

---

## Overview / 개요

The Splunk Security Alert System is a comprehensive Splunk application designed for enterprise-grade FortiGate firewall security monitoring. It provides real-time threat detection using the **EMS (Event-Metric-State) pattern**, ensuring that security notifications are sent to Slack **only on state changes**, eliminating duplicate alerts.

이 시스템은 기업용 FortiGate 방화벽 보안을 모니터링하기 위한 Splunk 애플리케이션입니다. **EMS(Event-Metric-State) 패턴**을 사용하여 보안 알림이 **상태 변경 시에만** Slack으로 전송되어 중복 알림을 제거합니다.

---

## Highlights / 주요 특징

| Feature | Description |
|---------|-------------|
| **15 Production Alerts** | VPN, HA, hardware, resource, brute-force, traffic, license, FMG sync coverage |
| **Zero Duplicate Notifications** | State-tracking via 11 CSV lookups, alerts fire only on state change |
| **SPL-First Architecture** | All logic in Splunk searches, no external processing required |
| **Air-Gapped Ready** | All Python dependencies bundled (`requests`, `urllib3`, `certifi`, ...) |
| **Single-Line Slack Messages** | Optimized for mobile push notifications (≤200 chars) |
| **Macro-Based Configuration** | Index name, LogID groups, thresholds centralized |

---

## System Architecture / 시스템 아키텍처

```mermaid
flowchart LR
    FG["FortiGate Firewall"] -->|"syslog"| SP["Splunk Indexer"]
    SP -->|"index=_internal"| AL["Alert Logic<br/>(SPL Macros)"]
    AL -->|"state change?| ST["State Tracker<br/>(11 CSV Lookups)"]
    ST -->|"yes"| SL["Slack Alert<br/>(slack.py)"]
    ST -->|"no"| END["No Action"]

    subgraph Production Alerts
        AL -->|VPN| VPN_AL
        AL -->|HA| HA_AL
        AL -->|Hardware| HW_AL
        AL -->|Resource| RES_AL
        AL -->|Brute Force| BF_AL
        AL -->|Traffic| TRAF_AL
        AL -->|License| LIC_AL
        AL -->|FMG Sync| FMG_AL
    end
```

### Repository Layout / 저장소 구조

```
splunk-security-alert-system/
├── .github/
│   └── workflows/           # 32 GitHub Actions workflows
├── _bot-scripts/            # Automation tooling
│   ├── scripts/             # Python automation (pr_review, repo_review, etc.)
│   ├── AGENTS.md            # Agent configuration
│   └── Dockerfile.*          # Container definitions
├── security_alert/          # Splunk app (deployable)
│   ├── default/             # 15 alert definitions, macros, transforms
│   ├── bin/                 # Slack alert action (slack.py)
│   ├── lib/python3/         # Bundled Python dependencies
│   ├── lookups/             # State trackers + LogID map (6091 entries)
│   └── metadata/
├── docs/                    # Deployment, release notes, quick start
├── tests/                   # Validation scripts
└── demo/                    # Demonstration assets
```

---

## Automation Inventory / 자동화 인벤토리

### GitHub Actions Workflows / GitHub Actions 워크플로우

| # | Workflow File | Purpose |
|---|---------------|---------|
| 01 | `01_branch-to-pr.yml` | Branch-to-PR automation |
| 02 | `02_issue-to-branch.yml` | Issue-to-branch automation |
| 03 | `03_pr-checks.yml` | PR validation checks |
| 04 | `04_actionlint.yml` | Action linting |
| 05 | `05_gitleaks.yml` | Secret scanning |
| 06 | `06_codeql.yml` | CodeQL security analysis |
| 07 | `07_dependency-review.yml` | Dependency vulnerability review |
| 08 | `08_scorecard.yml` | OpenSSF Scorecard assessment |
| 09 | `09_semantic-pr.yml` | Semantic PR validation |
| 10 | `10_pr-review.yml` | PR review automation |
| 12 | `12_dependabot-auto-merge.yml` | Dependabot auto-merge |
| 13 | `13_pr-auto-merge.yml` | PR auto-merge |
| 14 | `14_bot-auto-fix.yml` | Bot auto-fix capabilities |
| 15 | `15_merged-pr-cleanup.yml` | Post-merge cleanup |
| 18 | `18_issue-management.yml` | Issue management |
| 19 | `19_issue-backfill.yml` | Issue backfill |
| 20 | `20_readme-gen.yml` | README generation |
| 21 | `21_docs-sync.yml` | Documentation sync |
| 24 | `24_release-notes.yml` | Release notes generation |
| 25 | `25_release-publish.yml` | Release publishing |
| 29 | `29_downstream-health-check.yml` | Downstream health check |
| 37 | `37_ci-failure-issues.yml` | CI failure issue creation |
| 42 | `42_reusable-docs-sync.yml` | Reusable docs sync |
| 43 | `43_reusable-issue-management.yml` | Reusable issue management |
| 44 | `44_reusable-pr-checks.yml` | Reusable PR checks |
| 45 | `45_reusable-gitleaks.yml` | Reusable Gitleaks scan |
| 60 | `60_ci-auto-heal.yml` | CI auto-heal |
| - | `auto-merge.yml` | General auto-merge |
| - | `ci.yml` | CI pipeline |
| - | `labeler.yml` | PR label automation |
| - | `welcome.yml` | New contributor welcome |
| - | `security/11_pr-review.yml` | Security-focused PR review |

### Python Automation Scripts / Python 자동화 스크립트

| Script | Purpose |
|--------|---------|
| `generate_readme.py` | README generation (bilingual Korean/English) |
| `pr_review_runner.py` | PR review runner |
| `redact_exposed_secrets.py` | Secret redaction |
| `repo_review.py` | Repository review |

### Reusable Workflows / 재사용 가능한 워크플로우

- **Reusable Docs Sync** (`42_reusable-docs-sync.yml`) - Documentation synchronization
- **Reusable Issue Management** (`43_reusable-issue-management.yml`) - Issue operations
- **Reusable PR Checks** (`44_reusable-pr-checks.yml`) - PR validation
- **Reusable Gitleaks** (`45_reusable-gitleaks.yml`) - Secret scanning

---

## Quick Start / 빠른 시작

### Prerequisites / 사전 요구사항

- Splunk 8.0+
- Python 3.7+
- FortiGate firewall logs accessible via syslog

### Installation / 설치

```bash
# Clone the repository
git clone https://github.com/splunk/security-alert-system.git
cd security-alert-system

# Deploy to Splunk
cp -r security_alert $SPLUNK_HOME/etc/apps/

# Restart Splunk
$SPLUNK_HOME/bin/splunk restart
```

### Configuration / 구성

1. Configure syslog input for FortiGate devices
2. Update macro definitions in `security_alert/default/ops.conf`:
   - `index_name` - Your Splunk index
   - `logid_groups` - LogID mappings
   - `thresholds` - Alert thresholds

---

## Local Development / 로컬 개발

### Development Environment / 개발 환경

```bash
# Create virtual environment
python3 -m venv venv
source venv/bin/activate  # Linux/Mac
# or
.\venv\Scripts\activate   # Windows

# Install dependencies
pip install -r _bot-scripts/requirements-dev.txt
pip install -r _bot-scripts/requirements.txt
```

### Docker Support / Docker 지원

```bash
# GitHub App development
docker-compose -f _bot-scripts/docker-compose.github_app.yml up -d

# GitHub Action development
docker build -f _bot-scripts/Dockerfile.github_action -t action-dev .
```

### Running Tests / 테스트 실행

```bash
# Run PR review tests
python -m pytest _bot-scripts/scripts/pr_review_runner_test.py -v

# Run repo review
python _bot-scripts/scripts/repo_review.py --help
```

---

## Commands Reference / 명령어 참조

### Makefile Commands / Makefile 명령어

```bash
make help              # Show all available commands
make lint              # Run linting
make test              # Run tests
make build             # Build Docker images
make deploy            # Deploy to Splunk
```

### Automation Scripts / 자동화 스크립트

| Command | Description |
|---------|-------------|
| `python scripts/generate_readme.py` | Generate README.md |
| `python scripts/pr_review_runner.py --pr <number>` | Run PR review |
| `python scripts/redact_exposed_secrets.py <file>` | Redact secrets |
| `python scripts/repo_review.py` | Run repository review |

---

## Contributing / 기여하기

We welcome contributions! Please see [CONTRIBUTING.md](./CONTRIBUTING.md) for details.

Contribution guides available:
- [Root CONTRIBUTING.md](./CONTRIBUTING.md) - Main contribution guidelines
- [`_bot-scripts/CONTRIBUTING.md`](./_bot-scripts/CONTRIBUTING.md) - Bot automation contribution guidelines

### Automation Contribution / 자동화 기여

This repository uses **GitHub Actions automation** extensively. When contributing:

1. Follow the workflow naming convention (`NN_<component>.yml`)
2. Use reusable workflows where applicable
3. Ensure `actionlint` passes (workflow `04_actionlint.yml`)
4. Run secret scanning with Gitleaks (`05_gitleaks.yml`)

### Automated Checks / 자동화된 검사

All PRs must pass:
- ✅ Action lint (`04_actionlint.yml`)
- ✅ Gitleaks scan (`05_gitleaks.yml`)
- ✅ CodeQL analysis (`06_codeql.yml`)
- ✅ Dependency review (`07_dependency-review.yml`)
- ✅ Scorecard assessment (`08_scorecard.yml`)

---

## License / 라이선스

See [LICENSE](./LICENSE) for details.

---

## Support / 지원

- Documentation: [`docs/`](./docs/)
- Quick Start: [`docs/QUICK-START.md`](./docs/QUICK-START.md)
- Deployment: [`docs/DEPLOYMENT.md`](./docs/DEPLOYMENT.md)
- Release Notes: [`docs/RELEASE-NOTES.md`](./docs/RELEASE-NOTES.md)

---

**Maintained by** the Splunk Security Alert System Team
**Version** 2.0.4
```