# Security Alert (Splunk App) / 보안 알림 (Splunk 앱)

프로덕션급 Splunk 애드온으로, 통합 알림 관리 대시보드, 안내형 손쉬운 알림 빌더, 풀-기능 알림 빌더, 그리고 탐색형 데이터 탐색기 대시보드를 제공한다. 안전한 포맷팅 헬퍼와 Slack 커스텀 알림 액션을 함께 제공하며, Python 런타임 의존성을 사전 번들하여 외부 네트워크가 없는(air-gapped) Splunk 환경에서도 동작한다.

A production-grade Splunk add-on that delivers a unified Alert Management Dashboard, a guided Easy Alert Builder, a feature-complete Alert Builder, and an exploratory Data Explorer Dashboard. It ships with a safe-formatting helper and a Slack custom alert action, and bundles Python runtime dependencies so it runs on air-gapped Splunk deployments.

---

## 운영 상태 (Status)

| Runtime / 실행 환경 | Status / 상태 | Owner / 책임자 | Next action / 다음 작업 |
| --- | --- | --- | --- |
| Splunk 9.x | Production | App author | `security_alert/` 폴더를 `.tar.gz`로 묶어 `security_alert.spl` 패키지를 생성한 뒤 *Manage Apps*에서 설치 |
| Splunk 8.x | Compatible | App author | 설치 후 8.x 인스턴스에서 대시보드 렌더링을 검증 |
| Air-gapped Splunk | Supported | App author | 외부 Python 패키지 다운로드 불필요 (의존성 사전 번들 완료) |
| Python 3 runtime | Bundled | App author | `security_alert/lib/python3/` 에 urllib3, charset_normalizer, idna 포함 |
| Maintenance mode | Active | App author | `docs/RELEASE-NOTES.md` 와 `docs/LEGACY-CLEANUP-REPORT.md` 참조 |

---

## 빠른 흐름 요약 (Quick-flow summary)

1. **Build the package** — `security_alert/` 폴더를 `.tar.gz` 로 묶어 `security_alert.spl` 로 패키징한다.
2. **Install the package** — Splunk Web 의 *Manage Apps* 화면에서 업로드하거나 `$SPLUNK_HOME/etc/apps/` 에 압축 해제한다.
3. **Restart Splunk** — (필요 시) `splunk restart` 로 앱을 활성화한다.
4. **Configure routes** — `security_alert/default/alert_actions.conf` 에서 Slack 알림 액션을 활성화하고 webhook 을 등록한다.
5. **Author alerts** — **Easy Alert Builder** 또는 **Alert Builder** 대시보드 뷰에서 알림을 작성한다.
6. **Operate alerts** — **Alert Management Dashboard** 에서 분류(triage)하고, **Data Explorer Dashboard** 에서 심층 분석한다.

---

## 1. 목적 (Purpose)

`security_alert/` 는 임시 알림 작성을 반복 가능하고 감사 가능한 워크플로우로 전환하는 자급식(self-contained) Splunk 앱이다. 탐지 엔지니어, SOC 분석가, Splunk 관리자가 한 곳에서 알림을 설계·배포·검토할 수 있도록 다음 네 가지 기능을 통합한다.

- **알림 작성** — Easy Alert Builder(안내형)와 Alert Builder(고급형) 양쪽 진입점을 제공한다.
- **알림 운영** — Alert Management Dashboard 에서 활성 알림을 분류하고 상태를 추적한다.
- **데이터 탐색** — Data Explorer Dashboard 에서 원시 이벤트를 시간/필드 기준으로 드릴다운한다.
- **알림 라우팅** — Slack 커스텀 알림 액션으로 외부 협업 채널에 전달한다.

A self-contained Splunk app that converts ad-hoc alert authoring into a repeatable, auditable workflow. It unifies authoring, operation, exploration, and routing of Splunk alerts behind four entry points (two builders, two dashboards) plus a Slack custom action.

---

## 목차 (Table of Contents)

1. [목적 (Purpose)](#1-목적-purpose)
2. [패키지 구성 (Package Contents)](#2-패키지-구성-package-contents)
3. [먼저 읽을 파일 (First Files to Read)](#3-먼저-읽을-파일-first-files-to-read)
4. [API / 진입점 (API & Entry Points)](#4-api--진입점-api--entry-points)
5. [빠른 시작 (Quickstart)](#5-빠른-시작-quickstart)
6. [아키텍처 (Architecture)](#6-아키텍처-architecture)
7. [설정 (Configuration)](#7-설정-configuration)
8. [명령어 (Commands Reference)](#8-명령어-commands-reference)
9. [로컬 개발 및 테스트 (Local Development & Testing)](#9-로컬-개발-및-테스트-local-development--testing)
10. [유지보수 및 트러블슈팅 (Maintenance & Troubleshooting)](#10-유지보수-및-트러블슈팅-maintenance--troubleshooting)
11. [기여 (Contributing)](#11-기여-contributing)
12. [라이선스 (License)](#12-라이선스-license)

---

## 2. 패키지 구성 (Package Contents)

### 2.1 최상위 디렉터리 (Top-level)

| 경로 / Path | 용도 / Purpose |
| --- | --- |
| `security_alert/` | Splunk 앱 루트. `security_alert.spl` 로 직접 패키징된다. |
| `docs/` | 운영 문서 (Quick Start, Release Notes, 배포 가이드, 정리 리포트) |
| `resume/` | 앱 참조 문서 (API, Architecture, Deployment, Troubleshooting) |
| `demo/` | 데모 시연 자료 |
| `CONTRIBUTING.md` | 기여 절차 및 PR 정책 |
| `LICENSE` | 라이선스 전문 |

### 2.2 `security_alert/` 앱 내부 (App internals)

| 경로 / Path | 역할 / Role |
| --- | --- |
| `app.manifest` | Splunk 앱 매니페스트 (이름, 버전, 작성자, 지원 런타임) |
| `README.md` | 앱 단위 운영 노트 및 컨벤션 |
| `default/app.conf` | 앱 식별자, 라벨, UI 기본값 |
| `default/alert_actions.conf` | Slack 커스텀 알림 액션 정의 |
| `default/macros.conf` | 재사용 SPL 매크로 가드레일 |
| `default/props.conf` | 인덱스별 필드 추출 / 인덱싱 규칙 |
| `default/savedsearches.conf` | 알림 트리거가 되는 저장된 검색 |
| `default/transforms.conf` | 필드 변환 룰 |
| `default/data/ui/nav/default.xml` | 앱 내비게이션 메뉴 노출 |
| `default/data/ui/views/easy_alert_builder.xml` | 안내형 알림 작성기 뷰 |
| `default/data/ui/views/alert-builder.xml` | 고급 알림 작성기 뷰 |
| `default/data/ui/views/alert-management-dashboard.xml` | 알림 운영 대시보드 뷰 |
| `default/data/ui/views/data-explorer-dashboard.xml` | 데이터 탐색 대시보드 뷰 |
| `bin/safe_fmt.py` | 사용자 입력 문자열의 안전한 포맷팅 헬퍼 |
| `bin/slack.py` | Slack 커스텀 알림 액션 스크립트 |
| `bin/six.py` | Python 2/3 호환 레이어 |
| `lib/python3/urllib3/` | 사전 번들 HTTP 클라이언트 |
| `lib/python3/charset_normalizer-*/` | 사전 번들 응답 인코딩 감지 |
| `lib/python3/idna-*/` | 사전 번들 IDN 도메인 처리 |
| `metadata/default.meta` | 앱 객체 capabilities 부여 |

---

## 3. 먼저 읽을 파일 (First Files to Read)

방문 목적별로 먼저 살펴볼 파일을 안내한다. 새 운영자는 ①→⑤ 순서로 읽으면 된다.

| 순서 | 파일 / File | 읽을 이유 / Why |
| --- | --- | --- |
| 1 | [`security_alert/README.md`](security_alert/README.md) | 앱 단위 운영 노트와 컨벤션을 확인한다. |
| 2 | [`security_alert/app.manifest`](security_alert/app.manifest) | 앱 메타데이터(버전, 작성자, 플랫폼 호환성)를 확인한다. |
| 3 | [`security_alert/default/app.conf`](security_alert/default/app.conf) | 앱 라벨과 내비게이션 기본값을 확인한다. |
| 4 | [`docs/QUICK-START.md`](docs/QUICK-START.md) | 설치부터 첫 알림 작성까지의 빠른 경로를 확인한다. |
| 5 | [`resume/ARCHITECTURE.md`](resume/ARCHITECTURE.md) | 대시보드·액션 모듈 관계와 책임 경계를 이해한다. |
| 6 | [`docs/ALERT-REPOSITORY-XWIKI.md`](docs/ALERT-REPOSITORY-XWIKI.md) | 알림 카탈로그 정책(어떤 알림이 어디 정의되는지)을 확인한다. |
| 7 | [`docs/LEGACY-CLEANUP-REPORT.md`](docs/LEGACY-CLEANUP-REPORT.md) | 레거시 자산 정리 이력과 신규 영향을 검토한다. |

---

## 4. API / 진입점 (API & Entry Points)

Splunk 앱이므로 "진입점" 은 크게 세 가지 표면 — UI 대시보드, SPL 매크로, 커스텀 알림 액션 — 으로 구성된다. 외부 REST 핸들러는 본 앱에서 직접 노출하지 않는다.

### 4.1 대시보드 / 뷰 (Dashboards)

| 경로 / Path | 진입점 / Entry | 목적 / Purpose |
| --- | --- | --- |
| `default/data/ui/views/easy_alert_builder.xml` | Easy Alert Builder | 사전 정의된 템플릿으로 알림을 빠르게 작성 (안내형) |
| `default/data/ui/views/alert-builder.xml` | Alert Builder | 모든 SPL 필드를 자유롭게 설정 (풀-기능) |
| `default/data/ui/views/alert-management-dashboard.xml` | Alert Management Dashboard | 활성 알림 분류/상태/히스토리 운영 |
| `default/data/ui/views/data-explorer-dashboard.xml` | Data Explorer Dashboard | 원시 이벤트 시간/필드 드릴다운 |

### 4.2 SPL 매크로 (Macros)

| 파일 / File | 사용 예 / Usage | 설명 / Notes |
| --- | --- | --- |
| `default/macros.conf` | `\| \`security_alert_macro($field$)\`` | 공통 SPL 매크로. 구체 키는 파일 참조 |

### 4.3 커스텀 알림 액션 (Custom Alert Actions)

| 스크립트 / Script | 모드 / Mode | 책임 / Responsibility |
| --- | --- | --- |
| `bin/slack.py` | alert action | 검색 결과를 Slack webhook 으로 전송한다. |
| `bin/safe_fmt.py` | helper | 사용자 입력 문자열을 안전하게 새니타이즈·포맷팅한다. |
| `bin/six.py` | helper | Python 2/3 호환 레이어를 제공한다. |

### 4.4 외부 의존성 (External dependencies)

| 패키지 / Package | 출처 / Source | 용도 / Purpose |
| --- | --- | --- |
| urllib3 | bundled (`lib/python3/urllib3/`) | Slack webhook 등 HTTP 호출 |
| charset_normalizer | bundled (`lib/python3/charset_normalizer-*/`) | HTTP 응답 인코딩 감지 |
| idna | bundled (`lib/python3/idna-*/`) | IDN 도메인 처리 |

> 외부 PyPI 미러가 없는 에어갭 Splunk 인스턴스를 위해 위 의존성은 사전 번들되어 있다. 신규 의존성을 추가할 때는 동일 경로에 번들한 뒤 `app.manifest` 의 메타데이터를 함께 갱신한다.

---

## 5. 빠른 시작 (Quickstart)

로컬 또는 스테이징 Splunk 인스턴스에서 5 분 안에 첫 알림까지 도달하는 경로다.

### 5.1 사전 요구 사항 (Prerequisites)

- **Splunk Enterprise** 8.x 이상 (9.x 권장)
- **Splunk 관리자** 자격 증명 (앱 설치 및 검색 권한)
- **Slack** 워크스페이스와 incoming webhook URL (Slack 액션 사용 시)
- **패키징 도구** `tar` (Splunk *App Builder* 사용 시 불필요)

### 5.2 단계별 설치 (Step-by-step)

```bash
# 1) 패키지 빌드 — security_alert/ → security_alert.spl
cd <repo-root>
tar -czf security_alert.spl security_alert/

# 2) Splunk 인스턴스에 배치
#    Option A — CLI 직접 배치
mkdir -p "$SPLUNK_HOME/etc/apps"
tar -xzf security_alert.spl -C "$SPLUNK_HOME/etc/apps/"

#    Option B — Splunk Web 업로드
#    Settings → Apps → Manage Apps → "Install app from file" → security_alert.spl

# 3) Splunk 재시작 (첫 설치 또는 app.conf 변경 시)
"$SPLUNK_HOME/bin/splunk" restart

# 4) Slack 알림 액션 활성화 — local/alert_actions.conf 에 webhook 등록
mkdir -p "$SPLUNK_HOME/etc/apps/security_alert/local"
$EDITOR "$SPLUNK_HOME/etc/apps/security_alert/local/alert_actions.conf"
# 예시) security_alert_slack_webhook = https://hooks.slack.com/services/XXX/YYY/ZZZ
```

### 5.3 첫 알림 작성 (Author your first alert)

1. Splunk Web 에 로그인 후 상단 앱 메뉴에서 **Security Alert** 앱으로 전환한다.
2. **Easy Alert Builder** 대시보드를 열고 사전 정의된 템플릿을 선택한다.
3. 인덱스·시간 범위·트리거 조건을 입력한 뒤 **Save** 로 알림 이름을 지정한다.
4. *Trigger conditions* 단계에서 **Slack** 알림 액션을 선택한다.
5. *Saved Searches* 에서 새 알림이 `Enabled` 상태로 보이는지 확인한다.
6. **Alert Management Dashboard** 로 이동해 첫 알림이 정상 표시되는지 검증한다.

자세한 운영 절차는 [`docs/QUICK-START.md`](docs/QUICK-START.md) 와 [`resume/DEPLOYMENT.md`](resume/DEPLOYMENT.md) 를 참조한다.

---

## 6. 아키텍처 (Architecture)

### 6.1 컴포넌트 맵 (Component map)

| 계층 / Layer | 컴포넌트 / Component | 대표 파일 / File | 책임 / Responsibility |
| --- | --- | --- | --- |
| Presentation | Easy Alert Builder | `default/data/ui/views/easy_alert_builder.xml` | 안내형 알림 작성 UI |
| Presentation | Alert Builder | `default/data/ui/views/alert-builder.xml` | 풀-기능 알림 작성 UI |
| Presentation | Alert Management Dashboard | `default/data/ui/views/alert-management-dashboard.xml` | 알림 운영 / 분류 |
| Presentation | Data Explorer Dashboard | `default/data/ui/views/data-explorer-dashboard.xml` | 이벤트 드릴다운 |
| Navigation | 앱 내비게이션 | `default/data/ui/nav/default.xml` | 메뉴 진입점 노출 |
| Configuration | 앱 메타데이터 | `default/app.conf`, `app.manifest` | 앱 식별자·플랫폼 |
| Configuration | 알림 라우팅 정의 | `default/alert_actions.conf` | 외부 채널 라우팅 정의 |
| Search | 저장된 검색 | `default/savedsearches.conf` | 알림 트리거 검색 |
| Search | 매크로 / 변환 | `default/macros.conf`, `default/transforms.conf` | 재사용 SPL 룰 |
| Indexing | 필드 추출 | `default/props.conf` | 인덱스별 필드 처리 |
| Runtime | 커스텀 알림 액션 | `bin/slack.py`, `bin/safe_fmt.py`, `bin/six.py` | 알림 라우팅 실행 |
| Permissions | 앱 권한 | `metadata/default.meta` | 객체 capabilities 부여 |

### 6.2 요청 흐름 (Request flow)

1. 운영자가 Splunk Web 에서 **Security Alert** 앱으로 진입한다.
2. 앱 내비게이션이 4 개 뷰(Easy / Full / Management / Explorer)를 노출한다.
3. 작성기에서 제출한 검색 식이 `savedsearches.conf` 형태로 저장되고 즉시 *Enabled* 가 된다.
4. Splunk 스케줄러가 검색을 주기적으로 실행해 트리거 조건을 평가한다.
5. 조건 매칭 시 등록된 **alert action** (예: `slack.py`) 이 실행된다.
6. `bin/safe_fmt.py` 가 메시지를 안전하게 포맷팅한 뒤 `lib/python3/urllib3` 로 Slack webhook 을 호출한다.
7. Management Dashboard 가 결과 상태를 표시하고, Data Explorer 가 후속 분석 진입점을 제공한다.

상세 다이어그램과 모듈 경계는 [`resume/ARCHITECTURE.md`](resume/ARCHITECTURE.md) 와 [`resume/API.md`](resume/API.md) 를 참조한다.

---

## 7. 설정 (Configuration)

### 7.1 앱 메타데이터 (App metadata)

| 파일 / File | 권장 수정 위치 / Override location | 키 예시 / Key example |
| --- | --- | --- |
| `default/app.conf` | `local/app.conf` | `[install]` `is_configured = true` |
| `app.manifest` | 편집 비권장 (배포 스크립트에서 갱신) | `name`, `version`, `author` |

### 7.2 알림 라우팅 (Alert routing)

`default/alert_actions.conf` 는 커스텀 액션 정의만 포함한다. 환경 의존 값(예: webhook URL, 채널)은 `local/alert_actions.conf` 에서 정의해 기본 파일을 덮어쓴다.

| 항목 / Item | 위치 / Location | 예시 / Example |
| --- | --- | --- |
| Slack webhook URL | `local/alert_actions.conf` | `security_alert_slack_webhook = https://hooks.slack.com/services/...` |
| Slack 채널 기본값 | `local/alert_actions.conf` | `security_alert_slack_channel = #soc-alerts` |

### 7.3 검색 및 인덱싱 (Search & indexing)

| 파일 / File | 용도 / Purpose | 신규 작업 시 / When to extend |
| --- | --- | --- |
| `default/macros.conf` | 공통 SPL 매크로 | 가드레일 매크로 추가 시 |
| `default/transforms.conf` | 변환 룰 | 신규 룰 추가 시 stanza 명세 |
| `default/props.conf` | 인덱스 필드 처리 | 신규 인덱스 / 소스 추가 시 |
| `default/savedsearches.conf` | 알림 트리거 검색 | 신규 알림 검색 정의 시 |

### 7.4 권한 (Permissions)

| 파일 / File | 역할 / Role |
| --- | --- |
| `metadata/default.meta` | 앱 객체 기본 capabilities. `local/default.meta` 로 세분화 권한 부여 |

---

## 8. 명령어 (Commands Reference)

운영자가 반복적으로 호출하는 명령만 수록한다.

### 8.1 패키징 (Packaging)

```bash
# 표준 Splunk 앱 패키지 (.spl = gzip 으로 묶인 tar)
tar -czf security_alert.spl security_alert/
```

### 8.2 설치 (Install)

```bash
# Option A — CLI 직접 배치
mkdir -p "$SPLUNK_HOME/etc/apps"
tar -xzf security_alert.spl -C "$SPLUNK_HOME/etc/apps/"

# Option B — Splunk Web UI
# Settings → Apps → Manage Apps → "Install app from file" → security_alert.spl 업로드
```

### 8.3 서비스 제어 (Service control)

```bash
# 앱 활성화 (필요 시 Splunk 재시작)
"$SPLUNK_HOME/bin/splunk" restart

# 설정 검증
"$SPLUNK_HOME/bin/splunk" btool check --app=security_alert
```

### 8.4 알림 액션 단위 테스트 (Alert action dry-run)

```bash
# Slack webhook 단위 테스트 (앱 실행 환경에서)
python3 "$SPLUNK_HOME/etc/apps/security_alert/bin/slack.py" \
    --webhook "https://hooks.slack.com/services/XXX/YYY/ZZZ" \
    --channel "#soc-test" \
    --text "Security Alert dry-run"
```

> 운영 시 `--webhook` 인수에 실제 URL 을 입력한다. 데모 절차는 [`demo/README.md`](demo/README.md) 를 참조한다.

---

## 9. 로컬 개발 및 테스트 (Local Development & Testing)

### 9.1 개발 환경 (Dev environment)

| 항목 / Item | 권장 / Recommended |
| --- | --- |
| Splunk 버전 | 9.x (개발), 8.x (호환성 회귀) |
| Python | 3.7+ (앱에 번들된 urllib3 / charset_normalizer / idna 가 호환되는 버전) |
| 코드 에디터 | Simple XML 검증을 위해 Splunk Web 의 *Source* 뷰 활용 |
| 데이터 | 자체 인덱스(예: `sec_alert_dev`) 와 권한 있는 사용자 롤 |

### 9.2 수정 워크플로우 (Edit workflow)

1. 저장소에서 파일을 수정한다.
2. `$SPLUNK_HOME/etc/apps/security_alert/` 에 동기화한다 (심볼릭 링크 또는 수동 복사).
3. Splunk Web *Manage Apps* 에서 **Reload** 하거나 `splunk restart` 로 반영한다.
4. 영향받는 대시보드를 열어 회귀를 확인한다.

### 9.3 테스트 체크리스트 (Test checklist)

| 영역 / Area | 검증 / Verification |
| --- | --- |
| Easy Alert Builder | 템플릿 선택 → 저장 → 결과 알림이 *Saved Searches* 에 표시 |
| Alert Builder | 임의 SPL 작성 → 저장 → 트리거 동작 |
| Alert Management Dashboard | 신규 알림이 분류 큐에 정상 표시 |
| Data Explorer Dashboard | 시간·필드 드릴다운이 빈 결과에서 크래시 없이 동작 |
| Slack 액션 | 트리거된 알림이 webhook 으로 수신 |
| 오프라인 환경 | 외부망 차단 상태에서 의존성 임포트 실패가 없음 |

### 9.4 보안 노트 (Security notes)

- `bin/safe_fmt.py` 는 외부에서 들어온 문자열을 항상 새니타이즈한 뒤 출력한다. 새 알림 액션을 작성할 때도 동일한 헬퍼를 재사용한다.
- Slack webhook 같은 비밀 값은 절대 `default/` 하위에 두지 말고 `local/` 에 둔다.
- 외부 송신 데이터가 있다면 `app.manifest` 의 관련 항목을 갱신해 의존성을 명시한다.

---

## 10. 유지보수 및 트러블슈팅 (Maintenance & Troubleshooting)

### 10.1 운영 문서 (Operational docs)

| 문서 / Doc | 용도 / Purpose |
| --- | --- |
| [`docs/QUICK-START.md`](docs/QUICK-START.md) | 첫 설치부터 첫 알림까지의 빠른 경로 |
| [`docs/DEPLOYMENT.md`](docs/DEPLOYMENT.md) | 배포 절차 및 환경별 설정 |
| [`docs/RELEASE-NOTES.md`](docs/RELEASE-NOTES.md) | 버전별 변경 이력 |
| [`docs/LEGACY-CLEANUP-REPORT.md`](docs/LEGACY-CLEANUP-REPORT.md) | 레거시 자산 정리 결과 |
| [`docs/ALERT-REPOSITORY-XWIKI.md`](docs/ALERT-REPOSITORY-XWIKI.md) | 알림 카탈로그 정책 |
| [`resume/API.md`](resume/API.md) | 내부 API / 모듈 참조 |
| [`resume/ARCHITECTURE.md`](resume/ARCHITECTURE.md) | 아키텍처 세부 |
| [`resume/DEPLOYMENT.md`](resume/DEPLOYMENT.md) | 배포 세부 절차 |
| [`resume/TROUBLESHOOTING.md`](resume/TROUBLESHOOTING.md) | 일반 트러블슈팅 |

### 10.2 자주 발생하는 증상 (Common symptoms)

| 증상 / Symptom | 1차 확인 / First check | 참조 문서 / Reference |
| --- | --- | --- |
| 대시보드가 빈 화면 | `app.conf` 의 `is_configured`, 브라우저 캐시 | [`resume/TROUBLESHOOTING.md`](resume/TROUBLESHOOTING.md) |
| Slack 메시지 미수신 | webhook URL, 아웃바운드 네트워크, Splunk 실행 사용자 권한 | [`resume/TROUBLESHOOTING.md`](resume/TROUBLESHOOTING.md) |
| 알림이 트리거되지 않음 | `savedsearches.conf` 의 `disabled`, 스케줄, 권한 | [`resume/TROUBLESHOOTING.md`](resume/TROUBLESHOOTING.md) |
| 의존성 임포트 실패 | `lib/python3/` 디렉터리 누락 파일 검증 | [`docs/DEPLOYMENT.md`](docs/DEPLOYMENT.md) |

### 10.3 지원 채널 (Support channels)

- 내부 이슈 트래커: 저장소 *Issues* 메뉴
- 운영 문서 우선 참조: 위 표의 트러블슈팅 문서
- 보안 이슈: 저장소 메인테이너에게 비공개로 연락 (연락처는 저장소 메타데이터 참조)

---

## 11. 기여 (Contributing)

기여 절차, 코드 스타일, PR 정책은 [`CONTRIBUTING.md`](CONTRIBUTING.md) 를 따른다.

| 영역 / Area | 가이드 / Guide |
| --- | --- |
| 코드 스타일 | `bin/*.py` 는 PEP 8, 함수 독스트링 권장 |
| 커밋 메시지 | 한국어 / 영어 모두 허용, 단일 변경 요약 명시 |
| PR 정책 | 한 PR 당 한 변경 묶음, 대시보드 XML 변경 시 스크린샷 첨부 권장 |
| 신규 의존성 | `lib/python3/` 사전 번들 + `app.manifest` 갱신 필수 |

---

## 12. 라이선스 (License)

이 저장소는 [`LICENSE`](LICENSE) 파일의 내용을 따른다. 사내 배포 및 외부 재배포 시 라이선스 전문을 함께 검토한다.

---

## 책임자 및 연락처 (Maintainers / Points of Contact)

| 이름 / Name | 역할 / Role | 연락 / Contact |
| --- | --- | --- |
| App author | Owner (앱 작성자 / Splunk 관리자) | 저장소 메타데이터 참조 |

상세 책임과 에스컬레이션 절차는 [`CONTRIBUTING.md`](CONTRIBUTING.md) 와 저장소 메인테이너 설정을 따른다.

---

## 추가 문서 (Further Documentation)

| 카테고리 / Category | 링크 / Link |
| --- | --- |
| 앱 단위 노트 | [`security_alert/README.md`](security_alert/README.md) |
| 빠른 시작 | [`docs/QUICK-START.md`](docs/QUICK-START.md) |
| 배포 | [`docs/DEPLOYMENT.md`](docs/DEPLOYMENT.md), [`resume/DEPLOYMENT.md`](resume/DEPLOYMENT.md) |
| API / 모듈 | [`resume/API.md`](resume/API.md) |
| 아키텍처 | [`resume/ARCHITECTURE.md`](resume/ARCHITECTURE.md) |
| 트러블슈팅 | [`resume/TROUBLESHOOTING.md`](resume/TROUBLESHOOTING.md) |
| 릴리스 노트 | [`docs/RELEASE-NOTES.md`](docs/RELEASE-NOTES.md) |
| 레거시 정리 | [`docs/LEGACY-CLEANUP-REPORT.md`](docs/LEGACY-CLEANUP-REPORT.md) |
| 알림 카탈로그 | [`docs/ALERT-REPOSITORY-XWIKI.md`](docs/ALERT-REPOSITORY-XWIKI.md) |
| 데모 자료 | [`demo/README.md`](demo/README.md) |