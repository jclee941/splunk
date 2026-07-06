# Security Alert (Splunk App) / 보안 알림 (Splunk 앱)

![Splunk 9.x](https://img.shields.io/badge/Splunk-9.x-orange?logo=splunk) ![Python 3](https://img.shields.io/badge/Python-3.x-3776AB?logo=python&logoColor=white) ![Air--gapped](https://img.shields.io/badge/Air--gapped-Supported-2E7D32) ![Maintenance](https://img.shields.io/badge/Maintenance-Active-blue)

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

## 빠른 흐름 요약 (Quick-flow summary)

1. **Build the package** — `security_alert/` 폴더를 `.tar.gz` 로 묶어 `security_alert.spl` 로 패키징한다.
2. **Install the package** — Splunk Web 의 *Manage Apps* 화면에서 업로드하거나 `$SPLUNK_HOME/etc/apps/` 에 압축 해제한다.
3. **Restart Splunk** — (필요 시) `splunk restart` 로 앱을 활성화한다.
4. **Configure routes** — `security_alert/default/alert_actions.conf` 에서 Slack 알림 액션을 활성화하고 webhook 을 등록한다.
5. **Author alerts** — **Easy Alert Builder** 또는 **Alert Builder** 대시보드 뷰에서 알림을 작성한다.
6. **Operate alerts** — **Alert Management Dashboard** 에서 분류(triage)하고, **Data Explorer Dashboard** 에서 심층 분석한다.

## 목차 (Table of contents)

- [1. 목적 (Purpose)](#1-목적-purpose)
- [2. 패키지 구성 (Package contents)](#2-패키지-구성-package-contents)
- [3. 상태 (Status)](#3-상태-status)
- [4. 먼저 읽을 파일 (First files to read)](#4-먼저-읽을-파일-first-files-to-read)
- [5. 엔트리 포인트 (Entry points)](#5-엔트리-포인트-entry-points)
- [6. 빠른 시작 (Quickstart)](#6-빠른-시작-quickstart)
- [7. 구성 (Configuration)](#7-구성-configuration)
- [8. 명령어 (Commands reference)](#8-명령어-commands-reference)
- [9. 로컬 개발 (Local development)](#9-로컬-개발-local-development)
- [10. 테스트 (Testing)](#10-테스트-testing)
- [11. 기여 가이드 (Contributing)](#11-기여-가이드-contributing)
- [12. 관리자 및 문의 (Maintainers / Contact)](#12-관리자-및-문의-maintainers--contact)
- [13. 추가 문서 (Further documentation)](#13-추가-문서-further-documentation)
- [14. 라이선스 (License)](#14-라이선스-license)

---

## 1. 목적 (Purpose)

`security_alert/` 는 Splunk 환경에서 임시 알림 작성을 반복 가능하고 감사 가능한 워크플로우로 전환하는 자급식(self-contained) Splunk 앱이다. 탐지 엔지니어, SOC 분석가, 그리고 Splunk 관리자가 단일 진입점에서 알림을 설계·배포·검토할 수 있도록 다음 네 가지 사용 시나리오를 지원한다.

`security_alert/` is a self-contained Splunk app that turns ad-hoc alert authoring into a repeatable, auditable workflow. It supports four primary use cases for detection engineers, SOC analysts, and Splunk administrators from a single entry point.

| Use case / 사용 시나리오 | Entry surface / 진입점 | Operator outcome / 결과 |
| --- | --- | --- |
| Guided authoring | Easy Alert Builder (손쉬운 알림 빌더) | 템플릿 기반으로 신규 알림을 빠르게 생성 |
| Full authoring | Alert Builder (알림 빌더) | 스케줄, 임계값, 알림 액션을 세밀하게 구성 |
| Triage and review | Alert Management Dashboard (알림 관리 대시보드) | 발생 알림을 한 화면에서 분류하고 후속 조치 |
| Exploratory analysis | Data Explorer Dashboard (데이터 탐색기 대시보드) | 데이터 분포, 상관관계, 이상 징후를 자유롭게 탐색 |

이 앱은 Python 3 런타임 의존성(urllib3, charset_normalizer, idna)을 사전 번들하므로, 폐쇄망(air-gapped) Splunk 환경에서도 별도 패키지 다운로드 없이 동작한다.

**누가 쓰는가 (Who uses it).** Detection engineering 팀, SOC 분석가, Splunk 플랫폼 관리자, 그리고 사내 보안 운영 거버넌스를 책임지는 팀이 주 사용자다. 알림 작성·검토·분석·전달의 전 과정을 한 앱 안에서 수행할 수 있다.

## 2. 패키지 구성 (Package contents)

저장소 최상위에는 `security_alert/` Splunk 앱과 운영·이력 문서 자산이 함께 들어 있다. 패키징 산출물은 `security_alert/` 디렉터리에서 빌드하는 `security_alert.spl` 하나다.

| Path / 경로 | Purpose / 용도 |
| --- | --- |
| `security_alert/app.manifest` | 앱 이름·버전·작성자 등 메타데이터 |
| `security_alert/metadata/default.meta` | 객체 단위 권한 및 가시성 제어 |
| `security_alert/default/app.conf` | 앱 자체 설정 (라벨, 네비게이션 노출 여부 등) |
| `security_alert/default/alert_actions.conf` | Slack 등 커스텀 알림 액션 정의 |
| `security_alert/default/macros.conf` | 재사용 검색 매크로 |
| `security_alert/default/props.conf` | 필드 추출·타임스탬프 규칙 |
| `security_alert/default/savedsearches.conf` | 알림이 되는 저장된 검색 목록 |
| `security_alert/default/transforms.conf` | 필드 변환 룰 |
| `security_alert/default/data/ui/nav/default.xml` | Splunk Web 좌측 네비게이션 |
| `security_alert/default/data/ui/views/alert-builder.xml` | 풀-기능 알림 빌더 대시보드 |
| `security_alert/default/data/ui/views/alert-management-dashboard.xml` | 알림 관리(트리이지) 대시보드 |
| `security_alert/default/data/ui/views/data-explorer-dashboard.xml` | 데이터 탐색기 대시보드 |
| `security_alert/default/data/ui/views/easy_alert_builder.xml` | 손쉬운 알림 빌더 대시보드 |
| `security_alert/bin/safe_fmt.py` | 안전한 문자열 포맷팅 헬퍼 (XSS 방지 등) |
| `security_alert/bin/slack.py` | Slack 인바운드 webhook 알림 액션 |
| `security_alert/bin/six.py` | Python 2/3 호환 셔(shim) |
| `security_alert/lib/python3/urllib3/` | 사전 번들된 urllib3 (air-gapped 지원) |
| `security_alert/lib/python3/charset_normalizer-3.4.4.dist-info/` | 사전 번들된 charset_normalizer 메타데이터 |
| `security_alert/lib/python3/idna-3.11.dist-info/` | 사전 번들된 idna 메타데이터 |
| `security_alert/README.md` | 앱 내부 보조 문서 |
| `docs/QUICK-START.md` | 설치 및 첫 알림 작성 절차 |
| `docs/DEPLOYMENT.md` | 배포 절차 |
| `docs/RELEASE-NOTES.md` | 릴리스 노트 |
| `docs/LEGACY-CLEANUP-REPORT.md` | 레거시 정리 보고서 |
| `docs/ALERT-REPOSITORY-XWIKI.md` | 보고·협력 절차 |
| `resume/API.md` | 앱이 노출하는 스크립트/시그니처 정리 |
| `resume/ARCHITECTURE.md` | 컴포넌트 의존성·데이터 흐름 |
| `resume/DEPLOYMENT.md` | 배포 환경별 권장값 |
| `resume/TROUBLESHOOTING.md` | 자주 발생하는 장애와 해결 |
| `demo/README.md` | 데모/시연 절차 |
| `CONTRIBUTING.md` | 기여 절차 |
| `LICENSE` | 라이선스 전문 |

## 3. 상태 (Status)

- **Production-ready** for Splunk 9.x. 작성·트리이지·분석 전 영역이 검증된 상태로 운영된다.
- **Compatible** with Splunk 8.x. 설치 직후 대시보드 렌더링 한 차례를 수동 확인하는 것을 권장한다.
- **No upstream network requirement.** 모든 Python 의존성은 `security_alert/lib/python3/` 아래에 사전 번들되어 있어 폐쇄망에서도 동일하게 동작한다.
- **Active maintenance.** 자세한 변경 이력은 [`docs/RELEASE-NOTES.md`](docs/RELEASE-NOTES.md), 정리된 레거시 자산 목록은 [`docs/LEGACY-CLEANUP-REPORT.md`](docs/LEGACY-CLEANUP-REPORT.md) 를 참조한다.
- **Not deprecated.** 현 시점의 타겟은 Splunk 9.x 베이스라인이며, 버그 수정 전용 모드로 전환된 적 없다.

## 4. 먼저 읽을 파일 (First files to read)

운영자가 가장 먼저 살펴봐야 할 파일은 다음과 같다. 위에서부터 순서대로 읽으면 패키지의 형태와 실행 경로를 빠르게 파악할 수 있다.

| Order | File | Why / 이유 |
| --- | --- | --- |
| 1 | [`security_alert/default/app.conf`](security_alert/default/app.conf) | 앱 이름·라벨·네비게이션 노출 여부 확인 |
| 2 | [`security_alert/app.manifest`](security_alert/app.manifest) | 앱 버전·작성자·필수 Splunk 버전 |
| 3 | [`security_alert/default/alert_actions.conf`](security_alert/default/alert_actions.conf) | 활성화된 알림 액션(특히 Slack) 정의 |
| 4 | [`security_alert/default/savedsearches.conf`](security_alert/default/savedsearches.conf) | 부트스트랩 시 등록되는 알림 목록 |
| 5 | [`security_alert/default/data/ui/nav/default.xml`](security_alert/default/data/ui/nav/default.xml) | UI 진입점 구조 |
| 6 | [`security_alert/bin/slack.py`](security_alert/bin/slack.py) | Slack webhook 발송 경로 |
| 7 | [`docs/QUICK-START.md`](docs/QUICK-START.md) | 설치와 첫 알림 작성 절차 |

## 5. 엔트리 포인트 (Entry points)

이 앱은 단일 백엔드 패키지(`security_alert.spl`)에 다섯 개의 진입점을 노출한다. 모두 Splunk Web 과 저장된 검색 호출 경로로 접근한다.

| Entry | Kind | Path / 파일 | 인증 요구 |
| --- | --- | --- | --- |
| Easy Alert Builder | Dashboard | [`security_alert/default/data/ui/views/easy_alert_builder.xml`](security_alert/default/data/ui/views/easy_alert_builder.xml) | Splunk 로그인 |
| Alert Builder | Dashboard | [`security_alert/default/data/ui/views/alert-builder.xml`](security_alert/default/data/ui/views/alert-builder.xml) | Splunk 로그인 |
| Alert Management Dashboard | Dashboard | [`security_alert/default/data/ui/views/alert-management-dashboard.xml`](security_alert/default/data/ui/views/alert-management-dashboard.xml) | Splunk 로그인 |
| Data Explorer Dashboard | Dashboard | [`security_alert/default/data/ui/views/data-explorer-dashboard.xml`](security_alert/default/data/ui/views/data-explorer-dashboard.xml) | Splunk 로그인 |
| Slack 알림 액션 | Custom alert action | [`security_alert/bin/slack.py`](security_alert/bin/slack.py) | webhook URL 구성 필요 |

작성 워크플로우는 다음 흐름을 따른다.

1. 운영자가 `Easy Alert Builder` 또는 `Alert Builder` 로 진입한다.
2. 검색 조건·임계값·스케줄이 저장된 검색(`savedsearches.conf`)에 기록된다.
3. 저장된 검색의 트리거 시점에 `slack.py` 가 호출되어 Slack 채널로 메시지를 전송한다.
4. 발생 알림은 `Alert Management Dashboard` 에서 분류하고, `Data Explorer Dashboard` 에서 추가 분석한다.

## 6. 빠른 시작 (Quickstart)

폐쇄망 Splunk 인스턴스에서도 동일하게 동작하도록 모든 의존성을 사전 번들한 패키지 빌드 방식을 따른다.

### 6.1 패키지 빌드 (Build)

```bash
# 저장소 루트에서 실행
cd security_alert
tar -czf ../security_alert.spl .
cd ..
ls -lh security_alert.spl
```

### 6.2 패키지 설치 (Install)

GUI 경로:
1. Splunk Web 로그인 → *Settings → Apps → Manage Apps*.
2. **Install app from file** 로 `security_alert.spl` 업로드.

CLI 경로:
```bash
# Splunk 호스트에서 실행
cp security_alert.spl "$SPLUNK_HOME/etc/apps/"
tar -xzf "$SPLUNK_HOME/etc/apps/security_alert.spl" -C "$SPLUNK_HOME/etc/apps/"
splunk restart
```

### 6.3 알림 액션 구성 (Configure Slack)

`security_alert/default/alert_actions.conf` 를 그대로 두거나, 환경별 차이는 동일 경로의 `local/alert_actions.conf` 에서 오버라이드한다.

```ini
[slack]
enabled = 1
webhook_url = https://hooks.slack.com/services/<TEAM>/<CHANNEL>/<TOKEN>
channel = #sec-alerts
username = Splunk
icon_emoji = :warning:
```

### 6.4 첫 알림 작성 (Author your first alert)

1. Splunk Web 좌측 네비게이션에서 **Security Alert** → **Easy Alert Builder** 선택.
2. 인덱스·소스·임계값을 입력하고 저장.
3. **Manage Alerts** 에서 트리거 여부 확인.
4. **Alert Management Dashboard** 에서 분류 메모를 남긴다.

## 7. 구성 (Configuration)

모든 설정 파일은 `$SPLUNK_HOME/etc/apps/security_alert/default/` 하위에 있다. 환경별 차이는 동일 경로의 `local/` 디렉터리에서 오버라이드한다.

| File | Controls / 제어 대상 |
| --- | --- |
| `app.conf` | 앱 라벨, 권한, 네비게이션 노출 |
| `alert_actions.conf` | Slack 등 알림 액션의 활성화 여부와 webhook |
| `savedsearches.conf` | 부트스트랩 시 등록되는 알림(저장된 검색) |
| `macros.conf` | 검색 단계에서 재사용하는 매크로 |
| `props.conf` | 인덱서·검색 시점 필드 처리 |
| `transforms.conf` | 필드 변환 룰 |
| `metadata/default.meta` | 객체별 ACL (`capabilities`) |

### 7.1 권한 (Capabilities)

대시보드 자체의 접근 제어는 Splunk의 일반 역할(role) 메커니즘을 따른다. 알림 액션 객체 단위의 ACL 은 `metadata/default.meta` 의 `capabilities` 설정으로 보호된다. 자세한 매핑은 [`security_alert/metadata/default.meta`](security_alert/metadata/default.meta) 를 직접 확인한다.

### 7.2 의존성 (Dependencies)

Python 의존성은 외부 PyPI에 연결하지 않는다. 폐쇄망에서도 아래만으로 동작한다.

| Library | Version | Source |
| --- | --- | --- |
| urllib3 | bundled | [`security_alert/lib/python3/urllib3/`](security_alert/lib/python3/urllib3/) |
| charset_normalizer | 3.4.4 | [`security_alert/lib/python3/charset_normalizer-3.4.4.dist-info/`](security_alert/lib/python3/charset_normalizer-3.4.4.dist-info/) |
| idna | 3.11 | [`security_alert/lib/python3/idna-3.11.dist-info/`](security_alert/lib/python3/idna-3.11.dist-info/) |

## 8. 명령어 (Commands reference)

운영자가 일상적으로 사용하는 Splunk CLI 와 빌드 명령은 다음과 같다.

| Command | 용도 |
| --- | --- |
| `splunk restart` | 앱 로드/재로드를 위해 Splunk 전체 재시작 |
| `splunk reload deploy-server` | (선택) 배포 서버에서 앱 강제 재배포 |
| `splunk search '\| savedsearch "<name>"' -auth <user>:<pw>` | 저장된 검색을 즉석에서 시험 실행 |
| `tar -czf security_alert.spl -C security_alert .` | 패키지 재빌드 |
| `splunk cmd python etc/apps/security_alert/bin/safe_fmt.py "..."` | 포맷팅 헬퍼 수동 호출 (디버깅) |
| `splunk cmd python etc/apps/security_alert/bin/slack.py` | Slack 알림 액션 dry-run |

## 9. 로컬 개발 (Local development)

### 9.1 심볼릭 링크 워크플로우 (Symlink workflow)

```bash
# macOS / Linux
ln -sfn "$(pwd)/security_alert" "$SPLUNK_HOME/etc/apps/security_alert"
splunk restart
```

소스 수정 즉시 Splunk 가 변경을 인식한다. 대시보드 XML 캐시 갱신을 위해 브라우저에서 `?refresh=1` 을 붙여 재로드한다.

### 9.2 Python 헬퍼 디버깅

`security_alert/bin/` 의 스크립트는 Splunk 의 제한된 `PYTHONPATH` 위에서 동작한다. 로컬 디버깅 시 다음 환경 변수를 설정한다.

```bash
export PYTHONPATH="$(pwd)/security_alert/lib/python3:$(pwd)/security_alert/bin"
python3 security_alert/bin/safe_fmt.py "test <b>payload</b>"
python3 security_alert/bin/slack.py --help
```

### 9.3 대시보드 XML 작성 규칙

- 신규 대시보드는 `security_alert/default/data/ui/views/<name>.xml` 경로에 저장한다.
- 네비게이션 노출은 [`security_alert/default/data/ui/nav/default.xml`](security_alert/default/data/ui/nav/default.xml) 에 항목을 추가한다.
- 권한·ACL 토큰은 [`security_alert/metadata/default.meta`](security_alert/metadata/default.meta) 의 `[]` 블록에서 함께 갱신한다.

## 10. 테스트 (Testing)

이 저장소는 표준 `pytest` 와 `splunk-appinspect` 를 권장한다. CI는 별도 정의되어 있지 않으므로 변경 PR 제출 전 아래 4개 레이어를 로컬에서 모두 통과시켜야 한다.

| Layer | Tool | 명령어 |
| --- | --- | --- |
| Python helpers | pytest | `pytest security_alert/bin/` |
| App packaging | splunk-appinspect | `splunk-appinspect inspect security_alert.spl` |
| Savedsearch load | curl + splunkd REST | `curl -k -u admin:pw https://<host>:8089/services/saved/searches` |
| Slack 액션 | 수동 트리거 | 작성한 알림을 `\| savedsearch` 로 1회 실행 후 webhook 도착 확인 |

## 11. 기여 가이드 (Contributing)

기여 절차와 PR 규칙은 [`CONTRIBUTING.md`](CONTRIBUTING.md) 를 따른다. 요약하면 다음과 같다.

1. 이슈 또는 작업 항목을 먼저 등록한다.
2. 기능 브랜치(`feat/<name>`, `fix/<name>`)를 분기한다.
3. 로컬에서 10. 테스트 절차의 4개 레이어를 모두 통과시킨다.
4. PR 본문에 검증 결과(테스트 출력, 앱검사 결과)를 첨부한다.

## 12. 관리자 및 문의 (Maintainers / Contact)

- **Owner** — App author (저장소 작성자). 외부 연락처는 사내 운영 정책에 따르며 본 README 에는 노출하지 않는다.
- **Issue tracker** — 저장소 내 *Issues* 탭을 사용한다.
- **Security disclosures** — 자세한 흐름은 [`docs/ALERT-REPOSITORY-XWIKI.md`](docs/ALERT-REPOSITORY-XWIKI.md) 를 참조한다.

## 13. 추가 문서 (Further documentation)

| Document | Audience / 대상 |
| --- | --- |
| [`docs/QUICK-START.md`](docs/QUICK-START.md) | 신규 운영자 — 첫 설치와 첫 알림 작성 |
| [`docs/DEPLOYMENT.md`](docs/DEPLOYMENT.md) | Splunk 관리자 — 배포·업그레이드 절차 |
| [`resume/API.md`](resume/API.md) | 작성자 — 앱이 노출하는 스크립트·시그니처 정리 |
| [`resume/ARCHITECTURE.md`](resume/ARCHITECTURE.md) | 작성자 — 컴포넌트 의존성·데이터 흐름 |
| [`resume/DEPLOYMENT.md`](resume/DEPLOYMENT.md) | 작성자 — 배포 환경별 권장값 |
| [`resume/TROUBLESHOOTING.md`](resume/TROUBLESHOOTING.md) | 운영자 — 자주 발생하는 장애와 해결 |
| [`docs/RELEASE-NOTES.md`](docs/RELEASE-NOTES.md) | 모든 사용자 — 변경 이력 |
| [`docs/LEGACY-CLEANUP-REPORT.md`](docs/LEGACY-CLEANUP-REPORT.md) | 운영자 — 정리된 레거시 자산 목록 |
| [`docs/ALERT-REPOSITORY-XWIKI.md`](docs/ALERT-REPOSITORY-XWIKI.md) | 보안팀 — 보고·협력 절차 |
| [`demo/README.md`](demo/README.md) | 발표자 — 데모 시연 절차 |

## 14. 라이선스 (License)

본 저장소는 [`LICENSE`](LICENSE) 파일에 명시된 조건 하에 배포된다. 사내·외부 배포 정책은 해당 파일을 우선 참조한다.