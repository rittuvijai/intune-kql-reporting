# Intune KQL Reporting Library

Advanced KQL queries for Microsoft Intune reporting in Azure Log Analytics. Built for endpoint administrators who are tired of clicking through the Intune admin center for answers that a query gives in seconds.

## What this is

A curated, production-minded set of KQL queries against the Intune tables in Log Analytics:

| Table | What it tells you |
|---|---|
| `IntuneDevices` | Point-in-time device inventory snapshot (compliance, OS, join type, encryption, ownership) |
| `IntuneDeviceComplianceOrg` | Compliance evaluation history per device |
| `IntuneOperationalLogs` | Enrollment / app / policy operations with success-failure results (JSON `Properties`) |

## Prerequisites

1. In the Intune admin center: **Reports > Diagnostic settings** — send to a Log Analytics workspace.
2. Enable the log categories: **Devices**, **DeviceComplianceOrg**, **OperationalLogs** (AuditLogs optional).
3. Allow up to 24h for the first data to land.

## Query design notes

- `IntuneDevices` is a **snapshot export**, not an event stream. Every query takes the latest record per device with `summarize arg_max(TimeGenerated, *) by DeviceId`.
- `LastContact` / `CreatedDate` arrive as **strings** — queries convert with `todatetime()`.
- `IntuneOperationalLogs.Properties` is a JSON string — queries parse with `parse_json()`. Exact property names inside `Properties` can vary by event; each query notes where to adjust.
- Tunables (lookback windows, OS build lists, thresholds) are declared as `let` statements at the top of each query.
- String comparisons use case-insensitive operators (`=~`, `in~`) because Intune is inconsistent with casing.

## Queries

| # | File | Answers |
|---|---|---|
| 01 | `device-compliance-posture.kql` | Compliance % by OS — the one-slide posture view |
| 02 | `noncompliant-devices-actionable.kql` | Actionable non-compliant list, stalest first |
| 03 | `stale-devices-cleanup-candidates.kql` | 30/60-day silent devices — cleanup targets |
| 04 | `encryption-coverage.kql` | BitLocker/FileVault coverage by OS and ownership |
| 05 | `os-version-drift.kql` | Devices behind current OS builds — patch targeting list |
| 06 | `enrollment-failures.kql` | Enrollment failures grouped by error code |
| 07 | `app-install-failures.kql` | Top failing apps with error codes and affected device counts |
| 08 | `compliance-trend-30d.kql` | 30-day compliance trend (timechart-ready) |
| 09 | `join-type-ca-readiness.kql` | Join-type distribution + non-compliant Entra-joined devices (CA readiness) |
| 10 | `mobile-risk-posture.kql` | Jailbroken/rooted or unsupervised mobile devices |

## Usage

Paste any `.kql` file into Log Analytics / Azure Monitor Logs query editor, adjust the `let` tunables, and run. Queries ending in `| render timechart` are built for workbook tiles.

## Author

Rittu Vijai — Senior Cloud Ops & Automation Engineer (Intune · Entra ID · Azure Monitor/KQL)
