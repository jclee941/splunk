# GitHub Automation Bot — CLIProxyAPI v2.0

[![CLIProxyAPI](https://img.shields.io/badge/CLIProxyAPI-v2.0-c0ffee?style=flat-square)](https://cliproxy.jclee.me)
[![Workflows](https://img.shields.io/badge/Workflows-33%20Active-brightgreen?style=flat-square)](#automation-inventory--자동화-인벤토리)
[![Python](https://img.shields.io/badge/Python-3.x-3776AB?style=flat-square)](https://www.python.org/)
[![License](https://img.shields.io/badge/License-Proprietary-blue?style=flat-square)](./LICENSE)

> **English** | [**한국어**](#개요)

---

## Table of Contents

- [Overview](#overview--개요)
- [Features](#features--주요-기능)
- [Architecture](#architecture--아키텍처)
- [Automation Inventory](#automation-inventory--자동화-인벤토리)
- [Quick Start](#quick-start--빠른-시작)
- [Local Development](#local-development--로컬-개발)
- [Commands Reference](#commands-reference--명령어-참조)
- [Contribution Guide](#contribution-guide--기여-가이드)

---

## Overview | 개요

This repository contains the **CLIProxyAPI v2.0 GitHub Automation Bot** — a comprehensive system of **33 GitHub Actions workflows** and Python-based automation tools that provide automated code review, issue management, PR handling, documentation generation, and release automation for GitHub repositories.

이 저장소는 **CLIProxyAPI v2.0 GitHub 자동화 봇**을 포함하고 있으며, GitHub 저장소에 대한 자동화된 코드 리뷰, 이슈 관리, PR 처리, 문서 생성 및 릴리스 자동화를 제공하는 **33개의 GitHub Actions 워크플로우**와 Python 기반 자동화 도구의 종합 시스템입니다.

### Key Characteristics | 주요 특성

| Characteristic 특성 | Description 설명 |
|---|---|
| **Architecture** 아키텍처 | Python-based GitHub App + GitHub Actions |
| **Automation Scope** 자동화 범위 | 33 workflows covering PR, issues, security, releases |
| **Core Scripts** 핵심 스크립트 | `scripts/` (Python automation tools) |
| **External Services** 외부 서비스 | CLIProxyAPI (`cliproxy.jclee.me`), PR Agent (`qodo-ai/pr-agent`) |
| **Logging** 로깅 | ELK stack (`<homelab-elk>`) |

---

## Features | 주요 기능

### Automated Code Review | 자동화 코드 리뷰

- **PR Agent Integration**: AI-powered code reviews using `qodo-ai/pr-agent`
- **Automated PR Review**: Multiple workflows (`10_pr-review.yml`, `security/11_pr-review.yml`) orchestrate AI-driven code analysis
- **Custom Review Scripts**: Python-based review runners (`pr_review_runner.py`) provide supplementary analysis

### Issue Management | 이슈 관리

- **Automated Issue Handling**: Workflows process, classify, and route issues automatically
- **Issue Classification**: ML-based classification (`91_issue-classification.yml`) and backfill (`19_issue-backfill.yml`)
- **CI Failure Tracking**: Auto-creation of issues for CI failures (`37_ci-failure-issues.yml`)

### PR Automation | PR 자동화

- **Branch-to-PR**: Automatic branch-to-PR conversion (`01_branch-to-pr.yml`)
- **Auto-Merge**: Smart PR merging with dependency awareness (`12_dependabot-auto-merge.yml`, `13_pr-auto-merge.yml`, `auto-merge.yml`)
- **Auto-Fix**: Bot-powered code fixes (`14_bot-auto-fix.yml`)
- **Semantic PR**: Enforced semantic versioning commits (`09_semantic-pr.yml`)

### Security Scanning | 보안 스캐닝

- **Gitleaks**: Secret detection and prevention (`05_gitleaks.yml`, `45_reusable-gitleaks.yml`)
- **CodeQL**: Static code analysis (`06_codeql.yml`)
- **Dependency Review**: Vulnerability scanning (`07_dependency-review.yml`)
- **Scorecard**: Security score assessment (`08_scorecard.yml`)

### Documentation Automation | 문서 자동화

- **README Generation**: Automatic README updates (`20_readme-gen.yml`)
- **Docs Sync**: Cross-repository documentation synchronization (`21_docs-sync.yml`, `42_reusable-docs-sync.yml`)

### Release Automation | 릴리스 자동화

- **Release Notes**: Automated changelog generation (`24_release-notes.yml`)
- **Release Publishing**: Streamlined release workflow (`25_release-publish.yml`)
- **Downstream Health Check**: Post-release monitoring (`29_downstream-health-check.yml`)

### CI/CD Health | CI/CD 상태 관리

- **Actionlint**: Workflow syntax validation (`04_actionlint.yml`)
- **CI Auto-Heal**: Self-healing CI pipelines (`60_ci-auto-heal.yml`)
- **PR Checks**: Comprehensive PR validation (`03_pr-checks.yml`, `44_reusable-pr-checks.yml`)

---

## Architecture | 아키텍처

```mermaid
flowchart TB
    subgraph "GitHub Events"
        PR["Pull Request"]
        Issue["Issue"]
        Push["Push"]
        Schedule["Scheduled"]
    end

    subgraph "GitHub Actions Engine"
        W1["01_branch-to-pr.yml"]
        W2["02_issue-to-branch.yml"]
        W3["03_pr-checks.yml"]
        W4["04_actionlint.yml"]
        W5["05_gitleaks.yml"]
        W6["06_codeql.yml"]
        W7["07_dependency-review.yml"]
        W8["08_scorecard.yml"]
        W9["09_semantic-pr.yml"]
        W10["10_pr-review.yml"]
        W12["12_dependabot-auto-merge.yml"]
        W13["13_pr-auto-merge.yml"]
        W14["14_bot-auto-fix.yml"]
        W15["15_merged-pr-cleanup.yml"]
        W18["18_issue-management.yml"]
        W19["19_issue-backfill.yml"]
        W20["20_readme-gen.yml"]
        W21["21_docs-sync.yml"]
        W24["24_release-notes.yml"]
        W25["25_release-publish.yml"]
        W29["29_downstream-health-check.yml"]
        W37["37_ci-failure-issues.yml"]
        W42["42_reusable-docs-sync.yml"]
        W43["43_reusable-issue-management.yml"]
        W44["44_reusable-pr-checks.yml"]
        W45["45_reusable-gitleaks.yml"]
        W60["60_ci-auto-heal.yml"]
        W91["91_issue-classification.yml"]
        Labeler["labeler.yml"]
        Welcome["welcome.yml"]
        AutoMerge["auto-merge.yml"]
        CI["ci.yml"]
        Sec11["security/11_pr-review.yml"]
    end

    subgraph "CLIProxyAPI Bot"
        App["GitHub App"]
        Scripts["Python Scripts<br/>scripts/"]
    end

    subgraph "External Services"
        PRAgent["PR Agent<br/>qodo-ai/pr-agent"]
        ProxyAPI["CLIProxyAPI<br/>cliproxy.jclee.me"]
        ELK["ELK Stack<br/>&lt;homelab-elk&gt;"]
    end

    PR --> W1
    PR --> W3
    PR --> W10
    PR --> W13
    PR --> W14
    PR --> W44
    PR --> Sec11
    PR --> AutoMerge

    Issue --> W2
    Issue --> W18
    Issue --> W19
    Issue --> W37
    Issue --> W43
    Issue --> W91

    Push --> W4
    Push --> W5
    Push --> W6
    Push --> W7
    Push --> W8
    Push --> W9
    Push --> W60
    Push --> CI

    Schedule --> W12
    Schedule --> W15
    Schedule --> W20
    Schedule --> W21
    Schedule --> W24
    Schedule --> W25
    Schedule --> W29
    Schedule --> W42
    Schedule --> W45

    W10 --> PRAgent
    Sec11 --> PRAgent
    W44 --> PRAgent
    W45 --> Scripts

    PRAgent --> ProxyAPI
    Scripts --> ProxyAPI
    App --> ProxyAPI

    ProxyAPI --> ELK
    ProxyAPI --> "CLIProxyAPI<br/>bot.jclee.me"
```

---

## Automation Inventory | 자동화 인벤토리

### Workflow Files | 워크플로우 파일 (33)

#### PR Workflows | PR 워크플로우

| Workflow File | Description 설명 |
|---|---|
| `01_branch-to-pr.yml` | Converts branches to pull requests automatically |
| `03_pr-checks.yml` | Runs comprehensive checks on pull requests |
| `10_pr-review.yml` | Orchestrates AI-powered PR reviews |
| `13_pr-auto-merge.yml` | Automatically merges approved PRs |
| `14_bot-auto-fix.yml` | Applies bot-generated code fixes |
| `15_merged-pr-cleanup.yml` | Cleans up after PR merge |
| `44_reusable-pr-checks.yml` | Reusable PR validation workflow |
| `security/11_pr-review.yml` | Security-focused PR review |
| `auto-merge.yml` | Generic auto-merge handler |

#### Issue Management Workflows | 이슈 관리 워크플로우

| Workflow File | Description 설명 |
|---|---|
| `02_issue-to-branch.yml` | Creates branches from issues |
| `18_issue-management.yml` | General issue processing and routing |
| `19_issue-backfill.yml` | Backfills issue metadata |
| `37_ci-failure-issues.yml` | Creates issues from CI failures |
| `43_reusable-issue-management.yml` | Reusable issue management workflow |
| `91_issue-classification.yml` | ML-based issue classification |

#### Security Workflows | 보안 워크플로우

| Workflow File | Description 설명 |
|---|---|
| `05_gitleaks.yml` | Secret scanning with Gitleaks |
| `06_codeql.yml` | CodeQL static analysis |
| `07_dependency-review.yml` | Dependency vulnerability scanning |
| `08_scorecard.yml` | Security scorecard assessment |
| `45_reusable-gitleaks.yml` | Reusable Gitleaks configuration |

#### Release & Deployment Workflows | 릴리스 및 배포 워크플로우

| Workflow File | Description 설명 |
|---|---|
| `24_release-notes.yml` | Generates release notes automatically |
| `25_release-publish.yml` | Publishes releases |
| `29_downstream-health-check.yml` | Monitors downstream dependencies |
| `42_reusable-docs-sync.yml` | Cross-repo documentation sync |

#### Documentation Workflows | 문서화 워크플로우

| Workflow File | Description 설명 |
|---|---|
| `20_readme-gen.yml` | Auto-generates README documentation |
| `21_docs-sync.yml` | Synchronizes documentation across repos |

#### CI/CD Workflows | CI/CD 워크플로우

| Workflow File | Description 설명 |
|---|---|
| `04_actionlint.yml` | Validates GitHub Actions syntax |
| `09_semantic-pr.yml` | Enforces semantic PR conventions |
| `12_dependabot-auto-merge.yml` | Auto-merges Dependabot PRs |
| `60_ci-auto-heal.yml` | Self-healing CI pipeline |
| `ci.yml` | Main CI workflow |

#### Community & Labeling Workflows | 커뮤니티 및 라벨링 워크플로우

| Workflow File | Description 설명 |
|---|---|
| `labeler.yml` | Auto-labels issues and PRs |
| `welcome.yml` | Welcomes new contributors |

### Python Automation Tools | Python 자동화 도구

The `scripts/` directory contains Python-based automation tools used by the workflows:

| Script | Description 설명 |
|---|---|
| `pr_review_runner.py` | Orchestrates PR review pipeline with PR Agent |
| `pr_review_runner_test.py` | Unit tests for PR review runner |
| `generate_readme.py` | Generates README.md documentation |
| `check_private_ips.py` | Scans for hardcoded private IP addresses (RFC1918) |
| `check_private_ips_test.py` | Unit tests for private IP detection |
| `check_workflow_scripts.py` | Validates workflow script compliance |
| `check_workflow_scripts_test.py` | Unit tests for workflow validation |
| `check_hardcode_scan_patterns_test.py` | Tests for hardcoded pattern detection |
| `redact_exposed_secrets.py` | Detects and redacts exposed secrets |
| `repo_review.py` | Comprehensive repository health review |
| `issue_classification_workflow_test.py` | Tests for issue classification workflow |
| `issue_classifier_js_test.py` | JavaScript-based issue classification tests |
| `readme_mermaid_test.py` | Validates Mermaid diagram syntax in README |

---

## Quick Start | 빠른 시작

### Prerequisites | 사전 요구사항

- Python 3.x
- Docker (optional, for containerized development)
- A GitHub App registered for the automation bot

### Installation | 설치

```bash
# Clone the repository
git clone https://github.com/<org>/CLIProxyAPI.git
cd CLIProxyAPI

# Install Python dependencies
pip install -r _bot-scripts/requirements.txt
pip install -r _bot-scripts/requirements-dev.txt
```

### Configuration | 설정

1. **GitHub App Setup**: Register a GitHub App and configure the webhook URL
2. **Environment Variables**: Set required environment variables for authentication
3. **CLIProxyAPI Endpoint**: Configure the CLIProxyAPI endpoint (`https://cliproxy.jclee.me/v1`)

For detailed deployment instructions, see [`docs/DEPLOYMENT.md`](./docs/DEPLOYMENT.md).

---

## Local Development | 로컬 개발

### Running the Bot | 봇 실행

```bash
cd _bot-scripts

# Run the GitHub App locally
python -m scripts.pr_review_runner
```

### Running Tests | 테스트 실행

```bash
cd _bot-scripts

# Run all Python tests
python -m pytest scripts/ -v

# Run specific test files
python -m pytest scripts/pr_review_runner_test.py -v
python -m pytest scripts/check_private_ips_test.py -v
python -m pytest scripts/readme_mermaid_test.py -v
```

### Docker-based Development | Docker 기반 개발

```bash
cd _bot-scripts

# Build the GitHub Action container
docker build -f Dockerfile.github_action -t cli-proxy-action .

# Build the GitHub App container
docker build -f Dockerfile.github_app -t cli-proxy-app .

# Run with docker-compose
docker-compose -f docker-compose.github_app.yml up
```

---

## Commands Reference | 명령어 참조

### Makefile Commands | Makefile 명령어

```bash
cd _bot-scripts

make help        # Display available Makefile targets
make install     # Install dependencies
make test        # Run test suite
make lint        # Run linters
make clean       # Clean build artifacts
```

### Python Scripts | Python 스크립트

```bash
# PR Review
python scripts/pr_review_runner.py

# Security Scanning
python scripts/check_private_ips.py [--path <directory>]
python scripts/check_workflow_scripts.py
python scripts/redact_exposed_secrets.py

# Documentation
python scripts/generate_readme.py

# Repository Review
python scripts/repo_review.py
```

---

## Repository Structure | 저장소 구조

```
CLIProxyAPI/
├── _bot-scripts/          # Main bot application
│   ├── scripts/           # Python automation tools
│   │   ├── pr_review_runner.py
│   │   ├── check_private_ips.py
│   │   ├── generate_readme.py
│   │   └── ...
│   ├── Dockerfile.github_action
│   ├── Dockerfile.github_app
│   ├── docker-compose.github_app.yml
│   ├── requirements.txt
│   ├── requirements-dev.txt
│   └── ...
├── .github/
│   └── workflows/         # GitHub Actions workflows (33 total)
│       ├── 01_branch-to-pr.yml
│       ├── 03_pr-checks.yml
│       ├── 10_pr-review.yml
│       └── ...
├── docs/                  # Documentation
│   ├── DEPLOYMENT.md
│   ├── QUICK-START.md
│   ├── RELEASE-NOTES.md
│   └── ...
├── tests/                # Integration tests
├── demo/                 # Demo materials
└── security_alert/       # Security alert module
```

---

## Contribution Guide | 기여 가이드

Contributions are welcome! Please follow these guidelines:

### Reporting Issues | 이슈 보고

1. Check existing issues before creating new ones
2. Use appropriate issue templates
3. Include reproduction steps and expected behavior

### Submitting Changes | 변경 제출

1. Fork the repository
2. Create a feature branch from `main`
3. Follow the existing code style and conventions
4. Add tests for new functionality
5. Ensure all tests pass before submitting

### Development Standards | 개발 표준

- Python code follows PEP 8 style guidelines
- GitHub Actions workflows are validated with actionlint
- All scripts include appropriate inline documentation
- Commit messages follow semantic conventions

### Related Documentation | 관련 문서

- [`CONTRIBUTING.md`](./CONTRIBUTING.md) — General contribution guidelines
- [`_bot-scripts/CONTRIBUTING.md`](./_bot-scripts/CONTRIBUTING.md) — Bot-specific guidelines
- [`_bot-scripts/AGENTS.md`](./_bot-scripts/AGENTS.md) — Agent architecture
- [`docs/DEPLOYMENT.md`](./docs/DEPLOYMENT.md) — Deployment instructions
- [`docs/QUICK-START.md`](./docs/QUICK-START.md) — Quick start guide

---

## License | 라이선스

Proprietary. See [`LICENSE`](./LICENSE) for details.

---

## External Links | 외부 링크

- **CLIProxyAPI**: <https://cliproxy.jclee.me>
- **PR Agent**: <https://github.com/qodo-ai/pr-agent>
- **Bot Dashboard**: <https://bot.jclee.me>

---

*Last updated: 2025*
