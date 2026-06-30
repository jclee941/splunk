# Architecture Summary

## System Overview

**Splunk Security Alert System v2.0.4** - FortiGate security event monitoring with Slack notifications.

> Component-level architecture showing data flow across system boundaries.

#### Diagram summary 1

- Type: flowchart
- Component: FortiGate syslog (FG)
- Component: Splunk Index (IDX)
- Component: Search Scheduler (SCH)
- Component: Macros (MAC)
- Component: Alert Definitions (ALT)
- Component: State Trackers (STT)
- Component: Python Scripts (PYT)
- Component: Slack Webhook (SLK)
- FortiGate syslog (FG) -> Splunk Index (IDX)
- Splunk Index (IDX) -> Search Scheduler (SCH)
- Search Scheduler (SCH) -> Alert Definitions (ALT)
- Alert Definitions (ALT) -> Macros (MAC)
- Alert Definitions (ALT) -> State Trackers (STT)
- Alert Definitions (ALT) -> Python Scripts (PYT)
- Python Scripts (PYT) -> Slack Webhook (SLK)

## Core Components

### 1. Alert Engine
- **15 Alert Definitions** in `default/savedsearches.conf`
- **SPL-based** real-time monitoring (1-minute intervals)
- **EMS Pattern** for state-based deduplication

### 2. State Tracking System
- **11 CSV-based state trackers** in `lookups/`
- **Binary states**: UP/DOWN, FAIL/OK, NORMAL/ABNORMAL
- **Atomic updates** via `outputlookup append=t`

### 3. Notification Pipeline
- **Slack Integration** via official alert action
- **Plain text formatting** (single-line messages)
- **Channel routing**: #security-firewall-alert

### 4. Data Sources
- **FortiGate syslog** → Splunk index (`index=fw`)
- **6,091 LogID mappings** in CSV lookup
- **Macro-based** LogID grouping (15 categories)

## Architecture Patterns

### State Tracking Pattern (EMS)
#### Diagram summary 2

- Type: flowchart
- Events (E) -> Current State (CS)
- Current State (CS) -> Compare with Previous (CP)
- Compare with Previous (CP) -> State Changed? (SC)
- State Changed? (SC) -> Alert + Update State (A)
- State Changed? (SC) -> Skip (SK)


### Alert Workflow
#### Diagram summary 3

- Type: flowchart
- SPL Search (SS) -> Enrich (EN)
- Enrich (EN) -> Deduplicate (DD)
- Deduplicate (DD) -> State Compare (SC)
- State Compare (SC) -> Format Message (FM)
- Format Message (FM) -> Slack (SL)


## Key Technologies

- **Splunk Enterprise** 9.x+ (alert engine)
- **Python 3** (Slack integration, bundled dependencies)
- **CSV Lookups** (state persistence)
- **Slack Webhooks** (notification delivery)

## Dependencies

**All bundled** in `lib/python3/` (air-gapped deployment):
- requests (2.32.5)
- urllib3 (2.5.0)
- certifi (2025.10.5)
- charset-normalizer (3.4.4)
- idna (3.11)

## Deployment Model

- **Self-contained** Splunk app
- **No external services** required
- **Configuration via files** (no UI setup)
- **Horizontal scaling** via search head clustering

## Performance Characteristics

- **Real-time latency**: <60 seconds (rt-10m windows)
- **State tracker size**: 100-1000 rows per CSV
- **Alert frequency**: 1-minute cron (15 concurrent searches)
- **Slack rate limit**: Respects webhook limits

## Security

- **Webhook URLs** in `local/alert_actions.conf` (gitignored)
- **No credentials** in code (FortiManager integration disabled)
- **Read-only** Splunk searches (no data modification)
