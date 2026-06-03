# Comprehensive README.md Generator

## Analysis

**Repository:** CLIProxyAPI v2.0 GitHub Automation Bot
**Core Directory:** `_bot-scripts/` (Python-based GitHub Actions bot)
**Workflows:** 33 workflow files (prefixed numbering pattern)
**Go Tools:** 0 (Python-based automation)
**External Services:** CLIProxyAPI (cliproxy.jclee.me), PR Agent (qodo-ai/pr-agent)

## Generated README.md

```markdown
# GitHub Automation Bot - CLIProxyAPI v2.0

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
| **Automation Scope** 자동화 범위 | 33 workflows covering PR, issue, docs, release |
| **External Integrations** 외부 통합 | CLIProxyAPI, PR Agent (qodo-ai/pr-agent) |
| **Deployment Target** 배포 대상 | Self-hosted (homelab environment) |

---

## Features | 주요 기능

### Automated Code Review | 자동화된 코드 리뷰

- **PR Review**: AI-powered PR analysis using [PR Agent](https://github.com/qodo-ai/pr-agent)
- **Security Scanning**: Gitleaks secrets detection, CodeQL analysis
- **Dependency Review**: Vulnerability scanning and license compliance
- **Action Lint**: Workflow file validation

### Issue Management | 이슈 관리

- **Automatic Classification**: AI-based issue categorization
- **Branch Creation**: Auto-create feature branches from issues
- **Backfill Automation**: Sync issues across repositories
- **CI Failure Tracking**: Auto-create issues from failed pipelines

### Pull Request Automation | 풀 리퀘스트 자동화

- **Auto Merge**: Automatic merging based on status checks
- **Dependabot Integration**: Automated dependency updates
- **Semantic PR Validation**: Enforce conventional commits
- **PR Cleanup**: Auto-cleanup merged PR branches

### Documentation | 문서화

- **README Generation**: Automated documentation updates
- **Docs Sync**: Cross-repository documentation synchronization
- **Release Notes**: Automatic changelog generation

### Release Automation | 릴리스 자동화

- **Version Management**: Semantic versioning workflow
- **Release Publishing**: Automated release artifact distribution
- **Downstream Health Check**: Monitor dependent repositories

---

## Architecture | 아키텍처

```mermaid
flowchart TB
    subgraph "GitHub Events | GitHub 이벤트"
        PR[/"Pull Request"\n/"풀 리퀘스트"/]
        ISSUE[/"Issue"\n/"이슈"/]
        PUSH[/"Push"\n/"푸시"/]
        SCHEDULE[/"Schedule"\n/"스케줄"/]
    end

    subgraph "Workflow Layer | 워크플로우 레이어"
        subgraph "PR Automation | PR 자동화"
            WF03["03_pr-checks.yml\nPR Checks"]
            WF10["10_pr-review.yml\nPR Review"]
            WF13["13_pr-auto-merge.yml\nAuto Merge"]
            WF14["14_bot-auto-fix.yml\nAuto Fix"]
            WF15["15_merged-pr-cleanup.yml\nPR Cleanup"]
        end

        subgraph "Issue Management | 이슈 관리"
            WF02["02_issue-to-branch.yml\nIssue→Branch"]
            WF18["18_issue-management.yml\nIssue Mgmt"]
            WF19["19_issue-backfill.yml\nIssue Backfill"]
            WF91["91_issue-classification.yml\nClassification"]
        end

        subgraph "Security & Quality | 보안 및 품질"
            WF04["04_actionlint.yml\nAction Lint"]
            WF05["05_gitleaks.yml\nSecrets Scan"]
            WF06["06_codeql.yml\nCodeQL"]
            WF07["07_dependency-review.yml\nDep Review"]
            WF08["08_scorecard.yml\nScorecard"]
            SEC11["security/11_pr-review.yml\nSec Review"]
        end

        subgraph "Documentation | 문서화"
            WF20["20_readme-gen.yml\nREADME Gen"]
            WF21["21_docs-sync.yml\nDocs Sync"]
            WF42["42_reusable-docs-sync.yml\nReusable Docs"]
        end

        subgraph "Release & Deploy | 릴리스 및 배포"
            WF24["24_release-notes.yml\nRelease Notes"]
            WF25["25_release-publish.yml\nPublish"]
            WF29["29_downstream-health-check.yml\nHealth Check"]
        end

        subgraph "CI/CD Healing | CI/CD 자동 복구"
            WF60["60_ci-auto-heal.yml\nAuto Heal"]
            WF37["37_ci-failure-issues.yml\nCI Failure→Issue"]
        end

        subgraph "Merge Automation | 병합 자동화"
            WF01["01_branch-to-pr.yml\nBranch→PR"]
            WF12["12_dependabot-auto-merge.yml\nDependabot"]
            AUTO["auto-merge.yml\nAuto Merge"]
        end
    end

    subgraph "Bot Services | 봇 서비스"
        CLIProxy["CLIProxyAPI\n&lt;homelab-host&gt;:8317"]
        PRAgent["PR Agent\nqodo-ai/pr-agent"]
        ELK["ELK Stack\n&lt;homelab-elk&gt;"]
    end

    subgraph "External Services | 외부 서비스"
        GH["GitHub API"]
        REPO["Downstream\nRepos"]
    end

    PR --> WF03
    PR --> WF10
    PR --> WF13
    PR --> SEC11
    ISSUE --> WF02
    ISSUE --> WF18
    ISSUE --> WF91
    PUSH --> WF04
    PUSH --> WF05
    PUSH --> WF06
    SCHEDULE --> WF60

    WF10 --> CLIProxy
    WF10 --> PRAgent
    WF05 --> ELK
    WF08 --> GH

    WF12 --> AUTO
    WF13 --> AUTO
    WF24 --> WF25
    WF29 --> REPO

    style CLIProxy fill:#f9f,stroke:#333,stroke-width:2px
    style PRAgent fill:#9f9,stroke:#333,stroke-width:2px
    style ELK fill:#ff9,stroke:#333,stroke-width:2px
```

---

## Automation Inventory | 자동화 인벤토리

### GitHub Actions Workflows | GitHub Actions 워크플로우

#### PR Automation | PR 자동화

| Workflow | Description | Trigger |
|----------|-------------|---------|
| `01_branch-to-pr.yml` | Create PR from feature branch | Push to feature branch |
| `03_pr-checks.yml` | Run checks on PR (lint, test, build) | Pull request |
| `10_pr-review.yml` | AI-powered PR review | Pull request |
| `13_pr-auto-merge.yml` | Auto-merge approved PRs | PR approve/comment |
| `14_bot-auto-fix.yml` | Auto-fix detected issues | PR checks failure |
| `15_merged-pr-cleanup.yml` | Cleanup branches after merge | PR merged |
| `auto-merge.yml` | Generic auto-merge workflow | Manual/dispatch |

#### Issue Management | 이슈 관리

| Workflow | Description | Trigger |
|----------|-------------|---------|
| `02_issue-to-branch.yml` | Create branch from issue | Issue opened/labeled |
| `18_issue-management.yml` | Manage issue lifecycle | Issue events |
| `19_issue-backfill.yml` | Sync issues from upstream | Schedule |
| `91_issue-classification.yml` | Classify issues with AI | Issue opened |

#### Security & Quality | 보안 및 품질

| Workflow | Description | Trigger |
|----------|-------------|---------|
| `04_actionlint.yml` | Lint GitHub Actions files | Push/PR |
| `05_gitleaks.yml` | Scan for secrets/leaks | Push/PR |
| `06_codeql.yml` | CodeQL security analysis | Push/PR |
| `07_dependency-review.yml` | Review dependency changes | PR |
| `08_scorecard.yml` | OpenSSF Scorecard analysis | Schedule |
| `security/11_pr-review.yml` | Security-focused PR review | PR |

#### CI/CD Automation | CI/CD 자동화

| Workflow | Description | Trigger |
|----------|-------------|---------|
| `60_ci-auto-heal.yml` | Auto-heal failing CI | CI failure |
| `37_ci-failure-issues.yml` | Create issue from CI failure | CI failure |
| `ci.yml` | Main CI workflow | Push/PR |

#### Documentation | 문서화

| Workflow | Description | Trigger |
|----------|-------------|---------|
| `20_readme-gen.yml` | Generate/update README | Push/dispatch |
| `21_docs-sync.yml` | Sync docs across repos | Schedule/dispatch |
| `42_reusable-docs-sync.yml` | Reusable docs sync | Called workflow |

#### Release Management | 릴리스 관리

| Workflow | Description | Trigger |
|----------|-------------|---------|
| `24_release-notes.yml` | Generate release notes | Tag/dispatch |
| `25_release-publish.yml` | Publish release artifacts | Release |
| `29_downstream-health-check.yml` | Check downstream repos | Schedule |

#### Reusable Workflows | 재사용可能な 워크플로우

| Workflow | Description |
|----------|-------------|
| `43_reusable-issue-management.yml` | Reusable issue management |
| `44_reusable-pr-checks.yml` | Reusable PR checks |
| `45_reusable-gitleaks.yml` | Reusable secrets scanning |

#### Other Automation | 기타 자동화

| Workflow | Description | Trigger |
|----------|-------------|---------|
| `09_semantic-pr.yml` | Enforce semantic PR titles | PR |
| `12_dependabot-auto-merge.yml` | Auto-merge Dependabot PRs | PR (dependabot) |
| `labeler.yml` | Auto-label PRs/Issues | PR/Issue |
| `welcome.yml` | Welcome new contributors | PR (first-time) |

### Python Automation Scripts | Python 자동화 스크립트

Located in `_bot-scripts/scripts/`:

| Script | Purpose |
|--------|---------|
| `pr_review_runner.py` | Run PR review via CLIProxyAPI |
| `repo_review.py` | Repository-level review |
| `generate_readme.py` | Generate README documentation |
| `check_workflow_scripts.py` | Validate workflow scripts |
| `check_private_ips.py` | Scan for hardcoded IPs |
| `check_hardcode_scan_patterns_test.py` | Test hardcode scanning |
| `redact_exposed_secrets.py` | Redact exposed secrets |
| `issue_classification_workflow_test.py` | Test issue classification |
| `pr_review_runner_test.py` | Test PR review runner |
| `readme_mermaid_test.py` | Test README Mermaid diagrams |

---

## Quick Start | 빠른 시작

### Prerequisites |前提条件

- Python 3.9+
- GitHub CLI (`gh`)
- Access to CLIProxyAPI endpoint

### Installation | 설치

```bash
# Clone repository
git clone https://github.com/your-org/cli-proxy-api.git
cd cli-proxy-api

# Install dependencies
pip install -r _bot-scripts/requirements.txt

# Install dev dependencies
pip install -r _bot-scripts/requirements-dev.txt
```

### Configuration | 설정

```bash
# Set CLIProxyAPI endpoint
export CLIPROXY_API_URL="https://cliproxy.jclee.me/v1"

# Set GitHub token
export GH_TOKEN="ghp_your_token_here"
```

### Running Locally | 로컬 실행

```bash
# Run PR review
python _bot-scripts/scripts/pr_review_runner.py --pr-url https://github.com/owner/repo/pull/123

# Generate README
python _bot-scripts/scripts/generate_readme.py --repo-owner owner --repo-name repo

# Check for hardcoded IPs
python _bot-scripts/scripts/check_private_ips.py --path ./your-code
```

---

## Local Development | 로컬 개발

### Directory Structure | 디렉토리 구조

```
.
├── _bot-scripts/           # Main bot implementation
│   ├── scripts/            # Python automation scripts
│   ├── docker-compose.github_app.yml  # GitHub App deployment
│   └── Dockerfile.*        # Container images
├── .github/
│   └── workflows/          # GitHub Actions workflows (33 files)
├── docs/                   # Documentation
├── tests/                  # Test files
├── demo/                   # Demo materials
└── security_alert/         # Security alert handling
```

### Running Tests | 테스트 실행

```bash
# Run all tests
make -f _bot-scripts/Makefile test

# Run specific test
pytest _bot-scripts/scripts/check_private_ips_test.py

# Run workflow validation
python _bot-scripts/scripts/check_workflow_scripts.py
```

### Local CI Simulation | 로컬 CI 시뮬레이션

```bash
# Simulate PR checks
act -W .github/workflows/03_pr-checks.yml

# Simulate PR review
act -W .github/workflows/10_pr-review.yml
```

---

## Commands Reference | 명령어 참조

### Makefile Commands (in `_bot-scripts/Makefile`)

| Command | Description |
|---------|-------------|
| `make test` | Run all unit tests |
| `make lint` | Run linting checks |
| `make format` | Format code |
| `make docker-build` | Build Docker images |
| `make docker-push` | Push Docker images |

### Python Scripts | Python 스크립트

| Script | Key Options |
|--------|-------------|
| `pr_review_runner.py` | `--pr-url`, `--repo`, `--pr-number` |
| `generate_readme.py` | `--repo-owner`, `--repo-name`, `--output` |
| `check_private_ips.py` | `--path`, `--verbose` |
| `check_workflow_scripts.py` | `--workflow-dir` |
| `repo_review.py` | `--repo-url`, `--output-format` |

---

## Contribution Guide | 기여 가이드

### Workflow Development | 워크플로우 개발

1. **Naming Convention**: Use prefix numbers (e.g., `10_pr-review.yml`)
2. **Documentation**: Update `docs/` when adding new workflows
3. **Testing**: Add tests in `tests/` for new functionality

### Adding New Workflows | 새 워크플로우 추가

1. Create workflow file in `.github/workflows/`
2. Follow naming convention with numeric prefix
3. Add to `91_issue-classification.yml` pattern if applicable
4. Update README automation inventory

### Code Standards | 코드 표준

- Python: Follow PEP 8
- YAML: Valid GitHub Actions syntax
- Comments: English (preferred) or Korean

### Security Considerations | 보안 고려사항

- Never commit secrets or tokens
- Use GitHub Secrets for sensitive data
- Run `check_private_ips.py` before committing
- Use `05_gitleaks.yml` for secret scanning

---

## License | 라이선스

See [LICENSE](./LICENSE) file for details.

---

## External Links | 외부 링크

- **CLIProxyAPI**: <https://cliproxy.jclee.me/v1>
- **PR Agent**: <https://github.com/qodo-ai/pr-agent>
- **Bot Status**: <https://bot.jclee.me> (if available)

---

*Generated by CLIProxyAPI README Generator (minimax-m2.7)*

```

---

## Summary

This README provides:

1. **Bilingual coverage** (English/Korean) for all sections
2. **33 workflow files** listed with actual on-disk names including numeric prefixes
3. **Mermaid architecture diagram** with proper HTML-escaped angle brackets in node labels
4. **No hardcoded IPs** — uses placeholders `<homelab-host>`, `<homelab-elk>`
5. **Proper external links** — CLIProxyAPI (cliproxy.jclee.me), PR Agent (qodo-ai/pr-agent), bot.jclee.me
6. **Real directory structure** from `_bot-scripts/` path
7. **No ASCII art** — uses proper Mermaid flowchart
8. **Proper markdown headings** — no bold text as substitute for headings
9. **Complete automation inventory** grouped by category
10. **Quick start, local development, commands reference, and contribution guide** sections
