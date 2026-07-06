# Security Alert (Splunk App)

![Splunk App](https://img.shields.io/badge/Splunk-App-blueviolet) ![Python 3](https://img.shields.io/badge/Python-3.x-3776AB) [![License](https://img.shields.io/badge/license-internal-lightgrey)](LICENSE)

## 한국어 요약

`security_alert/` 디렉터리에 있는 Splunk 보안 알림 앱입니다. 검색 기반 알림 작성, 알림 관리 대시보드, 데이터 탐색기, 손쉬운 알림 빌더를 제공하며, `bin/slack.py`로 Slack 알림 전송, `bin/safe_fmt.py`로 메시지 포맷 안전화, `bin/six.py`로 Python 2/3 호환을 지원합니다. 운영·설계 문서는 `resume/`, 프로젝트 차원의 배포/릴리스 노트/레거시 정리 보고는 `docs/`, 데모는 `demo/`에 있습니다.

핵심 진입점은 `security_alert/default/app.conf`(앱 메타데이터), `security_alert/default/alert_actions.conf`(알림 액션), `security_alert/default/savedsearches.conf`(저장된 검색), `security_alert/bin/slack.py`(외부 통지) 입니다.

## English Summary

This repository hosts a Splunk add-on, `security_alert/`, that provides security alerting capabilities: an alert builder, an alert management dashboard, a data explorer, and an "easy" alert builder view. The Python helper scripts handle Slack notifications (`bin/slack.py`), safe message formatting (`bin/safe_fmt.py`), and Python 2/3 compatibility (`bin/six.py`). Operational and architectural documentation lives under `resume/`. Project-level deployment notes, release notes, and the legacy cleanup report are in `docs/`. A small demo is available in `demo/`.

The primary entry points are the Splunk conf files under `security_alert/default/` and the bundled Python libraries under `security_alert/lib/python3/`.

## 상태(Status) — 한눈에 보기 / At a glance

| 영역 / Area | 값 / Value |
| --- | --- |
| 제품 / Product | Splunk 보안 알림 앱 / Splunk security alerting app |
| 앱 경로 / App path | `security_alert/` |
| 앱 매니페스트 / App manifest | `security_alert/app.manifest` |
| 대시보드 / Dashboards | `alert-builder`, `alert-management-dashboard`, `data-explorer-dashboard`, `easy_alert_builder` |
| 알림 액션 / Alert actions | `security_alert/default/alert_actions.conf` |
| 저장된 검색 / Saved searches | `security_alert/default/savedsearches.conf` |
| 통합 / Integrations | Slack (`bin/slack.py`) |
| 문서 / Documentation | `resume/`, `docs/`, `demo/` |
| 라이선스 / License | 저장소 `LICENSE` 참조 / see `LICENSE` |

## 빠른 흐름 요약 / Quick Flow

1. `security_alert/default/savedsearches.conf`로 탐지 검색을 정의합니다.
2. `security_alert/default/alert_actions.conf`로 알림 액션과 Slack 매개변수를 연결합니다.
3. `security_alert/bin/slack.py`가 알림을 포맷팅(`bin/safe_fmt.py`) 후 Slack으로 전송합니다.
4. 운영자는 `data/ui/views/alert-management-dashboard.xml`로 알림을 모니터링합니다.
5. 새 알림은 `data/ui/views/easy_alert_builder.xml` 또는 `alert-builder.xml`로 작성합니다.

## 목차 / Table of Contents

- [저장소 구성 / Repository Contents](#저장소-구성--repository-contents)
- [먼저 읽을 파일 / First Files to Read](#먼저-읽을-파일--first-files-to-read)
- [진입점과 API / Entry Points and API](#진입점과-api--entry-points-and-api)
- [빠른 시작 / Quickstart](#빠른-시작--quickstart)
- [설정 / Configuration](#설정--configuration)
- [명령 참조 / Commands Reference](#명령-참조--commands-reference)
- [로컬 개발 / Local Development](#로컬-개발--local-development)
- [테스트 / Testing](#테스트--testing)
- [기여 / Contributing](#기여--contributing)
- [유지보수자 / Maintainers](#유지보수자--maintainers)
- [추가 문서 / Further Documentation](#추가-문서--further-documentation)
- [라이선스 / License](#라이선스--license)

## 저장소 구성 / Repository Contents

| 경로 / Path | 용도 / Purpose |
| --- | --- |
| `LICENSE` | 저장소 라이선스 / Repository license |
| `CONTRIBUTING.md` | 기여 가이드 / Contribution guide |
| `README.md` | 본 문서 / This file |
| `resume/` | 운영·아키텍처·API 문서 / Operational & architecture docs |
| `docs/` | 배포·릴리스·레거시 정리 보고서 / Deployment, release, legacy cleanup |
| `demo/` | 데모 자료 / Demo assets |
| `security_alert/` | Splunk 앱 본체 / Splunk app payload |
| `security_alert/app.manifest` | Splunk 앱 매니페스트 / App manifest |
| `security_alert/bin/` | 알림 스크립트 / Alert scripts (Python) |
| `security_alert/default/` | Splunk 설정 파일 / Splunk conf files |
| `security_alert/metadata/` | 앱 메타데이터 / App metadata |
| `security_alert/data/ui/nav/` | 네비게이션 / Navigation XML |
| `security_alert/data/ui/views/` | 대시보드 / Dashboards |
| `security_alert/lib/python3/` | 번들 Python 라이브러리 / Bundled Python libs |

## 먼저 읽을 파일 / First Files to Read

운영자/개발자 모두 다음 순서로 읽는 것을 권장합니다.

1. `security_alert/app.manifest` — 앱 이름, 버전, 작성자.
2. `security_alert/default/app.conf` — 앱 ID, 라벨, UI 가시성.
3. `security_alert/default/alert_actions.conf` — 알림 액션 정의.
4. `security_alert/default/savedsearches.conf` — 저장된 검색과 스케줄.
5. `security_alert/bin/slack.py` — 외부 통지 경로.
6. `resume/ARCHITECTURE.md` — 전체 아키텍처 (있을 경우).
7. `docs/QUICK-START.md` — 빠른 시작 절차.
8. `docs/DEPLOYMENT.md` — 배포 절차.

## 진입점과 API / Entry Points and API

Splunk 앱 구조상 진입점은 파일 단위입니다.

| 진입점 / Entry Point | 역할 / Role |
| --- | --- |
| `security_alert/default/alert_actions.conf` | 알림 액션 등록 / Alert action registration |
| `security_alert/default/savedsearches.conf` | 탐지 검색 정의 / Detection search definitions |
| `security_alert/default/macros.conf` | 재사용 SPL 매크로 / Reusable SPL macros |
| `security_alert/default/transforms.conf` | 필드 추출/재작성 / Field transforms |
| `security_alert/default/props.conf` | 인덱싱/파싱 규칙 / Indexing & parsing rules |
| `security_alert/bin/slack.py` | Slack 알림 스크립트 / Slack notifier |
| `security_alert/bin/safe_fmt.py` | 메시지 포맷 안전화 / Safe formatter |
| `security_alert/bin/six.py` | Py2/Py3 호환 / Compatibility shim |
| `security_alert/data/ui/nav/default.xml` | 네비게이션 / App navigation |
| `security_alert/data/ui/views/*.xml` | 대시보드 4종 / Four dashboard views |

외부 표면은 Splunk의 표준 알림 액션 인터페이스와, `bin/slack.py`가 노출하는 명령행 인자입니다. 자세한 스키마는 `resume/API.md`를 참고하세요.

## 빠른 시작 / Quickstart

1. 저장소를 클론합니다.
2. `security_alert/` 디렉터리를 Splunk 앱 디렉터리(예: `$SPLUNK_HOME/etc/apps/`)에 복사합니다.
3. Splunk를 재시작하거나 배포 서버를 통해 앱을 배포합니다.
4. Splunk Web에서 **Security Alert** 앱을 선택합니다.
5. **Easy Alert Builder** 또는 **Alert Builder** 뷰에서 새 알림을 만듭니다.
6. `alert_actions.conf`에 Slack 웹훅 URL 등 자격 증명을 설정합니다.
7. **Alert Management Dashboard**에서 동작을 확인합니다.

자세한 단계는 `docs/QUICK-START.md` 및 `docs/DEPLOYMENT.md`를 참고하세요.

## 설정 / Configuration

| 파일 / File | 설정 항목 / Settings | 비고 / Notes |
| --- | --- | --- |
| `app.conf` | `id`, `label`, `version` | 앱 식별자 |
| `app.manifest` | `schemaVersion`, `info.id` | Splunk 8+ 매니페스트 |
| `alert_actions.conf` | `label`, `param.*`, `command` | 알림 액션 매개변수 |
| `savedsearches.conf` | `search`, `schedule`, `action.*` | 검색/스케줄/연결 액션 |
| `macros.conf` | `definition` | SPL 매크로 |
| `props.conf` | `EXTRACT-*`, `TRANSFORMS-*` | 필드 추출/재작성 |
| `transforms.conf` | `REGEX`, `FORMAT` | 변환 규칙 |
| `metadata/default.meta` | 권한/소유자 | 뷰·매크로 접근 제어 |

민감 값(웹훅 URL, 토큰)은 Splunk 자격 증명 관리 또는 암호화된 앱 디렉터리에 보관하세요.

## 명령 참조 / Commands Reference

| 명령 / Command | 위치 / Location | 용도 / Purpose |
| --- | --- | --- |
| `slack.py` | `security_alert/bin/` | Slack 알림 전송 |
| `safe_fmt.py` | `security_alert/bin/` | 메시지 포맷 안전화 |
| `six.py` | `security_alert/bin/` | Python 2/3 호환 유틸 |

각 스크립트의 인자/옵션은 `resume/API.md` 또는 스크립트 상단 docstring을 참고하세요.

## 로컬 개발 / Local Development

- Splunk 인스턴스(또는 컨테이너 이미지)에서 `$SPLUNK_HOME/etc/apps/security_alert`로 앱을 마운트합니다.
- `bin/` 스크립트는 `lib/python3/`의 번들 의존성(예: `urllib3`)을 사용합니다. 시스템 패키지를 추가할 때는 동일한 경로에 두세요.
- UI 변경은 `data/ui/views/*.xml`과 `data/ui/nav/default.xml`을 수정합니다.
- 개발 후 Splunk 재시작 또는 `_bump`로 캐시 무효화가 필요할 수 있습니다.

## 테스트 / Testing

- 저장소에는 명시적 테스트 디렉터리가 포함되어 있지 않습니다. `bin/` 스크립트에 단위 테스트를 추가할 경우, Splunk 외부에서 실행 가능하도록 의존성을 격리하세요.
- 대시보드 XML은 Splunk 인스턴스에서 수동 검증합니다.
- 알림 액션은 `savedsearches.conf`에 임시 검색을 만들어 종단간으로 확인합니다.

## 기여 / 기여 / Contributing

기여 절차는 `CONTRIBUTING.md`를 따릅니다. 큰 변경 전에는 이슈 또는 PR로 먼저 협의해 주세요.

## 유지보수자 / Maintainers

본 저장소의 유지보수 책임자는 `security_alert/app.manifest` 및 `security_alert/default/app.conf`의 작성자 필드를 참고하세요. 사내 연락처는 사정 상 본 README에 기재하지 않습니다.

## 추가 문서 / Further Documentation

- 운영/아키텍처: `resume/ARCHITECTURE.md`, `resume/API.md`, `resume/DEPLOYMENT.md`, `resume/TROUBLESHOOTING.md`
- 프로젝트 문서: `docs/QUICK-START.md`, `docs/DEPLOYMENT.md`, `docs/RELEASE-NOTES.md`, `docs/LEGACY-CLEANUP-REPORT.md`, `docs/ALERT-REPOSITORY-XWIKI.md`
- 데모: `demo/README.md`

## 라이선스 / License

저장소 루트의 `LICENSE` 파일을 참조하세요. 내부 사용 조건이 명시된 경우, 외부 배포 시 별도 허가가 필요할 수 있습니다.