# Security Alert Manager (Splunk App) / 보안 알림 관리 (Splunk 앱)

> Splunk에서 보안 이벤트를 수집·정규화하고, 알림(alert)을 정의한 뒤 Slack으로 전달하는 알림 관리 앱입니다. 본 저장소는 앱 본체(`security_alert/`)와 운영 문서(`docs/`, `resume/`)를 함께 제공합니다.

A Splunk add-on that defines, manages, and dispatches security alerts (with Slack delivery) from a single searchable workspace.

---

## 빠른 상태 요약 / Quick Status

| 항목 | 값 |
| --- | --- |
| 제품 종류 | Splunk App (`.conf` 기반 + 번들 Python 3) |
| 대상 플랫폼 | Splunk Enterprise / Splunk Cloud (8.x 이상 권장) |
| 핵심 디렉터리 | `security_alert/` (앱 본체) |
| 알림 채널 | Slack (Webhook) |
| 빌드 진입점 | `security_alert/app.manifest` |
| 문서 진입점 | `docs/QUICK-START.md`, `resume/ARCHITECTURE.md` |
| 라이선스 | `LICENSE` |
| 기여 가이드 | `CONTRIBUTING.md` |
| 데모 | `demo/README.md` |
| 운영 상태 | 프로덕션 사용 가능 (단, 업그레이드 시 `docs/RELEASE-NOTES.md` 확인) |

## 한눈에 보는 흐름 / Flow at a Glance

1. Splunk 인덱서가 원본 로그를 색인합니다.
2. `security_alert/default/savedsearches.conf`가 트리거 조건을 정의합니다.
3. `security_alert/default/alert_actions.conf`가 매칭 시 동작을 지정합니다.
4. `security_alert/bin/slack.py`가 메시지를 포맷팅(`safe_fmt.py`) 후 Slack Webhook으로 전송합니다.
5. `security_alert/default/data/ui/views/` 대시보드에서 알림을 조회·관리합니다.

> 운영 단계, 트러블슈팅, 레거시 정리 내역은 각 문서 파일을 그대로 따라가세요.

---

## 목차 / Table of Contents

1. [용도와 사용자 / Purpose and Audience](#용도와-사용자--purpose-and-audience)
2. [주요 기능 / Features](#주요-기능--features)
3. [저장소 구성 / Package Contents](#저장소-구성--package-contents)
4. [아키텍처 / Architecture](#아키텍처--architecture)
5. [빠른 시작 / Quickstart](#빠른-시작--quickstart)
6. [설정 / Configuration](#설정--configuration)
7. [명령·엔트리포인트 / Commands and Entry Points](#명령·엔트리포인트--commands-and-entry-points)
8. [로컬 개발 / Local Development](#로컬-개발--local-development)
9. [테스트 / Testing](#테스트--testing)
10. [기여 / Contributing](#기여--contributing)
11. [유지보수자 / Maintainers](#유지보수자--maintainers)
12. [추가 문서 / Further Documentation](#추가-문서--further-documentation)
13. [라이선스 / License](#라이선스--license)

---

## 용도와 사용자 / Purpose and Audience

- **용도 / Purpose**: Splunk의 보안 이벤트에서 알림 규칙을 정의·실행하고, 그 결과를 Slack으로 전달하며, 대시보드에서 후속 분석을 수행하는 앱입니다.
- **대상 사용자 / Audience**:
  - SOC 분석가, 보안 엔지니어
  - Splunk 관리자(앱 설치·업그레이드 담당)
  - 알림 파이프라인을 코드 형태로 검토하려는 플랫폼 팀

## 주요 기능 / Features

| 기능 | 설명 |
| --- | --- |
| 알림 빌더 | `alert-builder`, `easy_alert_builder` 뷰로 알림 규칙 생성 |
| 알림 관리 | `alert-management-dashboard`에서 활성 알림 조회·해제 |
| 데이터 탐색 | `data-explorer-dashboard`에서 원본 이벤트 검색 |
| Slack 전달 | `bin/slack.py`가 Webhook으로 메시지 발송 |
| 안전한 포맷팅 | `bin/safe_fmt.py`로 메시지 이스케이프 처리 |
| 필드 추출 | `default/props.conf`, `default/transforms.conf`로 정규화 |
| 매크로 | `default/macros.conf`로 재사용 검색 토큰 제공 |
| 탐색 UI | `data/ui/nav/default.xml`로 내비게이션 정의 |

## 저장소 구성 / Package Contents

실제 최상위 구조를 반영한 요약입니다.

| 경로 | 역할 |
| --- | --- |
| `security_alert/app.manifest` | Splunk 앱 메타데이터 |
| `security_alert/default/app.conf` | 앱 설정(라벨, 권한) |
| `security_alert/default/alert_actions.conf` | 알림 동작 정의 |
| `security_alert/default/savedsearches.conf` | 저장된 검색(트리거) |
| `security_alert/default/props.conf` | 필드/시간 추출 규칙 |
| `security_alert/default/transforms.conf` | 필드 변환 규칙 |
| `security_alert/default/macros.conf` | 재사용 매크로 |
| `security_alert/default/data/ui/views/*.xml` | 대시보드/뷰 |
| `security_alert/default/data/ui/nav/default.xml` | 내비게이션 |
| `security_alert/bin/slack.py` | Slack Webhook 발송 스크립트 |
| `security_alert/bin/safe_fmt.py` | 메시지 포맷 유틸리티 |
| `security_alert/bin/six.py` | Python 2/3 호환 헬퍼 |
| `security_alert/metadata/default.meta` | 권한/소유자 메타 |
| `security_alert/lib/python3/urllib3/` | 번들 HTTP 클라이언트 |
| `security_alert/lib/python3/charset_normalizer-*/` | 번들 인코딩 감지 |
| `security_alert/lib/python3/idna-*/` | 번들 IDN 변환 |
| `docs/` | 운영·릴리스 문서 |
| `resume/` | 이관·아키텍처·API 문서 |
| `demo/` | 데모/시연 자료 |
| `CONTRIBUTING.md` | 기여 절차 |
| `LICENSE` | 라이선스 원문 |

## 아키텍처 / Architecture

| 계층 | 구성요소 | 설명 |
| --- | --- | --- |
| 입력 | Splunk 인덱서 | 원본 보안 로그 색인 |
| 정규화 | `props.conf`, `transforms.conf` | 필드 추출·치환 |
| 탐지 | `savedsearches.conf` | 조건 평가 및 트리거 |
| 동작 | `alert_actions.conf`, `bin/slack.py` | Slack Webhook 호출 |
| 포맷 | `bin/safe_fmt.py` | 메시지 안전 이스케이프 |
| UI | `data/ui/views/*.xml` | 빌더/관리/탐색 화면 |
| 메타 | `app.manifest`, `metadata/default.meta` | 앱 식별·권한 |

### 요청 흐름 (논리 단계)

1. 인덱서가 이벤트를 색인합니다.
2. `savedsearches.conf`의 스케줄에 따라 검색이 실행됩니다.
3. 매칭 결과는 `alert_actions.conf`에 정의된 동작으로 라우팅됩니다.
4. `bin/slack.py`가 `safe_fmt.py`로 메시지를 정리한 뒤 HTTP POST로 Slack Webhook을 호출합니다.
5. `metadata/default.meta`의 권한 정책에 따라 UI에서 알림이 노출됩니다.

상세 다이어그램과 호출 관계는 [`resume/ARCHITECTURE.md`](resume/ARCHITECTURE.md)에서 확인하세요.

## 빠른 시작 / Quickstart

### 1. 사전 요구 사항

| 항목 | 요구 |
| --- | --- |
| Splunk | 8.x 이상 |
| 디스크 | 앱 디렉터리당 충분한 여유 공간(번들 의존성 포함) |
| 네트워크 | Splunk → Slack Webhook URL 도달 가능 |
| 권한 | 앱 배포 권한, `metadata/default.meta`에 명시된 capability |

### 2. 설치

1. `security_alert/` 디렉터리를 `$SPLUNK_HOME/etc/apps/`에 복사합니다.
2. Splunk를 재시작하거나 “Manage Apps”에서 새로 고침합니다.
3. Slack Webhook URL을 앱 설정에 입력합니다(자세한 키는 `docs/QUICK-START.md` 참조).
4. `app.conf`의 권한/라벨을 환경에 맞게 조정합니다.

### 3. 첫 알림 발송

1. Splunk Web에서 앱의 **Easy Alert Builder**로 이동합니다.
2. 샘플 인덱스/소스를 선택해 규칙을 저장합니다.
3. 저장된 검색이 트리거되면 Slack 채널에 메시지가 도달하는지 확인합니다.

단계별 스크린샷과 토글 옵션은 [`docs/QUICK-START.md`](docs/QUICK-START.md)에 정리되어 있습니다.

## 설정 / Configuration

| 키 | 위치 | 용도 |
| --- | --- | --- |
| Slack Webhook URL | `default/app.conf` 또는 환경 변수 | 메시지 전송 대상 |
| 알림 라벨 | `default/alert_actions.conf` | 동작 이름/표시 |
| 트리거 조건 | `default/savedsearches.conf` | 크론/쿼리 |
| 필드 추출 | `default/props.conf` | 시간·필드 |
| 변환 | `default/transforms.conf` | 룩업/치환 |
| 매크로 | `default/macros.conf` | 재사용 토큰 |
| 권한 | `metadata/default.meta` | 역할/소유자 |
| 앱 메타 | `app.manifest` | 버전, 작성자 |

## 명령·엔트리포인트 / Commands and Entry Points

| 진입점 | 용도 |
| --- | --- |
| `security_alert/bin/slack.py` | 알림을 Slack으로 발송 |
| `security_alert/bin/safe_fmt.py` | 메시지 포맷 모듈 |
| `security_alert/bin/six.py` | 호환성 헬퍼 |
| `data/ui/views/alert-builder.xml` | 알림 규칙 빌더 UI |
| `data/ui/views/easy_alert_builder.xml` | 간편 빌더 UI |
| `data/ui/views/alert-management-dashboard.xml` | 알림 관리 대시보드 |
| `data/ui/views/data-explorer-dashboard.xml` | 데이터 탐색 대시보드 |
| `data/ui/nav/default.xml` | 앱 내비게이션 |

REST API와 외부 호출 명세는 [`resume/API.md`](resume/API.md)를 참고하세요.

## 로컬 개발 / Local Development

| 작업 | 절차 |
| --- | --- |
| 코드 수정 | `$SPLUNK_HOME/etc/apps/security_alert/`에서 직접 편집 |
| 앱 재로드 | Splunk Web → “Manage Apps” → Reload, 또는 재시작 |
| 의존성 변경 | `security_alert/lib/python3/` 내 번들 교체 |
| UI 변경 | `data/ui/views/*.xml` 편집 후 페이지 새로 고침 |
| 설정 검증 | `bin/splunk btool check`로 `.conf` 무결성 확인 |
| 문서 갱신 | `docs/`, `resume/` 하위 Markdown 수정 |

개발 환경 구성은 [`docs/DEPLOYMENT.md`](docs/DEPLOYMENT.md)를 참고하세요.

## 테스트 / Testing

| 항목 | 권장 방법 |
| --- | --- |
| 알림 발송 | 샘플 검색을 수동 실행해 Slack 수신 확인 |
| `.conf` 검증 | `btool check`로 문법 검사 |
| UI | 대시보드·빌더에서 권한별로 페이지 접근 확인 |
| 라이브러리 | `bin/slack.py` 단위 호출(드라이 런 모드) |
| 회귀 | `docs/RELEASE-NOTES.md`의 변경 이력과 대조 |

테스트 절차와 체크리스트는 [`resume/ARCHITECTURE.md`](resume/ARCHITECTURE.md)와 [`docs/QUICK-START.md`](docs/QUICK-START.md)를 함께 확인하세요.

## 기여 / Contributing

1. `CONTRIBUTING.md`의 절차와 코딩 규약을 따릅니다.
2. 변경 사항은 가능한 한 `docs/RELEASE-NOTES.md`에 한 줄로 요약합니다.
3. 새로운 알림 동작을 추가할 때 `alert_actions.conf`, `bin/`, UI 뷰를 한 묶음으로 갱신합니다.
4. PR 전 로컬 Splunk 인스턴스에서 스모크 테스트를 수행합니다.

## 유지보수자 / Maintainers

| 역할 | 위치 |
| --- | --- |
| 코드 오너 | `security_alert/app.manifest`의 `author` 필드 |
| 문서 오너 | `docs/`, `resume/` 하위 문서 |
| 이슈 트래커 | 저장소 Issues 탭 |

자세한 책임 범위는 [`CONTRIBUTING.md`](CONTRIBUTING.md)와 `docs/RELEASE-NOTES.md`의 변경 이력을 참고하세요.

## 추가 문서 / Further Documentation

| 문서 | 경로 |
| --- | --- |
| 빠른 시작 | [`docs/QUICK-START.md`](docs/QUICK-START.md) |
| 배포 가이드 | [`docs/DEPLOYMENT.md`](docs/DEPLOYMENT.md) |
| 레거시 정리 보고서 | [`docs/LEGACY-CLEANUP-REPORT.md`](docs/LEGACY-CLEANUP-REPORT.md) |
| 릴리스 노트 | [`docs/RELEASE-NOTES.md`](docs/RELEASE-NOTES.md) |
| 알림 저장소 XWiki | [`docs/ALERT-REPOSITORY-XWIKI.md`](docs/ALERT-REPOSITORY-XWIKI.md) |
| 아키텍처 | [`resume/ARCHITECTURE.md`](resume/ARCHITECTURE.md) |
| API 명세 | [`resume/API.md`](resume/API.md) |
| 배포(이관) | [`resume/DEPLOYMENT.md`](resume/DEPLOYMENT.md) |
| 트러블슈팅 | [`resume/TROUBLESHOOTING.md`](resume/TROUBLESHOOTING.md) |
| 데모 | [`demo/README.md`](demo/README.md) |

## 라이선스 / License

본 저장소는 [`LICENSE`](LICENSE) 파일에 명시된 라이선스를 따릅니다. 외부에 번들된 `urllib3`, `charset_normalizer`, `idna` 등 서드파티 라이브러리는 각 `dist-info/` 하위의 라이선스 표기를 그대로 유지합니다.