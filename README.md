# Security Alert (Splunk App) / 보안 알림 (Splunk 앱)

A production-grade Splunk add-on that delivers a unified **Alert Management Dashboard**, a guided **Easy Alert Builder**, a feature-complete **Alert Builder**, and an exploratory **Data Explorer Dashboard**. The app ships with a safe-formatting helper and a Slack custom alert action, and bundles its Python runtime dependencies so it runs on air-gapped Splunk deployments.

프로덕션급 Splunk 애드온으로, 통합 **알림 관리 대시보드**, 안내형 **손쉬운 알림 빌더**, 풀-기능 **알림 빌더**, 그리고 탐색형 **데이터 탐색기 대시보드**를 제공합니다. 안전한 포맷팅 헬퍼와 Slack 커스텀 알림 액션을 함께 제공하며, Python 런타임 의존성을 번들하여 격리(air-gapped) Splunk 환경에서도 동작합니다.

---

## Table of Contents / 목차

1. [Overview / 개요](#1-overview--개요)
2. [Features / 주요 기능](#2-features--주요-기능)
3. [Architecture / 아키텍처](#3-architecture--아키텍처)
4. [Repository Layout / 저장소 구조](#4-repository-layout--저장소-구조)
5. [Quick Start / 빠른 시작](#5-quick-start--빠른-시작)
6. [Configuration / 설정](#6-configuration--설정)
7. [Component Reference / 구성 요소 참조](#7-component-reference--구성-요소-참조)
8. [Local Development / 로컬 개발](#8-local-development--로컬-개발)
9. [Testing / 테스트](#9-testing--테스트)
10. [Operations / 운영](#10-operations--운영)
11. [Documentation Index / 문서 인덱스](#11-documentation-index--문서-인덱스)
12. [Contributing / 기여](#12-contributing--기여)
13. [License / 라이선스](#13-license--라이선스)

---

## 1. Overview / 개요

`security_alert/` is a self-contained Splunk app that converts ad-hoc alerting into a repeatable, auditable workflow. It targets SOC analysts, detection engineers, and Splunk administrators who need to:

- Author alerts through either a guided **Easy Alert Builder** or a full **Alert Builder** UI.
- Triage, acknowledge, and audit alerts from one **Alert Management Dashboard**.
- Investigate the events behind alerts in a **Data Explorer Dashboard**.
- Route alerts to Slack using the bundled `bin/slack.py` custom alert action.
- Operate on isolated Splunk instances, because the Python runtime dependencies (`urllib3`, `idna`, `charset_normalizer`, `six`) are vendored under `security_alert/lib/python3/`.

`security_alert/`은(는) 임시 알림 작성을 반복 가능하고 감사 가능한 워크플로우로 변환하는 독립형 Splunk 앱입니다. 대상 사용자는 다음과 같은 요구사항을 가진 SOC 분석가, 탐지 엔지니어, Splunk 관리자입니다.

- **Easy Alert Builder** 안내 UI 또는 풀 기능 **Alert Builder** UI로 알림을 작성합니다.
- 단일 **Alert Management Dashboard**에서 알림을 분류·승인·감사합니다.
- **Data Explorer Dashboard**에서 알림의 근거가 되는 이벤트를 조사합니다.
- 번들된 `bin/slack.py` 커스텀 알림 액션으로 알림을 Slack으로 라우팅합니다.
- Python 런타임 의존성(`urllib3`, `idna`, `charset_normalizer`, `six`)이 `security_alert/lib/python3/`에 동봉되어 있으므로 인터넷에 연결되지 않은 격리 Splunk 인스턴스에서도 동작합니다.

### Intended Audience / 대상 사용자

| Role / 역할 | Use Case / 사용 사례 |
|---|---|
| SOC Analyst / SOC 분석가 | Triage alerts, acknowledge and track resolution from one dashboard / 단일 대시보드에서 알림 분류·승인·해결 추적 |
| Detection Engineer / 탐지 엔지니어 | Author and tune alerts via Easy Alert Builder or full Alert Builder / Easy Alert Builder 또는 풀 Alert Builder로 알림 작성 및 튜닝 |
| Splunk Administrator / Splunk 관리자 | Deploy a self-contained app to air-gapped Splunk instances / 인터넷이 분리된 Splunk 인스턴스에 자체 완비 앱 배포 |
| Incident Responder / 인시던트 대응자 | Pivot from an alert into the Data Explorer for root-cause investigation / 알림에서 Data Explorer로 피벗하여 근본 원인 조사 |

---

## 2. Features / 주요 기능

| Feature / 기능 | Description / 설명 |
|---|---|
| **Alert Management Dashboard** / 알림 관리 대시보드 | Single-pane triage, acknowledgement, and audit view / 단일 화면에서 분류·승인·감사 |
| **Easy Alert Builder** / 손쉬운 알림 빌더 | Guided UI for non-experts / 비전문가용 안내형 작성 UI |
| **Alert Builder** / 알림 빌더 | Full authoring surface with all Splunk alert knobs / 모든 Splunk 알림 옵션을 갖춘 풀 작성 화면 |
| **Data Explorer Dashboard** / 데이터 탐색기 대시보드 | Drill-down exploration of the events that drove an alert / 알림을 유발한 이벤트의 드릴다운 탐색 |
| **Slack Custom Alert Action** / Slack 커스텀 알림 액션 | `bin/slack.py` ships notifications to Slack channels / `bin/slack.py`가 Slack 채널로 알림 전송 |
| **Safe Formatter** / 안전한 포맷터 | `bin/safe_fmt.py` prevents payload/content mishaps in alert messages / `bin/safe_fmt.py`가 알림 메시지에서 페이로드/콘텐츠 오류를 방지 |
| **Air-Gapped Runtime** / 격리 환경 런타임 | `urllib3`, `idna`, `charset_normalizer`, `six` vendored under `lib/python3/` / `lib/python3/`에 의존성 동봉 |
| **Saved Searches & Macros** / 저장된 검색 & 매크로 | Reusable primitives wired into the dashboards / 대시보드에 연결된 재사용 가능한 프리미티브 |

---

## 3. Architecture / 아키텍처

The app is a Splunk "private" add-on: search-time and alert-time logic lives entirely inside `security_alert/default/`, while the Python alert action and helpers live under `security_alert/bin/`. End-user UIs are Splunk views (XML) registered in `default/data/ui/nav/default.xml`.

이 앱은 Splunk "private" 애드온입니다. 검색/알림 로직은 `security_alert/default/`에, Python 알림 액션과 헬퍼는 `security_alert/bin/`에 모두 포함되어 있습니다. 최종 사용자 UI는 `default/data/ui/nav/default.xml`에 등록된 Splunk 뷰(XML)입니다.

### 3.1 Component Map / 구성 요소 맵

| Component / 구성 요소 | Path / 경로 | Role / 역할 |
|---|---|---|
| App manifest / 앱 매니페스트 | `security_alert/app.manifest` | App identity, version, author / 앱 ID/버전/작성자 |
| Default metadata / 기본 메타데이터 | `security_alert/metadata/default.meta` | Capability defaults / 기본 capability |
| App configuration / 앱 설정 | `security_alert/default/app.conf` | UI labels, setup info / UI 레이블, 설정 정보 |
| Alert actions / 알림 액션 | `security_alert/default/alert_actions.conf` | Registers `slack` and other custom actions / `slack` 등 커스텀 액션 등록 |
| Macros / 매크로 | `security_alert/default/macros.conf` | Reusable search macros / 재사용 검색 매크로 |
| Field transforms / 필드 변환 | `security_alert/default/transforms.conf` | Field-level enrichments / 필드 레벨 보강 |
| Field props / 필드 속성 | `security_alert/default/props.conf` | Field indexing/rex behaviour / 필드 인덱싱/rex 동작 |
| Saved searches / 저장된 검색 | `security_alert/default/savedsearches.conf` | Reusable alerts and reports / 재사용 알림/리포트 |
| Views / 뷰 | `security_alert/default/data/ui/views/*.xml` | Dashboards and alert builder UIs / 대시보드 및 알림 빌더 UI |
| Nav / 내비게이션 | `security_alert/default/data/ui/nav/default.xml` | Sidebar entries / 사이드바 항목 |
| Slack alert action / Slack 알림 액션 | `security_alert/bin/slack.py` | Outbound Slack delivery / 외부 Slack 전송 |
| Safe formatter / 안전한 포맷터 | `security_alert/bin/safe_fmt.py` | Sanitises alert payloads / 알림 페이로드 정제 |
| `six` shim / `six` 보강 | `security_alert/bin/six.py` | Python 2/3 compatibility / Python 2/3 호환성 |
| Vendored runtime / 동봉 런타임 | `security_alert/lib/python3/` | `urllib3`, `idna`, `charset_normalizer`, `six` bundles / 런타임 의존성 번들 |

### 3.2 Request Flow / 요청 흐름

1. **Author / 작성** — Analyst uses **Easy Alert Builder** or **Alert Builder** view (`data/ui/views/`) to author a saved search.
   분석가는 **Easy Alert Builder** 또는 **Alert Builder** 뷰(`data/ui/views/`)를 사용해 저장된 검색을 작성합니다.
2. **Persist / 저장** — Search is written to `default/savedsearches.conf` (or the user-level equivalent) and the alert metadata is registered in `alert_actions.conf`.
   검색은 `default/savedsearches.conf`에 저장되며 알림 메타데이터는 `alert_actions.conf`에 등록됩니다.
3. **Trigger / 트리거** — Splunk fires the saved search on schedule and invokes configured alert actions.
   Splunk는 일정에 따라 저장된 검색을 실행하고 설정된 알림 액션을 호출합니다.
4. **Deliver / 전달** — `bin/slack.py` runs with arguments supplied by `alert_actions.conf`, sanitises content via `bin/safe_fmt.py`, and POSTs to Slack.
   `bin/slack.py`가 `alert_actions.conf`에서 전달된 인자로 실행되어 `bin/safe_fmt.py`로 콘텐츠를 정제 후 Slack에 POST합니다.
5. **Triage / 분류** — Analyst opens the **Alert Management Dashboard** to acknowledge / close alerts.
   분석가는 **Alert Management Dashboard**를 열어 알림을 승인/종료합니다.
6. **Investigate / 조사** — From an alert row the analyst pivots to the **Data Explorer Dashboard** to inspect raw events.
   분석가는 알림 행에서 **Data Explorer Dashboard**로 피벗해 원본 이벤트를 확인합니다.

---

## 4. Repository Layout / 저장소 구조

The repository root contains the Splunk app, supporting documentation, and a demo scratchpad. The structure below mirrors the on-disk layout.

저장소 루트는 Splunk 앱과 보조 문서, 데모 공간을 포함합니다. 아래 구조는 디스크 레이아웃을 그대로 반영합니다.

```
.
├── CONTRIBUTING.md
├── LICENSE
├── README.md
├── resume/
│   ├── API.md
│   ├── ARCHITECTURE.md
│   ├── DEPLOYMENT.md
│   └── TROUBLESHOOTING.md
├── docs/
│   ├── ALERT-REPOSITORY-XWIKI.md
│   ├── DEPLOYMENT.md
│   ├── LEGACY-CLEANUP-REPORT.md
│   ├── QUICK-START.md
│   └── RELEASE-NOTES.md
├── demo/
│   └── README.md
└── security_alert/
    ├── app.manifest
    ├── bin/
    │   ├── safe_fmt.py
    │   ├── six.py
    │   └── slack.py
    ├── metadata/
    │   └── default.meta
    ├── default/
    │   ├── alert_actions.conf
    │   ├── app.conf
    │   ├── macros.conf
    │   ├── props.conf
    │   ├── savedsearches.conf
    │   ├── transforms.conf
    │   └── data/
    │       └── ui/
    │           ├── nav/
    │           │   └── default.xml
    │           └── views/
    │               ├── alert-builder.xml
    │               ├── alert-management-dashboard.xml
    │               ├── data-explorer-dashboard.xml
    │               └── easy_alert_builder.xml
    └── lib/
        └── python3/
            ├── idna-3.11.dist-info/
            ├── urllib3/
            │   ├── __init__.py
            │   ├── _base_connection.py
            │   ├── _collections.py
            │   ├── _request_methods.py
            │   ├── _version.py
            │   ├── connection.py
            │   ├── connectionpool.py
            │   ├── exceptions.py
            │   ├── fields.py
            │   ├── filepost.py
            │   ├── poolmanager.py
            │   ├── py.typed
            │   ├── response.py
            │   ├── util/
            │   ├── http2/
            │   └── contrib/
            ├── charset_normalizer-3.4.4.dist-info/
            └── (additional vendored distributions)
```

### 4.1 Top-Level Directories / 최상위 디렉터리

| Directory / 디렉터리 | Purpose / 용도 |
|---|---|
| `security_alert/` | The Splunk app package you install into `$SPLUNK_HOME/etc/apps/` / `$SPLUNK_HOME/etc/apps/`에 설치할 Splunk 앱 패키지 |
| `docs/` | Long-form documentation: deployment, release notes, quick start, wiki, cleanup report / 운영 문서(배포, 릴리스 노트, 빠른 시작, 위키, 정리 보고서) |
| `resume/` | Resumed spec documents: API, architecture, deployment, troubleshooting / 재개 사양 문서(API, 아키텍처, 배포, 트러블슈팅) |
| `demo/` | Scratchpad for demonstrations / 데모용 작업 공간 |
| `CONTRIBUTING.md` | Contribution guidelines / 기여 가이드라인 |
| `LICENSE` | Repository licence / 저장소 라이선스 |

---

## 5. Quick Start / 빠른 시작

These steps assume a Linux/Unix Splunk host with shell access. Paths are shown with the placeholder `$SPLUNK_HOME`.

이 단계는 셸 접근 권한이 있는 Linux/Unix Splunk 호스트를 가정합니다. 경로는 플레이스홀더 `$SPLUNK_HOME`를 사용해 표기했습니다.

### 5.1 Install / 설치

1. Copy the app directory into place / 앱 디렉터리를 대상 위치로 복사합니다.

   ```bash
   cp -r security_alert "$SPLUNK_HOME/etc/apps/"
   chown -R splunk:splunk "$SPLUNK_HOME/etc/apps/security_alert"
   ```

2. Restart Splunk so the new app is picked up / 새 앱이 적용되도록 Splunk을 재시작합니다.

   ```bash
   "$SPLUNK_HOME/bin/splunk" restart
   ```

3. Confirm the app is loaded / 앱이 로드되었는지 확인합니다.

   ```bash
   "$SPLUNK_HOME/bin/splunk" display app | grep security_alert
   ```

### 5.2 Use the App / 앱 사용

- Open Splunk Web → **Apps** → **Security Alert** to land on the Alert Management Dashboard.
  Splunk Web → **앱** → **Security Alert**로 이동하면 알림 관리 대시보드가 표시됩니다.
- Use **Easy Alert Builder** to author a first alert in a guided workflow.
  **Easy Alert Builder**를 사용해 안내 워크플로우로 첫 번째 알림을 작성합니다.
- Use **Alert Builder** when you need full control over schedule, trigger conditions, and actions.
  일정/트리거 조건/액션을 완전히 제어하려면 **Alert Builder**를 사용합니다.
- Pivot into **Data Explorer Dashboard** from any alert row.
  알림 행에서 **Data Explorer Dashboard**로 피벗합니다.

### 5.3 Wire Up Slack / Slack 연결

1. Configure the `slack` alert action in `default/alert_actions.conf` (or via Splunk Web → Settings → Alert actions).
   `default/alert_actions.conf` (또는 Splunk Web → 설정 → 알림 액션)에서 `slack` 알림 액션을 설정합니다.
2. Supply a webhook URL or API token through the secret store available to your deployment.
   배포 환경에서 사용 가능한 비밀 저장소를 통해 webhook URL 또는 API 토큰을 제공합니다.
3. Trigger a test saved search and confirm delivery.
   테스트용 저장된 검색을 실행해 전송을 확인합니다.

---

## 6. Configuration / 설정

### 6.1 App-Level Configuration / 앱 레벨 설정

File / 파일: `security_alert/default/app.conf`

| Key / 키 | Purpose / 용도 |
|---|---|
| `[install]` stanza / `[install]` 스탠자 | App packaging metadata / 앱 패키지 메타데이터 |
| `[ui]` stanza / `[ui]` 스탠자 | App-level UI labels and is_visible / 앱 레벨 UI 레이블 및 is_visible |
| `[launcher]` stanza / `[launcher]` 스탠자 | Default view launched on entry / 진입 시 열리는 기본 뷰 |

### 6.2 Alert Actions / 알림 액션

File / 파일: `security_alert/default/alert_actions.conf`

| Key / 키 | Purpose / 용도 |
|---|---|
| `[slack]` stanza / `[slack]` 스탠자 | Registers the `bin/slack.py` alert action / `bin/slack.py` 알림 액션 등록 |
| `param.webhook_url` | Slack webhook destination / Slack webhook 대상 |
| `param.channel` | Target Slack channel / 대상 Slack 채널 |
| `param.format` | Output format (`plain`, `mrkdwn`, etc.) / 출력 형식(`plain`, `mrkdwn` 등) |

### 6.3 Saved Searches / 저장된 검색

File / 파일: `security_alert/default/savedsearches.conf`

| Key / 키 | Purpose / 용도 |
|---|---|
| `search` | SPL query to schedule / 예약할 SPL 쿼리 |
| `cron_schedule` | Trigger schedule / 트리거 일정 |
| `dispatchAs` | User context for execution / 실행 사용자 컨텍스트 |
| `action.slack` | Whether to invoke the `slack` alert action / `slack` 알림 액션 호출 여부 |
| `action.risk` | Built-in risk action binding / 내장 risk 액션 바인딩 |

### 6.4 Macros / 매크로

File / 파일: `security_alert/default/macros.conf`

| Macro / 매크로 | Purpose / 용도 |
|---|---|
| Reusable SPL building blocks | Shorten detection queries and dashboards / 탐지 쿼리/대시보드 단축 |

### 6.5 Field Extraction & Enrichment / 필드 추출 및 보강

| File / 파일 | Purpose / 용도 |
|---|---|
| `default/props.conf` | Field extraction / 필드 추출 |
| `default/transforms.conf` | Lookups, regex transforms / 룩업, 정규식 변환 |

### 6.6 Permissions / 권한

File / 파일: `security_alert/metadata/default.meta`

| Capability / capability | Default / 기본값 | Notes / 참고 |
|---|---|---|
| Read access for views / 뷰 읽기 | App-internal / 앱 내부 | Configured via `default.meta` |
| Write access for savedsearches / 저장된 검색 쓰기 | Per Splunk role / Splunk 역할별 | Adjust via `authorize.conf` on the Splunk side / Splunk 쪽 `authorize.conf`에서 조정 |

---

## 7. Component Reference / 구성 요소 참조

### 7.1 Views (UI) / 뷰 (UI)

Located at `security_alert/default/data/ui/views/`.

| View / 뷰 | File / 파일 | Purpose / 용도 |
|---|---|---|
| `alert-builder` | `alert-builder.xml` | Full authoring surface / 풀 작성 화면 |
| `easy_alert_builder` | `easy_alert_builder.xml` | Guided wizard for analysts / 분석가용 안내 마법사 |
| `alert-management-dashboard` | `alert-management-dashboard.xml` | Triage and audit alerts / 알림 분류/감사 |
| `data-explorer-dashboard` | `data-explorer-dashboard.xml` | Drill-down exploration / 드릴다운 탐색 |

### 7.2 Alert Actions (Python) / 알림 액션 (Python)

Located at `security_alert/bin/`.

| Script / 스크립트 | Purpose / 용도 | Key Inputs / 주요 입력 |
|---|---|---|
| `slack.py` | Posts alert payload to Slack / 알림 페이로드를 Slack에 게시 | webhook URL, channel, formatted message |
| `safe_fmt.py` | Sanitises fields / values before posting / 게시 전 필드/값 정제 | raw payload |
| `six.py` | Python 2/3 compatibility shim / Python 2/3 호환성 보강 | N/A |

### 7.3 Vendored Runtime / 동봉 런타임

Located at `security_alert/lib/python3/`.

| Distribution / 배포본 | Version / 버전 | Reason / 사유 |
|---|---|---|
| `urllib3` | (bundled) | Required by `slack.py` HTTP delivery / `slack.py` HTTP 전송에 필요 |
| `idna` | `3.11` | Required by `urllib3` / `urllib3` 의존 |
| `charset_normalizer` | `3.4.4` | Required by `urllib3` / `urllib3` 의존 |
| `six` | (bundled in `bin/`) | Python 2/3 compatibility / Python 2/3 호환성 |

> **Note / 참고** All runtime distributions are present on disk so that the app can be deployed to air-gapped Splunk instances without pip access. / 모든 런타임 배포본이 디스크에 함께 제공되므로 pip 접근 없이 격리 Splunk 인스턴스에 배포할 수 있습니다.

### 7.4 Saved Searches & Macros / 저장된 검색 & 매크로

| Type / 유형 | Source File / 원본 파일 | Used By / 사용처 |
|---|---|---|
| Saved searches / 저장된 검색 | `default/savedsearches.conf` | Alert Builder, demo flows |
| Macros / 매크로 | `default/macros.conf` | Dashboards and detection queries / 대시보드 및 탐지 쿼리 |

---

## 8. Local Development / 로컬 개발

### 8.1 Edit-Install Loop / 편집-설치 루프

1. Clone the repository / 저장소를 클론합니다.

   ```bash
   git clone <repo-url> security-alert
   cd security-alert
   ```

2. Symlink the app into a local Splunk for fast iteration / 빠른 반복을 위해 앱을 로컬 Splunk에 심볼릭 링크합니다.

   ```bash
   ln -s "$(pwd)/security_alert" "$SPLUNK_HOME/etc/apps/security_alert"
   ```

3. Edit files in place under `security_alert/` / `security_alert/` 아래에서 직접 편집합니다.
4. Restart, or use debug refresh where applicable / Splunk을 재시작하거나 가능하면 디버그 새로 고침을 사용합니다.

   ```bash
   "$SPLUNK_HOME/bin/splunk" restart
   ```

### 8.2 Coding Conventions / 코딩 규칙

| Area / 영역 | Convention / 규칙 |
|---|---|
| Python (alert actions) | Splunk-supported Python 3, no external pip calls at runtime / Splunk 지원 Python 3, 런타임에 외부 pip 호출 금지 |
| Saved searches | One concern per stanza, descriptive `name` / 스탠자당 하나의 관심사, 설명적인 `name` 사용 |
| Views / dashboards | Pure SimpleXML under `data/ui/views/` / `data/ui/views/` 순수 SimpleXML |
| Configuration | Stanza ordering kept stable for diff readability / 가독성을 위해 스탠자 순서 유지 |

### 8.3 Vendor Updates / 의존성 업데이트 절차

1. Use a controlled dev environment to populate updated packages / 통제된 개발 환경에서 업데이트된 패키지를 채웁니다.
2. Drop new distribution folders under `security_alert/lib/python3/`.
   `security_alert/lib/python3/` 아래에 새 배포본 폴더를 추가합니다.
3. Re-verify the app starts cleanly with no `ImportError` in `splunkd.log`.
   `splunkd.log`에 `ImportError`가 없는지 확인하며 앱이 깨끗하게 시작하는지 재확인합니다.

---

## 9. Testing / 테스트

| Test Type / 테스트 유형 | How / 방법 | Goal / 목표 |
|---|---|---|
| Smoke test / 스모크 테스트 | Trigger one saved search and verify Slack delivery / 저장된 검색을 1회 트리거해 Slack 전달 확인 | Confirms end-to-end wiring / 종단 간 연결 확인 |
| View render / 뷰 렌더링 | Open each dashboard in Splunk Web after install / 설치 후 Splunk Web에서 각 대시보드 열기 | Confirms XML and nav registration / XML/내비 등록 확인 |
| Unit test safe_fmt / `safe_fmt` 단위 테스트 | Exercise `bin/safe_fmt.py` with malicious payloads from a clean shell / 깨끗한 셸에서 위험 페이로드로 `bin/safe_fmt.py` 실행 | Confirms content sanitisation / 콘텐츠 정제 확인 |
| Dependency test / 의존성 테스트 | Import `urllib3`, `idna`, `charset_normalizer`, `six` from `lib/python3/` in a sandboxed Python / 샌드박스 Python에서 `lib/python3/`의 의존성 import | Confirms vendored runtime is intact / 동봉 런타임 무결성 확인 |

If the repository grows a `tests/` tree later, prefer pytest with fixtures that stub Splunk's `splunk.Intersplunk` and HTTP calls.

나중에 저장소에 `tests/` 트리가 추가되면 Splunk의 `splunk.Intersplunk` 및 HTTP 호출을 스텁하는 픽스처와 함께 pytest를 우선 사용하세요.

---

## 10. Operations / 운영

### 10.1 Logs / 로그

| Source / 소스 | Location / 위치 | Look For / 확인 사항 |
|---|---|---|
| `splunkd.log` | `$SPLUNK_HOME/var/log/splunkd.log` | App load errors, `ImportError`, parse errors / 앱 로드 오류, `ImportError`, 파싱 오류 |
| Python script logs / Python 스크립트 로그 | Same `splunkd.log` or custom handler / 동일한 `splunkd.log` 또는 커스텀 핸들러 | Stack traces from `bin/*.py` / `bin/*.py`의 스택 트레이스 |

### 10.2 Common Tasks / 일반 작업

| Task / 작업 | Command / 명령 |
|---|---|
| Reload app config only / 앱 설정만 다시 로드 | `"$SPLUNK_HOME/bin/splunk" reload app security_alert` (where supported) |
| Restart Splunk / Splunk 재시작 | `"$SPLUNK_HOME/bin/splunk" restart` |
| Inspect loaded views / 로드된 뷰 확인 | `"$SPLUNK_HOME/bin/splunk" display app security_alert` |

### 10.3 Upgrade Strategy / 업그레이드 전략

1. Stop Splunk / Splunk 정지.
2. Back up the existing `security_alert/` / 기존 `security_alert/` 백업.
3. Replace with the new package / 새 패키지로 교체.
4. Re-apply any operator-only edits (`authorize.conf`, password storage).
   운영자 전용 수정(`authorize.conf`, 비밀번호 저장)을 다시 적용합니다.
5. Restart and re-run smoke tests.
   재시작 후 스모크 테스트를 다시 실행합니다.

---

## 11. Documentation Index / 문서 인덱스

| Document / 문서 | Path / 경로 | Audience / 대상 |
|---|---|---|
| README (this file) | `README.md` | All readers / 모든 사용자 |
| Quick Start / 빠른 시작 | `docs/QUICK-START.md` | First-time installers / 최초 설치자 |
| Deployment Guide / 배포 가이드 | `docs/DEPLOYMENT.md`, `resume/DEPLOYMENT.md` | Admins / 관리자 |
| Architecture Spec / 아키텍처 사양 | `resume/ARCHITECTURE.md` | Engineers / 엔지니어 |
| API Reference / API 참조 | `resume/API.md` | Integrators / 통합 담당자 |
| Troubleshooting / 트러블슈팅 | `resume/TROUBLESHOOTING.md` | Operators / 운영자 |
| Release Notes / 릴리스 노트 | `docs/RELEASE-NOTES.md` | All readers / 모든 사용자 |
| Legacy Cleanup Report / 레거시 정리 보고서 | `docs/LEGACY-CLEANUP-REPORT.md` | Maintainers / 유지보수 담당자 |
| XWiki Integration / XWiki 통합 | `docs/ALERT-REPOSITORY-XWIKI.md` | Wiki maintainers / 위키 유지보수 담당자 |
| Demo Notes / 데모 노트 | `demo/README.md` | Presenters / 발표자 |
| Contribution Guide / 기여 가이드 | `CONTRIBUTING.md` | Contributors / 기여자 |

---

## 12. Contributing / 기여

1. Read `CONTRIBUTING.md` for house rules and review expectations.
   집 규칙과 리뷰 기대치를 위해 `CONTRIBUTING.md`를 읽어주세요.
2. Fork and create a feature branch / 포크 후 기능 브랜치를 생성합니다.
3. Keep changes scoped: one concern per commit.
   변경 범위를 좁게 유지하고 커밋당 하나의 관심사를 다루세요.
4. Verify that `splunkd.log` stays clean after install/reload.
   설치/재로드 후 `splunkd.log`에 오류가 없는지 확인합니다.
5. Update the relevant doc under `docs/` or `resume/` whenever a public surface changes.
   공개 인터페이스가 변경될 때마다 `docs/` 또는 `resume/`의 관련 문서를 업데이트합니다.

---

## 13. License / 라이선스

Distributed under the terms of the `LICENSE` file at the repository root. See `LICENSE` for the full text.

저장소 루트의 `LICENSE` 파일에 명시된 조건에 따라 배포됩니다. 전문은 `LICENSE`를 참조하세요.