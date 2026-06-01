# Splunk Security Alert System for FortiGate

[![fortigate-security](https://img.shields.io/badge/FortiGate-Security%20Alerts-00A9CE?style=flat-square)](https://docs.fortinet.com/product/fortigate/)
[![splunk](https://img.shields.io/badge/SPL-Splunk%20Search%20Language-FF9900?style=flat-square)](https://docs.splunk.com/Documentation/SPL)
[![License](https://img.shields.io/badge/License-Proprietary-blue?style=flat-square)](./LICENSE)
[![Workflows](https://img.shields.io/badge/Workflows-32%20Active-brightgreen?style=flat-square)](#automation-inventory)
[![Bot](https://img.shields.io/badge/Bot-CLIProxyAPI%20v2.0-c0ffee?style=flat-square)](https://cliproxy.jclee.me)

> **English** | [한국어](#개요)

---

## Overview

This repository contains the **Splunk Security Alert System** designed for FortiGate firewall monitoring. It provides **15 production-ready security alerts** with state-aware alerting capabilities that ensure zero duplicate notifications.

### Key Characteristics

| Characteristic | Description |
|----------------|-------------|
| **Alert Count** | 15 production alerts for FortiGate security events |
| **Architecture** | SPL-First: All logic implemented in Splunk Search Processing Language |
| **Alerting Model** | State-aware with deduplication to prevent alert storms |
| **Notification** | Single-line Slack messages for efficient incident response |
| **Configuration** | Macro-based for easy customization and air-gapped deployment |
| **Deployment** | Air-gapped environment ready |

---

## Features

### Core Capabilities

- **15 Production Security Alerts** covering critical FortiGate events:
  - Brute force detection
  - Intrusion prevention system (IPS) alerts
  - Malware detection
  - Policy violations
  - Anomaly detection
  - And more...

- **State-Aware Alerting Engine**
  - Tracks alert state across evaluation windows
  - Eliminates duplicate notifications
  - Maintains alert history for correlation

- **SPL-First Architecture**
  - All detection logic in pure SPL
  - Portable across Splunk deployments
  - Version controlled and reviewable

- **Macro-Based Configuration**
  - Centralized parameter management
  - Easy threshold tuning
  - Environment-specific overrides

- **Slack Integration**
  - Concise single-line message format
  - Actionable alert data
  - Direct correlation links

### Automation Features

This repository includes sophisticated GitHub automation:

| Category | Count | Description |
|----------|-------|-------------|
| CI/CD Workflows | 32 | End-to-end automation pipelines |
| Python Automation Scripts | 5 | PR review, secret redaction, README generation |
| Security Scanning | 4 | Gitleaks, CodeQL, Dependency Review, Scorecard |
| Auto-Fix Capabilities | 1 | Bot-powered automatic remediation |

---

## Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                    GitHub Repository                            │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────────┐   │
│  │ Workflows/   │  │ _bot-scripts/│  │ docs/                │   │
│  │ (32 files)   │  │ (Python CLI)  │  │ (Deployment guides)  │   │
│  └──────────────┘  └──────────────┘  └──────────────────────┘   │
│         │                 │                     │                │
│         ▼                 ▼                     ▼                │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │              CLIProxyAPI (cliproxy.jclee.me)             │   │
│  │    ┌─────────────┐  ┌──────────────┐  ┌─────────────┐    │   │
│  │    │ PR Review   │  │ Issue Mgmt   │  │ Auto-Fix    │    │   │
│  │    │ (qodo-ai)   │  │ (Custom)     │  │ (Bot)       │    │   │
│  │    └─────────────┘  └──────────────┘  └─────────────┘    │   │
│  └─────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

### Component Overview

| Component | Location | Purpose |
|-----------|----------|---------|
| **Workflows** | `.github/workflows/` | GitHub Actions automation |
| **Bot Scripts** | `_bot-scripts/` | Python automation tools |
| **Documentation** | `docs/` | Deployment and operational docs |
| **Security Alerts** | `security_alert/` | FortiGate alert definitions |
| **Tests** | `tests/` | Validation test suite |
| **Demo** | `demo/` | Example configurations |

---

## Automation Inventory

### GitHub Workflows (32 Total)

#### Pull Request Automation

| Workflow | Description |
|----------|-------------|
| `01_branch-to-pr.yml` | Automatically create PR from feature branch |
| `02_issue-to-branch.yml` | Create branch from issue assignment |
| `03_pr-checks.yml` | Comprehensive PR validation pipeline |
| `10_pr-review.yml` | Automated PR review using qodo-ai/pr-agent |
| `security/11_pr-review.yml` | Security-focused PR review |
| `13_pr-auto-merge.yml` | Automatic PR merge when checks pass |
| `14_bot-auto-fix.yml` | Bot-powered automatic issue remediation |
| `15_merged-pr-cleanup.yml` | Post-merge cleanup operations |

#### Security Scanning

| Workflow | Description |
|----------|-------------|
| `04_actionlint.yml` | GitHub Actions syntax validation |
| `05_gitleaks.yml` | Secret scanning in commits |
| `06_codeql.yml` | CodeQL static analysis |
| `07_dependency-review.yml` | Dependency vulnerability scanning |
| `08_scorecard.yml` | Security posture assessment |

#### Issue Management

| Workflow | Description |
|----------|-------------|
| `18_issue-management.yml` | Automated issue triage and labeling |
| `19_issue-backfill.yml` | Issue metadata enrichment |
| `37_ci-failure-issues.yml` | Auto-create issues from CI failures |
| `43_reusable-issue-management.yml` | Reusable issue management logic |

#### Documentation Automation

| Workflow | Description |
|----------|-------------|
| `20_readme-gen.yml` | Automated README generation |
| `21_docs-sync.yml` | Documentation synchronization |
| `42_reusable-docs-sync.yml` | Reusable doc sync logic |

#### Release Engineering

| Workflow | Description |
|----------|-------------|
| `24_release-notes.yml` | Automated release notes generation |
| `25_release-publish.yml` | Release publication pipeline |
| `29_downstream-health-check.yml` | Downstream dependency health monitoring |

#### Continuous Integration

| Workflow | Description |
|----------|-------------|
| `ci.yml` | Primary CI pipeline |
| `60_ci-auto-heal.yml` | Automatic CI failure remediation |
| `auto-merge.yml` | Dependabot auto-merge automation |
| `labeler.yml` | Automatic label management |
| `welcome.yml` | New contributor welcome messages |

#### Reusable Workflows

| Workflow | Description |
|----------|-------------|
| `42_reusable-docs-sync.yml` | Cross-repository doc sync |
| `43_reusable-issue-management.yml` | Cross-repository issue handling |
| `44_reusable-pr-checks.yml` | Reusable PR validation |
| `45_reusable-gitleaks.yml` | Reusable secret scanning |

### Python Automation Tools

Located in `_bot-scripts/scripts/`:

| Script | Purpose |
|--------|---------|
| `pr_review_runner.py` | Execute automated PR reviews |
| `repo_review.py` | Repository-wide review automation |
| `redact_exposed_secrets.py` | Secret redaction for безопасность |
| `check_private_ips.py` | IP address validation |
| `generate_readme.py` | README.md generation |

### External Integrations

| Service | Integration Point | Purpose |
|---------|-------------------|---------|
| **qodo-ai/pr-agent** | PR Review | AI-powered code review |
| **CLIProxyAPI** | cliproxy.jclee.me | Bot command proxy |
| **Bot Service** | bot.jclee.me | Notification delivery |

---

## Quick Start

### Prerequisites

- Python 3.10+
- Docker (for containerized execution)
- Access to Splunk Enterprise/Cloud
- FortiGate devices with logging enabled

### Installation

```bash
# Clone the repository
git clone https://github.com/YOUR_ORG/splunk-fortigate-alerts.git
cd splunk-fortigate-alerts

# Install Python dependencies
pip install -r _bot-scripts/requirements.txt

# Install development dependencies
pip install -r _bot-scripts/requirements-dev.txt
```

### Basic Usage

1. **Configure Splunk Connection**

   ```bash
   export SPLUNK_HOST=https://your-splunk-instance
   export SPLUNK_TOKEN=your-api-token
   ```

2. **Deploy Alerts**

   ```bash
   make deploy
   ```

3. **Verify Alert Deployment**

   ```bash
   make verify
   ```

---

## Local Development

### Development Environment Setup

```bash
# Start development environment
make dev-env

# Run all tests
make test

# Run specific test suite
make test-unit
make test-integration
```

### Docker-Based Development

```bash
# Build GitHub App container
docker build -f _bot-scripts/Dockerfile.github_app -t fortigate-bot:app .

# Build GitHub Action container
docker build -f _bot-scripts/Dockerfile.github_action -t fortigate-bot:action .

# Run with docker-compose
docker-compose -f _bot-scripts/docker-compose.github_app.yml up
```

### Code Quality

```bash
# Run linting
make lint

# Run actionlint on workflows
make actionlint

# Format code
make format
```

---

## Commands Reference

### Makefile Targets

| Command | Description |
|---------|-------------|
| `make help` | Display available targets |
| `make dev-env` | Set up development environment |
| `make test` | Run all tests |
| `make test-unit` | Run unit tests only |
| `make test-integration` | Run integration tests |
| `make lint` | Run code linters |
| `make format` | Format code |
| `make actionlint` | Validate GitHub Actions syntax |
| `make deploy` | Deploy alerts to Splunk |
| `make verify` | Verify deployment |
| `make clean` | Clean build artifacts |

### Python Scripts

```bash
# Generate README
python _bot-scripts/scripts/generate_readme.py --output README.md

# Run PR review
python _bot-scripts/scripts/pr_review_runner.py --pr-url https://github.com/owner/repo/pull/1

# Check for private IPs
python _bot-scripts/scripts/check_private_ips.py --file alert.spl

# Redact exposed secrets
python _bot-scripts/scripts/redact_exposed_secrets.py --input log.txt --output clean.txt

# Repository review
python _bot-scripts/scripts/repo_review.py --repo-path .
```

---

## Contribution Guide

Please read [CONTRIBUTING.md](./CONTRIBUTING.md) for details on our development workflow and contribution standards.

### Contributing to Automation

1. **Fork the repository**
2. **Create a feature branch**

   ```bash
   git checkout -b feature/your-automation-name
   ```

3. **Make your changes**
   - Follow workflow naming conventions (`NN_name.yml`)
   - Add tests for new automation
   - Update documentation
4. **Run validation**

   ```bash
   make actionlint
   make test
   ```

5. **Submit a Pull Request**

### Adding New Security Alerts

1. Create alert definition in `security_alert/`
2. Follow naming convention: `alert_<category>_<name>.spl`
3. Include metadata header:

   ```spl
   /// alert: ALERT_NAME
   /// description: Brief description
   /// severity: HIGH|MEDIUM|LOW
   /// category: Category Name
   ```

4. Add corresponding test case
5. Update ALERT-REPOSITORY documentation

### Workflow Development Guidelines

| Rule | Description |
|------|-------------|
| **Naming** | Use 2-digit prefix for ordering (01-99) |
| **Idempotency** | Workflows must be safely re-runnable |
| **Documentation** | Every workflow needs `name:` and `description:` |
| **Secrets** | Use GitHub secrets, never hardcode credentials |
| **Timeout** | Set appropriate timeout-minutes |

---

## License

Proprietary software. See [LICENSE](./LICENSE) and [NOTICE](./_bot-scripts/NOTICE) for details.

---

## Support

- **Documentation**: See `docs/` directory
- **Security Issues**: See [SECURITY.md](./_bot-scripts/SECURITY.md)
- **Bugs**: Open an issue with `bug` label

---

## Additional Resources

| Resource | Link |
|----------|------|
| Bot Documentation | <https://bot.jclee.me> |
| CLIProxyAPI | <https://cliproxy.jclee.me> |
| PR-Agent (qodo-ai) | <https://github.com/qodo-ai/pr-agent> |

---

# 한국어 (Korean)

## 개요

이 저장소는 **FortiGate 방화벽 모니터링**을 위한 Splunk 보안 경고 시스템입니다. **15개의 운영 환경 경고**를 제공하며, 상태 인식 경고 기능을 통해 중복 알림을 방지합니다.

### 주요 특성

| 특성 | 설명 |
|------|------|
| **경고 수** | FortiGate 보안 이벤트 15개 |
| **아키텍처** | SPL 우선: 모든 로직이 Splunk 검색 처리 언어로 구현 |
| **경고 모델** | 상태 인식 및 중복 제거 |
| **알림** | 간결한 Slack 단일 라인 메시지 |
| **구성** | 매크로 기반 손쉬운 사용자 정의 |
| **배포** | 네트워크 격리 환경 준비 완료 |

## 빠른 시작

### 前提 조건

- Python 3.10+
- Docker
- Splunk Enterprise/Cloud 액세스
- 로깅이 활성화된 FortiGate 디바이스

### 설치

```bash
# 저장소 클론
git clone https://github.com/YOUR_ORG/splunk-fortigate-alerts.git
cd splunk-fortigate-alerts

# Python 의존성 설치
pip install -r _bot-scripts/requirements.txt

# 개발 의존성 설치
pip install -r _bot-scripts/requirements-dev.txt
```

### 기본 사용법

1. **Splunk 연결 구성**

   ```bash
   export SPLUNK_HOST=https://your-splunk-instance
   export SPLUNK_TOKEN=your-api-token
   ```

2. **경고 배포**

   ```bash
   make deploy
   ```

3. **배포 확인**

   ```bash
   make verify
   ```

## 자동화 목록

### GitHub 워크플로우 (32개)

| 카테고리 | 파일 | 설명 |
|----------|------|------|
| **PR 자동화** | `01-15_*.yml` | Pull Request 관련 자동화 |
| **보안 스캔** | `04-08_*.yml` | Gitleaks, CodeQL, 스코어카드 |
| **이슈 관리** | `18-19_*.yml`, `37_*.yml` | 이슈 트라이징 및 자동 생성 |
| **문서화** | `20-21_*.yml`, `42_*.yml` | README 생성, 문서 동기화 |
| **릴리스** | `24-25_*.yml` | 릴리스 노트 및 배포 |
| **CI/CD** | `ci.yml`, `60_*.yml` | 지속적 통합 및 자동 복구 |

### Python 자동화 도구

| 스크립트 | 용도 |
|----------|------|
| `pr_review_runner.py` | 자동 PR 리뷰 실행 |
| `repo_review.py` | 저장소 전체 검토 |
| `redact_exposed_secrets.py` | 노출된 시크릿 수정 |
| `check_private_ips.py` | IP 주소 검증 |
| `generate_readme.py` | README 생성 |

## 기여 가이드

상세한 개발 워크플로우와 기여 표준은 [CONTRIBUTING.md](./CONTRIBUTING.md)를 참조하세요.

1. **저장소 포크**
2. **기능 브랜치 생성**: `git checkout -b feature/your-feature`
3. **변경 사항 작성**
4. **유효성 검사 실행**: `make actionlint && make test`
5. **Pull Request 제출**

---

*Generated with [generate_readme.py](./_bot-scripts/scripts/generate_readme.py) • Last updated: 2025-01*
