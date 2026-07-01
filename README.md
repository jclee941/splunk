# Security Alert (Splunk App) / 보안 알림 (Splunk 앱)

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

---

## 빠른 흐름 요약 (Quick-flow summary)

1. **Build the package** — `security_alert/` 폴더를 `.tar.gz` 로 묶어 `security_alert.spl` 로 패키징한다.
2. **Install the package** — Splunk Web 의 *Manage Apps* 화면에서 업로드하거나 `$SPLUNK_HOME/etc/apps/` 에 압축 해제한다.
3. **Restart Splunk** — (필요 시) `splunk restart` 로 앱을 활성화한다.
4. **Configure routes** — `security_alert/default/alert_actions.conf` 에서 Slack 알림 액션을 활성화하고 webhook 을 등록한다.
5. **Author alerts** — **Easy Alert Builder** 또는 **Alert Builder** 대시보드 뷰에서 알림을 작성한다.
6. **Operate alerts** — **Alert Management Dashboard** 에서 분류(triage)하고, **Data Explorer Dashboard** 에서 심층 분석한다.

---

## 1. 목적 (Purpose)

`security_alert/` 는 임시 알림 작성을 반복 가능하고 감사 가능한 워크플로우로 전환하는 자급식(self-contained) Splunk 앱이다. 탐지 엔지니어, SOC 분석가, Splunk 관리자가 한 곳에서 알림을 설계·배포·검토할 수 있도록 한다.

대상 사용자:

| User role / 사용자 역할 | Typical task / 주요 작업 |
| --- | --- |
| 탐지 엔지니어 (Detection engineer) | 일관된 형식과 검증된 메타데이터로 알림을 작성 |
| SOC 분석가 (SOC analyst) | 발송된 알림을 분류하고 후속 조치(triage) 를 수행 |
| Splunk 관리자 (Splunk admin) | 알림 자산을 안전하게 패키징하고 배포 |
| 위협 헌터 (Threat hunter) | 보안 데이터셋에서 이상 징후를 탐색 |

핵심 사용자 시나리오:

- **알림 작성 표준화** — Easy Alert Builder 의 안내형 폼으로 신규 알림을 빠르게 정의한다.
- **알림 운영 가시화** — Alert Management Dashboard 에서 상태·담당자·SLA 를 추적한다.
- **데이터 심층 분석** — Data Explorer Dashboard 에서 원시 이벤트를 자유롭게 검색한다.
- **외부 채널 연동** — `bin/slack.py` 커스텀 알림 액션을 통해 Slack 으로 알림을 전달한다.

---

## 2. 주요 기능 (Features)

- **통합 알림 관리 대시보드** (`alert-management-dashboard.xml`) — 알림의 상태, 소유자, 최근 발생 이력을 한 화면에서 추적한다.
- **안내형 손쉬운 알림 빌더** (`easy_alert_builder.xml`) — 단계별 안내로 신규 알림을 빠르게 작성한다.
- **풀-기능 알림 빌더** (`alert-builder.xml`) — 검색 조건, 평가 주기, 액션을 세밀하게 구성한다.
- **탐색형 데이터 탐색기** (`data-explorer-dashboard.xml`) — 필드·시간 범위를 자유롭게 조합해 심층 분석한다.
- **안전한 포맷팅 헬퍼** (`bin/safe_fmt.py`) — 외부 입력(웹훅 페이로드, 사용자 메시지) 의 인코딩을 안전하게 정규화한다.
- **Slack 커스텀 알림 액션** (`bin/slack.py` + `alert_actions.conf`) — Splunk 알림 트리거 시 Slack 으로 메시지를 전송한다.
- **검색 매크로 / 저장된 검색** (`macros.conf`, `savedsearches.conf`) — 재사용 가능한 탐지 로직을 모듈화한다.
- **필드 추출 / 변환 규칙** (`props.conf`, `transforms.conf`) — 원시 로그를 정규화하여 일관된 필드 체계를 보장한다.
- **앱 내비게이션** (`data/ui/nav/default.xml`) — Splunk 메인 메뉴에 대시보드 진입점을 노출한다.
- **번들된 Python 의존성** (`lib/python3/`) — urllib3, charset_normalizer, idna 를 사전 포함해 인터넷 없이도 동작한다.

---

## 3. 패키지 구성 (Package contents)

`security_alert/` 디렉터리 구조와 각 자산의 역할:

```
security_alert/
├── app.manifest                    # Splunk 앱 메타데이터 (이름, 버전, 작성자)
├── README.md                       # 앱 단위 안내문
├── bin/
│   ├── safe_fmt.py                 # 안전한 문자열 포맷팅 헬퍼
│   ├── slack.py                    # Slack 커스텀 알림 액션
│   └── six.py                      # Python 2/3 호환 shim
├── default/
│   ├── alert_actions.conf          # 커스텀 알림 액션 등록 (slack)
│   ├── app.conf                    # 앱 전역 설정 (라벨, 작성자)
│   ├── macros.conf                 # 재사용 검색 매크로
│   ├── props.conf                  # 필드 추출 / 인덱싱 규칙
│   ├── savedsearches.conf          # 저장된 알림 정의
│   └── transforms.conf             # 필드 재작성 / 룩업 변환
├── metadata/
│   └── default.meta                # 뷰·매크로 권한 메타데이터
├── data/ui/
│   ├── nav/default.xml             # Splunk 메인 메뉴 진입점
│   └── views/
│       ├── alert-builder.xml
│       ├── alert-management-dashboard.xml
│       ├── data-explorer-dashboard.xml
│       └── easy_alert_builder.xml
└── lib/python3/                    # 번들된 외부 의존성
    ├── urllib3/
    ├── charset_normalizer-3.4.4.dist-info/
    └── idna-3.11.dist-info/
```

| 자산 (Asset) | 파일 | 목적 |
| --- | --- | --- |
| 앱 메타 | `app.manifest` | Splunk가 앱을 식별하는 manifest |
| 알림 액션 | `bin/slack.py`, `default/alert_actions.conf` | 알림 발생 시 외부 시스템으로 송신 |
| 헬퍼 | `bin/safe_fmt.py` | 외부 입력의 안전한 정규화 / 인코딩 |
| 대시보드 뷰 | `data/ui/views/*.xml` | 운영자가 사용하는 4 종 화면 |
| 탐지 로직 | `default/savedsearches.conf`, `default/macros.conf` | 재사용 가능 알림·검색 정의 |
| 데이터 정규화 | `default/props.conf`, `default/transforms.conf` | 원시 이벤트 → 정규화 필드 |
| 내비게이션 | `data/ui/nav/default.xml` | Splunk 메뉴 진입점 노출 |
| 의존성 | `lib/python3/` | 인터넷이 없어도 동작하도록 사전 번들 |

---

## 4. 아키텍처 (Architecture)

운영자가 알림을 작성하고 외부 채널로 송신하기까지의 흐름:

1. **Operator** 가 Splunk Web 에 로그인해 **Easy Alert Builder** 또는 **Alert Builder** 뷰를 연다.
2. **Dashboard view** (XML) 가 `savedsearches.conf` / `macros.conf` 의 정의를 폼 형태로 노출한다.
3. **Authoring form** 으로 등록된 검색은 Splunk 의 `savedsearch` 메커니즘으로 저장된다.
4. **Splunk scheduler** 가 평가 주기에 따라 검색을 실행하고, 매칭 시 alert 를 발생시킨다.
5. **`alert_actions.conf`** 가 트리거된 alert 를 `bin/slack.py` 로 라우팅한다.
6. **`safe_fmt.py`** 가 외부 입력 문자열을 안전하게 정규화한다.
7. **`slack.py`** 가 번들된 `urllib3` / `charset_normalizer` / `idna` 를 사용해 Slack webhook 으로 POST 한다.
8. **SOC analyst** 가 **Alert Management Dashboard** 에서 알림을 분류한다.
9. **Detection engineer** 이 **Data Explorer Dashboard** 에서 동일 데이터셋을 다시 열어 원인 분석한다.

핵심 컴포넌트 매핑:

| 컴포넌트 | 위치 | 역할 |
| --- | --- | --- |
| View layer (XML) | `data/ui/views/*.xml` | 사용자 인터페이스 정의 |
| Splunk config | `default/*.conf` | 앱 동작 / 탐지 로직 정의 |
| Custom script | `bin/*.py` | 알림 송출·인코딩 헬퍼 |
| Bundled runtime | `lib/python3/` | 격리 환경에서 외부 호출 보장 |
| Navigation | `data/ui/nav/default.xml` | 메뉴 노출 |

---

## 5. 빠른 시작 (Quickstart)

### 5.1 패키징 (Packaging)

`security_alert/` 폴더를 `.spl` (Splunk 패키지) 로 묶는다. Splunk 패키지 컨벤션에 따라 압축 후 확장자만 `.spl` 로 변경한다.

```bash
cd security_alert
tar czf ../security_alert.spl .
cd ..
ls -lh security_alert.spl
```

### 5.2 설치 (Installation)

**옵션 A — Splunk Web UI**

1. Splunk Web 로그인 → *Apps* → *Manage Apps* 진입
2. *Install app from file* 클릭 → `security_alert.spl` 업로드
3. 업로드 완료 메시지 확인 → (필요 시) *Restart Splunk*

**옵션 B — 수동 배치 (air-gapped 권장)**

```bash
# Splunk 중지
$SPLUNK_HOME/bin/splunk stop

# 앱 디렉터리에 압축 해제
tar xzf security_alert.spl -C $SPLUNK_HOME/etc/apps/

# Splunk 재기동
$SPLUNK_HOME/bin/splunk start
```

### 5.3 설치 후 확인 (Post-install verification)

| 점검 항목 | 위치 / 명령 | 기대 결과 |
| --- | --- | --- |
| 앱 노출 여부 | Splunk Web → *Apps* 메뉴 | *Security Alert* 항목 표시 |
| 뷰 접근 | URL 입력 `/app/security_alert/easy_alert_builder` | 대시보드 렌더링 성공 |
| 모듈 임포트 | `$SPLUNK_HOME/bin/splunk python3 -c "import sys; sys.path.insert(0,'$SPLUNK_HOME/etc/apps/security_alert/lib/python3'); import urllib3; import charset_normalizer; import idna; print('ok')"` | `ok` 출력 |
| Slack 액션 등록 | Splunk Web → *Settings* → *Alert actions* | *slack* 액션 노출 |

### 5.4 첫 알림 작성 (Author your first alert)

1. Splunk Web → *Security Alert* → *Easy Alert Builder* 선택
2. 폼을 따라 이름, 검색, 평가 주기, 알림 채널 입력
3. *Save* 클릭 → `default/savedsearches.conf` 에 항목이 추가됨
4. *Alert Management Dashboard* 에서 방금 작성한 알림의 상태를 확인

자세한 절차는 [`docs/QUICK-START.md`](docs/QUICK-START.md) 와 [`security_alert/README.md`](security_alert/README.md) 를 참조한다.

---

## 6. 설정 (Configuration)

앱의 동작은 다음 conf 파일이 결정한다. 파일을 수정한 뒤 Splunk 를 재기동하거나, *Settings* → *Alert actions* 에서 새로 고친다.

| 설정 파일 | 역할 | 변경 빈도 | 비고 |
| --- | --- | --- | --- |
| `security_alert/default/app.conf` | 앱 라벨, 작성자, UI 설정 | 거의 없음 | 앱 식별 정보 |
| `security_alert/default/alert_actions.conf` | Slack 등 커스텀 알림 액션 등록 | 기능 추가 시 | stanzas 가 액션 단위 |
| `security_alert/default/savedsearches.conf` | 저장된 알림(검색) 정의 | 알림 추가 시 | Easy/Alert Builder 가 채움 |
| `security_alert/default/macros.conf` | 재사용 검색 매크로 | 탐지 로직 확장 시 | `define` 형태 |
| `security_alert/default/props.conf` | 필드 추출 / sourcetype 매핑 | 신규 소스 도입 시 | 인덱싱 정규화 |
| `security_alert/default/transforms.conf` | 필드 재작성·룩업 변환 | 정규화 규칙 추가 시 | `props.conf` 와 짝 |
| `security_alert/metadata/default.meta` | 뷰 / 매크로 권한 | 권한 변경 시 | 앱 / 글로벌 스코프 |

권한 참고:

| 권한 종류 | 기본값 | 위치 |
| --- | --- | --- |
| 앱 표시 | 모든 Splunk 사용자에게 노출 | Splunk role capability |
| 뷰 실행 | `metadata/default.meta` 의 stanza 별 권한에 따름 | 필요 시 역할별 조정 |
| conf 편집 | Splunk 관리자 (`admin`) | Splunk role capability |

---

## 7. 명령어 / 스크립트 (Commands & scripts)

`security_alert/bin/` 디렉터리의 실행 스크립트:

| 스크립트 | 호출 경로 | 용도 |
| --- | --- | --- |
| `safe_fmt.py` | `bin/safe_fmt.py` 또는 다른 스크립트에서 import | 외부 입력 문자열의 안전한 포맷팅·인코딩 |
| `slack.py` | Splunk alert engine 이 자동 호출 | `alert_actions.conf` 의 Slack 액션 핸들러 |
| `six.py` | import 대상 | Python 2 / 3 호환 shim |

스크립트를 Splunk 외부에서 직접 디버깅할 때는 번들된 런타임을 PYTHONPATH 에 포함한다.

```bash
export PYTHONPATH="$SPLUNK_HOME/etc/apps/security_alert/lib/python3:$PYTHONPATH"
python3 -c "from safe_fmt import *; help(safe_format)"
```

자세한 함수 시그니처는 [`resume/API.md`](resume/API.md) 에 정리되어 있다.

---

## 8. 로컬 개발 (Local development)

### 8.1 권장 개발 흐름

1. 저장소를 작업 디렉터리에 클론한다.
2. 변경할 자산(뷰·conf·스크립트) 을 식별한다.
3. `security_alert/` 를 `$SPLUNK_HOME/etc/apps/security_alert` 로 심볼릭 링크하거나, 빌드 후 설치한다.
4. Splunk Web 에서 즉시 결과를 확인한다.
5. 변경 후 *Settings* → *Alert actions* 새로 고침, 또는 Splunk 재기동한다.

### 8.2 심볼릭 링크 개발 (Development symlink)

```bash
ln -s "$(pwd)/security_alert" "$SPLUNK_HOME/etc/apps/security_alert"
$SPLUNK_HOME/bin/splunk restart
```

이 방식은 `tar` → `install` 사이클 없이 코드 수정만으로 반영된다.

### 8.3 디렉터리 컨벤션

| 영역 | 위치 | 규칙 |
| --- | --- | --- |
| 뷰 (UI) | `security_alert/data/ui/views/` | Simple XML 또는 Dashboard Studio |
| 설정 | `security_alert/default/` | 표준 Splunk conf 문법 |
| 스크립트 | `security_alert/bin/` | Python 3, 실행 권한 (`chmod +x`) |
| 의존성 | `security_alert/lib/python3/` | 반드시 앱 내부에서 import |

---

## 9. 테스트 (Testing)

| 영역 | 권장 검증 | 도구 |
| --- | --- | --- |
| 패키징 | `tar tzf security_alert.spl` 로 내부 구조 확인 | `tar` |
| 설치 | Splunk 재기동 후 앱 노출 확인 | Splunk Web |
| 뷰 렌더링 | 4 종 뷰 모두 정상 표시 | Splunk Web |
| 매크로 | `| maccro_name` 검색이 의도대로 동작 | Splunk Search bar |
| 알림 액션 | 테스트 알림 작성 → 수동 트리거 | Splunk *Settings* → *Alert actions* |
| Slack 송신 | webhook 으로 실제 메시지 수신 | Slack 채널 |
| 에어갭 시뮬레이션 | 네트워크 차단 후에도 모든 의존성 동작 | 오프라인 Splunk |

문제 발생 시 [`docs/TROUBLESHOOTING.md`](resume/TROUBLESHOOTING.md) 와 [`docs/LEGACY-CLEANUP-REPORT.md`](docs/LEGACY-CLEANUP-REPORT.md) 를 참조한다.

---

## 10. 문서 (Further documentation)

| 주제 | 문서 |
| --- | --- |
| 빠른 시작 | [`docs/QUICK-START.md`](docs/QUICK-START.md) |
| 배포 가이드 | [`docs/DEPLOYMENT.md`](docs/DEPLOYMENT.md), [`resume/DEPLOYMENT.md`](resume/DEPLOYMENT.md) |
| 아키텍처 | [`resume/ARCHITECTURE.md`](resume/ARCHITECTURE.md), [`docs/ALERT-REPOSITORY-XWIKI.md`](docs/ALERT-REPOSITORY-XWIKI.md) |
| API / 스크립트 시그니처 | [`resume/API.md`](resume/API.md) |
| 문제 해결 | [`resume/TROUBLESHOOTING.md`](resume/TROUBLESHOOTING.md) |
| 릴리스 노트 | [`docs/RELEASE-NOTES.md`](docs/RELEASE-NOTES.md) |
| 레거시 정리 이력 | [`docs/LEGACY-CLEANUP-REPORT.md`](docs/LEGACY-CLEANUP-REPORT.md) |
| 앱 자체 안내 | [`security_alert/README.md`](security_alert/README.md) |
| 데모 자산 | [`demo/README.md`](demo/README.md) |

---

## 11. 기여 (Contributing)

기여 절차는 [`CONTRIBUTING.md`](CONTRIBUTING.md) 를 참조한다. 주요 가이드라인:

- `security_alert/default/*.conf` 변경 시 의미 있는 stanza 이름을 사용한다.
- 신규 Python 모듈은 `security_alert/bin/` 에 두고, 외부 의존성은 가능한 `lib/python3/` 에 번들한다.
- 대시보드 뷰는 Simple XML / Dashboard Studio 컨벤션을 따른다.
- 커밋 메시지는 변경 영역을 명시한다 (예: `feat(alert-builder): add severity filter`).

---

## 12. 유지보수 및 지원 (Maintainers & support)

| 항목 | 안내 |
| --- | --- |
| Maintainer | App author (see `security_alert/app.manifest`) |
| Issue tracker | 저장소 내 issue tracker |
| Security disclosures | [`SECURITY_ALERT_README`](security_alert/README.md) 또는 issue tracker 의 security 라벨 |
| Operational questions | Splunk 의 검색·인덱스 기초 지식 참조 |

운영 중 보안 이슈를 발견하면 webhook 토큰, 비밀번호 등 자격증명을 커밋하지 않도록 한다.

---

## 13. 라이선스 (License)

이 저장소는 [`LICENSE`](LICENSE) 파일에 명시된 조건에 따라 배포된다. 앱 코드와 번들된 외부 의존성(urllib3, charset_normalizer, idna) 은 각각의 원본 라이선스를 따른다. 자세한 내용은 `security_alert/lib/python3/*/licenses/` 디렉터리를 참조한다.