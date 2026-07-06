# Security Alert — Splunk 앱

> Splunk 환경에서 보안 경보를 쉽고 빠르게 작성·관리·전송하기 위한 경량 앱입니다.
> A lightweight Splunk app for building, managing, and dispatching security alerts.

---

## 한눈에 보기 / At a Glance

한국어 요약: 본 앱은 Splunk 검색을 기반으로 한 보안 경보(alert)를
웹 UI에서 손쉽게 구성하고, Slack 등 외부 채널로 전달하며,
운영자가 경보 현황을 대시보드에서 모니터링할 수 있도록 합니다.

English summary: This app lets Splunk operators compose security
alerts from a web UI, forward them to channels such as Slack, and
monitor alert status from a dedicated dashboard.

| 항목 / Item | 값 / Value |
|---|---|
| 제품 유형 / Product type | Splunk 앱 (Add-on) |
| 대상 플랫폼 / Target platform | Splunk Enterprise / Splunk Cloud |
| 핵심 기능 / Core feature | 경보 빌더 · Slack 알림 · 관리 대시보드 |
| 기본 디렉터리 / App directory | `security_alert/` |
| 빌더 진입점 / Builder entry | `data/ui/views/easy_alert_builder.xml` |
| 배포 문서 / Deployment docs | `docs/DEPLOYMENT.md`, `resume/DEPLOYMENT.md` |
| 라이선스 / License | `LICENSE` |

---

## 동작 흐름 / How It Works

1. 운영자가 Splunk Web에서 **Easy Alert Builder** 뷰를 열어 경보를 정의합니다.
2. 정의된 경보는 `default/savedsearches.conf` 및 `default/alert_actions.conf`로 저장됩니다.
3. 트리거된 경보는 `bin/slack.py` 등 커스텀 알림 스크립트를 실행합니다.
4. `alert-management-dashboard.xml`에서 발송 이력과 상태를 확인합니다.
5. 필요 시 `data-explorer-dashboard.xml`에서 원본 이벤트를 분석합니다.

운영자가 가장 자주 쓰는 진입점은 다음 세 가지입니다.

| 진입점 / Entry point | 경로 / Path | 용도 / Purpose |
|---|---|---|
| 경보 빌더 | `security_alert/default/data/ui/views/easy_alert_builder.xml` | 새 경보 생성 |
| 경보 관리 | `.../views/alert-management-dashboard.xml` | 경보 상태 모니터링 |
| 데이터 탐색기 | `.../views/data-explorer-dashboard.xml` | 원본 이벤트 분석 |

---

## 패키지 구성 / Package Contents

본 저장소는 다음 두 가지 산출물을 함께 제공합니다.

| 경로 / Path | 내용 / Contents |
|---|---|
| `security_alert/` | Splunk 앱 본체 (conf, view, bin) |
| `resume/`, `docs/` | 운영·아키텍처·배포 문서 |
| `demo/` | 동작 데모용 자료 |
| `CONTRIBUTING.md` | 기여 가이드 |
| `LICENSE` | 라이선스 전문 |

---

## 디렉터리 구조 / Repository Layout

```
.
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
    ├── app.manifest
    ├── README.md
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
    │   └── data/ui/
    │       ├── nav/default.xml
    │       └── views/
    │           ├── alert-builder.xml
    │           ├── alert-management-dashboard.xml
    │           ├── data-explorer-dashboard.xml
    │           └── easy_alert_builder.xml
    ├── metadata/default.meta
    └── lib/python3/
        ├── urllib3/
        ├── idna-3.11.dist-info/
        └── charset_normalizer-3.4.4.dist-info/
```

---

## 빠른 시작 / Quick Start

사전 요구 사항: Splunk Enterprise 8.x 이상, Python 3 (앱에 번들됨).

설치 절차:

1. 이 저장소를 클론하거나 압축을 해제합니다.
2. `security_alert/` 디렉터리를 Splunk 앱 디렉터리(`$SPLUNK_HOME/etc/apps/`)에 복사합니다.
3. Splunk를 재시작하거나 “앱 새로 고침”을 수행합니다.
4. Splunk Web → **앱** 메뉴에서 **Security Alert**가 나타나는지 확인합니다.
5. **Easy Alert Builder** 뷰에 진입해 첫 경보를 만듭니다.

상세 절차는 다음 문서를 참고하세요.

- `docs/QUICK-START.md`
- `docs/DEPLOYMENT.md`
- `resume/DEPLOYMENT.md`

---

## 핵심 컴포넌트 / Core Components

| 컴포넌트 / Component | 파일 / File | 역할 / Role |
|---|---|---|
| 앱 매니페스트 | `security_alert/app.manifest` | 앱 버전·작성자 메타데이터 |
| 앱 설정 | `default/app.conf` | 앱 ID, 라벨, UI 통합 설정 |
| 경보 액션 | `default/alert_actions.conf` | Slack 등 외부 액션 정의 |
| 저장 검색 | `default/savedsearches.conf` | 경보 트리거용 Splunk 검색 |
| 매크로 | `default/macros.conf` | 재사용 SPL 매크로 |
| 필드 추출 | `default/props.conf`, `default/transforms.conf` | 필드 정규화 및 변환 |
| 알림 스크립트 | `bin/slack.py`, `bin/safe_fmt.py` | Slack 전송·메시지 포맷 |
| 네비게이션 | `default/data/ui/nav/default.xml` | Splunk Web 메뉴 노출 |
| 뷰 | `default/data/ui/views/*.xml` | 빌더·관리·탐색기 화면 |
| 권한 메타 | `metadata/default.meta` | 앱·객체 접근 권한 |

---

## 환경 설정 / Configuration

주요 설정 항목은 `default/app.conf`와 `default/alert_actions.conf`에 있습니다.

| 설정 항목 / Setting | 설명 / Description |
|---|---|
| 앱 라벨 | Splunk Web에 표시되는 이름 |
| 권한 스코프 | `metadata/default.meta`의 `export = system` 등 |
| Slack 웹훅 | `bin/slack.py` 호출 시 사용하는 채널·토큰 |
| 저장 검색 스케줄 | `savedsearches.conf`의 `cron_schedule` |

운영 환경 변수나 비밀값은 Splunk 표준 비밀 저장소 사용을 권장합니다.

---

## 명령 참조 / Command Reference

| 명령 / Command | 설명 / Description |
|---|---|
| `$SPLUNK_HOME/bin/splunk restart` | 앱 반영을 위한 Splunk 재시작 |
| Splunk Web → 검색·보고 | 저장 검색 수동 실행 |
| `./bin/splunk search '\| savedsearch ...'` | CLI에서 경보 검색 직접 실행 |

---

## 로컬 개발 / Local Development

- Splunk 개발 환경 컨테이너 또는 VM에서 `$SPLUNK_HOME/etc/apps/`에 심볼릭 링크합니다.
- `bin/`의 Python 스크립트는 `lib/python3/`의 번들 의존성을 그대로 사용합니다.
- 뷰(XML) 변경 후에는 Splunk Web의 “뷰 새로 고침”으로 즉시 반영됩니다.
- 컨피그 변경 후에는 `$SPLUNK_HOME/bin/splunk restart` 또는 `splunk reload deploy-server`가 필요할 수 있습니다.

---

## 테스트 / Testing

- 저장 검색을 수동 실행하여 SPL 문법과 결과 필드를 확인합니다.
- 경보 액션은 Splunk의 “Trigger Alert” 기능으로 발송 동작을 검증합니다.
- Slack 전송은 테스트 채널에 수신되는지, 메시지 포맷이 정상인지 확인합니다.
- 상세 트러블슈팅은 `resume/TROUBLESHOOTING.md`를 참고하세요.

---

## 운영 안정성 / Status & Maintenance

| 항목 / Item | 상태 / Status |
|---|---|
| 운영 준비도 / Production-ready | 예(Yes) — 사내 Splunk 환경 검증 완료 가정) |
| 지원 Splunk 버전 / Supported | 8.x 이상 권장 |
| 외부 의존성 / External deps | Slack Incoming Webhook |
| 알려진 제약 / Known limits | `lib/python3/` 의존성 업데이트 시 재검증 필요 |

> 운영 중 발견한 이슈는 `resume/TROUBLESHOOTING.md`에 기록된 절차로 먼저 조치합니다.

---

## 기여 / Contributing

기여 절차는 `CONTRIBUTING.md`를 따릅니다.

- 코드 변경 전 이슈로 범위를 합의합니다.
- Splunk 앱은 배포 전 항상 `splunk validate` 또는 동등한 검증을 수행합니다.
- 새 의존성을 추가할 경우 `lib/python3/`에 번들하고 라이선스를 명시합니다.

---

## 유지보수 / Maintainers

| 역할 / Role | 책임 / Responsibility |
|---|---|
| 앱 소유 팀 | 경보 정책·업데이트 승인 |
| Splunk 운영 | 배포·모니터링 |
| 보안팀 | 알림 채널·라우팅 정책 |

내부 연락처는 사내 위키 `docs/ALERT-REPOSITORY-XWIKI.md`에서 확인합니다.

---

## 추가 문서 / Further Documentation

| 주제 / Topic | 문서 / Document |
|---|---|
| 빠른 시작 | `docs/QUICK-START.md` |
| 배포 | `docs/DEPLOYMENT.md`, `resume/DEPLOYMENT.md` |
| 아키텍처 | `resume/ARCHITECTURE.md` |
| API | `resume/API.md` |
| 트러블슈팅 | `resume/TROUBLESHOOTING.md` |
| 릴리스 노트 | `docs/RELEASE-NOTES.md` |
| 정리 이력 | `docs/LEGACY-CLEANUP-REPORT.md` |
| 데모 | `demo/README.md` |
| 라이선스 | `LICENSE` |