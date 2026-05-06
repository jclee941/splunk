# Release Notes - Security Alert System v2.0.4

**Release Date**: 2025-11-07
**Package**: `security_alert-v2.0.4-production.tar.gz` (577 KB)
**Status**: Production Ready ✅

---

## 🎯 Overview

Security Alert System v2.0.4는 FortiGate 보안 이벤트를 모니터링하고 Slack으로 알림을 전송하는 독립 Splunk 앱입니다. 이번 버전은 EMS 기반 상태 추적, 번들 라이브러리, 그리고 포괄적인 문서화를 통해 프로덕션 배포에 최적화되었습니다.

---

## ✨ What's New

### 1. EMS 상태 추적 시스템
- **11개 CSV 상태 추적 파일**: 중복 알림 완벽 제거
- **상태 변화 감지**: DOWN→UP, FAIL→OK 등 양방향 알림
- **신규 추가**: `fmg_sync_state_tracker.csv` (FortiManager 동기화 추적)

### 2. Alert 018: FMG Out of Sync
- FortiManager 동기화 실패 감지
- Policy install 로그 분석
- 상태 추적을 통한 중복 알림 방지

### 3. 번들 Python 라이브러리
- **격리 환경 지원**: pip 설치 불필요
- **번들 포함**: requests, urllib3, charset-normalizer, certifi, idna
- **자동 경로 설정**: sys.path 자동 수정

### 4. 포괄적인 문서화
- **DEPLOYMENT.md**: 상세 배포 가이드 (한국어)
- **QUICK-START.md**: 5분 빠른 시작 가이드
- **트러블슈팅**: 5가지 일반 문제 해결 가이드

### 5. 문서 구조 개선
- 레거시 문서 아카이브 (`docs/archive/`)
- **docs/**: 개발자 및 배포 종합 가이드
- 깔끔한 루트 디렉토리

---

## 🚀 Key Features

### 15개 활성 알림
- **바이너리 상태** (4개): VPN, Hardware, Interface, HA
- **임계값 기반** (6개): CPU/Memory, Resource, Admin Login, Brute Force, Traffic, License
- **이벤트 기반** (5개): Config Change, System Reboot, FMG Sync

### 11개 상태 추적 파일
```
vpn_state_tracker.csv
hardware_state_tracker.csv
ha_state_tracker.csv
interface_state_tracker.csv
cpu_memory_state_tracker.csv
resource_state_tracker.csv
admin_login_state_tracker.csv
vpn_brute_force_state_tracker.csv
traffic_spike_state_tracker.csv
license_state_tracker.csv
fmg_sync_state_tracker.csv          # NEW
```

### Slack 통합
- 공식 Splunk Slack Alert Action 사용
- Plain Text 포맷 (attachment=none)
- UUID 자동 제거, 긴 값 truncate
- 이모지 포함, 구조화된 메시지

---

## 📦 Installation

### Quick Install (5 minutes)

```bash
# 1. Extract
cd /opt/splunk/etc/apps/
tar -xzf security_alert-v2.0.4-production.tar.gz

# 2. Set Permissions
chown -R splunk:splunk security_alert
chmod -R 755 security_alert/lib/

# 3. Configure Slack
mkdir -p security_alert/local
cat > security_alert/local/alert_actions.conf <<EOF
[slack]
param.webhook_url = https://hooks.slack.com/services/YOUR/WEBHOOK/URL
EOF

# 4. Restart Splunk
/opt/splunk/bin/splunk restart
```

**자세한 가이드**: `docs/DEPLOYMENT.md` 참조

---

## 🔧 Configuration

### 필수 설정
- **Slack Webhook**: `local/alert_actions.conf` 설정 필요
- **FortiGate Index**: 기본값 `index=fw` (변경 가능)

### 선택 설정
- **알림 비활성화**: `local/savedsearches.conf`에서 `enableSched = 0`
- **임계값 조정**: `local/macros.conf`에서 CPU/Memory 임계값 변경
- **Slack 채널 변경**: 알림별 채널 커스터마이징

---

## 📊 System Requirements

- **Splunk Enterprise**: 8.x 또는 9.x
- **Python**: 3.7+ (Splunk 기본 포함)
- **FortiGate 로그**: Splunk 인덱싱 필요
- **Disk Space**: 최소 10 MB (앱) + 상태 추적 파일

---

## 🔄 Upgrade from Previous Versions

### From v2.0.3 or Earlier

```bash
# 1. Backup current state trackers
tar -czf /backup/state_trackers_$(date +%Y%m%d).tar.gz \
  /opt/splunk/etc/apps/security_alert/lookups/*_state_tracker.csv

# 2. Backup local configs
tar -czf /backup/local_configs_$(date +%Y%m%d).tar.gz \
  /opt/splunk/etc/apps/security_alert/local/

# 3. Stop Splunk
/opt/splunk/bin/splunk stop

# 4. Remove old app
rm -rf /opt/splunk/etc/apps/security_alert

# 5. Install v2.0.4
tar -xzf security_alert-v2.0.4-production.tar.gz -C /opt/splunk/etc/apps/

# 6. Restore local configs
tar -xzf /backup/local_configs_*.tar.gz -C /

# 7. Restore state trackers (optional - start fresh recommended)
# tar -xzf /backup/state_trackers_*.tar.gz -C /

# 8. Set permissions
chown -R splunk:splunk /opt/splunk/etc/apps/security_alert

# 9. Start Splunk
/opt/splunk/bin/splunk start
```

**Note**: 상태 추적 파일을 복원하지 않으면 첫 실행 시 새로운 상태로 시작됩니다.

---

## ✅ Verification Checklist

배포 후 다음을 확인하세요:

```spl
# 1. App loaded
index=_internal source=*splunkd.log security_alert
| stats count by log_level

# 2. 15 alerts enabled
| rest /services/saved/searches
| search title="*Alert*"
| where disabled=0
| stats count

# 3. 11 state trackers
| rest /services/data/lookup-table-files
| search title="*state_tracker*"
| stats count

# 4. Bundled libraries working
# Run: cd /opt/splunk/etc/apps/security_alert && python3 -c "import sys; sys.path.insert(0, 'lib/python3'); import requests; print('OK')"

# 5. Slack integration
index=_internal source=*alert_actions.log action_name="slack"
| stats count by action_status
```

---

## 🐛 Known Issues

### Issue 1: CSV Lock Errors (Rare)
**Symptom**: `Error in 'outputlookup': The lookup table is locked`
**Workaround**: Splunk will automatically retry. If persistent, restart Splunk.
**Fix**: Ensured `append=t` mode in all alerts (v2.0.4)

### Issue 2: State Tracker Growth
**Impact**: CSV files may grow over time (normal behavior)
**Mitigation**: Monthly cleanup scheduled search included
**Monitoring**: Alert if file size > 1 MB

---

## 🔒 Security Considerations

### Hardcoded Credentials Removed
- **bin/fortigate_auto_response.py**: 하드코딩된 토큰 제거 (사용 안 함)
- **local/alert_actions.conf**: 사용자 환경에서 Slack webhook 설정

### Permissions
- `local/` 디렉토리: splunk:splunk, 600
- `lib/python3/`: splunk:splunk, 755
- `bin/*.py`: splunk:splunk, 755

---

## 📚 Documentation

| Document | Purpose | Language |
|----------|---------|----------|
| **security_alert/README.md** | User guide | Korean |
| **docs/DEPLOYMENT.md** | Deployment guide | Korean |
| **docs/QUICK-START.md** | 5-minute quick start | Korean |

**Archived**: 10 legacy documents in `docs/archive/`

---

## 🔄 Breaking Changes

### None

v2.0.4는 v2.0.3과 완전히 호환됩니다. 상태 추적 파일 형식이 동일하므로 기존 상태를 유지할 수 있습니다.

---

## 🎯 Roadmap

### Planned for v2.1.0 (Future)
- Dashboard improvements
- Additional FortiGate event types
- Enhanced auto-response actions
- Performance optimizations

---

## 📞 Support

**Repository**: https://github.com/qws941/splunk.git
**GitLab**: https://gitlab.jclee.me/jclee/splunk
**Maintainer**: NextTrade Security Team

---

## 📝 Changelog

### v2.0.4 (2025-11-07)
- ✅ Added Alert 018 (FMG Out of Sync)
- ✅ Added `fmg_sync_state_tracker.csv`
- ✅ Bundled Python dependencies (air-gapped support)
- ✅ Comprehensive documentation (DEPLOYMENT.md, QUICK-START.md)
- ✅ Archived 10 legacy documents
- ✅ Clean docs/ structure (docs/ 디렉토리 구조)
- ✅ Production-ready packaging

### v2.0.3 (2025-11-04)
- Fixed FMG sync SPL syntax
- Implemented EMS state tracking
- Slack message formatting improvements

### v2.0.1 (2025-11-03)
- Enhanced field parsing with coalesce()
- Fixed LogID definitions
- Added FMG install detection

---

## 🏆 Credits

**Developed by**: NextTrade Security Team
**AI Assistant**: Claude Code (Anthropic)
**License**: MIT

---

**🤖 Generated with [Claude Code](https://claude.com/claude-code)**
