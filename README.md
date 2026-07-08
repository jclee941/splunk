# Security Alert (Splunk Add-on)

![Platform: Splunk](https://img.shields.io/badge/platform-Splunk-65a637)
![Python: 3.x](https://img.shields.io/badge/python-3.x-blue)
[![Docs: resume/](https://img.shields.io/badge/docs-resume%2F-lightgrey)](resume/)

> Splunk 검색 헤드에서 보안 알림을 작성하고, 운영하고, 시각화하는 앱입니다.
> 알림 빌더, 알림 관리 대시보드, 데이터 탐색기, Easy Alert Builder 화면과 Slack 알림 모듈을 제공합니다.

[기여 가이드](CONTRIBUTING.md) · [라이선스](LICENSE) · [데모](demo/) · [릴리스 노트](docs/RELEASE-NOTES.md)

---

## 한 줄 요약

`security_alert/`는 Splunk 앱 디렉터리이며, `bin/`의 Python 스크립트와 `default/`의 Splunk 설정·뷰 파일로 보안 알림 라이프사이클을 다룹니다.

## 상태 (Status)

| 항목 | 값 | 출처 |
| --- | --- | --- |
| 제품 유형 | Splunk App | `security_alert/app.manifest` |
| 런타임 | Python 3 (번들 의존성 vendor) | `security_alert/lib/python3/` |
| 번들 라이브러리 | urllib3, idna 3.11, charset_normalizer 3.4.4 | `security_alert/lib/python3/` |
| 기본 알림 채널 | Slack (Webhook) | `security_alert/bin/slack.py` |
| 알림 빌더 | alert-builder, easy-alert-builder | `security_alert/default/data/ui/views/` |
| 운영 화면 | alert-management-dashboard, data-explorer-dashboard | `security_alert/default/data/ui/views/` |
| 빌더 안전 포맷터 | sandboxed 문자열 포맷 | `security_alert/bin/safe_fmt.py` |
| 보안 호환성 마이그레이션 | 완료 (legacy 정리) | `docs/LEGACY-CLEANUP-REPORT.md` |

## 운영 흐름 (Operational Flow)

1. 운영자가 Splunk Web에서 앱을 열고 **alert-builder** 또는 **easy-alert-builder**에서 검색·트리거 조건을 작성합니다.
2. 빌더가 `savedsearches.conf`에 저장된 정의를 생성하면 Splunk가 스케줄에 따라 검색을 실행합니다.
3. 이벤트 발화 시 `default/alert_actions.conf`의 액션이 트리거되고, Slack 알림은 `bin/slack.py`가 표준화된 페이로드로 전송합니다.
4. 운영자는 **alert-management-dashboard**에서 발화 이력을 확인하고 **data-explorer-dashboard**에서 원본 이벤트를 검토합니다.
5. 빌더 출력의 동적 부분은 `bin/safe_fmt.py`로 샌드박스 처리되어 포맷 인젝션을 차단합니다.

## 목차 (Contents)

- [패키지 구성](#패키지-구성)
- [먼저 읽을 파일](#먼저-읽을-파일)
- [빠른 시작](#빠른-시작)
- [진입점 (Entry Points)](#진입점)
- [아키텍처](#아키텍처)
- [설정 및 데이터 매핑](#설정-및-데이터-매핑)
- [로컬 개발 및 테스트](#로컬-개발-및-테스트)
- [배포 및 운영](#배포-및-운영)
- [문제 해결](#문제-해결)
- [기여 및 라이선스](#기여-및-라이선스)

---

## 패키지 구성 (Package Contents)

| 경로 | 역할 |
| --- | --- |
| `security_alert/app.manifest` | Splunk 앱 메타데이터, 버전, 의존성 선언 |
| `security_alert/default/app.conf` | 앱 식별자, 라벨, UI 설정 |
| `security_alert/default/nav/default.xml` | Splunk Web 네비게이션 항목 |
| `security_alert/default/data/ui/views/alert-builder.xml` | 고급 알림 빌더 화면 |
| `security_alert/default/data/ui/views/easy_alert_builder.xml` | 단순 모드 알림 빌더 |
| `security_alert/default/data/ui/views/alert-management-dashboard.xml` | 발화 알림 모니터링 |
| `security_alert/default/data/ui/views/data-explorer-dashboard.xml` | 원본 이벤트 탐색 |
| `security_alert/default/savedsearches.conf` | 부트스트랩용 저장 검색 |
| `security_alert/default/alert_actions.conf` | 커스텀 알림 액션 정의 |
| `security_alert/default/macros.conf` | 재사용 검색 매크로 |
| `security_alert/default/props.conf`, `transforms.conf` | 필드 추출·재작성 |
| `security_alert/bin/safe_fmt.py` | 샌드박스 문자열 포맷터 (빌더 출력) |
| `security_alert/bin/slack.py` | Slack Webhook 알림 전송기 |
| `security_alert/bin/six.py` | Python 2/3 호환 셰임 |
| `security_alert/lib/python3/` | urllib3, idna, charset_normalizer 번들 의존성 |
| `resume/` | 운영·아키텍처·API·트러블슈팅 문서 묶음 |
| `docs/` | 배포, 퀵스타트, 레거시 정리, 릴리스 노트 |
| `demo/` | 데모 시나리오 및 사용 예 |

## 먼저 읽을 파일

운영자가 처음 열람할 때 권장하는 순서입니다.

1. `resume/ARCHITECTURE.md` — 컴포넌트 책임과 데이터 흐름
2. `resume/API.md` — 빌더 출력 스키마, 알림 페이로드 스키마
3. `docs/QUICK-START.md` — Splunk 인스턴스에 앱 적재 후 첫 빌더 실행
4. `docs/DEPLOYMENT.md` 및 `resume/DEPLOYMENT.md` — 배포 절차와 사전 점검
5. `docs/RELEASE-NOTES.md` — 변경 이력과 호환성 노트

## 빠른 시작

Splunk 인스턴스가 이미 있고 `security_alert/` 디렉터리를 그대로 분배(앱)하는 흐름입니다.

| 단계 | 명령 또는 동작 |
| --- | --- |
| 앱 적재 | Splunk Web 또는 `$SPLUNK_HOME/etc/apps/`에 `security_alert/`를 복사 |
| 재시작 검증 | Splunkd 재시작 후 앱이 목록에 노출되는지 확인 |
| 빌더 진입 | Splunk Web → 앱 → **Alert Builder** 또는 **Easy Alert Builder** |
| Slack 연동 | 앱 컨텍스트에서 Webhook URL 설정 후 저장 검색에서 `slack` 액션 활성화 |
| 첫 알림 발화 | 저장 검색이 트리거되면 `alert-management-dashboard`에 표시 |

자세한 절차와 검증 체크리스트는 `docs/QUICK-START.md`와 `docs/DEPLOYMENT.md`에 있습니다.

## 진입점 (Entry Points)

| 표면 | 위치 | 호출 시점 |
| --- | --- | --- |
| Splunk Web UI 진입점 | `security_alert/default/nav/default.xml`, `data/ui/views/*.xml` | 사용자가 앱 선택 시 |
| 알림 액션 핸들러 | `security_alert/bin/slack.py` | 저장 검색 트리거 시 |
| 빌더 포맷 헬퍼 | `security_alert/bin/safe_fmt.py` | 빌더가 메시지 렌더링 시 |
| 부트스트랩 검색 | `security_alert/default/savedsearches.conf` | Splunk 스케줄러 시작 시 |
| 커스텀 알림 액션 등록 | `security_alert/default/alert_actions.conf` | 앱 적재 시 |
| 검색 매크로 | `security_alert/default/macros.conf` | 검색 파싱 시 |

## 아키텍처

핵심은 **UI 계층**(Splunk JS 뷰 4종), **설정 계층**(conf 파일), **실행 계층**(bin 스크립트), **번들 의존성**(lib/python3)으로 나뉩니다.

| 계층 | 구성 | 역할 |
| --- | --- | --- |
| UI | 4개 XML 뷰 + `nav/default.xml` | 빌더 입력, 대시보드 렌더링 |
| 설정 | `app.conf`, `savedsearches.conf`, `alert_actions.conf`, `macros.conf`, `props.conf`, `transforms.conf` | 검색·액션·필드 변환 정의 |
| 실행 | `bin/slack.py`, `bin/safe_fmt.py` | 알림 송신, 포맷 정제 |
| 번들 의존성 | urllib3, idna, charset_normalizer | HTTPS 호출 및 문자 인코딩 |

요청 흐름은 다음 순서로 동작합니다.

1. 빌더가 사용자 입력을 받아 `savedsearches.conf` 호환 정의를 구성합니다.
2. 저장 검색 실행 결과가 임계 조건을 만족하면 `alert_actions.conf`의 액션이 호출됩니다.
3. `slack` 액션은 `bin/slack.py`를 호출해 `urllib3`로 Webhook에 POST 요청을 보냅니다.
4. 빌더 출력의 사용자 입력은 `bin/safe_fmt.py`로 정제된 후 페이로드에 포함됩니다.
5. 운영 대시보드는 발화 이력과 원본 이벤트를 같은 앱 컨텍스트에서 읽어옵니다.

상세 컴포넌트 다이어그램은 `resume/ARCHITECTURE.md`를, 필드·페이로드 스키마는 `resume/API.md`를 참조하세요.

## 설정 및 데이터 매핑

| 설정 파일 | 편집 빈도 | 주 용도 |
| --- | --- | --- |
| `app.conf` | 낮음 | 앱 이름, 라벨, 권한 |
| `savedsearches.conf` | 잦음 | 알림으로 사용할 저장 검색 |
| `alert_actions.conf` | 중간 | 빌더가 만든 알림 액션 매핑 |
| `macros.conf` | 중간 | 검색 매크로 (예: `getAlertPayload`) |
| `props.conf` | 낮음 | 필드 추출 규칙 |
| `transforms.conf` | 낮음 | 값 재작성 룰 |

| 입력 | 정규화 위치 | 출력 |
| --- | --- | --- |
| 사용자 검색 문자열 | 빌더 입력 파서 | `search` 스탠자에 반영 |
| 사용자 알림 메시지 | `bin/safe_fmt.py` | 샌드박스 처리된 본문 |
| 이벤트 필드 | `props.conf` + `transforms.conf` | 표준화된 키 |
| Slack 채널·Webhook | 앱 컨텍스트 설정 | `bin/slack.py` 인자 |
| 빌더 폼 상태 | `easy_alert_builder.xml` | 단일 검색 스탠자 |

## 로컬 개발 및 테스트

| 작업 | 절차 |
| --- | --- |
| 앱 디렉터리 정합성 | 변경 후 `docs/DEPLOYMENT.md`의 점검 목록 수행 |
| 빌더 동작 | `docs/QUICK-START.md`의 수동 시나리오 재현 |
| Slack 알림 | `bin/slack.py --dry-run` 등 인자 옵션으로 페이로드 검증 |
| 정적 점검 | `security_alert/bin/`에서 import 경로(`lib/python3/`)가 깨지지 않는지 확인 |
| 데모 | `demo/README.md`의 시나리오 실행 |

## 배포 및 운영

- 배포 절차 및 환경 변수: `docs/DEPLOYMENT.md`, `resume/DEPLOYMENT.md`
- 배포 전 호환성 메모: `docs/RELEASE-NOTES.md`
- 외부 알림 저장소 연동 메모: `docs/ALERT-REPOSITORY-XWIKI.md`
- 이전 자산 정리 이력: `docs/LEGACY-CLEANUP-REPORT.md`

## 문제 해결

| 증상 | 1차 확인 문서 |
| --- | --- |
| 빌더가 정의를 저장하지 못함 | `resume/TROUBLESHOOTING.md` |
| Slack 알림이 도착하지 않음 | `resume/TROUBLESHOOTING.md`, `docs/DEPLOYMENT.md` |
| Splunk Web에 앱이 노출되지 않음 | `resume/DEPLOYMENT.md`, `docs/QUICK-START.md` |
| 매크로·필드 변환 미적용 | `resume/API.md`, `default/macros.conf` |

## 기여 및 라이선스

- 기여 절차: [CONTRIBUTING.md](CONTRIBUTING.md)
- 라이선스: [LICENSE](LICENSE)
- 사내 변경 제안은 `docs/RELEASE-NOTES.md` 양식을 따라 주세요.

## 더 읽어보기 (Further Documentation)

- 운영·아키텍처·API·트러블슈팅: `resume/ARCHITECTURE.md`, `resume/API.md`, `resume/DEPLOYMENT.md`, `resume/TROUBLESHOOTING.md`
- 배포·퀵스타트·릴리스 노트: `docs/DEPLOYMENT.md`, `docs/QUICK-START.md`, `docs/RELEASE-NOTES.md`
- 외부 저장소 연동 메모: `docs/ALERT-REPOSITORY-XWIKI.md`
- 데모 시나리오: `demo/README.md`