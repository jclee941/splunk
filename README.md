```markdown
# Splunk Security Alert System

[![Version](https://img.shields.io/badge/version-2.0.4-blue.svg)](./security_alert/app.manifest)
[![Splunk](https://img.shields.io/badge/Splunk-8.0%2B-orange.svg)](https://www.splunk.com/)
[![Python](https://img.shields.io/badge/python-3.7%2B-green.svg)](https://www.python.org/)
[![License](https://img.shields.io/badge/license-Internal-lightgrey.svg)](./LICENSE)

> FortiGate 방화벽 보안 이벤트를 실시간으로 감지하고, **EMS(Event-Metric-State) 패턴**을 통해 중복 없이 Slack으로 알림하는 프로덕션급 Splunk 앱입니다.

## 개요

본 프로젝트는 FortiGate 방화벽에서 발생하는 다양한 보안 및 시스템 이벤트를 Splunk에서 수집·분석하여, 상태 변화가 실제로 발생했을 때만 Slack으로 알림을 전송하는 종합 모니터링 솔루션입니다.  
SPL 중심의 아키텍처로 외부 처리 프로세스 없이 Splunk 내부 검색만으로 모든 로직을 처리하며, 폐쇄망(Air-Gapped) 환경에서도 번들된 Python 의존성으로 즉시 배포 가능합니다.

## 주요 기능

- **15종 프로덕션 알림** — VPN, HA, 하드웨어, 리소스, 무차별 대입(Brute-force), 트래픽 급증, 라이선스, FMG 동기화 등
- **중복 알림 제로** — 11개 CSV 기반 상태 추적(Lookup)으로 상태 변경 시에만 알림 발송
- **EMS(Event-Metric-State) 패턴** — 이벤트 발생이 아닌 상태 전환(OK ↔ CRITICAL 등) 기반 알림
- **SPL-First 아키텍처** — 모든 로직을 Splunk 검색으로 처리, 외부 연산 불필요
- **폐쇄망 지원** — `requests`, `urllib3`, `certifi` 등 Python 의존성 전체 번들링
- **모바일 푸시 최적화** — 200자 이내의 한 줄 Slack 메시지로 모바일 알림에 최적화
- **매크로 기반 설정** — 인덱스명, LogID 그룹, 임계값을 중앙 매크로에서 일괄 관리
- **Slack 통합** — 기본 제공되는 Custom Alert Action으로 즉시 연동

## 설치 방법

### 요구 사항

- Splunk Enterprise 8.0 이상
- Python 3.7 이상 (Splunk 내장 Python 3 환경)
- FortiGate Syslog 또는 HEC를 통한 데이터 수집

### Splunk App 배포

1. 저장소를 클론합니다.
   ```bash
   git clone <repository-url>
   cd splunk-security-alert
   ```

2. `security_alert` 디렉터리를 Splunk 앱 경로로 복사합니다.
   ```bash
   cp -r security_alert $SPLUNK_HOME/etc/apps/
   ```

3. Splunk를 재시작합니다.
   ```bash
   $SPLUNK_HOME/bin/splunk restart
   ```

4. Splunk 웹에서 **앱 관리** → `security_alert` → **권한 및 설정**을 확인합니다.

5. **Lookup 파일** (`lookups/*.csv`)의 초기 상태를 확인하고, 필요시 경로 및 권한을 조정합니다.

### Slack 연