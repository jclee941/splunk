# Security Alert (Splunk App) / 보안 알림 (Splunk 앱)

프로덕션급 Splunk 애드온으로, 통합 **알림 관리 대시보드**, 안내형 **손쉬운 알림 빌더**, 풀-기능 **알림 빌더**, 그리고 탐색형 **데이터 탐색기 대시보드**를 제공합니다. 안전한 포맷팅 헬퍼와 Slack 커스텀 알림 액션을 함께 제공하며, Python 런타임 의존성을 번들하여 격리(air-gapped) Splunk 환경에서도 동작합니다.

A production-grade Splunk add-on that delivers a unified Alert Management Dashboard, a guided Easy Alert Builder, a feature-complete Alert Builder, and an exploratory Data Explorer Dashboard. The app ships with a safe-formatting helper and a Slack custom alert action, and bundles its Python runtime dependencies so it runs on air-gapped Splunk deployments.

---

## Status / 운영 상태

| Runtime     | Status     | Owner      | Next action                                                  |
| ----------- | ---------- | ---------- | ------------------------------------------------------------ |
| Splunk 9.x  | Production | App author | `tar` the `security_alert/` folder, then install via *Manage Apps* |
| Splunk 8.x  | Compatible | App author | Validate dashboards against your 8.x instance after install |
| Air-gapped  | Supported  | App author | No external Python download required; dependencies are bundled |

---

## Quick-flow summary / 빠른 흐름 요약

1. Build the package — bundle the `security_alert/` folder into a `.tar.gz` named `security_alert.spl`.
2. Install the package — upload through Splunk Web or unpack into `$SPLUNK_HOME/etc/apps/`.
3. Configure routes — edit `default/alert_actions.conf` to enable the Slack action.
4. Author alerts — use the **Easy Alert Builder** or **Alert Builder** dashboard view.
5. Operate alerts — triage inside the **Alert Management Dashboard**; deep-dive in the **Data Explorer Dashboard**.

---

## 1. Purpose / 목적

`security_alert/` 는 임시 알림 작성을 반복 가능하고 감사 가능한 워크플로우로 전환하는 자급식 Splunk 앱입니다. 다음 사용자를 대상으로 합니다.

- 탐지 엔지니어 — 일관된 형식으로 알림을 작성해야 하는 사람
- SOC 분석가 — 알림을 분류하고 후속 조치를 취하는 사람
- Splunk 관리자 — 알림 자산을 안전하게 패키징하고 배포하는 사람

주요 사용자 시나리오는 다음과 같습니다.

| Capability                  | User role          | View / Entry point                              |
| --------------------------- | ------------------ | ----------------------------------------------- |
| Guided alert authoring      | SOC analyst        | `default/data/ui/views/easy_alert_builder.xml`  |
| Full alert authoring        | Detection engineer | `default/data/ui/views/alert-builder.xml`       |
| Alert triage and lifecycle   | SOC analyst        | `default/data/ui/views/alert-management-dashboard.xml` |
| Exploratory data review     | Detection engineer | `default/data/ui/views/data-explorer-dashboard.xml` |
| Slack-based fan-out         | All roles          | `bin/slack.py` invoked via `default/alert_actions.conf` |

---

## 2. Package Contents / 패키지 구성

| Path                          | Purpose / 설명                                                        |
| ----------------------------- | --------------------------------------------------------------------- |
| `security_alert/app.manifest` | Splunk App manifest declaring app identity and version                |
| `security_alert/default/`     | Splunk configuration files (`.conf`) shipped as defaults              |
| `security_alert/metadata/`    | Localized metadata, including `default.meta` capability declarations  |
| `security_alert/bin/`         | Custom alert scripts: `slack.py`, `safe_fmt.py`, `six.py`             |
| `security_alert/lib/python3/` | Bundled Python dependencies (urllib3, idna, charset_normalizer)       |
| `security_alert/default/data/ui/views/` | Splunk dashboard XML: alert builder, easy builder, management, explorer |
| `docs/`                       | Project documentation (quick start, deployment, release notes)        |
| `resume/`                     | Engineering resume artifacts and architecture notes                  |
| `demo/`                       | Demo material for offline walkthroughs                                |

---

## 3. First Files to Read / 먼저 읽을 파일

| Order | File                                           | Why read it / 읽는 이유                                  |
| ----- | ---------------------------------------------- | -------------------------------------------------------- |
| 1     | `security_alert/app.manifest`                  | Confirms app id, version, and supported Splunk versions  |
| 2     | `security_alert/default/app.conf`              | Author, label, and UI hookup                             |
| 3     | `security_alert/default/alert_actions.conf`    | Where the Slack alert action is declared and configured  |
| 4     | `security_alert/bin/slack.py`                  | Runtime behavior of the Slack fan-out                    |
| 5     | `security_alert/bin/safe_fmt.py`               | Safe string formatting helper used by alert scripts      |
| 6     | `docs/QUICK-START.md`                          | End-to-end install and first-alert walkthrough            |
| 7     | `docs/DEPLOYMENT.md`                           | Air-gapped install, file-system layout, and permissions  |

---

## 4. API / Entry Points / 진입 지점

| Surface                  | Type                    | Default location                                         |
| ------------------------ | ----------------------- | -------------------------------------------------------- |
| Slack custom alert       | Alert action script     | `bin/slack.py`                                            |
| Safe formatting helper   | Python utility module   | `bin/safe_fmt.py`                                         |
| Compatibility shim       | Python 2/3 helper       | `bin/six.py`                                              |
| Easy Alert Builder view  | Splunk dashboard XML    | `default/data/ui/views/easy_alert_builder.xml`           |
| Alert Builder view       | Splunk dashboard XML    | `default/data/ui/views/alert-builder.xml`                |
| Alert Management view    | Splunk dashboard XML    | `default/data/ui/views/alert-management-dashboard.xml`   |
| Data Explorer view       | Splunk dashboard XML    | `default/data/ui/views/data-explorer-dashboard.xml`      |
| Navigation                | XML nav file            | `default/data/ui/nav/default.xml`                        |

자세한 동작 계약은 [`resume/API.md`](resume/API.md) 와 [`resume/ARCHITECTURE.md`](resume/ARCHITECTURE.md) 를 참조하세요.

---

## 5. Architecture / 아키텍처

### 5.1 Components

| Component             | Role                                                        |
| --------------------- | ----------------------------------------------------------- |
| Dashboards (XML)      | Author and triage alerts inside Splunk Web                  |
| Alert action scripts  | Send alerts to external systems (for example Slack)        |
| Saved searches        | Drive alert evaluation on a schedule                        |
| Macros and transforms | Reusable SPL fragments and field transformations            |
| Bundled Python libs   | Offline-safe runtime (urllib3, idna, charset_normalizer)    |

### 5.2 Request flow (alert authoring → fan-out)

1. Analyst opens the **Easy Alert Builder** or **Alert Builder** view.
2. Form-driven XML captures search, trigger condition, and recipients.
3. Splunk persists the saved search and links it to the Slack alert action.
4. On trigger, Splunk invokes `bin/slack.py` with the action payload.
5. `slack.py` calls `safe_fmt.py` to render messages without unsafe formatting.
6. `slack.py` posts the payload through the bundled `urllib3` client.
7. The **Alert Management Dashboard** tracks trigger history and ownership.
8. The **Data Explorer Dashboard** supports ad-hoc investigation of the same data set.

---

## 6. Quickstart / 빠른 시작

### 6.1 Build

`security_alert/` 디렉터리만 묶어서 표준 Splunk 앱 아카이브 형식으로 만듭니다.

```bash
tar -czf security_alert.spl security_alert/
```

### 6.2 Install

| Method              | Steps                                                                                                |
| ------------------- | ---------------------------------------------------------------------------------------------------- |
| Splunk Web          | *Manage Apps* → *Install app from file* → select `security_alert.spl`                                |
| File system         | Extract into `$SPLUNK_HOME/etc/apps/`, then run `splunk restart`                                      |
| Air-gapped host     | Copy the bundle via removable media, then follow the file-system method above                        |

### 6.3 Configure the Slack action

`security_alert/default/alert_actions.conf` 을 검토하고, 채널, webhook, 그리고 토큰 값을 환경에 맞게 조정합니다. 자세한 절차는 [`docs/DEPLOYMENT.md`](docs/DEPLOYMENT.md) 를 참조하세요.

### 6.4 Author your first alert

1. Splunk Web 에서 앱 *Security Alert* 을 선택합니다.
2. *Easy Alert Builder* 를 열어 안내된 양식을 채웁니다.
3. 저장을 누르면 검색이 활성화되고 Slack 액션이 연결됩니다.
4. *Alert Management Dashboard* 에서 결과를 확인합니다.

### 6.5 Detailed walkthrough

| Topic              | Document                                          |
| ------------------ | ------------------------------------------------- |
| Install and first run | [`docs/QUICK-START.md`](docs/QUICK-START.md)   |
| Deployment and air-gap | [`docs/DEPLOYMENT.md`](docs/DEPLOYMENT.md)    |
| API and contracts  | [`resume/API.md`](resume/API.md)                   |
| Architecture notes | [`resume/ARCHITECTURE.md`](resume/ARCHITECTURE.md) |
| Troubleshooting    | [`resume/TROUBLESHOOTING.md`](resume/TROUBLESHOOTING.md) |

---

## 7. Configuration / 설정

| File                                     | Purpose / 설명                                    |
| ---------------------------------------- | ------------------------------------------------- |
| `security_alert/default/app.conf`        | App identity, label, UI hookup                    |
| `security_alert/default/alert_actions.conf` | Declares the Slack custom alert action          |
| `security_alert/default/macros.conf`     | Reusable SPL macros shared by dashboards          |
| `security_alert/default/props.conf`      | Field extraction and indexing-time rules          |
| `security_alert/default/transforms.conf` | Field transforms referenced by props and searches |
| `security_alert/default/savedsearches.conf` | Scheduled searches that drive alerting         |
| `security_alert/metadata/default.meta`   | Capability declarations for the app               |

운영 환경에서는 `default/` 를 직접 수정하지 말고 `local/` 에서 재정의하세요. 그러면 앱 업그레이드 시에도 변경 사항이 유지됩니다.

---

## 8. Commands Reference / 명령어 참조

| Command                                                       | Purpose / 설명                                   |
| ------------------------------------------------------------- | ------------------------------------------------ |
| `tar -czf security_alert.spl security_alert/`                  | Build the installable Splunk archive             |
| `splunk restart`                                              | Restart Splunk after installing the app          |
| `./bin/splunk display alert`                                   | Inspect a configured alert                       |
| `./bin/splunk search '\| rest /services/saved/searches'`       | List alerts defined in the app                  |

---

## 9. Local Development / 로컬 개발

| Step                       | Action                                                                                       |
| -------------------------- | -------------------------------------------------------------------------------------------- |
| 1. Pick a working copy    | Place `security_alert/` under `$SPLUNK_HOME/etc/apps/` on a dev Splunk instance               |
| 2. Iterate on dashboards   | Edit XML files under `default/data/ui/views/`, then refresh the browser view                  |
| 3. Iterate on Python       | Modify scripts under `bin/`; Splunk re-imports them on next invocation                       |
| 4. Validate bundled deps   | Confirm `lib/python3/` contents match the declared runtime requirements                      |
| 5. Rebuild the archive     | Re-run the `tar` command from §6.1 whenever the bundle should be redistributed               |

권한과 격리 배포 절차는 [`docs/DEPLOYMENT.md`](docs/DEPLOYMENT.md) 를 참조하세요.

---

## 10. Testing / 테스트

| Surface                 | How to validate / 검증 방법                                                     |
| ----------------------- | ------------------------------------------------------------------------------- |
| Slack alert script      | Run `python3 bin/slack.py` against a known fixture and confirm output           |
| Safe formatting helper  | Unit-call `safe_fmt.py` with adversarial inputs                                 |
| Saved searches          | Trigger manually inside Splunk and review the **Alert Management Dashboard**     |
| Dashboards              | Open each view in Splunk Web and confirm panels load without errors             |
| Bundled dependencies    | Verify import paths under `lib/python3/` resolve on a clean Splunk Python runtime |

문제 해결 절차는 [`resume/TROUBLESHOOTING.md`](resume/TROUBLESHOOTING.md) 를 참조하세요.

---

## 11. Maintainers / Points of Contact / 관리자 및 연락처

| Role                | Owner      | Notes                                                              |
| ------------------- | ---------- | ------------------------------------------------------------------ |
| App author          | App author | Primary contact for the app, dashboards, and alert actions         |
| Documentation       | App author | Owns `docs/` and `resume/` content                                 |
| Release engineering | App author | Owns packaging, version bumps, and distribution                   |

거버넌스 절차는 [`CONTRIBUTING.md`](CONTRIBUTING.md) 를 참조하세요.

---

## 12. Further Documentation / 추가 문서

| Topic                                | Document                                                |
| ------------------------------------ | ------------------------------------------------------- |
| Quick start                          | [`docs/QUICK-START.md`](docs/QUICK-START.md)            |
| Deployment and air-gap               | [`docs/DEPLOYMENT.md`](docs/DEPLOYMENT.md)              |
| Release notes                        | [`docs/RELEASE-NOTES.md`](docs/RELEASE-NOTES.md)        |
| Legacy cleanup report                | [`docs/LEGACY-CLEANUP-REPORT.md`](docs/LEGACY-CLEANUP-REPORT.md) |
| XWiki alert repository context       | [`docs/ALERT-REPOSITORY-XWIKI.md`](docs/ALERT-REPOSITORY-XWIKI.md) |
| Architecture notes                   | [`resume/ARCHITECTURE.md`](resume/ARCHITECTURE.md)      |
| API contracts                        | [`resume/API.md`](resume/API.md)                        |
| Deployment playbook                  | [`resume/DEPLOYMENT.md`](resume/DEPLOYMENT.md)          |
| Troubleshooting                      | [`resume/TROUBLESHOOTING.md`](resume/TROUBLESHOOTING.md) |
| Demo material                        | [`demo/README.md`](demo/README.md)                      |
| Contributing guide                   | [`CONTRIBUTING.md`](CONTRIBUTING.md)                    |
| License                              | [`LICENSE`](LICENSE)                                    |