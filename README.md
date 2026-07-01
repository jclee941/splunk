# Security Alert (Splunk App) / 보안 알림 (Splunk 앱)

프로덕션급 Splunk 애드온으로, 통합 **알림 관리 대시보드**, 안내형 **손쉬운 알림 빌더**, 풀-기능 **알림 빌더**, 그리고 탐색형 **데이터 탐색기 대시보드**를 제공합니다. 안전한 포맷팅 헬퍼와 Slack 커스텀 알림 액션을 함께 제공하며, Python 런타임 의존성을 번들하여 격리(air-gapped) Splunk 환경에서도 동작합니다.

A production-grade Splunk add-on that delivers a unified Alert Management Dashboard, a guided Easy Alert Builder, a feature-complete Alert Builder, and an exploratory Data Explorer Dashboard. The app ships with a safe-formatting helper and a Slack custom alert action, and bundles its Python runtime dependencies so it runs on air-gapped Splunk deployments.

---

## Status / 운영 상태

| Runtime / 실행 환경 | Status / 상태 | Owner / 책임자 | Next action / 다음 작업 |
| --- | --- | --- | --- |
| Splunk 9.x | Production | App author | `security_alert/` 폴더를 `.tar.gz`로 묶어 `security_alert.spl` 패키지를 만든 뒤 *Manage Apps* 메뉴에서 설치 |
| Splunk 8.x | Compatible | App author | 설치 후 8.x 인스턴스에서 대시보드 렌더링을 검증 |
| Air-gapped | Supported | App author | 외부 Python 패키지 다운로드 불필요(의존성 사전 번들) |
| Python 3 runtime | Bundled | App author | `security_alert/lib/python3/` 내 urllib3, charset_normalizer, idna 포함 |

---

## Quick-flow summary / 빠른 흐름 요약

1. **Build the package** — `security_alert/` 폴더를 `.tar.gz`로 묶어 `security_alert.spl`로 패키징합니다.
2. **Install the package** — Splunk Web의 *Manage Apps* 화면에서 업로드하거나 `$SPLUNK_HOME/etc/apps/`에 압축 해제합니다.
3. **Configure routes** — `default/alert_actions.conf`에서 Slack 알림 액션을 활성화합니다.
4. **Author alerts** — **Easy Alert Builder** 또는 **Alert Builder** 대시보드 뷰에서 알림을 작성합니다.
5. **Operate alerts** — **Alert Management Dashboard**에서 분류(triage)하고, **Data Explorer Dashboard**에서 심층 분석합니다.

---

## 1. Purpose / 목적

`security_alert/` 는 임시 알림 작성을 반복 가능하고 감사 가능한 워크플로우로 전환하는 자급식 Splunk 앱입니다.

대상 사용자:

| User role / 사용자 역할 | Typical task / 주요 작업 |
| --- | --- |
| 탐지 엔지니어 (Detection engineer) | 일관된 형식과 검증된 메타데이터로 알림을 작성 |
| SOC 분석가 (SOC analyst) | 알림을 분류하고 후속 조치(triage)를 수행 |
| Splunk 관리자 (Splunk admin) | 알림 자산을 안전하게 패키징하고 배포 |

핵심 사용자 시나리오:

- **알림 작성** — Easy Alert Builder 또는 Alert Builder에서 검색 조건, 트리거 조건, 알림 액션을 단계적으로 정의합니다.
- **알림 운영** — Alert Management Dashboard에서 알림 상태(발생/승인/종료)를 추적하고 소유자를 지정합니다.
- **데이터 탐색** — Data Explorer Dashboard에서 원시 이벤트를 시각화하고 알림 작성의 전제 조건을 확인합니다.
- **외부 통지** — Slack 커스텀 알림 액션으로 검증된 페이로드를 지정 채널에 게시합니다.

---

## 2. Package Contents / 패키지 구성

저장소 최상위 구조는 다음과 같습니다.

| Path / 경로 | Role / 역할 |
| --- | --- |
| `security_alert/` | 배포 단위인 Splunk 앱 본체 (`app.conf`, 대시보드, 액션, 번들 의존성 포함) |
| `security_alert/bin/` | 스크립트 트리거 진입점 (`safe_fmt.py`, `slack.py`, `six.py`) |
| `security_alert/default/` | Splunk 설정 파일(`alert_actions.conf`, `app.conf`, `macros.conf`, `props.conf`, `savedsearches.conf`, `transforms.conf`) |
| `security_alert/default/data/ui/nav/default.xml` | 앱 내비게이션 정의 |
| `security_alert/default/data/ui/views/` | 4종 대시보드/빌더 XML 뷰 |
| `security_alert/lib/python3/` | 격리 환경용 번들 Python 의존성(urllib3, charset_normalizer, idna) |
| `security_alert/metadata/default.meta` | 앱 메타데이터 및 권한 |
| `security_alert/app.manifest` | Splunk 앱 매니페스트 |
| `docs/` | 루트 레벨 운영 문서(QUICK-START, DEPLOYMENT, RELEASE-NOTES, LEGACY-CLEANUP-REPORT, ALERT-REPOSITORY-XWIKI) |
| `resume/` | 부가 문서 모음(API, ARCHITECTURE, DEPLOYMENT, TROUBLESHOOTING) |
| `demo/` | 데모 자료(자체 README 참조) |
| `CONTRIBUTING.md` | 기여 절차 |
| `LICENSE` | 라이선스 원문 |

---

## 3. First Files to Read / 먼저 읽을 파일

운영자가 가장 먼저 확인해야 할 파일은 다음과 같습니다.

| Order | File / 파일 | Reason / 이유 |
| --- | --- | --- |
| 1 | `security_alert/app.conf` | 앱 버전, 라벨, 라이선스 그룹 등 최상위 메타데이터 확인 |
| 2 | `security_alert/app.manifest` | Splunk AppInspect가 사용하는 매니페스트 정의 확인 |
| 3 | `security_alert/default/app.conf` | 앱 내부 설정(UI 가시성, 권한 스코프) 확인 |
| 4 | `security_alert/default/alert_actions.conf` | 사용자 정의 알림 액션(예: `slack`) 등록 여부 확인 |
| 5 | `security_alert/default/data/ui/nav/default.xml` | 사용자에게 노출되는 메뉴 구조 확인 |
| 6 | `docs/QUICK-START.md` | 5분 내 설치 → 첫 알림 작성 시나리오 |
| 7 | `docs/DEPLOYMENT.md` | 운영 환경 배포 절차(파일 권한, 인덱서 간 동기화 등) |
| 8 | `resume/ARCHITECTURE.md` | 컴포넌트 간 책임 분리와 데이터 흐름 |

---

## 4. API or Entry Points / API 및 진입점

이 앱은 REST API를 직접 노출하지 않습니다. 사용자는 Splunk UI 또는 검색 언어(SPL)을 통해 진입합니다.

### 4.1 Dashboard / Builder 진입점

| Entry / 진입점 | View XML / 뷰 XML | Purpose / 용도 |
| --- | --- | --- |
| Alert Management Dashboard | `default/data/ui/views/alert-management-dashboard.xml` | 운영 중인 알림의 상태, 소유자, 발생 추이를 한 화면에서 관리 |
| Data Explorer Dashboard | `default/data/ui/views/data-explorer-dashboard.xml` | 원시 이벤트 탐색 및 알림 작성 전 데이터 사전 확인 |
| Easy Alert Builder | `default/data/ui/views/easy_alert_builder.xml` | 최소 입력만으로 알림을 작성하는 안내형 빌더 |
| Alert Builder | `default/data/ui/views/alert-builder.xml` | 모든 필드를 노출하는 풀-기능 알림 빌더 |

### 4.2 Scripted entry points / 스크립트 진입점

| Script / 스크립트 | Role / 역할 | Invocation / 호출 위치 |
| --- | --- | --- |
| `security_alert/bin/safe_fmt.py` | 입력 문자열의 안전한 포맷팅(이스케이프/검증) 헬퍼 | 다른 스크립트에서 import 또는 직접 호출 |
| `security_alert/bin/slack.py` | Slack 커스텀 알림 액션 본체 | `default/alert_actions.conf`의 `command =` 항목에서 호출 |
| `security_alert/bin/six.py` | Python 2/3 호환성 유틸(`six` 라이브러리 사본) | `safe_fmt.py` 등 보조 모듈에서 import |

### 4.3 SPL macros / SPL 매크로

`security_alert/default/macros.conf`에 정의된 매크로는 검색 작성 시 재사용됩니다. 구체적인 매크로 이름과 본문은 동일 파일에서 직접 확인해야 합니다.

---

## 5. Quickstart / 빠른 시작

### 5.1 Build / 빌드

```bash
# 저장소 루트에서 실행
tar -czf security_alert.spl security_alert/
```

### 5.2 Install / 설치

Splunk Web 경로:

1. *Apps → Manage Apps → Install app from file* 메뉴 진입
2. `security_alert.spl` 업로드
3. *Restart Splunk* 가 필요한 경우 안내에 따라 재시작

수동 경로(파일 직접 배치):

```bash
# Splunk 인스턴스가 설치된 호스트에서 실행
$SPLUNK_HOME/bin/splunk stop
cp security_alert.spl $SPLUNK_HOME/etc/apps/
cd $SPLUNK_HOME/etc/apps
tar -xzf security_alert.spl
$SPLUNK_HOME/bin/splunk start
```

### 5.3 Configure / 설정

`security_alert/default/alert_actions.conf`를 검토하여 Slack 액션을 활성화합니다.

| Setting / 설정 항목 | Example / 예시 | Note / 비고 |
| --- | --- | --- |
| `alert_actions.conf` 내 Slack 액션 stanza의 `disabled` | `disabled = 0` | `1`이면 비활성, `0`이면 활성 |
| `command` | `command = slack.py` | 실행 스크립트 경로 |
| `param.*` | 채널/토큰 등 | 토큰 같은 비밀 값은 `local/` 디렉터리에 별도 파일로 보관 권장 |

비밀 값은 반드시 `local/alert_actions.conf`로 오버라이드하고, 저장소에는 커밋하지 않습니다.

### 5.4 First alert / 첫 알림 작성

1. Splunk Web에서 *Security Alert* 앱 진입
2. *Easy Alert Builder* 선택 → 이름, 검색어, 트리거 조건 입력
3. *Save* 후 *Activate*
4. *Alert Management Dashboard* 에서 새 알림이 노출되는지 확인

자세한 절차는 `docs/QUICK-START.md`를 참조하세요.

---

## 6. Configuration / 설정

### 6.1 Configuration files / 설정 파일

| File / 파일 | Scope / 적용 범위 | Owner / 책임자 |
| --- | --- | --- |
| `security_alert/default/app.conf` | 앱 전역 설정 | App author(기본값), Splunk admin(`local/`로 오버라이드) |
| `security_alert/default/alert_actions.conf` | 알림 액션 등록 및 파라미터 | Splunk admin |
| `security_alert/default/macros.conf` | SPL 매크로 | App author |
| `security_alert/default/props.conf` | 필드 추출/타임스탬프 규칙 | Splunk admin |
| `security_alert/default/savedsearches.conf` | 사전 정의된 검색/보고 | App author |
| `security_alert/default/transforms.conf` | 필드 변환 룰 | Splunk admin |
| `security_alert/metadata/default.meta` | 앱/뷰 권한 매트릭스 | App author |

### 6.2 Override pattern / 오버라이드 패턴

운영 환경에서는 `default/` 파일을 직접 수정하지 않고 동일 상대 경로의 `local/` 파일에서 필요한 키만 재정의합니다.

```text
security_alert/
├── default/
│   └── alert_actions.conf   # 수정하지 않음
└── local/
    └── alert_actions.conf   # 비밀 값, 환경별 채널 등을 여기에 정의
```

### 6.3 Bundled dependencies / 번들 의존성

| Package / 패키지 | Source / 출처 | Reason / 번들 사유 |
| --- | --- | --- |
| `urllib3` | `security_alert/lib/python3/urllib3/` | 격리 환경에서 외부 HTTP 호출을 위해 필요 |
| `charset_normalizer` | `security_alert/lib/python3/charset_normalizer-3.4.4.dist-info/` | 응답 인코딩 판별 |
| `idna` | `security_alert/lib/python3/idna-3.11.dist-info/` | 국제화 도메인 처리 |

번들 의존성은 Splunk Python 인터프리터 경로에서 그대로 임포트되며, 운영자가 별도로 `pip install`을 수행할 필요가 없습니다.

---

## 7. Commands Reference / 명령어 참조

앱 자체에는 CLI 명령이 없으며, Splunk CLI를 통해 다음 작업을 수행합니다.

| Task / 작업 | Command / 명령어 | Where / 실행 위치 |
| --- | --- | --- |
| 패키지 빌드 | `tar -czf security_alert.spl security_alert/` | 저장소 루트 |
| 앱 설치(수동) | `cp security_alert.spl $SPLUNK_HOME/etc/apps/` | Splunk 호스트 |
| 압축 해제(수동) | `tar -xzf security_alert.spl` | `$SPLUNK_HOME/etc/apps/` |
| Splunk 재시작 | `$SPLUNK_HOME/bin/splunk restart` | Splunk 호스트 |
| 앱 목록 확인 | `$SPLUNK_HOME/bin/splunk display app` | Splunk 호스트 |
| AppInspect 검증 | `splunk-appinspect inspect security_alert.spl` | 개발자 워크스테이션 |

앱 내부 스크립트는 Splunk 알림 액션 실행 흐름 안에서 호출됩니다.

---

## 8. Architecture / 아키텍처

### 8.1 Component map / 컴포넌트 맵

| Layer / 계층 | Component / 컴포넌트 | Responsibility / 책임 |
| --- | --- | --- |
| UI | `default/data/ui/nav/default.xml` | 앱 내비게이션 진입 제공 |
| UI | 4종 대시보드 XML (`alert-management-dashboard`, `data-explorer-dashboard`, `easy_alert_builder`, `alert-builder`) | 알림 작성/운영/탐색 시각화 |
| Configuration | `default/*.conf` | 동작/스키마 정의 |
| Logic | `bin/safe_fmt.py`, `bin/slack.py`, `bin/six.py` | 알림 액션 실행, 포맷팅, 호환성 |
| Runtime | `lib/python3/urllib3`, `lib/python3/charset_normalizer`, `lib/python3/idna` | 격리 환경 외부 통신 |
| Metadata | `app.conf`, `app.manifest`, `metadata/default.meta` | 버전/권한/매니페스트 |

### 8.2 Request flow / 요청 흐름

1. 운영자가 Splunk Web에서 앱 진입 → `default.xml`이 메뉴 노출
2. 대시보드 선택 → 해당 XML이 SimpleXML 뷰를 렌더링
3. 검색어 입력 후 알림 저장 → `savedsearches.conf` 형식으로 저장
4. 알림 트리거 발생 시 Splunk이 `alert_actions.conf`의 액션 실행
5. 액션이 `bin/slack.py` 등을 호출 → `bin/safe_fmt.py`로 페이로드 정제
6. `lib/python3/`의 번들 의존성으로 Slack 등 외부 시스템에 POST
7. 결과는 `Alert Management Dashboard`에서 사후 추적

---

## 9. Local Development / 로컬 개발

| Step / 단계 | Action / 작업 |
| --- | --- |
| 1 | Splunk Enterprise 9.x를 로컬에 설치 |
| 2 | 이 저장소를 클론 후 `security_alert/`를 `$SPLUNK_HOME/etc/apps/security_alert/`에 심볼릭 링크 또는 복사 |
| 3 | `$SPLUNK_HOME/bin/splunk restart` 로 앱 로드 |
| 4 | XML/Conf 수정 후 브라우저 새로고침으로 반영 확인(대시보드/뷰 캐시 무효화 필요 시 새로고침) |
| 5 | Python 스크립트 수정 후 `$SPLUNK_HOME/bin/splunk restart` 또는 액션 재실행으로 반영 |

권장 워크플로우:

- `default/`는 절대 직접 수정하지 않고, 개발 중에는 `local/` 오버라이드를 사용합니다.
- 변경 검증 후 `default/`에 반영해야 하는 경우 PR로 제출합니다.

---

## 10. Testing / 테스트

| Test type / 테스트 종류 | Approach / 접근 | Notes / 비고 |
| --- | --- | --- |
| 정적 검증 (Static) | `splunk-appinspect inspect security_alert.spl` | Splunk AppInspect 베이스라인 통과 |
| 단위 테스트 (Unit) | `bin/` 스크립트 import 후 함수 단위 호출 | 격리된 Python 가상환경에서 `urllib3` 등 번들 모듈을 모킹 |
| 통합 테스트 (Integration) | 로컬 Splunk 인스턴스에 설치 후 대시보드 렌더링 확인 | AppInspect + 수동 클릭 트리 |
| 알림 액션 테스트 | *Settings → Alert Actions* 에서 테스트 트리거 | Slack 등 외부 시스템 호출은 별도 테스트 채널에서 수행 |

테스트 절차는 `docs/RELEASE-NOTES.md` 및 `resume/TROUBLESHOOTING.md`의 권장 시나리오를 참고하세요.

---

## 11. Maintainers / Points of Contact / 유지보수자 및 연락처

| Role / 역할 | Responsibility / 책임 | Contact path / 연락 경로 |
| --- | --- | --- |
| App author (Primary maintainer) | 기능 개발, 릴리스, AppInspect 대응 | 저장소 *Issues* 탭 |
| Splunk admin (Operational owner) | 배포, 비밀 값 관리, 권한 부여 | 조직 내부 운영 채널 |
| Detection engineering reviewer | 알림 작성 UX/메타데이터 검토 | 조직 내부 리뷰 보드 |

공개 이슈 트래커를 통해 버그 제보와 기능 요청을 받습니다. 보안 취약점 제보는 공개 이슈 대신 별도 채널을 통해 비공개로 통지해야 합니다.

---

## 12. Further Documentation / 추가 문서

| Topic / 주제 | File / 파일 | Note / 비고 |
| --- | --- | --- |
| 5분 빠른 시작 | `docs/QUICK-START.md` | 설치부터 첫 알림까지 |
| 배포 절차 | `docs/DEPLOYMENT.md` | 멀티 인스턴스 배포, 권한 |
| 아키텍처 상세 | `resume/ARCHITECTURE.md` | 컴포넌트 다이어그램 |
| API 레퍼런스 | `resume/API.md` | 알림 액션 파라미터 레퍼런스 |
| 트러블슈팅 | `resume/TROUBLESHOOTING.md` | 자주 발생하는 증상과 해결책 |
| 릴리스 노트 | `docs/RELEASE-NOTES.md` | 버전별 변경 이력 |
| 레거시 정리 이력 | `docs/LEGACY-CLEANUP-REPORT.md` | 과거 자산 정리 보고서 |
| XWiki 연동 | `docs/ALERT-REPOSITORY-XWIKI.md` | 외부 위키 통합 메모 |
| 데모 자료 | `demo/README.md` | 데모 환경 구성 안내 |
| 기여 절차 | `CONTRIBUTING.md` | PR/이슈 작성 가이드 |
| 라이선스 | `LICENSE` | 사용 조건 |

---

## 13. Contribution Guide / 기여 가이드

기여 절차는 `CONTRIBUTING.md`를 따릅니다. 핵심 원칙:

- 이슈 등록 후 PR을 제출합니다.
- `default/` 파일을 직접 수정하기 전에 동일 키를 `local/`에서 검증한 뒤 반영합니다.
- 대시보드/뷰 XML 변경 시 Splunk 9.x와 8.x 모두에서 렌더링을 확인합니다.
- `splunk-appinspect`를 통과한 상태로 PR을 제출합니다.
- 비밀 값(Slack 토큰, 채널 ID 등)은 PR에 포함하지 않습니다.

---

## 14. License / 라이선스

본 저장소는 `LICENSE` 파일에 명시된 라이선스 조건 하에 배포됩니다. 사용·수정·재배포 전 라이선스 전문을 확인하세요.