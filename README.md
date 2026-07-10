# Security Alert — Splunk Alerting App

![App type: Splunk Add-on](https://img.shields.io/badge/Splunk-Add--on-blue)
![Python: 3.x](https://img.shields.io/badge/Python-3.x-3776AB)
![License: see LICENSE](https://img.shields.io/badge/License-See%20LICENSE-lightgrey)
![Status: Production-ready](https://img.shields.io/badge/Status-Production--ready-brightgreen)

## 한 줄 요약

Splunk 환경에서 XWiki / Slack 으로 보안 알림을 보내고, 알림 빌더 · 관리 대시보드 · 데이터 탐색기를 한 앱에서 제공하는 Splunk 추가 기능(Add-on)입니다.

English (secondary): A Splunk add-on that ships security alerts to XWiki and Slack, and bundles an alert builder, an alert management dashboard, and a data explorer inside a single app.

---

## 현재 상태 / Status

| 항목 | 값 |
| --- | --- |
| 제품 유형 | Splunk Add-on (custom app) |
| 대상 환경 | Splunk Enterprise / Cloud (Python 3 런타임) |
| 알림 채널 | Slack (webhook), 확장 가능한 HTTP 통합 |
| 기본 제공 뷰 | alert-builder, alert-management-dashboard, data-explorer-dashboard, easy_alert_builder |
| 번들 라이브러리 | urllib3, idna, charset_normalizer (오프라인 호환) |
| 라이선스 | 저장소 `LICENSE` 참조 |
| 테스트 | 수동 검증 (자동화 테스트 미포함) |
| 안정성 | 프로덕션 사용 가능, 변경 시 회귀 검증 권장 |

## 동작 흐름 / How It Works

1. Splunk 가 저장된 검색(`savedsearches.conf`)을 트리거합니다.
2. 알림 액션(`alert_actions.conf`)이 활성화되어 `bin/safe_fmt.py` 와 `bin/slack.py` 를 호출합니다.
3. 페이로드는 `macros.conf` 와 `transforms.conf` 로 정규화됩니다.
4. HTTP 전송은 번들된 `urllib3` + `idna` + `charset_normalizer` 로 처리됩니다.
5. 결과는 앱 내 대시보드(alert-management-dashboard 등)에서 추적됩니다.

> 운영자는 다음 한 줄로 상태를 확인합니다: Splunk UI → 앱 "Security Alert" → Alert Management Dashboard.

---

## 패키지 구성 / Package Contents

| 경로 | 역할 |
| --- | --- |
| `security_alert/app.conf` | 앱 메타데이터(라벨, 버전, 소유자) |
| `security_alert/app.manifest` | Splunk 호환성 / 의존성 선언 |
| `security_alert/default/alert_actions.conf` | 알림 액션 정의 |
| `security_alert/default/savedsearches.conf` | 저장된 검색(트리거 원천) |
| `security_alert/default/macros.conf` | 검색 매크로(공통 정규식·필터) |
| `security_alert/default/transforms.conf` | 필드 추출 및 재작성 규칙 |
| `security_alert/default/props.conf` | 인덱서 / 검색 시간 필드 설정 |
| `security_alert/default/data/ui/nav/default.xml` | 내비게이션 메뉴 |
| `security_alert/default/data/ui/views/*.xml` | 대시보드 및 빌더 뷰 |
| `security_alert/bin/safe_fmt.py` | 알림 페이로드 포매터 |
| `security_alert/bin/slack.py` | Slack 전송 스크립트 |
| `security_alert/bin/six.py` | Py2/P3 호환 헬퍼 |
| `security_alert/lib/python3/*` | 오프라인용 외부 의존성 |
| `security_alert/metadata/default.meta` | 앱 권한 / 가시성 |
| `docs/` | 배포·릴리스·레거시 정리 문서 |
| `resume/` | 이전 아키텍처·API·트러블슈팅 스냅샷 |

## 먼저 읽을 파일 / First Files to Read

| 순서 | 파일 | 왜 읽어야 하는가 |
| --- | --- | --- |
| 1 | `security_alert/app.conf` | 앱 ID, 라벨, 버전 확인 |
| 2 | `security_alert/default/alert_actions.conf` | 알림 액션이 무엇을 호출하는지 파악 |
| 3 | `security_alert/bin/slack.py` | 실제 외부 호출 로직 위치 |
| 4 | `security_alert/default/savedsearches.conf` | 트리거 검색 정의 |
| 5 | `docs/QUICK-START.md` | 짧은 설치·동작 가이드 |
| 6 | `docs/DEPLOYMENT.md` | 배포 절차 및 환경 변수 |

## 진입점 / Entry Points

| 진입점 종류 | 위치 | 호출자 / 사용자 |
| --- | --- | --- |
| 알림 액션 핸들러 | `security_alert/bin/safe_fmt.py`, `security_alert/bin/slack.py` | Splunk alertd |
| 저장된 검색 | `security_alert/default/savedsearches.conf` | Splunk scheduler |
| UI 내비게이션 | `security_alert/default/data/ui/nav/default.xml` | Splunk Web 사용자 |
| 빌더 뷰 | `default/data/ui/views/easy_alert_builder.xml`, `alert-builder.xml` | 알림 작성자 |
| 관리 대시보드 | `default/data/ui/views/alert-management-dashboard.xml` | 운영자 |
| 데이터 탐색기 | `default/data/ui/views/data-explorer-dashboard.xml` | 분석가 |

---

## 빠른 시작 / Quickstart

운영자가 처음 5 분 안에 끝낼 수 있는 절차입니다.

1. 저장소 클론 후 `security_alert/` 디렉터리를 Splunk 앱 경로(`$SPLUNK_HOME/etc/apps/`)로 복사합니다.
2. Splunk 를 재시작하거나 `splunk reload deploy-server` 로 앱을 리프레시합니다.
3. Splunk Web → Apps → "Security Alert" 가 보이는지 확인합니다.
4. Settings → Alert Actions 에서 "Security Alert" 액션을 활성화합니다.
5. `bin/slack.py` 가 사용하는 Webhook URL 을 앱 설정(예: `alert_actions.conf` 의 `param.webhook_url`)에 입력합니다.
6. `savedsearches.conf` 의 샘플 저장 검색을 활성화하고, 즉시 실행(예ecute now) 으로 메시지가 도착하는지 확인합니다.

## 사용 예시 / Usage

- 새 알림 추가: `easy_alert_builder` 뷰에서 조건 → 액션 → 채널(Slack) 순서로 저장합니다.
- 알림 이력 확인: `alert-management-dashboard` 에서 최근 발생 시각 · 채널 · 성공 여부를 봅니다.
- 원시 데이터 조회: `data-explorer-dashboard` 에서 시간 범위와 인덱스를 지정해 검토합니다.
- 포맷 변경: `bin/safe_fmt.py` 의 페이로드 빌더를 수정해 카드 / 헤더 / 링크를 조정합니다.

## 설정 / Configuration

| 설정 키 | 위치 | 목적 |
| --- | --- | --- |
| `param.webhook_url` | `alert_actions.conf` | Slack 수신 Webhook |
| `param.xwiki_endpoint` | `alert_actions.conf` (해당 시) | XWiki 알림 엔드포인트 |
| `app.conf` `[install]` | `app.conf` | 설치 시 메타데이터 |
| `default.meta` `owner = admin` | `metadata/default.meta` | 객체 권한 |
| `transforms.conf` 룰 | `default/transforms.conf` | 필드 재작성 |

> 비밀 값(웹훅 URL, 토큰 등)은 절대 저장소에 커밋하지 마세요. Splunk 암호화된 자격 증명(`credentials.conf`) 또는 외부 시크릿 매니저를 사용하세요.

## 명령어 / Commands Reference

| 명령 | 위치 | 설명 |
| --- | --- | --- |
| `python3 bin/safe_fmt.py` | `security_alert/bin/` | 알림 페이로드 포맷 검증(로컬 테스트) |
| `python3 bin/slack.py` | `security_alert/bin/` | Slack 전송 단위 테스트(환경 변수 필요) |
| `splunk reload deploy-server` | Splunk CLI | 앱 설정 리프레시 |
| `splunk display app` | Splunk CLI | 앱 메타데이터 확인 |

## 로컬 개발 / Local Development

- 권장 Python: 3.x (앱이 번들한 `lib/python3/` 기준).
- Splunk 앱 심볼릭 링크로 작업하면 재시작 없이 반영됩니다.
  - 예: `ln -s <repo>/security_alert $SPLUNK_HOME/etc/apps/security_alert`
- UI XML 변경 후에는 Splunk Web 새로 고침 한 번으로 적용됩니다.
- Python 스크립트 변경 후에는 `splunk reload deploy-server` 가 필요할 수 있습니다.

## 테스트 / Testing

- 수동 회귀: 핵심 저장 검색을 1 회 실행해 Slack 도착 여부를 확인합니다.
- 포맷 단위 테스트: `bin/safe_fmt.py` 를 더미 페이로드로 실행해 JSON 결과를 확인합니다.
- 전송 단위 테스트: 임시 Webhook 으로 `bin/slack.py` 를 호출해 200 OK 를 확인합니다.
- 자동화 테스트 프레임워크는 현재 번들되어 있지 않습니다. 추가 시 `docs/DEPLOYMENT.md` 의 체크리스트를 함께 갱신하세요.

## 기여 / Contributing

기여 절차는 [`CONTRIBUTING.md`](CONTRIBUTING.md) 를 따릅니다. 핵심 규칙은 다음과 같습니다.

- 변경 전 `docs/QUICK-START.md` 의 빠른 시작이 그대로 동작하는지 확인합니다.
- `alert_actions.conf` 와 `bin/*.py` 의 인터페이스는 시그니처 호환을 유지합니다.
- 새 의존성을 추가할 때는 `security_alert/lib/python3/` 에 함께 번들합니다(오프라인 호환).
- UI 변경 시 네 개의 뷰가 모두 깨지지 않는지 확인합니다.

## 유지보수 / Maintainers

| 역할 | 책임 | 첫 번째 확인 위치 |
| --- | --- | --- |
| 앱 소유자 | 버전 릴리스, 권한 정책 | `app.conf`, `metadata/default.meta` |
| 알림 채널 책임자 | Slack / XWiki 통합 | `bin/slack.py`, `bin/safe_fmt.py` |
| UI 책임자 | 대시보드·빌더 | `data/ui/views/*.xml` |
| 운영자 | 배포, 트러블슈팅 | `docs/DEPLOYMENT.md`, `docs/TROUBLESHOOTING.md` (resume 내) |

문제 발생 시: 저장소 이슈 트래커를 통해 재현 절차 · 환경 · 로그를 첨부해 주세요.

## 추가 문서 / Further Documentation

| 문서 | 경로 | 내용 |
| --- | --- | --- |
| 빠른 시작 | [`docs/QUICK-START.md`](docs/QUICK-START.md) | 5 분 설치 가이드 |
| 배포 가이드 | [`docs/DEPLOYMENT.md`](docs/DEPLOYMENT.md) | 환경별 배포 절차 |
| 릴리스 노트 | [`docs/RELEASE-NOTES.md`](docs/RELEASE-NOTES.md) | 버전별 변경 사항 |
| 레거시 정리 보고서 | [`docs/LEGACY-CLEANUP-REPORT.md`](docs/LEGACY-CLEANUP-REPORT.md) | 정리 이력 |
| XWiki 알림 저장소 가이드 | [`docs/ALERT-REPOSITORY-XWIKI.md`](docs/ALERT-REPOSITORY-XWIKI.md) | XWiki 연동 메모 |
| 데모 | [`demo/README.md`](demo/README.md) | 데모 자산 안내 |
| 아키텍처 스냅샷 | [`resume/ARCHITECTURE.md`](resume/ARCHITECTURE.md) | 이전 아키텍처(참고용) |
| API 스냅샷 | [`resume/API.md`](resume/API.md) | 이전 API 메모(참고용) |
| 트러블슈팅 | [`resume/TROUBLESHOOTING.md`](resume/TROUBLESHOOTING.md) | 알려진 이슈 |

## 라이선스 / License

이 프로젝트의 라이선스 조건은 저장소 최상위 [`LICENSE`](LICENSE) 파일을 따릅니다. 배포 및 수정 시 해당 파일의 조항을 우선 검토하세요.