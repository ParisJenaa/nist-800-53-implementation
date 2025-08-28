# Security Assessment Report (SAR) — MediTrack

## Method
Mix of Inspect, Test, and Interview. Evidence stored under /artifacts.

---

## AC-2 — Account Management
Objective: Accounts align to HR and follow JML (joiner/mover/leaver).
Steps:
1) Export Cognito users; export IAM users (CSV).  
2) Compare to HR roster (sample 10).  
3) Verify 2 recent leavers are disabled ≤24h.  
Expected: No orphans; leavers disabled on time.  
Result: Pass/Partial with notes.

## AC-3 — Access Enforcement
Objective: RBAC & data scoping enforced.
Steps:
1) Call admin-only API with Clinician token → expect 403.  
2) Clinician from Clinic A attempts to read Clinic B patient → expect 403.  
Evidence: Postman screenshots; Lambda logs (deny).  
Result: Pass.

## AC-17 — Remote Access
Objective: Admin access requires MFA and approved IPs.
Steps:
1) From non-corporate IP, attempt console login → expect deny.  
2) From corporate VPN, login without MFA → expect deny.  
Evidence: CloudTrail `ConsoleLogin` events and IAM policy.  
Result: Pass.

## IA-2 — Identification & Authentication
Objective: MFA enforced for admin and high-risk app actions.
Steps:
1) Create test Clinic Admin; confirm MFA setup required.  
2) Attempt data export without MFA challenge → expect block.  
Evidence: Screenshots; app logs show challenge.  
Result: Pass.

## IA-5 — Authenticator Management
Objective: Validate password/key rotation and secrets rotation.
Steps:
1) Check Secrets Manager rotation status (DB secret last changed <30d).  
2) Confirm CI access key rotated <90d.  
Evidence: Screenshots; ticket references.  
Result: Pass/Partial.

## AU-2 — Event Logging
Objective: Key events captured without PHI.
Steps:
1) Trigger 3 failed logins; verify CloudWatch entry within 5 min.  
2) Review sample logs to confirm no PHI fields.  
Result: Pass.

## AU-6 — Audit Review/Reporting
Objective: Alerts on suspicious activity.
Steps:
1) Generate 6 failed logins in 5 min → alarm fires → ticket created.  
2) Run Logs Insights saved query for privilege changes.  
Evidence: Alarm, ticket ID, query results.  
Result: Pass.

## CM-2 — Baseline Configuration
Objective: Settings match baseline doc.
Steps:
1) Compare RDS parameter group vs Baseline-Config.md (3 parameters).  
2) Verify CloudTrail is org/all-region.  
Result: Pass/Partial.

## CM-6 — Configuration Settings
Objective: Enforce secure values.
Steps:
1) Confirm S3 Block Public Access = ON.  
2) Confirm CloudFront min TLS ≥ 1.2 and HSTS enabled.  
Result: Pass.

## CP-9 — System Backup
Objective: Restores are successful within RTO/RPO.
Steps:
1) Restore last nightly snapshot to dev; run integrity checks (row counts).  
2) Record restoration time vs RTO (4h).  
Result: Pass if <4h and data intact.

## IR-4 — Incident Handling
Objective: Team can execute playbook.
Steps:
1) Tabletop: credential compromise scenario; follow checklist.  
2) Verify evidence capture and containment timeline.  
Result: Pass/Improvements noted.

## IR-6 — Incident Reporting
Objective: Timely, correct notifications.
Steps:
1) Draft mock SEV1 report using template; legal review within 2h.  
Result: Pass.

## SA-9 — External Services
Objective: Vendor risk controls current.
Steps:
1) Review SOC 2 date ≤12 months; confirm DPA, right to audit.  
2) Verify API TLS config (A or better).  
Result: Pass/Partial.

## SC-13 — Cryptographic Protection
Objective: Crypto enforced in transit/at rest.
Steps:
1) Attempt `sslmode=disable` to DB → connection fails.  
2) Confirm KMS keys have rotation enabled; review key policies.  
Result: Pass.

## SI-2 — Flaw Remediation
Objective: Vulnerabilities remediated per SLA.
Steps:
1) Review last month’s CI scans; sample 3 criticals → fixed ≤7d.  
2) Confirm RDS version updated in last cycle.  
Result: Pass/Partial.
