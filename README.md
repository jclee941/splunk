# Security Alert (Splunk App) / 보안 알림 (Splunk 앱)

A production-grade Splunk add-on that delivers a unified **Alert Management Dashboard**, a guided **Easy Alert Builder**, a feature-complete **Alert Builder**, and an exploratory **Data Explorer Dashboard**. The app ships with a safe-formatting helper and a Slack custom alert action, and bundles its Python runtime dependencies so it runs on air-gapped Splunk deployments.

프로덕션급 Splunk 애드온으로, 통합 **알림 관리 대시보드**, 안내형 **손쉬운 알림 빌더**, 풀-기능 **알림 빌더**, 그리고 탐색형 **데이터 탐색기 대시보드**를 제공합니다. 안전한 포맷팅 헬퍼와 Slack 커스텀 알림 액션을 함께 제공하며, Python 런타임 의존성을 번들하여 격리(air-gapped) Splunk 환경에서도 동작합니다.

| Runtime      | Status     | Owner        | Next action             |
| ------------ | ---------- | ------------ | ----------------------- |
| Splunk 9.x   | Production | App author   | `tar` the `security_alert/` folder, then install via *Manage Apps* |

---

## Quick-flow summary

1. Build the package → bundle `security_alert/` into a `.tar.gz` named `security_alert.spl`.
2. Install the package → upload through Splunk Web or unpack into `$SPLUNK_HOME/etc/apps/`.
3. Configure routes → edit `default/alert_actions.conf` to enable the Slack action.
4. Author alerts → use the **Easy Alert Builder** or **Alert Builder** dashboard view.
5. Operate alerts → triage inside the **Alert Management Dashboard**; deep-dive in the **Data Explorer Dashboard**.

---

## 1. Overview / 개요

`security_alert/` is a self-contained Splunk app that converts ad-hoc alerting into a repeatable, auditable workflow. It targets SOC analysts, detection engineers, and Splunk administrators who need to:

- Author alerts through either a guided **Easy Alert Builder** or a full **Alert Builder** UI.
- Triage, acknowledge, and audit alerts from one **Alert Management Dashboard**.
- Investigate the events behind alerts in a **Data Explorer Dashboard**.
- Route alerts to Slack using the bundled `bin/slack.py` custom alert action.
- Operate on isolated Splunk instances, because the Python runtime dependencies (`urllib3`, `idna`, `charset_normalizer`, `six`) are vendored under `security_alert/lib/python3/`.

### 1.1 Target users / 대상 사용자

| Role                      | Primary use                                       |
| ------------------------- | ------------------------------------------------- |
| SOC analyst               | Triage live alerts in the management dashboard   |
| Detection engineer        | Author and version alerts through builders        |
| Splunk administrator      | Install, configure, and back up the app           |
| IR / on-call responder    | Consume alerts delivered to Slack channels        |

### 1.2 Production-readiness

| Aspect                    | State                                                    |
| ------------------------- | -------------------------------------------------------- |
| Release channel           | Production-ready                                         |
| Air-gap support           | Supported (vendored Python libs)                         |
| Deprecated paths          | None published                                           |
| Compatibility             | Splunk Enterprise / Splunk Cloud 9.x                     |

---

## 2. Features / 주요 기능

| Component                    | Purpose                                                            |
| ---------------------------- | ------------------------------------------------------------------ |
| `bin/slack.py`               | Custom alert action that posts formatted alerts to Slack           |
| `bin/safe_fmt.py`            | Whitelisted formatter used by `slack.py` to render event payloads  |
| Alert Management Dashboard   | Unified triage surface for live alerts                             |
| Easy Alert Builder           | Guided wizard for non-power-user alert authoring                   |
| Alert Builder                | Full editor for advanced alert configuration                       |
| Data Explorer Dashboard      | Ad-hoc investigation surface tied to alert results                 |
| `savedsearches.conf`         | Scheduled and on-demand alert definitions                           |
| `transforms.conf`            | Lookup and field transformation configuration                      |
| `macros.conf` / `props.conf` | Search-time normalization building blocks                           |
| `alert_actions.conf`         | Wiring of the Slack action and built-in webhook fallbacks           |
| Vendored `lib/python3/`      | Offline-capable runtime (urllib3, idna, charset_normalizer, six)    |

---

## 3. Architecture / 아키텍처

### 3.1 Component map

| Layer            | Artifact                                                      | Role                                  |
| ---------------- | ------------------------------------------------------------- | ------------------------------------- |
| UI (XML views)   | `default/data/ui/views/*.xml`                                 | Dashboards and alert authoring UIs     |
| UI navigation    | `default/data/ui/nav/default.xml`                             | Menu entries and grouping             |
| Search knowledge | `default/macros.conf`, `default/props.conf`                   | Reusable search and field rules       |
| Alert logic      | `default/savedsearches.conf`, `default/transforms.conf`       | Scheduled and on-demand alert logic    |
| Actions          | `default/alert_actions.conf`, `bin/slack.py`, `bin/safe_fmt.py` | Outbound delivery and formatting    |
| App metadata     | `default/app.conf`, `metadata/default.meta`, `app.manifest`   | Identity, ACLs, package manifest       |
| Runtime          | `lib/python3/urllib3`, `lib/python3/idna`, `lib/python3/charset_normalizer`, `bin/six.py` | Vendored Python runtime |

### 3.2 Request flow (alert authoring and delivery)

1. Operator opens the **Easy Alert Builder** or **Alert Builder** view from the navigation menu.
2. The view writes a search entry to `default/savedsearches.conf` (or the user-level equivalent).
3. Splunk schedules the saved search; matching events flow through `macros.conf` and `props.conf`.
4. The `slack.py` custom alert action is invoked per-trigger.
5. `slack.py` delegates payload rendering to `safe_fmt.py` (whitelisted tokens only).
6. The formatted message is posted to the configured Slack channel.

### 3.3 Deployment topology

- Runs on a single Splunk search head or a search-head cluster.
- The `security_alert/` folder is the self-contained deployment unit.
- Vendored Python libs live under `lib/python3/`, isolated from system Python.

---

## 4. Repository Layout / 저장소 구조

| Path                              | Purpose                                       |
| --------------------------------- | --------------------------------------------- |
| `README.md`                       | This document                                 |
| `CONTRIBUTING.md`                 | Contribution guidelines                       |
| `LICENSE`                         | License terms                                 |
| `security_alert/`                 | The Splunk app package (deployment unit)      |
| `security_alert/bin/`             | Executable scripts (alert actions, helpers)   |
| `security_alert/default/`         | App config, saved searches, transforms, views |
| `security_alert/default/data/ui/` | XML dashboards and navigation                 |
| `security_alert/lib/python3/`     | Vendored Python runtime dependencies          |
| `security_alert/metadata/`        | ACL metadata for the app                     |
| `security_alert/app.manifest`     | Splunk app manifest                           |
| `docs/`                           | Reference documentation                       |
| `docs/QUICK-START.md`             | Onboarding walkthrough                        |
| `docs/DEPLOYMENT.md`              | Splunk deployment guide                       |
| `docs/RELEASE-NOTES.md`           | Versioned change log                         |
| `docs/LEGACY-CLEANUP-REPORT.md`   | History of cleanup activities                 |
| `docs/ALERT-REPOSITORY-XWIKI.md`  | Cross-team wiki reference                     |
| `resume/`                         | Preserved earlier doc set (API, architecture, deployment, troubleshooting) |
| `demo/`                           | Demo material                                 |

---

## 5. Quick Start / 빠른 시작

### 5.1 Prerequisites

- Splunk Enterprise or Splunk Cloud 9.x.
- Write access to `$SPLUNK_HOME/etc/apps/` (admin role or equivalent).
- Optional: an outbound Slack webhook URL or workspace credentials, depending on the alert action mode.

### 5.2 Install

| Step | Command / action                                                                                          |
| ---- | --------------------------------------------------------------------------------------------------------- |
| 1    | `cd` to the repository root.                                                                              |
| 2    | `tar -czf security_alert.spl security_alert/` to build the installable package.                            |
| 3    | In Splunk Web, go to *Apps → Manage Apps → Install app from file* and upload `security_alert.spl`.        |
| 4    | Restart Splunk if prompted.                                                                               |
| 5    | Open *Security Alert* from the app launcher to confirm the dashboards load.                                |

For air-gapped deployments, skip the network step; the vendored runtime under `lib/python3/` is sufficient.

### 5.3 Verify

- The new **Security Alert** menu item appears in the Splunk app bar.
- The four views (`alert-builder`, `easy_alert_builder`, `alert-management-dashboard`, `data-explorer-dashboard`) render without error.
- A test saved search fires and routes through `bin/slack.py` when a channel is configured.

---

## 6. Configuration / 설정

| File                                | Key parameters                                                                                |
| ----------------------------------- | --------------------------------------------------------------------------------------------- |
| `default/app.conf`                  | App `id`, `version`, label, owner, and restart flags                                          |
| `default/alert_actions.conf`        | Enable the Slack action; set webhook or token values                                         |
| `default/savedsearches.conf`        | Pre-built scheduled alerts and their cron schedules                                           |
| `default/transforms.conf`           | Lookup table definitions and field overrides                                                  |
| `default/macros.conf`               | Reusable search macros (`( … )` snippets)                                                    |
| `default/props.conf`                | Field extraction, indexing-time, and search-time settings                                     |
| `metadata/default.meta`             | App-level ACLs and capability gating                                                          |

### 6.1 Slack action

Configure outbound delivery by editing `default/alert_actions.conf` and (optionally) the alert action UI inputs inside the dashboard XML. Treat webhook URLs as secrets; store them in `local/` or `system/local/` rather than `default/`.

### 6.2 Recommended overrides

Use a `local/` directory inside the app to layer environment-specific changes on top of `default/`. Splunk loads `local/` last and wins precedence.

---

## 7. Component Reference / 구성 요소 참조

### 7.1 Custom alert actions (bin/)

| Script            | Role                                                            |
| ----------------- | --------------------------------------------------------------- |
| `slack.py`        | Posts a formatted alert payload to a Slack channel or webhook   |
| `safe_fmt.py`     | Whitelisted template renderer; sanitizes dynamic fields         |
| `six.py`          | Compatibility shim (vendored copy from the `six` project)       |

### 7.2 Dashboards (views)

| View                              | Purpose                                                |
| --------------------------------- | ------------------------------------------------------ |
| `alert-builder.xml`               | Full alert authoring editor                            |
| `easy_alert_builder.xml`          | Step-by-step wizard for non-power-users                |
| `alert-management-dashboard.xml`  | Triage queue, ack, audit                               |
| `data-explorer-dashboard.xml`     | Drill into the events that triggered or matched alerts |

### 7.3 Search knowledge

Saved searches in `default/savedsearches.conf` are the executable half of the app; dashboards present them, macros and props normalize the inputs, and transforms enrich results before they reach an alert action.

---

## 8. Local Development / 로컬 개발

| Step | Action                                                                                            |
| ---- | ------------------------------------------------------------------------------------------------- |
| 1    | Clone the repository and copy `security_alert/` into a sandboxed `$SPLUNK_HOME/etc/apps/`.        |
| 2    | Create a per-developer `security_alert/local/` to override settings without touching `default/`.   |
| 3    | Edit XML views with the Splunk Dashboard Editor or a text editor; reload dashboards from Splunk Web. |
| 4    | Iterate on `bin/slack.py` against a scratch saved search that you can run on-demand.              |
| 5    | Use the **Data Explorer Dashboard** to verify SPL changes before promoting them to production.    |

### 8.1 Coding conventions

- Keep `default/` immutable for shared baselines; mutate `local/` instead.
- Render any operator-supplied strings through `safe_fmt.py` before sending them to Slack.
- Treat `lib/python3/` as vendored; do not edit vendored packages.

---

## 9. Testing / 테스트

| Layer                  | Approach                                                                  |
| ---------------------- | ------------------------------------------------------------------------- |
| Search validation      | Run saved searches by hand against a fixture index                        |
| Renderer tests         | Exercise `safe_fmt.py` against known payloads to confirm whitelisting     |
| Dashboard reachability | Open each XML view in Splunk Web after a reload                           |
| Delivery check         | Configure a sandbox Slack channel and fire a test alert                   |
| Upgrade hygiene        | Compare `default/` against the prior release's `default/` before tagging   |

---

## 10. Operations / 운영

| Concern               | Guidance                                                                                       |
| --------------------- | ---------------------------------------------------------------------------------------------- |
| Backups               | Snapshot `security_alert/default/` and `security_alert/local/` between releases               |
| Upgrades              | Install the new package over the existing app; review `docs/RELEASE-NOTES.md` before upgrading |
| Air-gapped installs   | Use the bundled `lib/python3/` runtime; ensure outgoing connectivity exists for Slack delivery |
| Capacity              | Saved searches inherit Splunk scheduler limits; alert actions add per-trigger work             |
| Secret handling       | Store tokens and webhooks in `local/` or `system/local/`; never commit them to `default/`       |
| Cleanup history       | See `docs/LEGACY-CLEANUP-REPORT.md` for what was removed and why                              |

---

## 11. Documentation Index / 문서 인덱스

| Doc                                                         | Topic                                       |
| ----------------------------------------------------------- | ------------------------------------------- |
| `docs/QUICK-START.md`                                       | Onboarding walkthrough                      |
| `docs/DEPLOYMENT.md`                                        | Production deployment                       |
| `docs/RELEASE-NOTES.md`                                     | Change log and upgrade notes                |
| `docs/LEGACY-CLEANUP-REPORT.md`                             | Cleanup history                             |
| `docs/ALERT-REPOSITORY-XWIKI.md`                            | Cross-team wiki reference                   |
| `resume/API.md`                                             | API surface (preserved earlier doc set)     |
| `resume/ARCHITECTURE.md`                                    | Architecture notes (preserved earlier set)  |
| `resume/DEPLOYMENT.md`                                      | Deployment notes (preserved earlier set)    |
| `resume/TROUBLESHOOTING.md`                                 | Troubleshooting guide (preserved earlier)   |
| `demo/README.md`                                            | Demo material                               |

---

## 12. Contributing / 기여

See `CONTRIBUTING.md` for the full policy. Highlights:

- Keep patches scoped and reversible; prefer flag-gated changes.
- Update `docs/RELEASE-NOTES.md` when behavior changes.
- Do not commit secrets, internal hostnames, or environment-specific overrides.
- Run the local smoke checklist in *Local Development / Testing* before opening a review.

---

## 13. License / 라이선스

This project is released under the terms in `LICENSE`.