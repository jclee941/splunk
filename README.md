# Security Alert (Splunk App) / 보안 알림 (Splunk 앱)

프로덕션급 Splunk 애드온으로, 통합 **알림 관리 대시보드**, 안내형 **손쉬운 알림 빌더**, 풀-기능 **알림 빌더**, 그리고 탐색형 **데이터 탐색기 대시보드**를 제공합니다. 안전한 포맷팅 헬퍼와 Slack 커스텀 알림 액션을 함께 제공하며, Python 런타임 의존성을 번들하여 격리(air-gapped) Splunk 환경에서도 동작합니다.

A production-grade Splunk add-on that delivers a unified **Alert Management Dashboard**, a guided **Easy Alert Builder**, a feature-complete **Alert Builder**, and an exploratory **Data Explorer Dashboard**. The app ships with a safe-formatting helper and a Slack custom alert action, and bundles its Python runtime dependencies so it runs on air-gapped Splunk deployments.

---

## Status / 운영 상태

| Runtime      | Status     | Owner      | Next action                                               |
| ------------ | ---------- | ---------- | --------------------------------------------------------- |
| Splunk 9.x   | Production | App author | `tar` the `security_alert/` folder, then install via *Manage Apps* |
| Splunk 8.x   | Compatible | App author | Validate dashboards against your 8.x instance after install |
| Air-gapped   | Supported  | App author | No external Python download required; deps are bundled     |

---

## Quick-flow summary / 빠른 흐름 요약

1. Build the package → bundle `security_alert/` into a `.tar.gz` named `security_alert.spl`.
2. Install the package → upload through Splunk Web or unpack into `$SPLUNK_HOME/etc/apps/`.
3. Configure routes → edit `default/alert_actions.conf` to enable the Slack action.
4. Author alerts → use the **Easy Alert Builder** or **Alert Builder** dashboard view.
5. Operate alerts → triage inside the **Alert Management Dashboard**; deep-dive in the **Data Explorer Dashboard**.

---

## 1. Overview / 개요

`security_alert/` is a self-contained Splunk app that turns ad-hoc alerting into a repeatable, auditable workflow. It targets SOC analysts, detection engineers, and Splunk administrators who need to:

- Author alerts through a guided **Easy Alert Builder** or a full **Alert Builder** UI.
- Triage, acknowledge, and audit alerts from a single **Alert Management Dashboard**.
- Investigate the events behind alerts in a **Data Explorer Dashboard**.
- Route alerts to Slack using the bundled `bin/slack.py` custom alert action.
- Operate on isolated Splunk instances because the Python dependencies (`urllib3`, `idna`, `charset_normalizer`, `six`) are bundled under `lib/python3/`.

`security_alert/`는 즉흥적인 알림 작성을 반복 가능하고 감사 가능한 워크플로로 전환하는 자족형 Splunk 앱입니다. SOC 분석가, 디텍션 엔지니어, Splunk 관리자가 다음을 수행할 때 사용합니다.

- 안내형 **손쉬운 알림 빌더** 또는 풀 기능 **알림 빌더** UI로 알림 작성
- 단일 **알림 관리 대시보드**에서 알림 분류·승인·감사
- **데이터 탐색기 대시보드**에서 알림背后 이벤트 조사
- 번들된 `bin/slack.py` 커스텀 알림 액션으로 Slack 라우팅
- `lib/python3/` 아래에 Python 의존성(`urllib3`, `idna`, `charset_normalizer`, `six`)이 번들되어 있어 격리 환경에서도 동작

---

## 2. Package contents / 패키지 구성

| Path                                              | Role                                              |
| ------------------------------------------------- | ------------------------------------------------- |
| `security_alert/app.manifest`                     | Splunk app manifest (version, author, dependencies) |
| `security_alert/default/app.conf`                 | App metadata, UI visibility, label                 |
| `security_alert/default/alert_actions.conf`       | Enables the `slack` custom alert action            |
| `security_alert/default/savedsearches.conf`       | Saved searches that back the dashboards           |
| `security_alert/default/macros.conf`              | Reusable SPL macros                               |
| `security_alert/default/props.conf`               | Field extraction / indexing props                 |
| `security_alert/default/transforms.conf`          | Field transforms and lookups                      |
| `security_alert/default/data/ui/nav/default.xml`  | Navigation entry points                           |
| `security_alert/default/data/ui/views/alert-builder.xml` | Full Alert Builder dashboard              |
| `security_alert/default/data/ui/views/easy_alert_builder.xml` | Guided Easy Alert Builder dashboard   |
| `security_alert/default/data/ui/views/alert-management-dashboard.xml` | Triage and acknowledge alerts     |
| `security_alert/default/data/ui/views/data-explorer-dashboard.xml` | Investigate underlying events         |
| `security_alert/bin/safe_fmt.py`                  | Safe string-formatting helper used by alert actions |
| `security_alert/bin/slack.py`                     | Slack custom alert action                          |
| `security_alert/bin/six.py`                       | Python 2/3 compatibility shim                      |
| `security_alert/lib/python3/urllib3/`             | Bundled HTTP client                                |
| `security_alert/lib/python3/idna-3.11.*`          | Bundled IDNA library                               |
| `security_alert/lib/python3/charset_normalizer-3.4.4.*` | Bundled charset normalizer                  |
| `security_alert/metadata/default.meta`            | App-level permissions metadata                     |

### First files to read / 먼저 읽을 파일

운영자가 가장 먼저 확인해야 할 파일은 다음과 같습니다.

| Order | File                                                | Why                                              |
| ----- | --------------------------------------------------- | ------------------------------------------------ |
| 1     | `security_alert/app.manifest`                       | Declares app version and supported Splunk versions |
| 2     | `security_alert/default/app.conf`                   | Controls UI labels and visibility                 |
| 3     | `security_alert/default/alert_actions.conf`         | Enables the Slack alert action                    |
| 4     | `security_alert/bin/slack.py`                       | Slack custom alert action entry point             |
| 5     | `docs/QUICK-START.md`                               | Step-by-step install and first-run guide          |

---

## 3. Architecture / 아키텍처

### 3.1 Component map

| Layer        | Component                          | File(s)                                         |
| ------------ | ---------------------------------- | ----------------------------------------------- |
| Navigation   | Default nav                        | `default/data/ui/nav/default.xml`               |
| Dashboard UI | Easy Alert Builder                 | `default/data/ui/views/easy_alert_builder.xml`  |
| Dashboard UI | Alert Builder                      | `default/data/ui/views/alert-builder.xml`       |
| Dashboard UI | Alert Management Dashboard         | `default/data/ui/views/alert-management-dashboard.xml` |
| Dashboard UI | Data Explorer Dashboard            | `default/data/ui/views/data-explorer-dashboard.xml` |
| Search layer | Saved searches + macros            | `default/savedsearches.conf`, `default/macros.conf` |
| Indexing     | Props + transforms                 | `default/props.conf`, `default/transforms.conf` |
| Action layer | Custom alert action (Slack)        | `bin/slack.py`, `default/alert_actions.conf`    |
| Helpers      | Safe formatting, six shim          | `bin/safe_fmt.py`, `bin/six.py`                 |
| Runtime      | Bundled Python dependencies        | `lib/python3/urllib3/`, `idna-3.11.*`, `charset_normalizer-3.4.4.*` |

### 3.2 Author → triage → route flow

1. Author — analyst opens **Easy Alert Builder** or **Alert Builder**, fills in SPL/macros, and saves a saved search.
2. Persist — `savedsearches.conf` and `macros.conf` store the alert definition.
3. Trigger — Splunk fires the saved search and invokes the `slack` alert action declared in `alert_actions.conf`.
4. Format — `bin/safe_fmt.py` sanitizes the message payload before rendering.
5. Route — `bin/slack.py` posts the alert to Slack over HTTPS using the bundled `urllib3`.
6. Operate — SOC team triages the alert in **Alert Management Dashboard**.
7. Investigate — analyst pivots to **Data Explorer Dashboard** for event-level drill-down.

### 3.3 Bilingual role table

| Layer        | 한국어                              | English                                |
| ------------ | ----------------------------------- | -------------------------------------- |
| Navigation   | 기본 내비게이션                     | Default navigation                     |
| Dashboard UI | 손쉬운 알림 빌더                    | Easy Alert Builder                     |
| Dashboard UI | 알림 빌더                           | Alert Builder                          |
| Dashboard UI | 알림 관리 대시보드                  | Alert Management Dashboard             |
| Dashboard UI | 데이터 탐색기 대시보드              | Data Explorer Dashboard                |
| Action layer | Slack 커스텀 알림 액션              | Slack custom alert action              |
| Runtime      | 번들된 Python 런타임                | Bundled Python runtime                 |

---

## 4. Quickstart / 빠른 시작

### 4.1 Build the `.spl` package

```bash
# From the repository root
tar -czf security_alert.spl security_alert/
```

### 4.2 Install on Splunk

```bash
# Option A — Splunk Web
# Manage Apps → Install app from file → upload security_alert.spl

# Option B — manual unpack (replace SPLUNK_HOME with your install path)
SPLUNK_HOME=/opt/splunk
tar -xzf security_alert.spl -C "$SPLUNK_HOME/etc/apps/"
"$SPLUNK_HOME/bin/splunk" restart
```

### 4.3 First-run checklist

| Step | Action                                                            | Expected result                           |
| ---- | ----------------------------------------------------------------- | ----------------------------------------- |
| 1    | Open Splunk Web → Apps → Security Alert                           | App appears in the launcher               |
| 2    | Open **Alert Management Dashboard**                               | Empty triage view renders without errors  |
| 3    | Open **Easy Alert Builder** and save a sample alert               | A new saved search appears under *Alerts* |
| 4    | Trigger the alert manually                                        | Slack action fires (if configured)        |

---

## 5. Configuration / 설정

### 5.1 Enable the Slack alert action

`security_alert/default/alert_actions.conf` registers the `slack` action. To turn it on, copy the stanza into `local/alert_actions.conf` and set `disabled = 0`:

```ini
[slack]
disabled = 0
icon_path = $SPLUNK_HOME/etc/apps/security_alert/bin/slack.py
label = Send to Slack
param.webhook_url = https://hooks.slack.com/services/REPLACE/WITH/YOUR_WEBHOOK
param.channel = #soc-alerts
```

`bin/slack.py` reads these parameters through Splunk's alert action API and posts the rendered message via the bundled `urllib3` client.

### 5.2 Safe formatting

`bin/safe_fmt.py` exposes a guarded string-formatting helper that alert actions call before sending payloads to external systems. It prevents accidental template-injection when alert content contains user-controlled fields.

### 5.3 Permissions

`security_alert/metadata/default.meta` controls app-level capabilities. Override in `local/meta.conf` if your environment restricts roles further.

---

## 6. Commands reference / 명령어 참조

| Command                                                                           | Purpose                                  |
| --------------------------------------------------------------------------------- | ---------------------------------------- |
| `tar -czf security_alert.spl security_alert/`                                     | Build the Splunk package                  |
| `tar -xzf security_alert.spl -C "$SPLUNK_HOME/etc/apps/"`                         | Install manually                         |
| `"$SPLUNK_HOME/bin/splunk" restart`                                               | Restart Splunk after install             |
| `"$SPLUNK_HOME/bin/splunk" reload app security_alert`                             | Reload after editing `default/*.conf`    |
| `"$SPLUNK_HOME/bin/splunk" display alert_actions slack -auth <user>:<pass>`        | Inspect Slack action parameters          |

---

## 7. Local development / 로컬 개발

| Area              | Path                            | Tip                                                       |
| ----------------- | ------------------------------- | --------------------------------------------------------- |
| UI editing        | `security_alert/default/data/ui/views/` | Edit XML, then reload the app in Splunk Web       |
| Search tuning     | `security_alert/default/savedsearches.conf` | Use `| sos` or `| rest` to inspect saved searches |
| Action scripts    | `security_alert/bin/`           | Run scripts directly with `python3` for unit-level debug |
| Permissions       | `security_alert/metadata/`      | Copy `default.meta` to `local/meta.conf` before editing  |
| Runtime deps      | `security_alert/lib/python3/`   | Vendored — do not replace with system packages           |

---

## 8. Testing / 테스트

| Test target        | How                                                                |
| ------------------ | ------------------------------------------------------------------ |
| Slack action       | Trigger a saved search manually from *Settings → Searches, reports, and alerts* |
| Easy Alert Builder | Save a sample alert and confirm it appears in *Alerts*             |
| Alert Builder      | Submit a complex query and validate the saved SPL                  |
| Alert Management   | Acknowledge an alert and verify state persistence                  |
| Data Explorer      | Click into an alert and confirm event drill-down                   |

There is no automated test harness shipped in the repository. Add unit tests under `security_alert/bin/tests/` if you introduce new alert actions or helpers.

---

## 9. Maintainers / 관리자

| Role             | Contact channel                                  |
| ---------------- | ------------------------------------------------ |
| App author       | Repository owner (see git history)               |
| Issue triage     | GitHub Issues on this repository                 |
| Security reports | Follow `CONTRIBUTING.md` disclosure policy       |

---

## 10. Further documentation / 추가 문서

| Document                                      | Purpose                                        |
| --------------------------------------------- | ---------------------------------------------- |
| `resume/API.md`                               | Alert action and helper API surface             |
| `resume/ARCHITECTURE.md`                      | Deep-dive architecture notes                   |
| `resume/DEPLOYMENT.md`                        | Production deployment guidance                  |
| `resume/TROUBLESHOOTING.md`                   | Common failures and fixes                       |
| `docs/QUICK-START.md`                         | Step-by-step first-run guide                    |
| `docs/DEPLOYMENT.md`                          | Deployment scenarios                            |
| `docs/RELEASE-NOTES.md`                       | Version-by-version change log                   |
| `docs/ALERT-REPOSITORY-XWIKI.md`              | Cross-wiki integration notes                    |
| `docs/LEGACY-CLEANUP-REPORT.md`               | Historical cleanup record                       |
| `demo/README.md`                              | Demo environment instructions                   |
| `security_alert/README.md`                    | App-level README inside the package             |
| `CONTRIBUTING.md`                             | Contribution and disclosure policy              |
| `LICENSE`                                     | License terms                                   |

---

## 11. License / 라이선스

See `LICENSE` at the repository root.