# FortiGate Security Alert Repository

**Security Alert System v2.0.4** - XWiki 문서화

---

## 📋 목차

1. [개요](#개요)
2. [알림 목록](#알림-목록)
3. [알림 상세 (001-018)](#알림-상세)
4. [상태 추적 시스템](#상태-추적-시스템)
5. [LogID 참조](#logid-참조)
6. [트러블슈팅](#트러블슈팅)

---

## 개요

### 시스템 정보

- **버전**: v2.0.4
- **플랫폼**: Splunk Enterprise 8.x/9.x
- **데이터 소스**: FortiGate Firewall Logs (`index=fw`)
- **알림 채널**: Slack (#security-firewall-alert)
- **상태 추적**: EMS 방식 (11개 CSV 파일)

### 핵심 기능

- ✅ **중복 알림 제거**: EMS 상태 추적으로 상태 변화만 알림
- ✅ **양방향 감지**: DOWN→UP, FAIL→OK 복구 알림 지원
- ✅ **실시간 모니터링**: 1분 간격 실시간 검색
- ✅ **자동 메시지 포맷**: UUID 제거, 값 truncate, 구조화된 메시지

---

## 알림 목록

### 전체 알림 (15개)

| ID | 알림명 | 분류 | 상태 | 심각도 | State Tracker |
|----|--------|------|------|--------|---------------|
| **001** | Config Change | 이벤트 | ✅ Enabled | Medium | - |
| **002** | VPN Tunnel Down/Up | 바이너리 | ✅ Enabled | Critical | vpn_state_tracker.csv |
| ~~003~~ | ~~예약됨~~ | - | - | - | - |
| ~~004~~ | ~~예약됨~~ | - | - | - | - |
| ~~005~~ | ~~예약됨~~ | - | - | - | - |
| **006** | CPU/Memory Anomaly | 임계값 | ✅ Enabled | High | cpu_memory_state_tracker.csv |
| **007** | Hardware Failure/Restored | 바이너리 | ✅ Enabled | Critical | hardware_state_tracker.csv |
| **008** | HA State Change | 바이너리 | ✅ Enabled | High | ha_state_tracker.csv |
| ~~009~~ | ~~예약됨~~ | - | - | - | - |
| **010** | Resource Limit | 임계값 | ✅ Enabled | Medium | resource_state_tracker.csv |
| **011** | Admin Login Failed | 임계값 | ⚠️ Disabled | High | admin_login_state_tracker.csv |
| **012** | Interface Down/Up | 바이너리 | ✅ Enabled | Medium | interface_state_tracker.csv |
| **013** | SSL VPN Brute Force | 임계값 | ⚠️ Disabled | High | vpn_brute_force_state_tracker.csv |
| ~~014~~ | ~~예약됨~~ | - | - | - | - |
| **015** | Abnormal Traffic Spike | 임계값 | ✅ Enabled | Medium | traffic_spike_state_tracker.csv |
| **016** | System Reboot | 이벤트 | ✅ Enabled | Low | - |
| **017** | License Expiry Warning | 임계값 | ⚠️ Disabled | Low | license_state_tracker.csv |
| **018** | FMG Out of Sync | 이벤트 | ✅ Enabled | Medium | fmg_sync_state_tracker.csv |

### 분류별 통계

| 분류 | 개수 | 활성화 | 비활성화 |
|------|------|--------|----------|
| **바이너리 상태** | 4개 | 4 | 0 |
| **임계값 기반** | 6개 | 3 | 3 |
| **이벤트 기반** | 5개 | 3 | 2 |
| **전체** | **15개** | **10개** | **5개** |

---

## 알림 상세

### Alert 001: Config Change (설정 변경)

**분류**: 이벤트 기반
**상태**: ✅ Enabled
**심각도**: Medium

**목적**: FortiGate 설정 변경을 실시간으로 감지하고 변경 내역 추적

**LogID**:
- `0100044546` - CLI 설정 변경
- `0100044547` - GUI 설정 변경
- `0100044548-50` - FMG Install

**감지 조건**:
- 설정 변경 이벤트 발생
- UUID 전용 변경은 필터링 (`is_uuid_only_change = 0`)

**메시지 형식**:
```
{change_type} | {config_path} | FROM: {before_value} ➜ TO: {after_value}
```

**예시**:
```
🔒 Firewall Policy | firewall.policy.policy1 | FROM: 192.168.1.0/24 ➜ TO: 192.168.2.0/24
```

**SPL 핵심 로직**:
```spl
| eval action = case(
    like(msg, "%added%"), "➕ ADD",
    like(msg, "%deleted%"), "🗑️ DELETE",
    like(msg, "%modified%"), "✏️ MODIFY",
    true(), "📝 CHANGE")
| eval criticality = case(
    like(config_path, "%firewall%policy%"), "🔴 HIGH",
    like(config_path, "%vpn%"), "🟠 MEDIUM",
    true(), "🟢 LOW")
```

**Suppression**: 10분 (device, user, config_path)
**State Tracker**: 없음 (Suppression 사용)

---

### Alert 002: VPN Tunnel Down/Up (VPN 터널 상태)

**분류**: 바이너리 상태
**상태**: ✅ Enabled
**심각도**: Critical

**목적**: VPN 터널 장애 감지 및 복구 알림

**LogID**:
- **Down**: `0101037124` (Tunnel Down), `0101037131` (Phase1 Fail), `0101037134` (Phase2 Fail)
- **Up**: `0101037125` (Phase1 Up), `0101037132` (Tunnel Up)

**상태 전환**:
- `DOWN` → `UP` (복구 알림)
- `UP` → `DOWN` (장애 알림)

**메시지 형식**:
```
{vpn_type}: {vpn_name} | {event} | Remote: {remote_ip} | Reason: {failure_reason}
```

**예시**:
```
⚠️ VPN Tunnel Down: tunnel1
IPsec: tunnel1 | Phase1 Fail | Remote: 10.1.1.100 | Reason: Authentication failed

✅ VPN Tunnel Up: tunnel1
IPsec: tunnel1 | Tunnel Up | Remote: 10.1.1.100
```

**SPL 핵심 로직**:
```spl
| eval current_state = "DOWN"  # or "UP"
| stats latest(*) as * by device, vpn_name
| join type=left device vpn_name [
    | inputlookup vpn_state_tracker
    | rename state as previous_state ]
| eval state_changed = if(isnull(previous_state) OR previous_state!=current_state, 1, 0)
| where state_changed=1
| outputlookup append=t vpn_state_tracker
```

**State Tracker**: `vpn_state_tracker.csv`
**Suppression**: 없음 (EMS 상태 추적 사용)

---

### Alert 003-005: (예약됨)

현재 사용되지 않음. 향후 확장을 위해 예약됨.

---

### Alert 006: CPU/Memory Anomaly (CPU/메모리 이상)

**분류**: 임계값 기반
**상태**: ✅ Enabled
**심각도**: High

**목적**: CPU/Memory 사용률의 베이스라인 대비 급격한 변화 감지

**LogID**:
- `0104043001` - CPU 사용률 로그
- `0104043002` - Memory 사용률 로그

**감지 조건**:
- **임계값**: 베이스라인 대비 **20% 이상 편차**
- 베이스라인: 과거 데이터의 평균값 (eventstats 사용)

**상태 전환**:
- `ABNORMAL` ↔ `NORMAL`

**메시지 형식**:
```
{resource} Usage Spike | Current: {current}% (Baseline: {baseline}%) | +{deviation}% deviation
```

**예시**:
```
📈 CPU/Memory Anomaly: CPU
CPU Usage Spike | Current: 85% (Baseline: 60%) | +41.7% deviation
```

**SPL 핵심 로직**:
```spl
| eventstats avg(current_value) as baseline_avg by device, resource
| eval deviation_pct = round(((current_value - baseline_avg) / baseline_avg) * 100, 1)
| eval current_state = if(deviation_pct > 20, "ABNORMAL", "NORMAL")
```

**State Tracker**: `cpu_memory_state_tracker.csv`
**Suppression**: 없음 (EMS 상태 추적 사용)

---

### Alert 007: Hardware Failure/Restored (하드웨어 장애)

**분류**: 바이너리 상태
**상태**: ✅ Enabled
**심각도**: Critical

**목적**: 하드웨어 컴포넌트 장애 감지 및 복구 알림

**LogID**:
- **Failure**: `0103040001-03` (Fan/PSU/Temperature 장애)
- **Restored**: `0103040014-15` (복구)

**감지 조건**:
- Fan, PSU, Temperature, Disk 센서 상태 변화

**상태 전환**:
- `FAIL` → `OK` (복구 알림)
- `OK` → `FAIL` (장애 알림)

**메시지 형식**:
```
{component} {component_detail} | Status: {status} | {additional_info}
```

**예시**:
```
⚠️ Hardware Failure: Fan
Fan FAN 1 | Status: FAILED

✅ Hardware Restored: Fan
Fan FAN 1 | Status: RESTORED
```

**SPL 핵심 로직**:
```spl
| eval component = case(
    like(msg, "%fan%"), "Fan",
    like(msg, "%power%"), "PSU",
    like(msg, "%temp%"), "Temperature",
    true(), "Hardware")
| eval severity = case(
    component="PSU" OR component="Temperature", "🔴 CRITICAL",
    component="Fan", "🟠 HIGH",
    true(), "🟡 MEDIUM")
```

**State Tracker**: `hardware_state_tracker.csv`
**Suppression**: 없음 (EMS 상태 추적 사용)

---

### Alert 008: HA State Change (HA 상태 변경)

**분류**: 바이너리 상태
**상태**: ✅ Enabled
**심각도**: High

**목적**: HA 클러스터 상태 변경 감지 (역할 전환, 멤버 변경)

**LogID**:
- `0100020010` - HA 상태 변경
- `0104043544` - HA 역할 변경
- `0104043545` - HA 동기화 이벤트

**상태 전환**:
- HA 상태가 변경될 때마다 알림 (previous_state ≠ current_state)

**메시지 형식**:
```
{role} | Transition: {from_state} → {to_state} | Member: {member} | Reason: {reason}
```

**예시**:
```
🔄 HA State Change: fw01
PRIMARY | Transition: slave → master | Member: FW01 | Reason: Peer unreachable
```

**SPL 핵심 로직**:
```spl
| eval role = case(
    like(ha_state, "%master%"), "PRIMARY",
    like(ha_state, "%slave%"), "SECONDARY",
    like(ha_state, "%standalone%"), "STANDALONE",
    true(), "UNKNOWN")
| eval criticality = case(
    ha_state="standalone", "🔴 CRITICAL",
    like(transition, "%→%"), "🟠 CHANGE",
    true(), "🟢 NORMAL")
```

**State Tracker**: `ha_state_tracker.csv`
**Suppression**: 없음 (EMS 상태 추적 사용)

---

### Alert 009: (예약됨)

현재 사용되지 않음. 향후 확장을 위해 예약됨.

---

### Alert 010: Resource Limit (리소스 한계)

**분류**: 임계값 기반
**상태**: ✅ Enabled
**심각도**: Medium

**목적**: 시스템 리소스 사용률이 임계값 초과 시 알림

**LogID**:
- `0104043003` - Disk/Memory 사용률
- `0104043004` - Session/CPU 사용률

**감지 조건**:
- **임계값**: 75% 이상 사용률

**상태 전환**:
- `EXCEEDED` ↔ `NORMAL`

**메시지 형식**:
```
{resource_type} | Usage: {usage}% | Remaining: {remaining}
```

**예시**:
```
📊 Resource Limit: Disk
Disk | Usage: 85% | Remaining: 15GB
```

**SPL 핵심 로직**:
```spl
| eval usage_pct = tonumber(coalesce(usage_pct, "0"))
| eval severity = case(
    usage_pct >= 95, "🔴 CRITICAL",
    usage_pct >= 85, "🟠 HIGH",
    usage_pct >= 75, "🟡 MEDIUM",
    true(), "🟢 LOW")
| eval current_state = if(usage_pct >= 75, "EXCEEDED", "NORMAL")
```

**State Tracker**: `resource_state_tracker.csv`
**Suppression**: 없음 (EMS 상태 추적 사용)

---

### Alert 011: Admin Login Failed (관리자 로그인 실패)

**분류**: 임계값 기반
**상태**: ⚠️ Disabled (enableSched = 0)
**심각도**: High

**목적**: 관리자 계정에 대한 반복적인 로그인 실패 감지 (브루트포스 공격)

**LogID**:
- `0105032003` - Admin 로그인 실패
- `0105032004` - Admin 로그아웃
- `0105032005` - Admin 권한 부족
- `0105043001` - Admin 인증 실패

**감지 조건**:
- **임계값**: 5분 내 3회 이상 실패

**상태 전환**:
- `ATTACK` ↔ `NORMAL`

**메시지 형식**:
```
{attack_type} from {source_ip} | Users: {users} | {fail_count} failures in {duration}
```

**예시**:
```
🚨 Admin Login Attack: fw01
🟡 Failed Login from 192.168.1.100 | Users: admin, root, user1 | 5 failures in 00:03:15
```

**SPL 핵심 로직**:
```spl
| stats count as fail_count, values(user) as users by device, source_ip
| eval attack_type = case(
    fail_count >= 10, "🔴 Brute Force",
    fail_count >= 5, "🟠 Suspicious",
    fail_count >= 3, "🟡 Failed Login",
    true(), "🟢 Normal")
| eval current_state = if(fail_count >= 3, "ATTACK", "NORMAL")
```

**State Tracker**: `admin_login_state_tracker.csv`
**Suppression**: 없음 (EMS 상태 추적 사용)

**비활성화 이유**: 프로덕션 환경에서 false positive 높음. 필요 시 활성화.

---

### Alert 012: Interface Down/Up (인터페이스 상태)

**분류**: 바이너리 상태
**상태**: ✅ Enabled
**심각도**: Medium

**목적**: 네트워크 인터페이스 링크 상태 변화 감지

**LogID**:
- `0100032001` - Interface link down
- `0100020007` - Interface link up

**상태 전환**:
- `DOWN` → `UP` (복구 알림)
- `UP` → `DOWN` (장애 알림)

**메시지 형식**:
```
{if_type} Interface: {interface} | Status: {status} | {link_info}
```

**예시**:
```
⚠️ Interface Down: port1
WAN Interface: port1 | Status: DOWN

✅ Interface Up: port1
WAN Interface: port1 | Status: UP | 1000Mbps/full
```

**SPL 핵심 로직**:
```spl
| search msg="*down*" OR msg="*Down*"  # or "*up*" for recovery
| eval status = "DOWN"  # or "UP"
| rex field=interface "(?<if_type>port|wan|lan|dmz)"
| eval current_state = "DOWN"  # or "UP"
```

**State Tracker**: `interface_state_tracker.csv`
**Suppression**: 없음 (EMS 상태 추적 사용)

---

### Alert 013: SSL VPN Brute Force (SSL VPN 브루트포스)

**분류**: 임계값 기반
**상태**: ⚠️ Disabled (enableSched = 0)
**심각도**: High

**목적**: SSL VPN에 대한 브루트포스 공격 감지

**LogID**:
- `0101039424` - SSL VPN 로그인 실패
- `0101039426` - SSL VPN 인증 거부
- `0100032003` - SSL VPN 타임아웃

**감지 조건**:
- **임계값**: 10분 내 5회 이상 실패
- 다중 사용자 시도 감지 (user enumeration)

**상태 전환**:
- `ATTACK` ↔ `NORMAL`

**메시지 형식**:
```
{attack_type} from {source_ip} | {fail_count} failures | Users: {users} | Duration: {duration}
```

**예시**:
```
🚨 SSL VPN Attack: fw01
🟠 Brute Force from 203.0.113.50 | 12 failures | Users: admin, vpnuser1, test +more | Duration: 00:05:30
```

**SPL 핵심 로직**:
```spl
| stats count as fail_count, dc(user) as unique_users, values(user) as attempted_users
  by device, source_ip
| eval attack_type = case(
    fail_count >= 20, "🔴 Severe Brute Force",
    fail_count >= 10, "🟠 Brute Force",
    unique_users >= 5, "🟡 User Enumeration",
    fail_count >= 5, "⚠️ Suspicious Activity",
    true(), "🟢 Normal")
| eval current_state = if(fail_count >= 5, "ATTACK", "NORMAL")
```

**State Tracker**: `vpn_brute_force_state_tracker.csv`
**Suppression**: 없음 (EMS 상태 추적 사용)

**비활성화 이유**: VPN 로그 볼륨 높음. 필요 시 임계값 조정 후 활성화.

---

### Alert 014: (예약됨)

현재 사용되지 않음. 향후 확장을 위해 예약됨.

---

### Alert 015: Abnormal Traffic Spike (비정상 트래픽 급증)

**분류**: 임계값 기반
**상태**: ✅ Enabled
**심각도**: Medium

**목적**: 네트워크 트래픽 볼륨 급증 감지 (DDoS, 데이터 유출)

**LogID**:
- `0000000013` - Traffic start
- `0000000014` - Traffic end
- `0000000020` - Session start

**감지 조건**:
- **임계값**: 베이스라인 대비 **3배 이상 급증**
- 5분 단위 트래픽 집계
- 35분 롤링 윈도우로 베이스라인 계산

**상태 전환**:
- `SPIKE` ↔ `NORMAL`

**메시지 형식**:
```
{anomaly_type} from {source_ip} | Current: {traffic_mb}MB (Baseline: {baseline_mb}MB) | {multiplier}x spike | {sessions} sessions
```

**예시**:
```
📈 Traffic Spike: fw01
💾 Bandwidth Spike from 192.168.1.50 | Current: 1250MB (Baseline: 350MB) | 3.6x spike | 150 sessions
```

**SPL 핵심 로직**:
```spl
| bin _time span=5m
| stats sum(bytes) as total_bytes, count as session_count
  by _time, device, source_ip, protocol
| eventstats avg(total_bytes) as baseline_avg by device, source_ip, protocol
| eval spike_multiplier = round(total_bytes / baseline_avg, 1)
| eval current_state = if(spike_multiplier >= 3, "SPIKE", "NORMAL")
```

**State Tracker**: `traffic_spike_state_tracker.csv`
**Suppression**: 없음 (EMS 상태 추적 사용)

---

### Alert 016: System Reboot (시스템 재시작)

**분류**: 이벤트 기반
**상태**: ✅ Enabled
**심각도**: Low

**목적**: FortiGate 시스템 재시작/크래시 감지

**LogID**:
- `0100032002` - System reboot
- `0100032003` - System crash
- `0100032004` - Kernel panic

**감지 조건**:
- 시스템 재시작 이벤트 발생
- Reboot reason 파싱

**메시지 형식**:
```
{reboot_type} {event_type} | Initiated by: {initiated_by} | Reason: {reason} | Uptime: {uptime}
```

**예시**:
```
🔄 System Reboot: fw01
✅ Planned System Reboot | Initiated by: admin | Reason: Firmware upgrade | Uptime: 45 days
```

**SPL 핵심 로직**:
```spl
| eval reboot_type = case(
    like(lower(reboot_reason), "%upgrade%"), "✅ Planned",
    like(lower(reboot_reason), "%crash%"), "🔴 Crash",
    like(initiated_by, "admin"), "🟡 Manual",
    true(), "⚠️ Unexpected")
| eval criticality = case(
    event_type="System Crash", "🔴 CRITICAL",
    reboot_type="⚠️ Unexpected", "🟠 HIGH",
    true(), "🟢 INFO")
```

**State Tracker**: 없음 (Suppression 사용)
**Suppression**: 10분 (device)

---

### Alert 017: License Expiry Warning (라이센스 만료 경고)

**분류**: 임계값 기반
**상태**: ⚠️ Disabled (enableSched = 0)
**심각도**: Low

**목적**: FortiGate/FortiGuard 라이센스 만료 임박 알림

**LogID**:
- `0104043009` - FortiCare license expiry
- `0104043010` - FortiGuard license expiry
- `0100032011` - Subscription expiry

**감지 조건**:
- **임계값**: 30일 이내 만료

**상태 전환**:
- `WARNING` ↔ `NORMAL`

**메시지 형식**:
```
{license_category} expiring in {days} days (Expiry: {expiry_date}) | {action_required}
```

**예시**:
```
⚠️ License Expiring: FortiGuard Services
🔒 FortiGuard Services expiring in 14 days (Expiry: 2025-12-01) | ⚠️ Renew immediately
```

**SPL 핵심 로직**:
```spl
| rex field=msg "(?i)expires?\s+(?:in\s+)?(?<days_remaining>\d+)\s+days?"
| eval urgency = case(
    days_remaining <= 7, "🔴 URGENT (≤7 days)",
    days_remaining <= 14, "🟠 HIGH (≤14 days)",
    days_remaining <= 30, "🟡 MEDIUM (≤30 days)",
    true(), "🟢 OK")
| eval current_state = if(days_remaining <= 30, "WARNING", "NORMAL")
```

**State Tracker**: `license_state_tracker.csv`
**Suppression**: 없음 (EMS 상태 추적 사용)

**비활성화 이유**: 라이센스 이벤트 로그 빈도 낮음. 필요 시 활성화.

---

### Alert 018: FMG Out of Sync (FortiManager 동기화 실패)

**분류**: 이벤트 기반
**상태**: ✅ Enabled
**심각도**: Medium

**목적**: FortiManager와 FortiGate 간 설정 동기화 실패 감지

**LogID**:
- `0100044548-50` - FMG policy install
- `0104043545` - FMG sync status

**감지 조건**:
- 동기화 상태가 `OUT_OF_SYNC`, `INSTALL_FAILED`, `SYNC_FAILED`인 경우

**상태 전환**:
- `OUT_OF_SYNC` ↔ `SYNCHRONIZED`

**메시지 형식**:
```
FMG: {fmg_server} | Status: {sync_status} | {reason} | Target: {install_target}
```

**예시**:
```
📡 FMG Out of Sync: fw01 ↔️ fmg.example.com
FMG: fmg.example.com | Status: OUT_OF_SYNC | 📊 Revision Mismatch | Target: root
```

**SPL 핵심 로직**:
```spl
| eval sync_status = case(
    like(msg, "%out of sync%"), "OUT_OF_SYNC",
    like(msg, "%install%failed%"), "INSTALL_FAILED",
    like(msg, "%synchronized%"), "SYNCHRONIZED",
    true(), "UNKNOWN")
| eval current_state = if(sync_status="SYNCHRONIZED", "SYNCHRONIZED", "OUT_OF_SYNC")
| eval severity = case(
    sync_status="INSTALL_FAILED", "🔴 CRITICAL",
    sync_status="OUT_OF_SYNC", "🟠 HIGH",
    true(), "🟢 NORMAL")
```

**State Tracker**: `fmg_sync_state_tracker.csv`
**Suppression**: 15분 (device, fmg_server)

---

## 상태 추적 시스템

### EMS (Event Management System) 방식

모든 바이너리 상태 및 임계값 기반 알림은 **CSV 파일 기반 상태 추적**을 사용하여 중복 알림을 방지합니다.

### 상태 추적 로직

```spl
# 1. 현재 상태 계산
| eval current_state = if(condition, "FAIL", "OK")

# 2. 이전 상태 조회
| join type=left device [
    | inputlookup state_tracker
    | rename state as previous_state ]

# 3. 상태 변화 감지
| eval state_changed = if(isnull(previous_state) OR previous_state!=current_state, 1, 0)
| where state_changed=1

# 4. 상태 저장
| eval state = current_state
| outputlookup append=t state_tracker
```

### 11개 State Tracker 파일

| 파일명 | 알림 ID | 상태 값 | 용도 |
|--------|---------|---------|------|
| `vpn_state_tracker.csv` | 002 | DOWN, UP | VPN 터널 |
| `hardware_state_tracker.csv` | 007 | FAIL, OK | 하드웨어 |
| `ha_state_tracker.csv` | 008 | master, slave, standalone | HA 클러스터 |
| `interface_state_tracker.csv` | 012 | DOWN, UP | 네트워크 인터페이스 |
| `cpu_memory_state_tracker.csv` | 006 | ABNORMAL, NORMAL | CPU/Memory |
| `resource_state_tracker.csv` | 010 | EXCEEDED, NORMAL | 시스템 리소스 |
| `admin_login_state_tracker.csv` | 011 | ATTACK, NORMAL | 관리자 로그인 |
| `vpn_brute_force_state_tracker.csv` | 013 | ATTACK, NORMAL | VPN 브루트포스 |
| `traffic_spike_state_tracker.csv` | 015 | SPIKE, NORMAL | 트래픽 급증 |
| `license_state_tracker.csv` | 017 | WARNING, NORMAL | 라이센스 |
| `fmg_sync_state_tracker.csv` | 018 | OUT_OF_SYNC, SYNCHRONIZED | FMG 동기화 |

### CSV 파일 구조

```csv
device,component,state,last_seen,details
fw01,tunnel1,DOWN,1699123456,Phase1 negotiation failed
fw01,tunnel1,UP,1699123789,Tunnel restored
```

---

## LogID 참조

### 전체 LogID 목록 (알림 번호순)

#### Alert 001: Configuration Change
```
0100044546 - CLI configuration change
0100044547 - GUI configuration change
0100044548 - FMG policy install start
0100044549 - FMG policy install progress
0100044550 - FMG policy install complete
```

#### Alert 002: VPN Tunnel
```
0101037124 - VPN tunnel down
0101037125 - VPN Phase1 up
0101037131 - VPN Phase1 negotiation failed
0101037132 - VPN tunnel up
0101037134 - VPN Phase2 negotiation failed
```

#### Alert 006: CPU/Memory
```
0104043001 - CPU usage log
0104043002 - Memory usage log
```

#### Alert 007: Hardware
```
0103040001 - Hardware sensor failure
0103040002 - Hardware component error
0103040003 - Hardware sensor critical
0103040014 - Hardware sensor normal
0103040015 - Hardware component restored
```

#### Alert 008: HA State
```
0100020010 - HA state change
0104043544 - HA role change
0104043545 - HA sync event
```

#### Alert 010: Resource Limit
```
0104043003 - Disk/Memory limit
0104043004 - Session/CPU limit
```

#### Alert 011: Admin Login
```
0105032003 - Admin login failed
0105032004 - Admin logout
0105032005 - Admin permission denied
0105043001 - Admin authentication failed
```

#### Alert 012: Interface
```
0100032001 - Interface link down
0100020007 - Interface link up
```

#### Alert 013: SSL VPN
```
0101039424 - SSL VPN login failed
0101039426 - SSL VPN authentication denied
0100032003 - SSL VPN timeout
```

#### Alert 015: Traffic
```
0000000013 - Traffic start
0000000014 - Traffic end
0000000020 - Session start
```

#### Alert 016: System Reboot
```
0100032002 - System reboot
0100032003 - System crash
0100032004 - Kernel panic
```

#### Alert 017: License
```
0104043009 - FortiCare license expiry
0104043010 - FortiGuard license expiry
0100032011 - Subscription expiry
```

#### Alert 018: FMG Sync
```
0100044548 - FMG policy install start
0100044549 - FMG policy install progress
0100044550 - FMG policy install complete
0104043545 - FMG sync status
```

---

## 트러블슈팅

### 알림이 실행되지 않음

**증상**: 알림이 예약되었지만 실행되지 않음

**원인**:
- FortiGate 인덱스에 데이터 없음
- 알림이 비활성화됨 (enableSched = 0)
- 스케줄러 문제

**해결**:
```spl
# 1. FortiGate 데이터 확인
index=fw earliest=-1h
| stats count by logid, devname
| head 10

# 2. 알림 활성화 상태 확인
| rest /services/saved/searches
| search title="002_VPN_Tunnel_Down"
| table title, disabled, cron_schedule

# 3. 스케줄러 로그 확인
index=_internal source=*scheduler.log savedsearch_name="002_VPN_Tunnel_Down"
| table _time, status, result_count, run_time
```

---

### Slack으로 알림이 전송되지 않음

**증상**: 알림은 실행되지만 Slack 메시지 없음

**원인**:
- Webhook URL 미설정 또는 잘못됨
- Slack API 오류
- 네트워크 문제

**해결**:
```spl
# 1. Slack 전송 로그 확인
index=_internal source=*alert_actions.log action_name="slack" earliest=-1h
| table _time, savedsearch_name, action_status, stderr
| head 20

# 2. Webhook URL 확인
# cat /opt/splunk/etc/apps/security_alert/local/alert_actions.conf | grep webhook_url

# 3. 수동 테스트
# curl -X POST YOUR_WEBHOOK_URL \
#   -H 'Content-Type: application/json' \
#   -d '{"text":"Test message from Security Alert System"}'
```

---

### 중복 알림 발생

**증상**: 동일한 상태에 대해 반복 알림

**원인**:
- 상태 추적 CSV 파일 권한 문제
- `outputlookup` 실패
- CSV 잠금 오류

**해결**:
```bash
# 1. CSV 권한 확인
ls -la /opt/splunk/etc/apps/security_alert/lookups/*.csv
chown splunk:splunk /opt/splunk/etc/apps/security_alert/lookups/*.csv

# 2. 상태 추적 로그 확인
index=_internal source=*splunkd.log outputlookup error
| table _time, message

# 3. 상태 추적 데이터 확인
| inputlookup vpn_state_tracker
| stats count by device, state
```

---

### 상태 추적 CSV 크기 증가

**증상**: CSV 파일이 10MB 이상으로 증가

**원인**: 오래된 상태 데이터 누적

**해결**:
```spl
# 월별 정리 스케줄 (30일 이상 데이터 삭제)
| inputlookup vpn_state_tracker
| where last_seen > relative_time(now(), "-30d")
| outputlookup vpn_state_tracker

# 모든 상태 추적 파일 정리 스크립트
| rest /services/data/lookup-table-files
| search title="*state_tracker*"
| fields title
| map search="| inputlookup $title$ | where last_seen > relative_time(now(), \"-30d\") | outputlookup $title$"
```

---

## 참고 문서

- **README.md** - 사용자 문서 (한국어)
- **docs/DEPLOYMENT.md** - 배포 가이드 (한국어)
- **docs/QUICK-START.md** - 빠른 시작 가이드 (한국어)
- **docs/RELEASE-NOTES.md** - 릴리즈 노트 (한국어)

---

**버전**: v2.0.4
**마지막 업데이트**: 2025-11-07
**저장소**: https://gitlab.jclee.me/jclee/splunk
**알림 통계**: 15개 (활성 10개, 비활성 5개)
