# Security Alert (Splunk App) / 보안 알림 (Splunk 앱)

![Splunk](https://img.shields.io/badge/Splunk-9.x-FF6F0F?logo=splunk&logoColor=white)
![Python](https://img.shields.io/badge/Python-3-3776AB?logo=python&logoColor=white)
![Air--gapped](https://img.shields.io/badge/Air--gapped-Supported-2E7D32)
![Status](https://img.shields.io/badge/Status-Production-2E7D32)
[![License](https://img.shields.io/badge/License-See%20LICENSE-555555)](#LICENSE)

프로덕션급 Splunk 애드온으로, 통합 알림 관리 대시보드, 안내형 손쉬운 알림 빌더, 풀-기능 알림 빌더, 그리고 탐색형 데이터 탐색기 대시보드를 제공한다. 안전한 포맷팅 헬퍼와 Slack 커스텀 알림 액션을 함께 제공하며, Python 런타임 의존성을 사전 번들하여 외부 네트워크가 없는(air-gapped) Splunk 환경에서도 동작한다.

A production-grade Splunk add-on that delivers a unified Alert Management Dashboard, a guided Easy Alert Builder, a feature-complete Alert Builder, and an exploratory Data Explorer Dashboard. It ships with a safe-formatting helper and a Slack custom alert action, and bundles Python runtime dependencies so it runs on air-gapped Splunk deployments.

> README 요약은 한국어 우선으로 작성되었으며, 운영 표·빠른 흐름·각 절의 절 제목은 한·영 이중 표기이다.

---

## 운영 상태 (Status)

| Runtime / 실행 환경 | Status / 상태 | Owner / 책임자 | Next action / 다음 작업 |
| --- | --- | --- | --- |
| Splunk 9.x | Production | App author | `security_alert/` 폴더를 `.tar.gz`로 묶어 `security_alert.spl` 패키지를 생성한 뒤 *Manage Apps*에서 설치 |
| Splunk 8.x | Compatible | App author | 설치 후 8.x 인스턴스에서 대시보드 렌더링을 검증 |
| Air-gapped Splunk | Supported | App author | 외부 Python 패키지 다운로드 불필요 (의존성 사전 번들 완료) |
| Python 3 runtime | Bundled | App author | `security_alert/lib/python3/` 에 urllib3, charset_normalizer, idna 포함 |
| Slack custom alert action | Bundled | App author | `security_alert/default/alert_actions.conf` 에서 활성화 |
| Easy Alert Builder UI | Bundled | App author | `security_alert/default/data/ui/views/easy_alert_builder.xml` |
| Data Explorer Dashboard | Bundled | App author | `security_alert/default/data/ui/views/data-explorer-dashboard.xml` |
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

## 목차 (Table of Contents)

1. [패키지 구성 (Package Contents)](#1-패키지-구성-package-contents)
2. [첫 번째로 읽을 파일 (First Files to Read)](#2-첫-번째로-읽을-파일-first-files-to-read)
3. [아키텍처 (Architecture)](#3-아키텍처-architecture)
4. [진입 지점·API (Entry Points and API)](#4-진입-지점api-entry-points-and-api)
5. [빠른 시작 (Quickstart)](#5-빠른-시작-quickstart)
6. [구성 (Configuration)](#6-구성-configuration)
7. [명령어 참조 (Commands Reference)](#7-명령어-참조-commands-reference)
8. [로컬 개발 (Local Development)](#8-로컬-개발-local-development)
9. [테스트 (Testing)](#9-테스트-testing)
10. [기여 안내 (Contributing)](#10-기여-안내-contributing)
11. [운영자 (Maintainers)](#11-운영자-maintainers)
12. [추가 문서 (Further Documentation)](#12-추가-문서-further-documentation)
13. [라이선스 (License)](#13-라이선스-license)

---

## 1. 패키지 구성 (Package Contents)

저장소 최상위 디렉터리는 운영 문서(`docs/`, `resume/`)와 실제 Splunk 앱 디렉터리(`security_alert/`)로 구성된다. 데모 스크립트는 `demo/` 에, 알림 저장소는 `docs/ALERT-REPOSITORY-XWIKI.md` 에 정리되어 있다.

| Path | 역할 / Role |
| --- | --- |
| `CONTRIBUTING.md` | 기여 규칙 / Contribution policy |
| `LICENSE` | 라이선스 전문 / License text |
| `README.md` | 본 문서 / This document |
| `resume/` | 운영 이력·아키텍처·트러블슈팅 문서 모음 |
| `docs/` | 빠른 시작·배포·릴리스 노트·레거시 정리 보고서 |
| `demo/` | 데모 스크립트 및 시연 자료 |
| `security_alert/` | 실제 Splunk 앱 디렉터리 (배포 단위) |

`security_alert/` 디렉터리는 Splunk 가 인식하는 표준 레이아웃이다.

| Path | 역할 / Role |
| --- | --- |
| `security_alert/app.manifest` | 앱 메타데이터 / App manifest |
| `security_alert/metadata/default.meta` | 액세스 제어 메타 / Access control metadata |
| `security_alert/default/app.conf` | 앱 정의 / App identity, label, version |
| `security_alert/default/alert_actions.conf` | Slack 등 커스텀 알림 액션 / Custom alert actions |
| `security_alert/default/savedsearches.conf` | 저장된 검색 / Predefined saved searches |
| `security_alert/default/transforms.conf` | 필드 변환 / Field transforms |
| `security_alert/default/props.conf` | 필드 추출·인덱싱 규칙 / Field extraction rules |
| `security_alert/default/macros.conf` | 재사용 검색 매크로 / Search macros |
| `security_alert/default/data/ui/nav/default.xml` | 네비게이션 메뉴 / Sidebar navigation |
| `security_alert/default/data/ui/views/*.xml` | 대시보드·빌더 뷰 / Dashboard and builder views |
| `security_alert/bin/safe_fmt.py` | 안전한 포맷팅 헬퍼 / Safe formatting helper |
| `security_alert/bin/slack.py` | Slack 커스텀 알림 액션 스크립트 / Slack custom alert action |
| `security_alert/bin/six.py` | Python 2/3 호환 헬퍼 / Python compatibility helper |
| `security_alert/lib/python3/` | 사전 번들된 의존성 (urllib3, charset_normalizer, idna) / Bundled deps |

---

## 2. 첫 번째로 읽을 파일 (First Files to Read)

운영자가 우선 확인해야 할 문서와 진입 파일은 다음과 같다.

| 우선순위 / Priority | 파일 / File | 확인 목적 / Why read |
| --- | --- | --- |
| 1 | `docs/QUICK-START.md` | 5분 내 설치를 위한 최소 절차 / Minimal install steps |
| 2 | `docs/DEPLOYMENT.md` | 표준·오프라인 배포 절차 / Standard / offline deployment |
| 3 | `docs/RELEASE-NOTES.md` | 변경 이력·호환성 / Changelog and compatibility |
| 4 | `docs/LEGACY-CLEANUP-REPORT.md` | 정리된 레거시 항목과 결정 / Legacy cleanup record |
| 5 | `docs/ALERT-REPOSITORY-XWIKI.md` | 알림 규칙 카탈로그 / Alert rule catalog |
| 6 | `resume/ARCHITECTURE.md` | 앱 아키텍처·모듈 경계 / App architecture |
| 7 | `resume/API.md` | 대시보드·액션 노출 인터페이스 / Exposed interfaces |
| 8 | `resume/DEPLOYMENT.md` | 배포 운영 메모 / Ops notes for deployment |
| 9 | `resume/TROUBLESHOOTING.md` | 장애 진단 / Incident diagnosis |
| 10 | `security_alert/app.manifest` | 앱 버전·작성자·라이선스 선언 / Version, author, license |
| 11 | `security_alert/default/app.conf` | 앱 라벨·가시성 / App label and visibility |

---

## 3. 아키텍처 (Architecture)

### 3.1 요청 흐름 (Request flow)

1. **Operator / 분석가** 가 Splunk Web 에 로그인한 뒤 사이드바에서 *Security Alert* 항목을 선택한다.
2. **Navigation layer** — `security_alert/default/data/ui/nav/default.xml` 이 대시보드와 빌더를 노출한다.
3. **Dashboard view** — `easy_alert_builder.xml`, `alert_builder.xml`, `alert-management-dashboard.xml`, `data-explorer-dashboard.xml` 중 하나가 렌더링된다.
4. **Search engine** — `savedsearches.conf`, `macros.conf`, `props.conf`, `transforms.conf` 의 정의를 사용해 SPL 을 평가한다.
5. **Alert trigger** — 매칭된 이벤트는 `alert_actions.conf` 의 액션으로 라우팅된다.
6. **Custom action script** — `bin/slack.py` 가 `bin/safe_fmt.py` 로 메시지를 정제한 뒤 Slack webhook 으로 POST 한다.
7. **Bundled runtime** — `lib/python3/` 의 urllib3 / charset_normalizer / idna 가 외부 네트워크 없이 HTTP 호출을 처리한다.
8. **Triage & explore** — 분석가는 *Alert Management Dashboard* 에서 분류(triage)하고, *Data Explorer Dashboard* 에서 심층 분석한다.

### 3.2 모듈 경계 (Module boundaries)

| 계층 / Layer | 위치 / Location | 책임 / Responsibility |
| --- | --- | --- |
| Presentation / 대시보드 | `security_alert/default/data/ui/views/` | UI 렌더링과 입력 수집 / Render UI and collect input |
| Navigation / 네비게이션 | `security_alert/default/data/ui/nav/default.xml` | 메뉴 노출 / Expose menu entries |
| Configuration / 설정 | `security_alert/default/*.conf` | 앱·검색·액션·필드 규칙 / App / search / action / field rules |
| Custom action / 액션 | `security_alert/bin/slack.py`, `bin/safe_fmt.py`, `bin/six.py` | 외부 시스템 호출·안전한 출력 / External calls, safe output |
| Bundled deps / 의존성 | `security_alert/lib/python3/` | 오프라인 환경 의존성 / Offline runtime deps |
| Access control / 권한 | `security_alert/metadata/default.meta` | 객체 권한 / Object permissions |
| Identity / 식별 | `security_alert/app.manifest`, `default/app.conf` | 앱 메타데이터 / App metadata |

---

## 4. 진입 지점·API (Entry Points and API)

| 진입 지점 / Entry point | 종류 / Type | 기본 위치 / Default location |
| --- | --- | --- |
| Alert Management Dashboard | Splunk dashboard view | `security_alert/default/data/ui/views/alert-management-dashboard.xml` |
| Easy Alert Builder | Guided dashboard view | `security_alert/default/data/ui/views/easy_alert_builder.xml` |
| Alert Builder | Full dashboard view | `security_alert/default/data/ui/views/alert-builder.xml` |
| Data Explorer Dashboard | Exploratory view | `security_alert/default/data/ui/views/data-explorer-dashboard.xml` |
| Slack custom alert action | Alert action script | `security_alert/bin/slack.py` (설정: `alert_actions.conf`) |
| Safe formatting helper | Python module | `security_alert/bin/safe_fmt.py` |
| Python 2/3 compat shim | Python module | `security_alert/bin/six.py` |
| App identity | Splunk conf | `security_alert/default/app.conf`, `security_alert/app.manifest` |

상세 인터페이스 표(`inputs`, `outputs`, `error mode`)는 [resume/API.md](resume/API.md) 를 참조한다.

---

## 5. 빠른 시작 (Quickstart)

### 5.1 사전 요구 사항 (Prerequisites)

| 항목 / Item | 권장 / Recommended |
| --- | --- |
| Splunk Enterprise | 9.x (호환: 8.x) / 9.x (compatible: 8.x) |
| Python runtime | Splunk 번들 Python 3 / Splunk bundled Python 3 |
| 네트워크 | 에어갭 호환 (외부 다운로드 불필요) / Air-gap friendly |
| Splunk 권한 | `admin` 또는 `power` 롤 / `admin` or `power` role |

### 5.2 패키지 빌드 (Build the package)

```bash
tar -czf security_alert.spl security_alert/
```

### 5.3 설치 (Install)

Splunk Web 경로: *Apps → Manage Apps → Install app from file* 에서 `security_alert.spl` 을 업로드한다.

수동 경로:

```bash
# 예시 (운영 인스턴스 경로로 교체)
tar -xzf security_alert.spl -C "$SPLUNK_HOME/etc/apps/"
"$SPLUNK_HOME/bin/splunk" restart
```

상세 절차는 [docs/QUICK-START.md](docs/QUICK-START.md) 와 [docs/DEPLOYMENT.md](docs/DEPLOYMENT.md) 를 참조한다.

---

## 6. 구성 (Configuration)

### 6.1 Slack 알림 액션

| 키 / Key | 위치 / Location | 의 미 / Meaning |
| --- | --- | --- |
| `custom_actions` | `security_alert/default/alert_actions.conf` | 액션 노출 여부 / Whether the action is exposed |
| `param.webhook_url` | 동 상 / same | Slack incoming webhook URL |
| `param.channel` | 동 상 / same | 기본 채널 / Default channel |
| `param.username` | 동 상 / same | 표시 이름 / Display name |
| `param.icon_emoji` | 동 상 / same | 이모지 아이콘 / Emoji icon |

### 6.2 검색·필드 규칙

| 파일 / File | 의 미 / Meaning |
| --- | --- |
| `savedsearches.conf` | 사전 정의 알림 검색 / Predefined alert searches |
| `props.conf` | 필드 추출·타임스탬프 규칙 / Field extraction, timestamp rules |
| `transforms.conf` | 인덱스 별 변환 룰 / Per-index transforms |
| `macros.conf` | 재사용 SPL 매크로 / Reusable SPL macros |

### 6.3 권한 (Permissions)

`security_alert/metadata/default.meta` 가 객체 권한을 정의한다. 변경 시 Splunk `btool` 로 검증한다.

```bash
"$SPLUNK_HOME/bin/splunk" btool check --app=security_alert
```

---

## 7. 명령어 참조 (Commands Reference)

| 명령 / Command | 의 미 / Meaning |
| --- | --- |
| `tar -czf security_alert.spl security_alert/` | 앱 패키징 / Package the app |
| `"$SPLUNK_HOME/bin/splunk" restart` | 앱 활성화 / Activate the app |
| `"$SPLUNK_HOME/bin/splunk" btool check --app=security_alert` | 설정 검증 / Validate configs |
| `"$SPLUNK_HOME/bin/splunk" reload deploy-server` | (배포 서버) 설정 재적재 / (Deployment server) reload |
| `python3 security_alert/bin/safe_fmt.py --help` | 포맷팅 헬퍼 사용법 / Helper usage |
| `python3 security_alert/bin/slack.py --help` | Slack 액션 dry-run / Slack action dry-run |

예시 출력과 환경 변수는 [docs/DEPLOYMENT.md](docs/DEPLOYMENT.md) 와 [resume/DEPLOYMENT.md](resume/DEPLOYMENT.md) 를 참조한다.

---

## 8. 로컬 개발 (Local Development)

### 8.1 심볼릭 링크 설치 (Symlink install)

변경 사항을 빠르게 반영하려면 Splunk 앱 디렉터리에 심볼릭 링크를 건다.

```bash
ln -s "$(pwd)/security_alert" "$SPLUNK_HOME/etc/apps/security_alert"
"$SPLUNK_HOME/bin/splunk" restart
```

### 8.2 디렉터리 규칙 (Directory conventions)

| 추가 / Add | 위치 / Path | 의 미 / Meaning |
| --- | --- | --- |
| 새 대시보드 | `security_alert/default/data/ui/views/` | UI 확장 / Extend UI |
| 새 알림 액션 스크립트 | `security_alert/bin/` | 외부 시스템 연동 / External integration |
| 새 의존성 | `security_alert/lib/python3/` (sub-folder) | 오프라인 의존성 / Offline deps |
| 새 매크로 | `security_alert/default/macros.conf` | SPL 재사용 / Reusable SPL |

### 8.3 코드 스타일

- Python: PEP 8, Splunk SDK 스타일 가이드 준수.
- XML 대시보드: 들여쓰기 2칸, 의미 있는 `id` 사용.
- 설정 파일: 키 정렬, 섹션별 주석 유지.

자세한 규칙은 [CONTRIBUTING.md](CONTRIBUTING.md) 를 참조한다.

---

## 9. 테스트 (Testing)

| 종류 / Type | 도구 / Tool | 위치 / Location | 명령 예 / Command example |
| --- | --- | --- | --- |
| 설정 검증 / Config check | Splunk `btool` | 설치된 앱 | `"$SPLUNK_HOME/bin/splunk" btool check --app=security_alert` |
| 헬퍼 단위 / Helper unit | Python `unittest` | `demo/` 또는 `bin/` | `python3 -m unittest discover demo` |
| 알림 액션 dry-run | `bin/slack.py` | `bin/` | `python3 security_alert/bin/slack.py --dry-run` |
| 대시보드 렌더링 / Render | 수동 또는 Splunk UI 검사 | `data/ui/views/` | Splunk Web 에서 각 뷰 로드 |
| 호환성 / Compatibility | Splunk 8.x / 9.x 인스턴스 | 별도 검증 환경 | 각각 새로 고침 |

트러블슈팅 흐름은 [resume/TROUBLESHOOTING.md](resume/TROUBLESHOOTING.md) 를 참조한다.

---

## 10. 기여 안내 (Contributing)

1. 이슈 트래커에서 변경 범위를 먼저 논의한다.
2. 기능 브랜치에서 작업한다 (예: `feature/<short-name>`).
3. PR 본문에 영향 받는 대시보드 / 액션 / 설정을 명시한다.
4. 커밋 메시지는 변경 의도를 짧게 요약한다.
5. `btool` 통과와 수동 렌더링 확인 후 PR 을 제출한다.

자세한 규칙: [CONTRIBUTING.md](CONTRIBUTING.md).

---

## 11. 운영자 (Maintainers)

| 역할 / Role | 책임 / Responsibility | 위치 / Pointer |
| --- | --- | --- |
| App author | 앱 개발·릴리스 / Develop and release | `security_alert/app.manifest` |
| Splunk admin | 배포·업그레이드·권한 / Deploy, upgrade, permission | 내부 운영 채널 / internal ops |
| SOC analysts | 알림 작성·분류 / Author and triage alerts | `alert_builder.xml`, `alert-management-dashboard.xml` |
| Detection engineers | 검색·매크로 정비 / Maintain searches and macros | `savedsearches.conf`, `macros.conf` |

신고 채널 및 에스컬레이션 절차는 내부 운영 정책에 따른다.

---

## 12. 추가 문서 (Further Documentation)

| 문서 / Document | 의 미 / Meaning |
| --- | --- |
| [docs/QUICK-START.md](docs/QUICK-START.md) | 5분 설치 가이드 / 5-minute install guide |
| [docs/DEPLOYMENT.md](docs/DEPLOYMENT.md) | 표준·오프라인 배포 절차 / Standard / offline deployment |
| [docs/RELEASE-NOTES.md](docs/RELEASE-NOTES.md) | 릴리스 노트 / Release notes |
| [docs/LEGACY-CLEANUP-REPORT.md](docs/LEGACY-CLEANUP-REPORT.md) | 레거시 정리 보고서 / Legacy cleanup report |
| [docs/ALERT-REPOSITORY-XWIKI.md](docs/ALERT-REPOSITORY-XWIKI.md) | 알림 규칙 카탈로그 / Alert rule catalog |
| [resume/API.md](resume/API.md) | 노출 인터페이스 표 / Exposed interface table |
| [resume/ARCHITECTURE.md](resume/ARCHITECTURE.md) | 아키텍처 세부 / Detailed architecture |
| [resume/DEPLOYMENT.md](resume/DEPLOYMENT.md) | 배포 운영 메모 / Deployment ops notes |
| [resume/TROUBLESHOOTING.md](resume/TROUBLESHOOTING.md) | 트러블슈팅 / Troubleshooting |
| [demo/README.md](demo/README.md) | 데모 스크립트 / Demo scripts |

---

## 13. 라이선스 (License)

본 저장소는 [LICENSE](LICENSE) 파일에 명시된 조건 하에 배포된다. 앱 자체의 라이선스 선언은 `security_alert/app.manifest` 및 `security_alert/default/app.conf` 의 `license` 필드를 참조한다.

> 번들된 서드파티 의존성(`security_alert/lib/python3/` 아래 urllib3, charset_normalizer, idna 등)은 각 원 프로젝트의 라이선스를 따른다. 자세한 내용은 `lib/python3/*/licenses/` 하위 파일을 참조한다.