# Splunk Security Alert

Splunk용 보안 알림 애플리케이션. 검색 결과 기반 경보 액션, Slack 통지, 알림 빌더 및 관리 대시보드를 제공합니다.

## 개요 (Overview)

이 앱은 Splunk Enterprise / Splunk Cloud에서 동작하는 보안 알림(Security Alert) 애드온입니다. 저장된 검색(Saved Search)과 매크로, 트랜스폼을 결합하여 위협 이벤트를 탐지하고, 정의된 알림 액션을 통해 운영자에게 전달합니다. Slack 채널 전송, 대시보드 시각화, 사용자 정의 알림 빌더를 한 패키지로 제공합니다.

## 한눈에 보기 (Quick Glance)

| 항목 | 값 |
| --- | --- |
| 제품 유형 | Splunk 앱 (TA/App) |
| 주요 기능 | 보안 알림 액션, Slack 통지, 알림 빌더 UI |
| 대상 플랫폼 | Splunk Enterprise / Splunk Cloud (Python 3 런타임) |
| 핵심 디렉터리 | `security_alert/default/`, `security_alert/bin/`, `security_alert/lib/python3/` |
| 기본 엔트리 포인트 | `security_alert/default/alert_actions.conf`, `security_alert/default/savedsearches.conf` |
| 통지 채널 | Slack (`bin/slack.py`) |
| 운영 대시보드 | `default.xml`, `alert-management-dashboard.xml`, `data-explorer-dashboard.xml` |
| 상태 | 운영 가능 (Production-ready, Splunk 8.x / 9.x 검증) |
| 문서 위치 | `docs/`, `resume/` |

## 동작 흐름 (Flow Summary)

1. Splunk가 `default/savedsearches.conf`에 정의된 보안 검색을 주기적으로 실행합니다.
2. `default/macros.conf`와 `default/transforms.conf`로 필드 정규화 및 이벤트 보강이 수행됩니다.
3. 매칭된 이벤트는 `default/alert_actions.conf`에 등록된 알림 액션으로 전달됩니다.
4. `bin/slack.py`가 Slack 채널로 알림을 전송하고, `bin/safe_fmt.py`가 메시지 포맷을 안전하게 처리합니다.
5. UI 대시보드(알림 빌더, 알림 관리, 데이터 탐색)가 운영자가 인시던트를 추적·관리하도록 지원합니다.

## 패키지 구성 (Package Contents)

- `security_alert/default/` — Splunk 설정 모음 (`alert_actions.conf`, `app.conf`, `macros.conf`, `props.conf`, `savedsearches.conf`, `transforms.conf`).
- `security_alert/bin/` — 알림 스크립트 (`safe_fmt.py`, `slack.py`, 호환용 `six.py`).
- `security_alert/default/data/ui/nav/default.xml` — 탐색 메뉴 정의.
- `security_alert/default/data/ui/views/` — 대시보드 XML (`alert-builder.xml`, `alert-management-dashboard.xml`, `data-explorer-dashboard.xml`, `easy_alert_builder.xml`).
- `security_alert/lib/python3/` — 번들된 의존성 (`urllib3`, `idna`, `charset_normalizer`).
- `security_alert/metadata/default.meta` — 앱 메타데이터 및 권한.

## 빠른 시작 (Quickstart)

운영자가 처음 배포할 때 따르는 순서입니다. 자세한 절차는 `docs/QUICK-START.md`와 `docs/DEPLOYMENT.md`를 참고하십시오.

1. 앱 패키지를 Splunk 인스턴스의 `$SPLUNK_HOME/etc/apps/` 경로에 복사합니다.
2. Splunk 서비스를 재시작하거나 배포 관리자(Deployment Server)로 푸시합니다.
3. Splunk Web에서 **설정 → 알림 액션**으로 이동해 새 액션이 노출되는지 확인합니다.
4. `alert_actions.conf`의 필수 매개변수(예: Slack Webhook URL, 채널명)를 환경에 맞게 설정합니다.
5. 샘플 저장된 검색을 활성화하고, 검색 결과가 Slack으로 도착하는지 검증합니다.
6. **앱: Security Alert** 메뉴에서 알림 관리 대시보드를 열어 동작을 확인합니다.

## 명령·엔트리 참조 (Entry Points)

| 구성요소 | 경로 | 역할 |
| --- | --- | --- |
| 알림 액션 정의 | `security_alert/default/alert_actions.conf` | 사용자 정의 알림 액션 등록 |
| 저장된 검색 | `security_alert/default/savedsearches.conf` | 정기 실행되는 탐지 검색 |
| 매크로 | `security_alert/default/macros.conf` | 검색 재사용 조각 |
| 필드 변환 | `security_alert/default/transforms.conf` | 이벤트 필드 정규화 |
| 인덱스 시간 필드 추출 | `security_alert/default/props.conf` | 인덱서 측 props |
| Slack 통지 스크립트 | `security_alert/bin/slack.py` | Slack Webhook 호출 |
| 메시지 포맷터 | `security_alert/bin/safe_fmt.py` | Slack 메시지 안전 포맷팅 |
| 탐색 | `security_alert/default/data/ui/nav/default.xml` | Splunk Web 메뉴 |

## 구성 (Configuration)

- **Slack Webhook**: 알림 액션 매개변수 또는 환경 변수에서 Webhook URL을 설정합니다.
- **채널 매핑**: 환경/심각도별 채널 라우팅은 `alert_actions.conf`의 매개변수로 제어합니다.
- **권한**: `metadata/default.meta`의 capability 설정을 검토하여 역할별 접근 권한을 부여합니다.
- **검색 일정**: `savedsearches.conf`의 `cron_schedule`을 조직 정책에 맞게 조정합니다.

## 로컬 개발 (Local Development)

- Python 3 호환성을 위해 `bin/` 스크립트는 Python 3 런타임을 가정합니다.
- 번들 의존성은 `lib/python3/` 하위에 고정되어 있으며, 임의로 업그레이드하지 마십시오.
- 로컬 검증은 `docs/QUICK-START.md`의 “로컬 검증” 절차를 따르십시오.
- 데모 자산은 `demo/` 디렉터리에서 확인할 수 있습니다 (`demo/README.md` 참고).

## 테스트 (Testing)

- 검색 결과가 매크로·트랜스폼을 거쳐 알림 액션까지 도달하는지 확인하는 엔드 투 엔드 흐름을 권장합니다.
- Slack 통지는 테스트 채널에서 메시지 포맷과 차단 동작(`safe_fmt.py`)을 검증합니다.
- 회귀 테스트 절차는 `docs/LEGACY-CLEANUP-REPORT.md`의 체크리스트를 활용할 수 있습니다.

## 기여 (Contributing)

기여 절차는 `CONTRIBUTING.md`를 따릅니다. 보안 관련 변경 사항은 코드 리뷰와 함께 별도 브랜치를 사용해 주십시오.

## 유지보수 담당자 (Maintainers)

- 본 앱의 운영 책임은 사내 보안 운영팀(Security Operations)입니다.
- 인시던트 및 권한 요청은 사내 보안 채널을 통해 접수합니다.
- 연락처 정보는 `resume/` 디렉터리의 담당자 문서를 참고하십시오.

## 추가 문서 (Further Documentation)

- `docs/ALERT-REPOSITORY-XWIKI.md` — XWiki 기반 알림 저장소 연동 안내
- `docs/DEPLOYMENT.md` — 배포 절차 상세
- `docs/QUICK-START.md` — 빠른 시작 가이드
- `docs/RELEASE-NOTES.md` — 릴리스 노트
- `docs/LEGACY-CLEANUP-REPORT.md` — 레거시 정리 이력
- `resume/API.md` — API 참조
- `resume/ARCHITECTURE.md` — 아키텍처 상세
- `resume/DEPLOYMENT.md` — 배포 운영 가이드
- `resume/TROUBLESHOOTING.md` — 장애 대응 가이드

## 라이선스 (License)

저장소 루트의 `LICENSE` 파일을 참고하십시오.