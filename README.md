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
- Operate on isolated Splunk instances, because the Python runtime dependencies (`urllib3`, `idna`, `charset_normalizer`, `six`) are vendored under `lib/python3/`.

The app is delivered as a single folder plus optional documentation. It is designed to be dropped onto `$SPLUNK_HOME/etc/apps/security_alert/`, requires no external Python packages at runtime, and exposes its capabilities through four SimpleXML dashboards plus a custom alert action.

`security_alert/`는 임시 알림 작성을 반복 가능하고 감사 가능한 워크플로우로 전환하는 독립형 Splunk 앱입니다. 다음이 필요한 SOC 분석가, 탐지 엔지니어, Splunk 관리자를 대상으로 합니다.

- 안내형 **손쉬운 알림 빌더** 또는 풀-기능 **알림 빌더** UI를 통한 알림 작성
- 단일 **알림 관리 대시보드**에서의 알림 분류, 승인, 감사
- **데이터 탐색기 대시보드**에서의 알림 근거 이벤트 조사
- 번들된 `bin/slack.py` 커스텀 알림 액션을 사용한 Slack 라우팅
- `lib/python3/`에 Python 런타임 의존성(`urllib3`, `idna`, `charset_normalizer`, `six`)이 사전 동봉되어 있어 Splunk 격리 환경에서도 동작

본 앱은 단일 폴더와 보조 문서로 제공되며, `$SPLUNK_HOME/etc/apps/security_alert/`에 그대로 배치하여 설치하고 런타임에 외부 Python 패키지를 요구하지 않습니다. 4개의 SimpleXML 대시보드와 1개의 커스텀 알림 액션으로 기능을 노출합니다.

---

## 2. Features / 주요 기능

| Capability / 기능 | Surface / 노출 위치 | Description / 설명 |
|---|---|---|
| Unified triage console / 통합 분류 콘솔 | `default/data/ui/views/alert-management-dashboard.xml` | Acknowledge, mute, route, and audit alerts from a single dashboard. 단일 대시보드에서 알림 승인, 음소거, 라우팅, 감사. |
| Guided authoring / 안내형 작성 | `default/data/ui/views/easy_alert_builder.xml` | Step-by-step wizard for analysts who rarely author alerts. 알림을 거의 작성하지 않는 분석가를 위한 단계별 마법사. |
| Full authoring / 풀-기능 작성 | `default/data/ui/views/alert-builder.xml` | Feature-complete builder for detection engineers. 탐지 엔지니어용 풀-기능 빌더. |
| Event investigation / 이벤트 조사 | `default/data/ui/views/data-explorer-dashboard.xml` | Drill into the events behind a triggered alert. 트리거된 알림의 근거 이벤트를 드릴다운. |
| Slack delivery / Slack 전송 | `bin/slack.py` custom alert action | Posts alert payloads to Slack incoming webhooks. Slack 인커밍 웹훅으로 알림 페이로드 전송. |
| Safe message formatting / 안전한 메시지 포맷팅 | `bin/safe_fmt.py` | Escapes and sanitizes Markdown / Slack mrkdwn payloads. Markdown / Slack mrkdwn 페이로드의 이스케이프 및 새니타이즈. |
| Air-gapped operation / 격리 환경 동작 | `lib/python3/` | Vendored `urllib3`, `idna`, `charset_normalizer`, `six`. `urllib3`, `idna`, `charset_normalizer`, `six` 사전 동봉. |
| Reusable search primitives / 재사용 검색 프리미티브 | `default/macros.conf` | Shared SPL macro definitions. 공통 SPL 매크로 정의. |
| Field enrichment / 필드 보강 | `default/props.conf`, `default/transforms.conf` | Field extractions, lookups, and indexing-time transforms. 필드 추출, 룩업, 인덱싱 시점 변환. |
| Pre-built detections / 사전 구축 탐지 | `default/savedsearches.conf` | Ready-to-enable scheduled searches. 즉시 활성화 가능한 예약 검색. |
| Action registration / 액션 등록 | `default/alert_actions.conf` | Declares the Slack custom alert action. Slack 커스텀 알림 액션 선언. |
| App identity / 앱 식별 | `app.manifest` | Splunk app version, author, and dependencies. Splunk 앱 버전, 작성자, 의존성. |
| Navigation / 내비게이션 | `default/data/ui/nav/default.xml` | Top-level menu entries for the dashboards. 대시보드 상위 메뉴 항목. |
| ACL defaults / ACL 기본값 | `metadata/default.meta` | Default capabilities and ownership. 기본 권한 및 소유권. |

---

## 3. Architecture / 아키텍처

The app is composed of four cooperating layers that map to standard Splunk extension points.

본 앱은 표준 Splunk 확장 지점에 매핑되는 4개의 협력 계층으로 구성됩니다.

| Layer / 계층 | Files / 파일 | Role / 역할 |
|---|---|---|
| Presentation / 프레젠테이션 | `default/data/ui/views/*.xml`, `default/data/ui/nav/default.xml` | SimpleXML dashboards and navigation. SimpleXML 대시보드 및 내비게이션. |
| Detection / 탐지 | `default/savedsearches.conf`, `default/macros.conf`, `default/props.conf`, `default/transforms.conf` | Scheduled searches, reusable SPL, field extraction, and lookup logic. 예약 검색, 재사용 SPL, 필드 추출, 룩업 로직. |
| Action / 액션 | `default/alert_actions.conf`, `bin/slack.py`, `bin/safe_fmt.py`, `bin/six.py` | Triggered response, message assembly, outbound webhook delivery. 트리거 응답, 메시지 조립, 외부 웹훅 전송. |
| Platform / 플랫폼 | `app.conf`, `app.manifest`, `metadata/default.meta`, `lib/python3/` | App identity, permissions, vendored runtime. 앱 식별, 권한, 사전 동봉 런타임. |

### Request Flow / 요청 흐름

1. A saved search in `savedsearches.conf` evaluates SPL at its schedule. / `savedsearches.conf`의 저장된 검색이 스케줄에 따라 SPL을 평가합니다.
2. The search invokes the registered custom alert action declared in `alert_actions.conf`. / 검색은 `alert_actions.conf`에 선언된 등록된 커스텀 알림 액션을 호출합니다.
3. Splunk hands the results payload to `bin/slack.py`. / Splunk은 결과 페이로드를 `bin/slack.py`에 전달합니다.
4. `slack.py` calls `bin/safe_fmt.py` to sanitize fields for Slack mrkdwn rendering. / `slack.py`는 Slack mrkdwn 렌더링을 위해 `bin/safe_fmt.py`를 호출하여 필드를 새니타이즈합니다.
5. `slack.py` imports the vendored `urllib3` from `lib/python3/urllib3/` (no PyPI access required). / `slack.py`는 `lib/python3/urllib3/`에서 사전 동봉된 `urllib3`을 임포트합니다(PyPI 액세스 불필요).
6. `urllib3` loads its bundled siblings (`idna`, `charset_normalizer`, `six`) from the same tree. / `urllib3`은 동일한 트리에서 번들된 형제 패키지(`idna`, `charset_normalizer`, `six`)를 로드합니다.
7. The HTTP POST is sent to the configured Slack webhook URL. / 구성된 Slack 웹훅 URL로 HTTP POST가 전송됩니다.
8. Authored or modified alerts surface in `easy_alert_builder.xml`, `alert-builder.xml`, or `alert-management-dashboard.xml`. / 작성 또는 수정된 알림은 `easy_alert_builder.xml`, `alert-builder.xml`, 또는 `alert-management-dashboard.xml`에 표시됩니다.
9. Analysts pivot from the management dashboard to `data-explorer-dashboard.xml` for raw event inspection. / 분석가는 관리 대시보드에서 `data-explorer-dashboard.xml`로 피벗하여 원시 이벤트를 검사합니다.

### Runtime Status at a Glance / 런타임 상태 한눈에 보기

| Surface / 노출면 | Runtime role / 런타임 역할 | State source / 상태 출처 |
|---|---|---|
| Dashboards / 대시보드 | Read SimpleXML, dispatch SPL. SimpleXML 읽기, SPL 디스패치. | Splunk view server. Splunk 뷰 서버. |
| Saved searches / 저장된 검색 | Scheduled by `savedsearches.conf`. `savedsearches.conf`로 예약. | `splunkd` scheduler. `splunkd` 스케줄러. |
| Custom alert action / 커스텀 알림 액션 | Spawned as a Python subprocess. Python 서브프로세스로 생성. | `bin/slack.py` stdout/stderr/log. `bin/slack.py` stdout/stderr/log. |
| Vendored Python / 사전 동봉 Python | Loaded via `lib/python3/...` sys.path. `lib/python3/...` sys.path로 로드. | Filesystem only, no PyPI. 파일시스템만 사용, PyPI 미사용. |

---

## 4. Repository Layout / 저장소 구조

```
/
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
    ├── README.md
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
            ├── urllib3/
            ├── http2/
            ├── contrib/
            ├── util/
            ├── idna-3.11.dist-info/
            ├── charset_normalizer-3.4.4.dist-info/
            └── (six vendored alongside)
```

### Top-Level Purpose / 최상위 디렉터리 목적

| Path / 경로 | Purpose / 목적 |
|---|---|
| `security_alert/` | The Splunk app itself. Place under `$SPLUNK_HOME/etc/apps/`. Splunk 앱 본체. `$SPLUNK_HOME/etc/apps/`에 배치. |
| `docs/` | Long-form guidance: deployment, quick start, release notes, legacy cleanup. 장기 가이드: 배포, 빠른 시작, 릴리스 노트, 레거시 정리. |
| `resume/` | Restored historical design notes: API, architecture, deployment, troubleshooting. 복원된 과거 설계 노트: API, 아키텍처, 배포, 트러블슈팅. |
| `demo/` | Demo environment pointer for evaluation. 평가를 위한 데모 환경 진입점. |
| `CONTRIBUTING.md`, `LICENSE`, `README.md` | Repository-level meta files. 저장소 메타 파일. |

---

## 5. Quick Start / 빠른 시작

### Prerequisites / 사전 요구 사항

| Requirement / 요구 사항 | Notes / 비고 |
|---|---|
| Splunk Enterprise or Cloud / Splunk Enterprise 또는 Cloud | Any recent 8.x or 9.x line. 최근 8.x 또는 9.x 라인. |
| File-system write access / 파일 시스템 쓰기 권한 | To `$SPLUNK_HOME/etc/apps/`. `$SPLUNK_HOME/etc/apps/`에 쓰기 권한. |
| Slack incoming webhook URL / Slack 인커밍 웹훅 URL | Required only for the Slack custom alert action. Slack 커스텀 알림 액션에만 필요. |
| No PyPI access required / PyPI 액세스 불필요 | All Python dependencies are vendored. 모든 Python 의존성이 사전 동봉되어 있습니다. |

### Install / 설치

1. Copy the `security_alert/` folder to your Splunk apps directory. / `security_alert/` 폴더를 Splunk 앱 디렉터리에 복사합니다.

   ```bash
   cp -r security_alert/ "$SPLUNK_HOME/etc/apps/"
   ```

2. Restart Splunk, or reload the app from the UI. / Splunk을 재시작하거나 UI에서 앱을 리로드합니다.

   ```bash
   "$SPLUNK_HOME/bin/splunk" restart
   ```

3. Verify the app is loaded. / 앱이 로드되었는지 확인합니다.

   ```bash
   "$SPLUNK_HOME/bin/splunk" display app security_alert
   ```

4. Navigate in Splunk Web to **Security Alert** in the app menu. / Splunk Web 앱 메뉴에서 **Security Alert**로 이동합니다.

5. (Optional, Slack only) Configure the webhook URL via the alert action settings on each saved search that should deliver to Slack. / (선택, Slack만) Slack으로 전송해야 하는 각 저장된 검색의 알림 액션 설정에서 웹훅 URL을 구성합니다.

For a longer walkthrough, see `docs/QUICK-START.md` and `docs/DEPLOYMENT.md`. / 자세한 단계는 `docs/QUICK-START.md` 및 `docs/DEPLOYMENT.md`를 참조하세요.

---

## 6. Configuration / 설정

The app's tunable surface is entirely contained in `security_alert/default/` plus the alert action parameters set per saved search.

앱의 조정 가능 노출면은 `security_alert/default/`에 완전히 포함되어 있으며, 저장된 검색별로 설정된 알림 액션 매개변수로 보완됩니다.

| File / 파일 | Purpose / 목적 | Operator action / 운영자 작업 |
|---|---|---|
| `default/app.conf` | App metadata, UI label, launcher. 앱 메타데이터, UI 레이블, 런처. | Edit `label`, `version`, `[ui]` settings if rebranding. 리브랜딩 시 `label`, `version`, `[ui]` 설정 편집. |
| `default/alert_actions.conf` | Registers the Slack custom alert action. Slack 커스텀 알림 액션 등록. | Configure per-action options such as webhook URL. 웹훅 URL 등 액션별 옵션 구성. |
| `default/savedsearches.conf` | Scheduled detection logic. 예약 탐지 로직. | Enable, disable, schedule, or modify searches. 검색 활성화, 비활성화, 예약, 수정. |
| `default/macros.conf` | Shared SPL snippets (e.g. `index=`, time windows). 공통 SPL 스니펫(예: `index=`, 시간 윈도우). | Adjust to match your index naming. 인덱스 명명에 맞춰 조정. |
| `default/props.conf` | Field extractions, sourcetypes, transforms hooks. 필드 추출, 소스타입, 변환 훅. | Extend with your regex / KV rules. 정규식 / KV 규칙 확장. |
| `default/transforms.conf` | Lookup tables and field renames. 룩업 테이블 및 필드 이름 변경. | Add CSV lookups in `lookups/`. `lookups/`에 CSV 룩업 추가. |
| `metadata/default.meta` | Default permissions and ownership. 기본 권한 및 소유권. | Tighten ACLs after deployment. 배포 후 ACL 강화. |
| `app.manifest` | Splunk Cloud-compatible manifest. Splunk Cloud 호환 매니페스트. | Update on upgrades. 업그레이드 시 갱신. |

### Alert Action Parameters / 알림 액션 매개변수

The Slack custom alert action exposes fields declared in `alert_actions.conf`. Typical parameters include:

Slack 커스텀 알림 액션은 `alert_actions.conf`에 선언된 필드를 노출합니다. 일반적인 매개변수는 다음과 같습니다.

| Parameter / 매개변수 | Description / 설명 |
|---|---|
| Webhook URL / 웹훅 URL | Slack incoming webhook endpoint. Slack 인커밍 웹훅 엔드포인트. |
| Channel override / 채널 재정의 | Optional `#channel` override. 선택적 `#channel` 재정의. |
| Message template / 메시지 템플릿 | SPL result fields mapped into Slack blocks. SPL 결과 필드를 Slack 블록에 매핑. |
| Mention list / 멘션 목록 | Users or groups to @-mention. @멘션할 사용자 또는 그룹. |
| Severity threshold / 심각도 임계값 | Minimum severity to deliver. 전송할 최소 심각도. |

---

## 7. Component Reference / 구성 요소 참조

### Binaries / 바이너리

| File / 파일 | Language / 언어 | Purpose / 목적 |
|---|---|---|
| `bin/slack.py` | Python 3 | Splunk custom alert action that delivers alerts to Slack. Splunk 커스텀 알림 액션, 알림을 Slack으로 전송. |
| `bin/safe_fmt.py` | Python 3 | Escapes and formats Splunk result fields safely for Slack mrkdwn. Splunk 결과 필드를 Slack mrkdwn용으로 안전하게 이스케이프/포맷. |
| `bin/six.py` | Python 3 | Python 2/3 compatibility shim used by Splunk's modular alert action contract. Splunk 모듈러 알림 액션 계약에 사용되는 Python 2/3 호환 셰임. |

### SimpleXML Views / SimpleXML 뷰

| File / 파일 | Audience / 대상 사용자 | Purpose / 목적 |
|---|---|---|
| `default/data/ui/views/alert-management-dashboard.xml` | SOC analysts, alert leads / SOC 분석가, 알림 책임자 | Triage queue, acknowledgement, status tracking. 분류 큐, 승인, 상태 추적. |
| `default/data/ui/views/easy_alert_builder.xml` | New alert authors / 신규 알림 작성자 | Step-by-step guided alert creation wizard. 단계별 안내형 알림 생성 마법사. |
| `default/data/ui/views/alert-builder.xml` | Detection engineers / 탐지 엔지니어 | Full authoring surface for advanced SPL and logic. 고급 SPL 및 로직을 위한 풀 작성 화면. |
| `default/data/ui/views/data-explorer-dashboard.xml` | Analysts / 분석가 | Free-form exploration of events feeding an alert. 알림에 이벤트를 자유 형식으로 탐색. |

### Vendored Libraries / 사전 동봉 라이브러리

| Package / 패키지 | Version Marker / 버전 표시 | Consumed by / 사용처 |
|---|---|---|
| `urllib3` | `urllib3/_version.py` (vendored tree) | `bin/slack.py` HTTPS POSTs. `bin/slack.py` HTTPS POST. |
| `idna` | `idna-3.11.dist-info/METADATA` | Hostname handling inside `urllib3`. `urllib3` 내부의 호스트명 처리. |
| `charset_normalizer` | `charset_normalizer-3.4.4.dist-info/METADATA` | Encoding detection inside `urllib3`. `urllib3` 내부의 인코딩 감지. |
| `six` | `bin/six.py` | Python 2/3 compat used by Splunk's alert action runner. Splunk 알림 액션 러너가 사용하는 Python 2/3 호환. |

### Dashboards and Navigation / 대시보드 및 내비게이션

The top-level navigation is defined in `default/data/ui/nav/default.xml` and lists the four dashboards. Each dashboard is a standalone SimpleXML view; cross-dashboard drilldowns are driven by tokens set within each view.

최상위 내비게이션은 `default/data/ui/nav/default.xml`에 정의되어 4개의 대시보드를 나열합니다. 각 대시보드는 독립 SimpleXML 뷰이며, 대시보드 간 드릴다운은 각 뷰에서 설정된 토큰에 의해 구동됩니다.

---

## 8. Local Development / 로컬 개발

### Recommended Layout / 권장 레이아웃

| Step / 단계 | Action / 작업 |
|---|---|
| 1 | Clone the repository alongside your Splunk apps directory. 저장소를 Splunk 앱 디렉터리 옆에 클론합니다. |
| 2 | Symlink `security_alert/` into `$SPLUNK_HOME/etc/apps/` to avoid copying on each change. 변경할 때마다 복사하지 않도록 `$SPLUNK_HOME/etc/apps/`에 `security_alert/`를 심볼릭 링크합니다. |
| 3 | Restart Splunk, or use the in-app reload. Splunk을 재시작하거나 앱 내 리로드를 사용합니다. |
| 4 | Iterate on dashboards, `.conf` files, and `bin/*.py`. 대시보드, `.conf` 파일, `bin/*.py`를 반복 작업합니다. |

### Development Workflow / 개발 워크플로우

1. Modify SimpleXML in `default/data/ui/views/`. / `default/data/ui/views/`의 SimpleXML을 수정합니다.
2. Modify detection logic in `default/savedsearches.conf`, `default/macros.conf`, `default/props.conf`, `default/transforms.conf`. / `default/savedsearches.conf`, `default/macros.conf`, `default/props.conf`, `default/transforms.conf`에서 탐지 로직을 수정합니다.
3. Modify `bin/slack.py` and `bin/safe_fmt.py` for delivery behavior. / 전송 동작은 `bin/slack.py` 및 `bin/safe_fmt.py`를 수정합니다.
4. Validate by triggering the saved search manually with `| sendalert slack ...` from the Search & Reporting app. / Search & Reporting 앱에서 `| sendalert slack ...`로 저장된 검색을 수동 트리거하여 검증합니다.
5. Read `splunkd` logs under `$SPLUNK_HOME/var/log/splunk/` to verify delivery. / `$SPLUNK_HOME/var/log/splunk/`의 `splunkd` 로그를 읽어 전송을 확인합니다.

### Python Editing / Python 편집

Because the runtime is vendored, keep imports inside `bin/` and `lib/python3/` paths only. Do not add `requirements.txt` or assume PyPI access.

런타임이 사전 동봉되어 있으므로 `bin/` 및 `lib/python3/` 경로 내의 임포트만 유지하세요. `requirements.txt`를 추가하거나 PyPI 액세스를 가정하지 마세요.

---

## 9. Testing / 테스트

| Test surface / 테스트 영역 | Method / 방법 |
|---|---|
| Saved searches / 저장된 검색 | Trigger manually with `| run savedsearch` or scheduled preview. `| run savedsearch` 또는 예약 미리보기로 수동 트리거. |
| Alert action delivery / 알림 액션 전송 | Invoke `bin/slack.py` with a fixture payload against a test Slack workspace. 테스트 Slack 워크스페이스에 픽스처 페이로드로 `bin/slack.py`를 호출. |
| Safe formatting / 안전 포맷팅 | Unit-test `bin/safe_fmt.py` with adversarial inputs (Markdown, control characters, very long strings). 적대적 입력(Markdown, 제어 문자, 매우 긴 문자열)으로 `bin/safe_fmt.py` 단위 테스트. |
| Dashboard rendering / 대시보드 렌더링 | Open each dashboard with sample data and verify token resolution. 샘플 데이터로 각 대시보드를 열어 토큰 해석을 확인. |
| Air-gapped install / 격리 설치 | Run with network disabled to confirm vendored deps resolve. 네트워크가 비활성화된 상태에서 실행하여 사전 동봉 의존성이 해결되는지 확인. |

For the historical test notes and operational checks, see `resume/TROUBLESHOOTING.md` and `docs/LEGACY-CLEANUP-REPORT.md`. / 과거 테스트 노트와 운영 점검 사항은 `resume/TROUBLESHOOTING.md` 및 `docs/LEGACY-CLEANUP-REPORT.md`를 참조하세요.

---

## 10. Operations / 운영

| Concern / 고려 사항 | Guidance / 가이드 |
|---|---|
| Upgrade / 업그레이드 | Replace `security_alert/`, then restart Splunk. Verify `app.manifest` `version`. `security_alert/`를 교체한 다음 Splunk을 재시작합니다. `app.manifest` `version`을 확인합니다. |
| Rollback / 롤백 | Restore the previous `security_alert/` tree from backup or version control. 백업 또는 버전 관리에서 이전 `security_alert/` 트리를 복원합니다. |
| Logs / 로그 | `splunkd` logs at `$SPLUNK_HOME/var/log/splunk/`. `$SPLUNK_HOME/var/log/splunk/`의 `splunkd` 로그. |
| Permissions / 권한 | Tune `metadata/default.meta` for role-based access. 역할 기반 액세스를 위해 `metadata/default.meta` 조정. |
| Network egress / 네트워크 송신 | Outbound HTTPS to the configured Slack webhook domain only. 구성된 Slack 웹훅 도메인으로의 아웃바운드 HTTPS만 허용. |
| Backups / 백업 | Treat the `security_alert/` directory as the canonical source of truth. `security_alert/` 디렉터리를 정식 소스로 취급. |

For deployment procedures and incident handling, see `docs/DEPLOYMENT.md`, `resume/DEPLOYMENT.md`, and `resume/TROUBLESHOOTING.md`. / 배포 절차와 인시던트 처리는 `docs/DEPLOYMENT.md`, `resume/DEPLOYMENT.md`, `resume/TROUBLESHOOTING.md`를 참조하세요.

---

## 11. Documentation Index / 문서 인덱스

| Document / 문서 | Path / 경로 | Scope / 범위 |
|---|---|---|
| API reference / API 참조 | `resume/API.md` | Alert action and helper function contracts. 알림 액션 및 헬퍼 함수 계약. |
| Architecture / 아키텍처 | `resume/ARCHITECTURE.md` | Historical architecture narrative. 과거 아키텍처 서술. |
| Deployment / 배포 | `resume/DEPLOYMENT.md`, `docs/DEPLOYMENT.md` | Install and upgrade procedures. 설치 및 업그레이드 절차. |
| Troubleshooting / 트러블슈팅 | `resume/TROUBLESHOOTING.md` | Known issues and recovery steps. 알려진 문제 및 복구 단계. |
| Quick start / 빠른 시작 | `docs/QUICK-START.md` | Concise first-run guide. 간결한 첫 실행 가이드. |
| Release notes / 릴리스 노트 | `docs/RELEASE-NOTES.md` | Per-version change log. 버전별 변경 로그. |
| Legacy cleanup / 레거시 정리 | `docs/LEGACY-CLEANUP-REPORT.md` | Notes on retired code paths. 폐기된 코드 경로에 대한 메모. |
| XWiki alert repository / XWiki 알림 저장소 | `docs/ALERT-REPOSITORY-XWIKI.md` | Companion documentation pointer. 동반 문서 진입점. |
| Demo / 데모 | `demo/README.md` | Demo environment setup. 데모 환경 구성. |

---

## 12. Contributing / 기여

Please read `CONTRIBUTING.md` at the repository root before opening an issue or pull request. The document covers:

이슈 또는 풀 리퀘스트를 열기 전에 저장소 루트의 `CONTRIBUTING.md`를 읽어주세요. 다음 내용을 다룹니다.

- Code style for Python and SimpleXML. Python 및 SimpleXML 코드 스타일.
- How to update vendored libraries without breaking air-gapped installs. 격리 설치를 깨뜨리지 않고 사전 동봉 라이브러리를 업데이트하는 방법.
- Dashboard change-review checklist. 대시보드 변경 검토 체크리스트.
- Backwards compatibility expectations for `.conf` schemas. `.conf` 스키마의 하위 호환성 기대치.

---

## 13. License / 라이선스

This repository is distributed under the terms described in the `LICENSE` file at the repository root. Vendored third-party libraries retain their original licenses (see `lib/python3/<package>-*.dist-info/licenses/` for each package).

본 저장소는 저장소 루트의 `LICENSE` 파일에 명시된 조건에 따라 배포됩니다. 사전 동봉된 타사 라이브러리는 각 라이선스를 유지합니다(각 패키지에 대해서는 `lib/python3/<package>-*.dist-info/licenses/` 참조).