# Security Alert (Splunk App)

[![Splunk](https://img.shields.io/badge/Splunk-Add--on-65A637)]()
[![Python 3](https://img.shields.io/badge/Python-3.x-3776AB)]()
[![Status](https://img.shields.io/badge/Status-Production-2EA44F)]()
[![License](https://img.shields.io/badge/License-See%20LICENSE-blue)]()

## 개요 (Overview)

보안 이벤트 알림과 알림 관리를 위한 Splunk 앱입니다. 커스텀 알림 액션,
Slack Webhook 연동, 알림 빌더 UI, 알림 관리 대시보드, 데이터 탐색
대시보드를 제공하여 SOC(Security Operations Center) 팀이 알림을 생성·관리·
대응할 수 있도록 돕습니다.

A Splunk app for security event alerting and notification management.
It offers custom alert actions, Slack Webhook integration, an alert
builder UI, an alert management dashboard, and a data explorer dashboard
so SOC teams can create, manage, and respond to alerts.

## 한눈에 보기 (At a Glance)

| 항목 (Item) | 값 (Value) |
| --- | --- |
| 제품 유형 (Type) | Splunk 앱 / Add-on |
| 대상 플랫폼 (Platform) | Splunk Enterprise, Splunk Cloud |
| 런타임 (Runtime) | Splunk App Framework, Python 3 |
| 배포 경로 (Install Path) | `$SPLUNK_HOME/etc/apps/security_alert/` |
| 주요 알림 채널 (Channel) | Slack Webhook |
| UI 모듈 (UI Modules) | 알림 빌더, 알림 관리, 데이터 탐색 |
| 상태 (Status) | 프로덕션 준비 (Production-ready) |
| 라이선스 (License) | [LICENSE](LICENSE) 참조 |

## 처리 흐름 (Processing Flow)

1. `savedsearches.conf`에 정의된 보안 검색이 이벤트를 탐지합니다.
2. `alert_actions.conf`의 커스텀 알림 액션이 트리거됩니다.
3. `bin/slack.py`가 번들된 `urllib3`로 Slack Webhook을 호출합니다.
4. `bin/safe_fmt.py`가 메시지를 안전하게 포맷합니다.
5. UI 대시보드에서 알림 이력과 상태를 확인합니다.
6. `transforms.conf` 및 `macros.conf`로 후속 정제·집계가 수행됩니다.

## 목차 (Table of Contents)

- [기능 (Features)](#features)
- [저장소 구조 (Repository Layout)](#repository-layout)
- [구성 요소 (Package Contents)](#package-contents)
- [시작하기 (Quickstart)](#quickstart)
- [명령어 참조 (Commands Reference)](#commands-reference)
- [로컬 개발 (Local Development)](#local-development)
- [테스트 (Testing)](#testing)
- [문제 해결 (Troubleshooting)](#troubleshooting)
- [유지보수 (Maintainers)](#maintainers)
- [추가 문서 (Further Documentation)](#further-documentation)
- [기여 가이드 (Contributing)](#contributing)
- [라이선스 (License)](#license)

## 기능 (Features)

- 커스텀 알림 액션: `default/alert_actions.conf`
- Slack Webhook 통합: `bin/slack.py`
- 안전한 메시지 포맷팅: `bin/safe_fmt.py`
- 알림 빌더 UI: `default/data/ui/views/alert-builder.xml`,
  `easy_alert_builder.xml`
- 알림 관리 대시보드:
  `default/data/ui/views/alert-management-dashboard.xml`
- 데이터 탐색 대시보드:
  `default/data/ui/views/data-explorer-dashboard.xml`
- 검색 매크로: `default/macros.conf`
- 필드 추출 및 변환: `default/props.conf`, `default/transforms.conf`
- 번들 Python 의존성: `urllib3`, `idna`, `charset_normalizer`

## 저장소 구조 (Repository Layout)

| 경로 (Path) | 설명 (Description) |
| --- | --- |
| `security_alert/` | Splunk 앱 본체 |
| `security_alert/bin/` | 알림 스크립트 (Slack, 포맷) |
| `security_alert/default/` | 기본 `.conf` 설정 모음 |
| `security_alert/default/data/ui/` | 내비게이션 및 XML 뷰 |
| `security_alert/lib/python3/` | 번들 Python 라이브러리 |
| `security_alert/metadata/` | 앱 메타데이터 |
| `resume/` | 운영·아키텍처 문서 묶음 |
| `docs/` | 배포, 릴리스, 레거시 보고서 |
| `demo/` | 데모 자료 |

## 구성 요소 (Package Contents)

| 파일 (File) | 역할 (Role) |
| --- | --- |
| `app.conf` | Splunk 앱 정의 및 권한 |
| `app.manifest` | 앱 매니페스트 정보 |
| `alert_actions.conf` | 커스텀 알림 액션 등록 |
| `savedsearches.conf` | 예약된 보안 검색 |
| `props.conf` | 필드 추출 규칙 |
| `transforms.conf` | 데이터 변환 규칙 |
| `macros.conf` | 재사용 가능한 검색 매크로 |
| `bin/slack.py` | Slack Webhook 전송기 |
| `bin/safe_fmt.py` | 메시지 안전 포맷터 |
| `bin/six.py` | Python 2/3 호환 유틸 |

## 시작하기 (Quickstart)

### 사전 요구 사항 (Prerequisites)

- Splunk Enterprise 8.x 이상 또는 Splunk Cloud
- 앱 디렉토리에 대한 쓰기 권한
- Slack 알림 사용 시 Slack Incoming Webhook URL

### 설치 (Install)

1. 저장소를 작업 디렉토리로 클론합니다.

   ```bash
   git clone <repository-url>
   cd <repository-directory>
   ```

2. `security_alert/` 디렉토리를 Splunk 앱 경로로 복사합니다.

   ```bash
   cp -r security_alert/ "$SPLUNK_HOME/etc/apps/"
   ```

3. Splunk를 재시작합니다.

   ```bash
   "$SPLUNK_HOME/bin/splunk" restart
   ```

4. Splunk Web에서 앱을 활성화합니다.

### 최소 설정 (Minimal Configuration)

- `default/app.conf`의 `id`, `version`, `[ui]` 항목을 검토합니다.
- `default/alert_actions.conf`의 알림 파라미터를 확인합니다.
- Slack 사용 시 `bin/slack.py`의 webhook 설정 또는 환경 변수를
  지정합니다.

## 명령어 참조 (Commands Reference)

| 작업 (Task) | 명령어 (Command) |
| --- | --- |
| Splunk 재시작 | `"$SPLUNK_HOME/bin/splunk" restart` |
| 앱 메타 표시 | `"$SPLUNK_HOME/bin/splunk" display app security_alert` |
| btool 검증 | `"$SPLUNK_HOME/bin/splunk" btool check --app=security_alert` |
| Slack 스크립트 점검 | `"$SPLUNK_HOME/bin/splunk" cmd python3 security_alert/bin/slack.py --check` |
| 로그 확인 | `tail -f "$SPLUNK_HOME/var/log/splunk/splunkd.log"` |

## 로컬 개발 (Local Development)

- 작업 위치: `$SPLUNK_HOME/etc/apps/security_alert/`
- `.conf` 변경 후 Splunk 재시작 또는 캐시 새로 고침이 필요할 수 있습니다.
- Python 스크립트는 Splunk 번들 Python 3 인터프리터로 실행됩니다.
- 새로운 외부 의존성은 `lib/python3/`에 번들링합니다.
- UI XML은 Splunk Web에서 즉시 반영되지 않을 수 있으므로 캐시를 확인합니다.

## 테스트 (Testing)

- 저장된 검색을 수동으로 실행해 알림 트리거를 검증합니다.
- `bin/slack.py` 호출을 단위 테스트로 분리해 메시지 포맷을 확인합니다.
- UI 대시보드는 Splunk Web에서 렌더링과 권한 노출을 확인합니다.
- 새 의존성 추가 시 Splunk 번들 환경에서 import 테스트를 수행합니다.

## 문제 해결 (Troubleshooting)

| 증상 (Symptom) | 확인 항목 (Check) |
| --- | --- |
| 앱이 목록에 없음 | 디렉토리 권한, `app.conf` 문법 |
| Slack 전송 실패 | Webhook URL, 네트워크, `slack.py` 로그 |
| 알림 미트리거 | `savedsearches.conf` 스케줄, 권한 |
| UI 페이지 오류 | `data/ui/nav/default.xml` 참조 무결성 |
| 의존성 누락 | `lib/python3/` 번들 상태, Python 버전 |

상세 절차는 [resume/TROUBLESHOOTING.md](resume/TROUBLESHOOTING.md)와
[docs/DEPLOYMENT.md](docs/DEPLOYMENT.md)를 참조하세요.

## 유지보수 (Maintainers)

- 보안 운영팀 (Security Operations Team)
- 내부 위키: [docs/ALERT-REPOSITORY-XWIKI.md](docs/ALERT-REPOSITORY-XWIKI.md)
- 변경 요청 및 책임 범위는 위키 페이지를 따릅니다.

## 추가 문서 (Further Documentation)

| 문서 (Document) | 경로 (Path) |
| --- | --- |
| API 참조 | [resume/API.md](resume/API.md) |
| 아키텍처 | [resume/ARCHITECTURE.md](resume/ARCHITECTURE.md) |
| 배포 가이드 | [resume/DEPLOYMENT.md](resume/DEPLOYMENT.md) |
| 문제 해결 | [resume/TROUBLESHOOTING.md](resume/TROUBLESHOOTING.md) |
| 빠른 시작 | [docs/QUICK-START.md](docs/QUICK-START.md) |
| 배포 (docs) | [docs/DEPLOYMENT.md](docs/DEPLOYMENT.md) |
| 릴리스 노트 | [docs/RELEASE-NOTES.md](docs/RELEASE-NOTES.md) |
| 레거시 정리 보고서 | [docs/LEGACY-CLEANUP-REPORT.md](docs/LEGACY-CLEANUP-REPORT.md) |
| 저장소 알림 위키 | [docs/ALERT-REPOSITORY-XWIKI.md](docs/ALERT-REPOSITORY-XWIKI.md) |
| 데모 | [demo/README.md](demo/README.md) |

## 기여 가이드 (Contributing)

기여 절차는 [CONTRIBUTING.md](CONTRIBUTING.md)를 따릅니다. 보안과 관련된
변경은 보안팀의 사전 검토가 필요하며, UI XML과 `.conf` 변경은 배포 전
스테이징 환경에서 검증해야 합니다.

## 라이선스 (License)

본 저장소의 라이선스 조건은 [LICENSE](LICENSE) 파일을 참조하세요.