# 보안 알림 앱 (Security Alert)

> Splunk 환경에서 보안 알림을 손쉽게 만들고, 관리하고, 외부 채널(Slack)로 전달하기 위한 Splunk 앱입니다.

## 한 줄 요약

운영자가 SPL 코드 작성 없이도 알림 규칙을 정의하고, 통합 대시보드에서 탐지·알림 흐름을 추적할 수 있는 Splunk 기반 보안 알림 패키지입니다.
(A Splunk-native security alerting package that lets operators define alert rules without writing SPL and track detection-to-notification flows from a unified dashboard.)

## 상태(Status at a glance)

| 항목 | 값 | 메모 |
|---|---|---|
| 제품 유형 | Splunk 앱 (`app.conf`, `app.manifest`) | `security_alert/` 디렉터리가 Splunk 앱 루트 |
| 알림 채널 | Slack Webhook | `bin/slack.py` |
| 빌더 UI | Easy Alert Builder, Alert Builder | `data/ui/views/` |
| 대시보드 | Alert Management, Data Explorer | `data/ui/views/` |
| 스케줄 평가 | `savedsearches.conf` | Splunk 스케줄러 기반 |
| 메시지 포맷 | `bin/safe_fmt.py` | 알림 페이로드 정규화 |
| 호환성 헬퍼 | `bin/six.py` | Python 2/3 호환 |
| 벤더 라이브러리 | urllib3, idna, charset_normalizer | `lib/python3/` |
| 운영 준비도 | 운영 가능 (테스트 권장) | 단위 테스트는 저장소에 미포함 |
| 라이선스 | `LICENSE` 참조 | 루트 |
| 추가 문서 | `docs/`, `resume/`, `demo/` | 하단 표 참고 |

## 빠른 흐름(Quick Flow)

1. Splunk 앱 디렉터리에 `security_alert/` 배치 후 Splunk 재시작
2. Splunk Web에서 Easy Alert Builder / Alert Builder 열어 규칙 조건 입력
3. 작성한 규칙이 `savedsearches.conf`에 저장 검색으로 등록됨
4. Splunk 스케줄러가 주기적으로 SPL을 실행 → 결과를 산출
5. 임계값/조건 충족 시 `alert_actions.conf`에 정의된 액션 트리거
6. 사용자 정의 액션이 `bin/slack.py`를 호출, `safe_fmt.py`로 안전하게 포맷 후 Slack Webhook으로 POST
7. 동일 이벤트는 Alert Management Dashboard / Data Explorer Dashboard에서 사후 조회

## 목차(Table of Contents)

- [개요(Overview)](#개요overview)
- [주요 기능(Features)](#주요-기능features)
- [아키텍처(Architecture)](#아키텍처architecture)
- [빠른 시작(Quick Start)](#빠른-시작quick-start)
- [설정(Configuration)](#설정configuration)
- [명령 참조(Command Reference)](#명령-참조command-reference)
- [로컬 개발(Local Development)](#로컬-개발local-development)
- [테스트(Testing)](#테스트testing)
- [기여(Contributing)](#기여contributing)
- [유지보수자(Maintainers)](#유지보수자maintainers)
- [추가 문서(Further Documentation)](#추가-문서further-documentation)
- [라이선스(License)](#라이선스license)

## 개요(Overview)

Security Alert 앱은 Splunk 내에서 보안 알림을 만들고 운영하는 작업을 단순화하기 위해 설계되었습니다. 분석가는 `Easy Alert Builder` 또는 `Alert Builder`를 통해 코드 작성 없이 알림 규칙을 정의할 수 있고, 정의된 저장 검색과 알림 흐름은 통합 대시보드에서 가시화됩니다. 알림은 표준 Splunk 알림 액션뿐 아니라 사용자 정의 액션(`bin/slack.py`)을 통해 Slack 같은 외부 시스템으로도 전달됩니다.

English: Security Alert is a Splunk app that simplifies authoring, operating, and reviewing security alerts inside Splunk. Analysts define rules through `Easy Alert Builder` or `Alert Builder` without writing SPL, observe scheduled search and alert action execution from a single dashboard, and forward notifications to external channels (e.g., Slack) via custom alert actions.

## 주요 기능(Features)

| 영역 | 기능 |
|---|---|
| 규칙 작성 | Easy Alert Builder UI, Alert Builder UI (SPL 작성 없이 알림 정의) |
| 스케줄링 | `savedsearches.conf` 기반 주기 평가(Splunk 스케줄러) |
| 알림 액션 | 표준 액션 + 사용자 정의 액션(`bin/slack.py`) |
| 안전 포맷팅 | `bin/safe_fmt.py`로 메시지 정규화 |
| 가시화 | Alert Management Dashboard, Data Explorer Dashboard |
| 필드 처리 | `props.conf` / `transforms.conf` 기반 인덱스/필드 룰 |
| 재사용 SPL | `macros.conf` |
| 권한 | `metadata/default.meta`로 앱/오브젝트 접근 제어 |
| 벤더 의존성 | `lib/python3/`에 urllib3, idna, charset_normalizer 사전 동봉 |

## 아키텍처(Architecture)

### 컴포넌트(Components)

| 경로 | 역할 |
|---|---|
| `security_alert/app.manifest` | Splunk 앱 매니페스트(버전, 호환성 표기) |
| `security_alert/default/app.conf` | Splunk 앱 메타데이터 |
| `security_alert/default/savedsearches.conf` | 알림 트리거용 저장 검색(SPL) |
| `security_alert/default/alert_actions.conf` | 알림 액션 이름과 스크립트 매핑 |
| `security_alert/default/macros.conf` | 재사용 SPL 매크로 |
| `security_alert/default/props.conf` | 필드 추출·타임·props 정의 |
| `security_alert/default/transforms.conf` | 필드 변환 룰 |
| `security_alert/bin/safe_fmt.py` | 메시지 포맷 유틸리티 |
| `security_alert/bin/slack.py` | Slack Webhook 송신 스크립트 |
| `security_alert/bin/six.py` | Python 2/3 호환 헬퍼 |
| `security_alert/data/ui/nav/default.xml` | Splunk 네비게이션 |
| `security_alert/data/ui/views/alert-builder.xml` | Alert Builder 뷰 |
| `security_alert/data/ui/views/easy_alert_builder.xml` | Easy Alert Builder 뷰 |
| `security_alert/data/ui/views/alert-management-dashboard.xml` | Alert Management Dashboard |
| `security_alert/data/ui/views/data-explorer-dashboard.xml` | Data Explorer Dashboard |
| `security_alert/lib/python3/urllib3` | HTTP 클라이언트(벤더) |
| `security_alert/lib/python3/idna` | IDNA 인코딩(벤더) |
| `security_alert/lib/python3/charset_normalizer` | 문자셋 추정(벤더) |
| `security_alert/metadata/default.meta` | 앱 권한 메타데이터 |

### 알림 흐름(Alert Flow)

1. 운영자가 `Easy Alert Builder`에서 규칙 조건 입력 → `savedsearches.conf` 항목 생성
2. Splunk 스케줄러가 해당 저장 검색을 주기적으로 실행 → 결과 산출
3. 결과가 임계값/조건에 부합하면 `alert_actions.conf`에 등록된 알림 액션이 트리거됨
4. 사용자 정의 액션이 `bin/slack.py`를 호출 → `safe_fmt.py`로 안전한 메시지 포맷 후 Slack Webhook으로 HTTP POST
5. 동일 이벤트는 `Alert Management Dashboard`와 `Data Explorer Dashboard`에서 사후 조회·추적

## 빠른 시작(Quick Start)

선결 조건: Splunk Enterprise 인스턴스, Slack 워크스페이스와 Webhook URL, Splunk 앱 디렉터리 접근 권한.

1. 이 저장소를 클론하거나 빌드 산출물을 받습니다.
2. `security_alert/` 폴더를 Splunk 앱 디렉터리(예: `$SPLUNK_HOME/etc/apps/`)에 복사합니다.
3. Splunk를 재시작하거나 분산 환경이라면 `$SPLUNK_HOME/bin/splunk reload deploy-server`로 설정을 다시 로드합니다.
4. Splunk Web → 앱 목록에서 `Security Alert`를 활성화한 뒤 메인 페이지로 진입합니다.
5. Alert Actions(또는 환경에 맞는 conf)에서 Slack Webhook URL을 설정하고, `bin/slack.py`가 이를 사용하도록 구성합니다.
6. `Easy Alert Builder`에서 첫 규칙을 만들어 알림이 Slack으로 정상 수신되는지 확인합니다.

자세한 절차는 [`docs/QUICK-START.md`](docs/QUICK-START.md)와 [`docs/DEPLOYMENT.md`](docs/DEPLOYMENT.md)를 참고하세요.

## 설정(Configuration)

| 구성 항목 | 위치 | 설명 |
|---|---|---|
| 앱 메타데이터 | `security_alert/default/app.conf` | 앱 이름, 버전, UI 라벨 |
| 저장 검색(알림 트리거) | `security_alert/default/savedsearches.conf` | 알림 트리거 SPL 정의 |
| 알림 액션 | `security_alert/default/alert_actions.conf` | 액션 이름과 스크립트 매핑 |
| 매크로 | `security_alert/default/macros.conf` | 공통 SPL 토큰 |
| 필드 변환 | `security_alert/default/transforms.conf`, `security_alert/default/props.conf` | 인덱스/필드 룰 |
| 권한 | `security_alert/metadata/default.meta` | 앱/오브젝트 접근 제어 |
| Slack Webhook | 환경변수 또는 별도 conf | `bin/slack.py`가 참조 |

운영 환경에서는 Webhook URL 같은 비밀값을 절대 conf 본문에 평문으로 두지 말고, 환경변수 또는 Splunk 암호 저장소에 보관하세요. 자세한 절차는 [`docs/DEPLOYMENT.md`](docs/DEPLOYMENT.md)와 [`resume/DEPLOYMENT.md`](resume/DEPLOYMENT.md)에 정리되어 있습니다.

## 명령 참조(Command Reference)

| 명령 | 설명 |
|---|---|
| 앱 디렉터리로 복사 | 예: `cp -r security_alert "$SPLUNK_HOME/etc/apps/"` |
| Splunk 단일 노드 재시작 | `$SPLUNK_HOME/bin/splunk restart` |
| 분산 환경 설정 재로드 | `$SPLUNK_HOME/bin/splunk reload deploy-server` |
| 메시지 포맷 단독 점검 | `python3 security_alert/bin/safe_fmt.py ...` |
| Slack 송신 단독 점검 | `python3 security_alert/bin/slack.py ...` (Webhook URL 환경변수 사용) |

## 로컬 개발(Local Development)

- 코드 변경 위치
  - `security_alert/bin/*.py` — 알림 액션 스크립트
  - `security_alert/default/*.conf` — Splunk 설정
  - `security_alert/data/ui/views/*.xml` — 대시보드·빌더 UI
  - `security_alert/data/ui/nav/default.xml` — 네비게이션
- UI를 수정한 뒤에는 Splunk Web에서 새로 고침하거나 앱을 재시작합니다.
- 신규 Python 의존성은 가능하면 표준 라이브러리를 우선합니다. 외부 의존성이 불가피하면 `security_alert/lib/python3/`에 벤더링합니다(이 저장소는 이미 `urllib3`, `idna`, `charset_normalizer`를 벤더링).
- conf 변경 후에는 잘못된 스키마가 앱 로드 실패로 이어질 수 있으므로 Splunk 로그를 확인합니다.
- 트러블슈팅 절차는 [`resume/TROUBLESHOOTING.md`](resume/TROUBLESHOOTING.md)와 [`resume/ARCHITECTURE.md`](resume/ARCHITECTURE.md)를 참고하세요.

## 테스트(Testing)

- 현재 저장소에는 단위 테스트 프레임워크가 포함되어 있지 않습니다. 향후 `pytest` 도입 시 권장 검증 항목은 다음과 같습니다.
  - `bin/safe_fmt.py`의 포맷 출력 스냅샷
  - `bin/slack.py`의 페이로드 직렬화 검증(목 HTTP 서버로 POST 캡처)
  - `savedsearches.conf` / `alert_actions.conf` 구문 검증 스크립트
- 통합 회귀는 Easy Alert Builder → 저장 검색 → 알림 액션 → Slack 도착의 전체 흐름을 주기적으로 수동 점검합니다.

## 기여(Contributing)

기여 절차는 [`CONTRIBUTING.md`](CONTRIBUTING.md)를 참고하세요. PR 제출 전 다음을 확인합니다.

- 변경 파일을 `$SPLUNK_HOME/etc/apps/security_alert/`에 반영한 뒤 Splunk 재시작 또는 설정 재로드로 동작 검증
- 신규 conf stanza의 스키마 유효성 확인(잘못된 항목은 앱 부팅 실패로 이어짐)
- UI XML의 잘림/인코딩 문제 점검
- 비밀값(Webhook URL 등)을 평문으로 커밋하지 않았는지 재확인

## 유지보수자(Maintainers)

이 저장소는 보안 운영 도구로 Splunk를 사용하는 팀에서 유지보수합니다. 변경 요청이나 장애 접수는 저장소 이슈 트래커 또는 사내 운영 채널을 통해 받습니다. XWiki 기반 문서 게시 절차는 [`docs/ALERT-REPOSITORY-XWIKI.md`](docs/ALERT-REPOSITORY-XWIKI.md)를 참고하세요.

## 추가 문서(Further Documentation)

| 문서 | 용도 |
|---|---|
| [`docs/QUICK-START.md`](docs/QUICK-START.md) | 빠른 시작 절차 |
| [`docs/DEPLOYMENT.md`](docs/DEPLOYMENT.md) | 배포 절차 |
| [`docs/RELEASE-NOTES.md`](docs/RELEASE-NOTES.md) | 릴리스 노트 |
| [`docs/LEGACY-CLEANUP-REPORT.md`](docs/LEGACY-CLEANUP-REPORT.md) | 레거시 정리 이력 |
| [`docs/ALERT-REPOSITORY-XWIKI.md`](docs/ALERT-REPOSITORY-XWIKI.md) | XWiki 문서 게시 절차 |
| [`resume/API.md`](resume/API.md) | 알림 액션 API |
| [`resume/ARCHITECTURE.md`](resume/ARCHITECTURE.md) | 상세 아키텍처 |
| [`resume/DEPLOYMENT.md`](resume/DEPLOYMENT.md) | 배포 심화 |
| [`resume/TROUBLESHOOTING.md`](resume/TROUBLESHOOTING.md) | 트러블슈팅 |
| [`demo/README.md`](demo/README.md) | 데모 시나리오 |
| [`security_alert/README.md`](security_alert/README.md) | Splunk 앱 내부 README |

## 라이선스(License)

이 저장소는 [`LICENSE`](LICENSE) 파일의 조건에 따라 배포됩니다. 사용 전 라이선스 전문을 확인하세요.