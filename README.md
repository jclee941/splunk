# Splunk Security Alert System

[![Version](https://img.shields.io/badge/version-2.0.4-blue.svg)](./security_alert/app.manifest)
[![Splunk](https://img.shields.io/badge/Splunk-8.0%2B-orange.svg)](https://www.splunk.com/)
[![Python](https://img.shields.io/badge/python-3.7%2B-green.svg)](https://www.python.org/)
[![License](https://img.shields.io/badge/license-Internal-lightgrey.svg)](#)

> Production-ready Splunk app for **FortiGate security monitoring** with state-aware alerting and Slack integration.

FortiGate 방화벽의 보안 이벤트를 실시간 감지하고 **EMS(Event-Metric-State) 패턴**으로 중복 알림 없이 Slack으로 전송하는 Splunk 앱입니다.

---

## Highlights

- **15 Production Alerts** — VPN, HA, hardware, resource, brute-force, traffic, license, FMG sync
- **Zero Duplicate Notifications** — State-tracking via 11 CSV lookups, alerts fire only on state change
- **SPL-First Architecture** — All logic in Splunk searches, no external processing required
- **Air-Gapped Ready** — All Python dependencies bundled (`requests`, `urllib3`, `certifi`, ...)
- **Single-Line Slack Messages** — Optimized for mobile push notifications (≤200 chars)
- **Macro-Based Configuration** — Index name, LogID groups, thresholds centralized

---

## Repository Layout

```
splunk/
├── security_alert/        # Splunk app (deployable)
│   ├── default/           # All 15 alert definitions, macros, transforms
│   ├── bin/               # Slack alert action (slack.py)
│   ├── lib/python3/       # Bundled Python dependencies
│   ├── lookups/           # State trackers + LogID map (6091 entries)
│   └── metadata/
├── docs/                  # Deployment, release notes, quick start
├── demo/                  # Demonstration assets
├── resume/                # Architecture deep-dives, API, troubleshooting
├── tests/                 # Validation scripts
└── CLAUDE.md              # AI assistant guidance (full architecture reference)
```

---

## Quick Start

### 1. Install the app

```bash
# Copy to Splunk apps directory
cd /opt/splunk/etc/apps/
tar -xzf security_alert-v2.0.4-production.tar.gz
chown -R splunk:splunk security_alert/

# Restart Splunk
/opt/splunk/bin/splunk restart
```

### 2. Configure Slack webhook (REQUIRED)

```bash
mkdir -p security_alert/local
cat > security_alert/local/alert_actions.conf <<'EOF'
[slack]
param.webhook_url = https://hooks.slack.com/services/YOUR/WEBHOOK/URL
EOF
chmod 600 security_alert/local/alert_actions.conf
```

### 3. Override FortiGate index (optional)

```ini
# security_alert/local/macros.conf
[fortigate_index]
definition = index=your_firewall_index
```

See [`docs/QUICK-START.md`](./docs/QUICK-START.md) and [`docs/DEPLOYMENT.md`](./docs/DEPLOYMENT.md) for full setup.

---

## Alert Catalog (15 alerts)

### Binary State (4) — alerts on UP↔DOWN transitions

| Alert | Detects | Severity |
|---|---|---|
| `002_VPN_Tunnel_Down/Up` | VPN tunnel state change | Critical |
| `007_Hardware_Failure/Restored` | PSU / fan / disk failures | Critical |
| `008_HA_State_Change` | HA primary/secondary role swap | High |
| `012_Interface_Down/Up` | Network interface state | Medium-High |

### Threshold-Based (6) — alerts when crossing baseline / static threshold

| Alert | Threshold | State |
|---|---|---|
| `006_CPU_Memory_Anomaly` | ±20% deviation from 24h baseline | ABNORMAL |
| `010_Resource_Limit` | ≥75% utilization | EXCEEDED |
| `011_Admin_Login_Failed` | ≥3 failures / 10min | ATTACK |
| `013_SSL_VPN_Brute_Force` | ≥5 failures / 10min | ATTACK |
| `015_Abnormal_Traffic_Spike` | 3× baseline traffic | SPIKE |
| `017_License_Expiry_Warning` | ≤30 days to expiry | WARNING |

### Event-Based (5) — fire on event with suppression window

- `001_Config_Change` (10min suppress)
- `016_System_Reboot` (30min suppress)
- `018_FMG_Out_Of_Sync` (15min suppress)
- (additional event-based alerts in `default/savedsearches.conf`)

---

## EMS State-Tracking Pattern

The core pattern that prevents duplicate notifications:

```spl
`fortigate_index` `logids_<category>`
| eval current_state = if(condition, "FAIL", "OK")
| stats latest(*) as * by device, component
| join type=left device component [
    | inputlookup state_tracker | rename state as previous_state
]
| eval state_changed = if(isnull(previous_state) OR previous_state!=current_state, 1, 0)
| where state_changed=1                          ◄── only fire on transition
| eval state = current_state
| outputlookup append=t state_tracker            ◄── append=t for atomic write
```

**Why it matters:**
- Without `where state_changed=1` → Slack flooded every cron tick
- Without `append=t` → CSV lock errors with concurrent alerts

Full architectural rationale in [`CLAUDE.md`](./CLAUDE.md).

---

## State Trackers (11 CSV files)

Located in `security_alert/lookups/`:

```
vpn_state_tracker.csv          hardware_state_tracker.csv
ha_state_tracker.csv           interface_state_tracker.csv
cpu_memory_state_tracker.csv   resource_state_tracker.csv
admin_login_state_tracker.csv  vpn_brute_force_state_tracker.csv
traffic_spike_state_tracker.csv license_state_tracker.csv
fmg_sync_state_tracker.csv
```

Plus the master reference: `fortigate_logid_notification_map.csv` (6091 LogID → category/severity mappings).

---

## Validation

```bash
# Validate config syntax
splunk btool savedsearches list --debug

# Test alert query directly
splunk search "`fortigate_index` `logids_vpn_tunnel` earliest=-1h | head 10"

# Inspect scheduler execution
splunk search 'index=_internal source=*scheduler.log savedsearch_name="*Alert*" | stats count by status'

# Inspect Slack delivery
splunk search 'index=_internal source=*alert_actions.log action_name="slack" | stats count by action_status'
```

See [`tests/README.md`](./tests/README.md) for the full validation harness.

---

## Documentation

| Document | Purpose |
|---|---|
| [`CLAUDE.md`](./CLAUDE.md) | Full architecture, SPL patterns, troubleshooting |
| [`docs/QUICK-START.md`](./docs/QUICK-START.md) | 5-minute setup |
| [`docs/DEPLOYMENT.md`](./docs/DEPLOYMENT.md) | Production deployment guide |
| [`docs/RELEASE-NOTES.md`](./docs/RELEASE-NOTES.md) | Version history |
| [`docs/ALERT-REPOSITORY-XWIKI.md`](./docs/ALERT-REPOSITORY-XWIKI.md) | XWiki alert documentation |
| [`resume/ARCHITECTURE.md`](./resume/ARCHITECTURE.md) | System architecture |
| [`resume/API.md`](./resume/API.md) | Alert API reference |
| [`resume/TROUBLESHOOTING.md`](./resume/TROUBLESHOOTING.md) | Common issues + fixes |

---

## Security Notes

- `local/alert_actions.conf` (Slack webhook URL) **must** be `chmod 600`
- `bin/fortigate_auto_response.py` is **disabled** but contains placeholder credentials — review before production
- `.gitignore` excludes `local/*`, `*.log`, `__pycache__/`, and runtime state CSVs

---

## Version

**v2.0.4** (2025-11-07)

- EMS state tracking applied to all 15 alerts (11 trackers)
- Official Slack Alert Action integration (plain-text format)
- All 15 alerts production-enabled (incl. Alert 018 FMG sync)
- Bundled Python dependencies for air-gapped deployments

---

## License

Internal use. See repository owner for redistribution terms.


<!-- LLM review probe 1777811213 — auto-removed after verification -->
