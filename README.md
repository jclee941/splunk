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

`security_alert/`는 일회성 알림 작성을 반복 가능하고 감사 가능한 워크플로우로 전환하는 자체 완비형 Splunk 앱입니다. 다음과 같은 요구사항을 가진 SOC 분석가, 탐지 엔지니어, Splunk 관리자를 대상으로 합니다.

- 안내형 **손쉬운 알림 빌더** 또는 풀-기능 **알림 빌더** UI를 통한 알림 작성
- 단일 **알림 관리 대시보드**에서의 알림 분류·인지·감사
- **데이터 탐색기 대시보드**에서의 알림背后的 이벤트 조사
- 번들된 `bin/slack.py` 커스텀 알림 액션을 통한 Slack 라우팅
- `security_alert/lib/python3/` 아래에 `urllib3`, `idna`, `charset_normalizer`, `six` Python 런타임 의존성을 동봉하여 인터넷이 차단된 Splunk 인스턴스에서의 운영

### 1.1 Intended Audience / 대상 사용자

| Role / 역할 | Primary Use Case / 주요 사용 사례 |
|---|---|
| SOC Analyst / SOC 분석가 | Triage and acknowledge alerts in the management dashboard / 관리 대시보드에서 알림 분류 및 인지 |
| Detection Engineer / 탐지 엔지니어 | Author and tune alerts in the Easy / Advanced Builder / 손쉬운/고급 빌더에서 알림 작성 및 튜닝 |
| Splunk Administrator / Splunk 관리자 | Install, configure, and operate the app on search heads / 검색 헤드에 앱 설치·설정·운영 |
| Incident Responder / 침해 대응 담당자 | Pivot from an alert to raw events in the Data Explorer / 데이터 탐색기에서 알림에서 원시 이벤트로 이동 |

---

## 2. Features / 주요 기능

| Area / 영역 | Feature / 기능 | Description / 설명 |
|---|---|---|
| Alert Authoring / 알림 작성 | Easy Alert Builder / 손쉬운 알림 빌더 | Step-by-step wizard for non-power users / 비숙련 사용자를 위한 단계별 마법사 |
| Alert Authoring / 알림 작성 | Alert Builder / 알림 빌더 | Full-power SPL-first authoring UI / SPL 중심의 풀 기능 작성 UI |
| Alert Triage / 알림 분류 | Alert Management Dashboard / 알림 관리 대시보드 | Single pane of glass for triage, ownership, and audit / 분류·소유권·감사용 단일 화면 |
| Investigation / 조사 | Data Explorer Dashboard / 데이터 탐색기 대시보드 | Pivot from an alert into the underlying events / 알림에서 원본 이벤트로 드릴다운 |
| Notification / 알림 전달 | Slack Custom Alert Action / Slack 커스텀 알림 액션 | `bin/slack.py` posts formatted messages to Slack webhooks / `bin/slack.py`가 Slack 웹훅에 포맷팅된 메시지 게시 |
| Templating / 템플릿 | Safe Format Helper / 안전한 포맷 헬퍼 | `bin/safe_fmt.py` sanitizes user input before substitution / `bin/safe_fmt.py`가 치환 전 사용자 입력 sanitize |
| Compatibility / 호환성 | Offline Operation / 오프라인 운영 | All Python dependencies bundled under `lib/python3/` / 모든 Python 의존성 `lib/python3/`에 동봉 |
| Compatibility / 호환성 | Python 2/3 Bridge / Python 2/3 브리지 | `bin/six.py` provides compatibility utilities / `bin/six.py`가 호환성 유틸리티 제공 |

---

## 3. Architecture / 아키텍처

The app follows the standard Splunk add-on layout. The view layer renders dashboards, the configuration layer defines searchable knowledge, and the script layer (custom alert actions) provides outbound integrations.

이 앱은 표준 Splunk 애드온 레이아웃을 따릅니다. 뷰 레이어가 대시보드를 렌더링하고, 설정 레이어가 검색 가능한 지식을 정의하며, 스크립트 레이어(커스텀 알림 액션)가 아웃바운드 통합을 제공합니다.

### 3.1 Layered View / 레이어 구성

| Layer / 레이어 | Path / 경로 | Role / 역할 |
|---|---|---|
| Navigation / 탐색 | `security_alert/default/data/ui/nav/default.xml` | Registers dashboards in the Splunk app menu / Splunk 앱 메뉴에 대시보드 등록 |
| Views / 뷰 | `security_alert/default/data/ui/views/*.xml` | Defines dashboards and forms / 대시보드 및 폼 정의 |
| Knowledge / 지식 객체 | `security_alert/default/*.conf` | Saved searches, macros, props, transforms / 저장된 검색, 매크로, props, transforms |
| App Metadata / 앱 메타데이터 | `security_alert/app.manifest`, `security_alert/metadata/default.meta` | App identity and capability model / 앱 정체성 및 기능 모델 |
| Custom Alert Actions / 커스텀 알림 액션 | `security_alert/bin/slack.py`, `security_alert/bin/safe_fmt.py` | Outbound integrations triggered by alerts / 알림에 의해 트리거되는 아웃바운드 통합 |
| Bundled Runtime / 번들 런타임 | `security_alert/lib/python3/*` | Vendored third-party Python packages / 동봉된 서드파티 Python 패키지 |

### 3.2 User Request Flow / 사용자 요청 흐름

1. A user opens the app from the Splunk launcher and lands on the **Alert Management Dashboard**. / 사용자가 Splunk 런처에서 앱을 열어 **알림 관리 대시보드**에 진입합니다.
2. From the navigation, the user opens the **Easy Alert Builder** or **Alert Builder** to author a new detection. / 탐색 메뉴에서 **손쉬운 알림 빌더** 또는 **알림 빌더**를 열어 새로운 탐지를 작성합니다.
3. The builder writes a saved search plus a `props.conf` / `transforms.conf` / `macros.conf` knowledge bundle, registered in `default/savedsearches.conf`. / 빌더는 `default/savedsearches.conf`에 등록된 저장된 검색과 `props.conf` / `transforms.conf` / `macros.conf` 지식 묶음을 생성합니다.
4. When the saved search fires, the configured custom alert action invokes `bin/slack.py`, which formats the payload through `bin/safe_fmt.py` and POSTs it to the Slack webhook. / 저장된 검색이 실행되면 설정된 커스텀 알림 액션이 `bin/slack.py`를 호출하고, `bin/safe_fmt.py`를 통해 페이로드를 포맷하여 Slack 웹훅에 POST 합니다.
5. Analysts triage the alert back in the **Alert Management Dashboard** and pivot into the **Data Explorer Dashboard** for raw event analysis. / 분석가는 다시 **알림 관리 대시보드**에서 알림을 분류하고 **데이터 탐색기 대시보드**로 이동하여 원시 이벤트를 분석합니다.

### 3.3 View Catalog / 뷰 목록

| View File / 뷰 파일 | Purpose / 용도 |
|---|---|
| `alert-management-dashboard.xml` | Triage, ownership, and audit surface / 분류·소유권·감사 화면 |
| `easy_alert_builder.xml` | Guided authoring wizard / 안내형 작성 마법사 |
| `alert-builder.xml` | Full authoring UI / 풀 기능 작성 UI |
| `data-explorer-dashboard.xml` | Exploratory event investigation / 탐색형 이벤트 조사 |

---

## 4. Repository Layout / 저장소 구조

```
.
├── CONTRIBUTING.md
├── LICENSE
├── README.md
├── demo/
│   └── README.md
├── docs/
│   ├── ALERT-REPOSITORY-XWIKI.md
│   ├── DEPLOYMENT.md
│   ├── LEGACY-CLEANUP-REPORT.md
│   ├── QUICK-START.md
│   └── RELEASE-NOTES.md
├── resume/
│   ├── API.md
│   ├── ARCHITECTURE.md
│   ├── DEPLOYMENT.md
│   └── TROUBLESHOOTING.md
└── security_alert/
    ├── README.md
    ├── app.manifest
    ├── bin/
    │   ├── safe_fmt.py
    │   ├── six.py
    │   └── slack.py
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
    ├── lib/
    │   └── python3/
    │       ├── charset_normalizer-3.4.4.dist-info/
    │       ├── idna-3.11.dist-info/
    │       ├── urllib3/
    │       └── ... (vendored third-party packages / 동봉된 서드파티 패키지)
    └── metadata/
        └── default.meta
```

| Path / 경로 | Purpose / 용도 |
|---|---|
| `security_alert/app.manifest` | App identity, version, author / 앱 정체성, 버전, 작성자 |
| `security_alert/metadata/default.meta` | Capability model for app objects / 앱 객체의 기능 모델 |
| `security_alert/bin/` | Executable scripts and custom alert actions / 실행 스크립트 및 커스텀 알림 액션 |
| `security_alert/default/` | Splunk configuration files (read-only baseline) / Splunk 설정 파일(읽기 전용 기준선) |
| `security_alert/local/` (created on install) | Per-instance overrides created at install time / 설치 시 생성되는 인스턴스별 재정의 |
| `security_alert/lib/python3/` | Vendored Python packages for offline use / 오프라인 사용을 위한 동봉 Python 패키지 |
| `docs/` | Operational and release documentation / 운영 및 릴리스 문서 |
| `resume/` | Architecture, API, deployment, and troubleshooting references / 아키텍처, API, 배포, 트러블슈팅 참고 자료 |
| `demo/` | Demo scripts and walkthroughs / 데모 스크립트 및 워크스루 |

---

## 5. Quick Start / 빠른 시작

### 5.1 Prerequisites / 사전 요구 사항

| Requirement / 요구 사항 | Version / 버전 | Notes / 비고 |
|---|---|---|
| Splunk Enterprise / Splunk 엔터프라이즈 | 8.x or later / 8.x 이상 | Search head environment / 검색 헤드 환경 |
| Splunk Cloud / Splunk 클라우드 | Compatible tier / 호환 티어 | Validate with your account team / 계정 팀에 확인 |
| Disk Space / 디스크 공간 | ≥ 50 MB | For bundled Python libraries / 번들 Python 라이브러리용 |
| Network egress (optional) / 네트워크 송신 (선택) | Slack webhook reachable | Required only if Slack action is enabled / Slack 액션 사용 시에만 필요 |

### 5.2 Installation / 설치

1. **Download or clone the repository** / **저장소 다운로드 또는 클론**

   ```bash
   git clone <repository-url> security-alert
   cd security-alert
   ```

2. **Package the app (optional)** / **앱 패키징 (선택)**

   ```bash
   tar czf security_alert.tgz security_alert/
   ```

3. **Install on Splunk** / **Splunk에 설치**

   | Method / 방법 | Steps / 단계 |
   |---|---|
   | Splunk Web UI / Splunk 웹 UI | Settings → Apps → Install app from file → select `security_alert.tgz` / 설정 → 앱 → 파일에서 앱 설치 → `security_alert.tgz` 선택 |
   | CLI / CLI | `splunk install app security_alert.tgz -auth <user>:<password>` |
   | Manual / 수동 | Copy `security_alert/` to `$SPLUNK_HOME/etc/apps/` and restart Splunk / `security_alert/`를 `$SPLUNK_HOME/etc/apps/`에 복사하고 Splunk 재시작 |

4. **Restart Splunk** / **Splunk 재시작**

   ```bash
   $SPLUNK_HOME/bin/splunk restart
   ```

5. **Verify the app appears** / **앱 표시 확인**

   - Open the Apps menu in Splunk Web and confirm **Security Alert** is listed. / Splunk 웹의 앱 메뉴를 열고 **Security Alert**가 표시되는지 확인합니다.
   - Open the **Alert Management Dashboard** to confirm the view renders. / **알림 관리 대시보드**를 열어 뷰가 렌더링되는지 확인합니다.

### 5.3 First Alert / 첫 알림 생성

1. Navigate to **Easy Alert Builder** in the app menu. / 앱 메뉴에서 **손쉬운 알림 빌더**로 이동합니다.
2. Fill in the wizard fields: name, index, schedule, search filter. / 마법사 필드(알림 이름, 인덱스, 스케줄, 검색 필터)를 입력합니다.
3. Click **Save**. The saved search appears in **Alert Management Dashboard**. / **저장**을 클릭합니다. 저장된 검색이 **알림 관리 대시보드**에 표시됩니다.
4. (Optional) Attach the bundled Slack action: edit the saved search, add an alert action, and supply a webhook URL. / (선택) 번들된 Slack 액션을 첨부합니다. 저장된 검색을 편집하고 알림 액션을 추가한 후 웹훅 URL을 입력합니다.

---

## 6. Configuration / 설정

All baseline configuration lives under `security_alert/default/`. Per-instance overrides belong in `security_alert/local/`, which is not committed to source control.

모든 기준 설정은 `security_alert/default/`에 있습니다. 인스턴스별 재정의는 소스 제어에 포함되지 않는 `security_alert/local/`에 저장됩니다.

### 6.1 Configuration Files / 설정 파일

| File / 파일 | Purpose / 용도 |
|---|---|
| `app.conf` | App identity, label, and UI visibility / 앱 정체성, 라벨, UI 노출 |
| `alert_actions.conf` | Declares the bundled custom alert action (Slack) / 번들된 커스텀 알림 액션(Slack) 선언 |
| `savedsearches.conf` | Default saved searches and scheduled alerts / 기본 저장된 검색 및 예약된 알림 |
| `macros.conf` | Reusable SPL macro definitions / 재사용 가능한 SPL 매크로 정의 |
| `props.conf` | Field extractions and timestamp rules / 필드 추출 및 타임스탬프 규칙 |
| `transforms.conf` | Lookup and field transformation definitions / 룩업 및 필드 변환 정의 |
| `metadata/default.meta` | Object-level capabilities / 객체 수준 기능 |
| `default/data/ui/nav/default.xml` | Navigation entries / 탐색 메뉴 항목 |
| `default/data/ui/views/*.xml` | Dashboard and form definitions / 대시보드 및 폼 정의 |

### 6.2 Slack Action Parameters / Slack 액션 파라미터

| Parameter / 파라미터 | Source / 출처 | Description / 설명 |
|---|---|---|
| `webhook_url` | `alert_actions.conf` (per saved search override) | Slack incoming webhook URL / Slack 인커밍 웹훅 URL |
| `channel` | `alert_actions.conf` | Target channel override (if supported by the webhook) / 대상 채널 재정의(웹훅이 지원하는 경우) |
| `message_template` | `alert_actions.conf` | Template string passed to `safe_fmt.py` / `safe_fmt.py`에 전달되는 템플릿 문자열 |

### 6.3 Capability Model / 기능 모델

`security_alert/metadata/default.meta` defines who can read or write app objects. The capability declarations follow Splunk's `default.meta` convention.

`security_alert/metadata/default.meta`는 앱 객체를 읽거나 쓸 수 있는 권한을 정의합니다. 기능 선언은 Splunk의 `default.meta` 규약을 따릅니다.

| Capability / 기능 | Typical Role / 일반 역할 |
|---|---|
| `read` | All authenticated users / 모든 인증된 사용자 |
| `write` | Power users, detection engineers / 파워 유저, 탐지 엔지니어 |
| `delete` | Splunk administrators / Splunk 관리자 |

---

## 7. Component Reference / 구성 요소 참조

### 7.1 Custom Alert Actions / 커스텀 알림 액션

| Script / 스크립트 | Purpose / 용도 |
|---|---|
| `bin/slack.py` | Posts alert results to a Slack incoming webhook / 알림 결과를 Slack 인커밍 웹훅에 게시 |
| `bin/safe_fmt.py` | Sanitizes and substitutes user input inside message templates / 메시지 템플릿에서 사용자 입력 sanitize 및 치환 |
| `bin/six.py` | Python 2 / 3 compatibility shim / Python 2/3 호환성 셰임 |

### 7.2 Views / 뷰

| View / 뷰 | Audience / 대상 |
|---|---|
| `easy_alert_builder.xml` | First-time authors, junior analysts / 초보 작성자, 주니어 분석가 |
| `alert-builder.xml` | Power users and detection engineers / 파워 유저 및 탐지 엔지니어 |
| `alert-management-dashboard.xml` | SOC operations / SOC 운영 |
| `data-explorer-dashboard.xml` | Incident responders / 침해 대응 담당자 |

### 7.3 Bundled Python Runtime / 번들 Python 런타임

| Package / 패키지 | Version / 버전 | Role / 역할 |
|---|---|---|
| `urllib3` | vendored / 동봉 | HTTP client used by `slack.py` / `slack.py`에서 사용하는 HTTP 클라이언트 |
| `idna` | 3.11 | Internationalized domain name support / 국제화 도메인 이름 지원 |
| `charset_normalizer` | 3.4.4 | Encoding detection for outbound payloads / 송신 페이로드의 인코딩 감지 |
| `six` | vendored / 동봉 | Python 2/3 compatibility helpers / Python 2/3 호환성 헬퍼 |

---

## 8. Local Development / 로컬 개발

### 8.1 Development Environment / 개발 환경

| Tool / 도구 | Purpose / 용도 |
|---|---|
| Splunk Enterprise (developer license) / Splunk 엔터프라이즈(개발자 라이선스) | Runtime / 런타임 |
| Python 3.x | Reference interpreter for `bin/*.py` / `bin/*.py`의 참조 인터프리터 |
| Code editor with XML support / XML 지원 코드 편집기 | Editing `.conf` and dashboard XML / `.conf` 및 대시보드 XML 편집 |

### 8.2 Iterative Workflow / 반복 워크플로우

1. **Edit source files** in your working copy. / 작업 사본에서 **소스 파일 편집**합니다.
2. **Sync to Splunk** by symlinking or copying the app directory: / **앱 디렉터리를 심볼릭 링크하거나 복사하여 Splunk에 동기화**합니다.

   ```bash
   ln -s "$(pwd)/security_alert" "$SPLUNK_HOME/etc/apps/security_alert"
   ```

3. **Reload the app** without restarting Splunk: / Splunk를 재시작하지 않고 **앱 리로드**:

   ```bash
   $SPLUNK_HOME/bin/splunk reload app security_alert
   ```

4. **Hard refresh the browser** to pick up view changes. / 뷰 변경 사항을 반영하려면 **브라우저를 강제 새로 고침**합니다.
5. **Tail the log** when testing custom alert actions: / 커스텀 알림 액션을 테스트할 때 **로그 모니터링**:

   ```bash
   tail -f $SPLUNK_HOME/var/log/splunk/splunkd.log
   ```

### 8.3 Editing Best Practices / 편집 모범 사례

| Area / 영역 | Practice / 모범 사례 |
|---|---|
| Dashboards / 대시보드 | Use Simple XML; validate against Splunk's schema / Simple XML 사용; Splunk 스키마로 검증 |
| Saved searches / 저장된 검색 | Keep `disabled = 0` in `default/`, set `enabled = 1` in `local/` for site-specific toggles / `default/`에는 `disabled = 0` 유지, 사이트별 토글은 `local/`에서 `enabled = 1` 설정 |
| Custom alert actions / 커스텀 알림 액션 | Always run user input through `safe_fmt.py` / 사용자 입력은 항상 `safe_fmt.py`를 통과 |
| Bundled libraries / 번들 라이브러리 | Never modify vendored packages; update via a documented process / 동봉 패키지를 수정하지 말고 문서화된 프로세스로 업데이트 |

---

## 9. Testing / 테스트

| Test Type / 테스트 유형 | Approach / 접근법 | Location / 위치 |
|---|---|---|
| Static checks / 정적 검사 | Lint Python with `python -m py_compile` / Python을 `python -m py_compile`로 린트 | `security_alert/bin/` |
| XML validation / XML 검증 | Validate dashboard XML against Splunk's Simple XML schema / 대시보드 XML을 Splunk Simple XML 스키마로 검증 | `security_alert/default/data/ui/views/` |
| Smoke test / 스모크 테스트 | Install the app on a dev instance, open each view / 개발 인스턴스에 앱 설치, 각 뷰 열기 | Splunk Web / Splunk 웹 |
| Custom action test / 커스텀 액션 테스트 | Fire a saved search, confirm Slack delivery (use a test webhook) / 저장된 검색 실행, Slack 전달 확인(테스트 웹훅 사용) | `bin/slack.py` |
| Knowledge validation / 지식 객체 검증 | Run `btool check` for each `.conf` / 각 `.conf`에 대해 `btool check` 실행 | `security_alert/default/` |

### 9.1 Manual Test Checklist / 수동 테스트 체크리스트

1. App loads in the launcher. / 앱이 런처에 로드됩니다.
2. All four views render without error. / 네 개의 뷰가 오류 없이 렌더링됩니다.
3. Easy Alert Builder produces a saved search visible in the management dashboard. / 손쉬운 알림 빌더가 관리 대시보드에 표시되는 저장된 검색을 생성합니다.
4. Saved search fires on schedule and triggers the Slack action. / 저장된 검색이 일정에 따라 실행되어 Slack 액션을 트리거합니다.
5. Data Explorer Dashboard returns results for the chosen index. / 데이터 탐색기 대시보드가 선택한 인덱스에 대한 결과를 반환합니다.

---

## 10. Operations / 운영

### 10.1 Observability Surface / 관측 가능성 표면

| Signal / 신호 | Source / 출처 | Operator Action / 운영자 조치 |
|---|---|---|
| Custom alert action logs / 커스텀 알림 액션 로그 | `splunkd.log`, `python.log` | Filter on `component=AlertActions` / `component=AlertActions` 필터 |
| Saved search run history / 저장된 검색 실행 기록 | Splunk scheduler UI / Splunk 스케줄러 UI | Inspect skipped or failed firings / 건너뛰기 또는 실패한 실행 확인 |
| Knowledge object diff / 지식 객체 차이 | `local/` vs `default/` | Reconcile with `btool list` / `btool list`로 조정 |

### 10.2 Common Operator Tasks / 일반 운영 작업

| Task / 작업 | Reference / 참고 |
|---|---|
| Update the app / 앱 업데이트 | `docs/DEPLOYMENT.md` |
| Diagnose alert delivery issues / 알림 전달 문제 진단 | `resume/TROUBLESHOOTING.md` |
| Review legacy behavior / 레거시 동작 검토 | `docs/LEGACY-CLEANUP-REPORT.md` |
| Roll out a new release / 새 릴리스 배포 | `docs/RELEASE-NOTES.md` |

### 10.3 Upgrade Procedure / 업그레이드 절차

1. Stop active alert actions referencing the old app version. / 이전 앱 버전을 참조하는 활성 알림 액션을 중지합니다.
2. Replace `security_alert/` under `$SPLUNK_HOME/etc/apps/`. / `$SPLUNK_HOME/etc/apps/` 아래의 `security_alert/`를 교체합니다.
3. Compare `local/` to the new `default/` to spot deprecated stanzas. / 새 `default/`와 `local/`를 비교하여 사용 중단된 스탠자를 확인합니다.
4. Restart Splunk and re-validate the four views. / Splunk를 재시작하고 네 개의 뷰를 재검증합니다.

---

## 11. Documentation Index / 문서 인덱스

| Document / 문서 | Path / 경로 | Audience / 대상 |
|---|---|---|
| API Reference / API 참조 | `resume/API.md` | Developers integrating with the alert action / 알림 액션과 통합하는 개발자 |
| Architecture Deep Dive / 아키텍처 심층 분석 | `resume/ARCHITECTURE.md` | Engineers / 엔지니어 |
| Deployment Guide / 배포 가이드 | `resume/DEPLOYMENT.md` | Splunk administrators / Splunk 관리자 |
| Troubleshooting / 트러블슈팅 | `resume/TROUBLESHOOTING.md` | Operators / 운영자 |
| Quick Start / 빠른 시작 | `docs/QUICK-START.md` | First-time installers / 최초 설치자 |
| Deployment Notes / 배포 노트 | `docs/DEPLOYMENT.md` | Splunk administrators / Splunk 관리자 |
| Release Notes / 릴리스 노트 | `docs/RELEASE-NOTES.md` | All users / 모든 사용자 |
| Alert Repository XWiki Notes / 알림 저장소 XWiki 노트 | `docs/ALERT-REPOSITORY-XWIKI.md` | Documentation maintainers / 문서 유지보수자 |
| Legacy Cleanup Report / 레거시 정리 보고서 | `docs/LEGACY-CLEANUP-REPORT.md` | Maintainers / 유지보수자 |
| Demo Walkthrough / 데모 워크스루 | `demo/README.md` | Trainers and evaluators / 교육자 및 평가자 |
| App-level README / 앱 수준 README | `security_alert/README.md` | End users installed in Splunk / Splunk에 설치된 최종 사용자 |
| Contributing / 기여 | `CONTRIBUTING.md` | Contributors / 기여자 |
| License / 라이선스 | `LICENSE` | All / 전체 |

---

## 12. Contributing / 기여

Contributions are welcome. Before opening a pull request:

기여를 환영합니다. 풀 리퀘스트를 열기 전에 다음을 확인합니다.

1. Read `CONTRIBUTING.md` for coding standards and review process. / 코딩 표준 및 리뷰 프로세스는 `CONTRIBUTING.md`를 읽어 주세요.
2. Keep configuration changes in `default/` minimal; prefer `local/` overrides for site-specific values. / `default/`의 설정 변경은 최소화하고 사이트별 값은 `local/` 재정의 사용을 권장합니다.
3. Do not commit changes to vendored Python packages under `security_alert/lib/python3/`. / `security_alert/lib/python3/` 아래의 동봉 Python 패키지 변경은 커밋하지 마세요.
4. Update the relevant doc in `docs/` or `resume/` whenever you add a new view, knowledge stanza, or alert action. / 새 뷰, 지식 스탠자, 알림 액션을 추가할 때마다 `docs/` 또는 `resume/`의 관련 문서를 업데이트하세요.
5. Run the manual test checklist in section 9.1 before requesting review. / 리뷰 요청 전에 9.1절의 수동 테스트 체크리스트를 실행하세요.

---

## 13. License / 라이선스

This project is released under the terms described in the `LICENSE` file at the root of the repository.

이 프로젝트는 저장소 루트의 `LICENSE` 파일에 명시된 조건에 따라 배포됩니다.