# HospitalDR - tiered disaster-recovery drill (offline-capable)

Problem: keep critical hospital services recoverable even when the main data centre fails.
Synthetic data only. No internet is needed at runtime.

## Run (Windows / Mac / Linux)
```
pip install cryptography
python app.py
```
Open http://127.0.0.1:8000, sign in, click **Reset & load synthetic data**, then **Run full drill**.

Demo logins (synthetic environment only): `asha / admin@123` (Admin), `ravi / ops@123` (Operator), `meena / audit@123` (Auditor), `kiran / doc@123` (Clinician: can use patient records and admit patients, cannot run backups or restores).

**If you are upgrading from an earlier copy:** delete the old `data` folder (or click **Reset & load synthetic data** once), because the patient database now has more fields.

## Full patient records
The **Patient records** tab (Admin, Operator, Clinician) searches patients and shows one complete record assembled from all four tiers:
- Tier 0: identity, demographics, blood group, emergency contact, allergies (with an emergency summary at the top)
- Tier 1: diagnoses, latest vitals, medications, recent lab results
- Tier 2: billing and appointments
- Tier 3: archived reports

Each part carries its tier's status. During an outage you can watch the record come back in priority order: turn on **Slow-motion restore (demo pacing)**, click **Fail main data centre**, then **Restore**. Tier 0 returns first. Slow-motion adds 3 s pauses between tiers and the report says so. Opening a record is written to the audit log (the Auditor cannot view records). **Admit a new patient** adds a made-up patient to Tier 0 (the app adds the SYNTH- prefix); it is blocked while the primary is down, as it would be in a real outage.

Optional settings (environment variables):
- `DR_DIR=D:\dr_site` sends backups to another disk/USB.
- `HTTPS=1` serves the dashboard over TLS with a self-signed certificate (data in transit). Windows PowerShell: `$env:HTTPS=1; python app.py`. The browser warns once; choose Advanced, then Continue.
- `REGION_PRIMARY` / `REGION_DR` change the declared data-residency labels (defaults: Mumbai / Hyderabad, simulated).
- `HOST=0.0.0.0` exposes it on the hospital LAN (demo PINs only; change them first).

## Mapping to the solution goals and success metrics
| Requirement | Where it is |
|---|---|
| Tiered recovery objective per system, justified clinically and by criticality | **Recovery objectives** tab (9 systems, 4 tiers) |
| Full patient information protected by the tiers | **Patient records** tab (identity, allergies, vitals, medications, labs, billing, archive) |
| Repeatable by a 3-person team, no specialist knowledge | One-click **Run full drill**, **Runbook** tab, **Drill history** card |
| 1. Measured recovery time and data loss vs target, live | Drill report table (RTO, RPO, rows lost, PASS/FAIL) |
| 2. Monthly standby cost relative to hospital budget | **Cost estimate** tab (editable assumptions) |
| 3. Runbook completeness - can a judge follow it | **Runbook** tab and `RUNBOOK.md` |
| 4. How failures are detected and reported | Health monitor (1 s), event log with detection latency, **Download report (.md)** |
| Data in India, encrypted at rest and in transit, synthetic only | **Security** tab (live checks; residency is declared, TLS via `HTTPS=1`) |
| Backup/restore limited to named roles and logged | Roles enforced on the server; **Audit log** tab (hash-chained) |

## What is real vs simulated
- Real: tiered SQLite databases, encrypted (Fernet: AES + HMAC) and checksummed backups, versioning, verification on restore, measured RTO/RPO, hash-chained audit log, role checks enforced server-side.
- Simulated: the "main data centre" and "DR site" are two folders on one machine; failure = deleting the primary files. Production would use two physical sites in India (for example Mumbai and Hyderabad regions) with TLS in transit and keys in a KMS.
- Scheduled demo backup intervals are shortened (10-60 s). The real targets/schedules are shown on each tier card.

## Files
- `app.py` - server, backup, restore, drill, monitor, audit log
- `static/index.html` - dashboard (no external libraries or CDNs, works offline)
- `RUNBOOK.md` - step-by-step recovery procedure


