# AC-2 — Account Management
Family: Access Control
Intent: Ensure only authorized users and admins have accounts; manage lifecycle (joiner/mover/leaver).
Design:
- App users in **Cognito**; admins in **IAM** (no root use). Break-glass admin is password-vaulted, checked quarterly.
- **Joiner:** HR ticket → manager approval → account created in Cognito (role: Clinician/ClinicAdmin).  
- **Mover:** RBAC change recorded in `RBAC-Mapping.xlsx`; effective within 24h.  
- **Leaver:** HR exit notice triggers disable within **24h** (Cognito & any IAM user).
- Orphan detection: monthly script lists users with no recent login (90d) for review.
Procedures:
1) Verify approval → create user → assign role/group.  
2) For movers: update Cognito group; for admins: change IAM group (least privilege).  
3) For leavers: disable user; remove keys; schedule delete after 30d.
Configuration/Evidence:
- Access Control Policy; screenshots of Cognito group rules; IAM groups + policy JSON (`artifacts/configs/iam/`).
- HR ticket samples (redacted).
Verification: Quarterly access review; reconcile against HR roster; spot-check 5 users.  
Role: Security Admin  
Status: Implemented

# AC-3 — Access Enforcement
Family: Access Control
Intent: Enforce least privilege and role-based access at API and data layers.
Design:
- API Gateway authorizer validates Cognito JWT scopes (e.g., `role:clinician`, `role:admin`).
- Lambda checks row-level access (clinic_id scoping).  
- IAM policies restrict DevOps/Sec Admin to project resources via tags (`Project=MediTrack`).
Procedures:
- RBAC mapping maintained in repo; code enforces scope checks in services.
- Database roles: `app_read`, `app_write` with **parameterized queries** only.
Evidence:
- Code snippet showing scope check; DB GRANT statements; IAM policy with condition on `aws:ResourceTag/Project`.
Verification: Unit tests for forbidden actions; manual test: try admin-only endpoint as Clinician → expect 403.  
Role: Dev Team + Sec Admin  
Status: Implemented

# AC-17 — Remote Access
Family: Access Control
Intent: Secure administrative access to cloud resources.
Design:
- Admin console/API access restricted to **corporate VPN IPs** (AWS IAM condition on `aws:SourceIp`) and **MFA required**.
- No SSH/RDP in design (serverless). Bastion/SSM not required; if future EC2 used → **SSM Session Manager** only.
Procedures:
- Maintain allowed CIDR list; rotate VPN certificates quarterly.
- Break-glass exception (mobile incident) logged and approved by IR Lead.
Evidence:
- IAM policy conditions; screenshot of org SCP or identity policy; VPN ACL docs.
Verification: Attempt console login from non-allowed IP → denied; verify CloudTrail shows denied `ConsoleLogin`.  
Role: Security Admin  
Status: Implemented

# IA-2 — Identification & Authentication (Org Users)
Family: Identification & Authentication
Intent: Positively identify users/admins; require MFA.
Design:
- Cognito: email as username, **TOTP MFA required** for Clinic Admin; optional for Clinician with risk-based enforcement (requires MFA for export/download functions).
- IAM: **MFA enforced** via policy; access keys disabled for console-only roles.
Procedures:
- Password policy: min 14 chars, deny common passwords, lockout after 10 attempts/5 min.
- Device re-registration on suspected compromise; recovery via helpdesk identity proofing.
Evidence:
- Cognito password/MFA settings screenshot; IAM `aws:MultiFactorAuthPresent` policy.
Verification: Create test user; confirm MFA prompt; attempt export without MFA → blocked.  
Role: Security Admin  
Status: Implemented

# IA-5 — Authenticator Management
Family: Identification & Authentication
Intent: Govern password and key lifecycle.
Design:
- Cognito password policy (14+, upper/lower/number/special, 90-day rotation for admins).  
- IAM access keys not used except for CI user; CI key rotated every 90 days via pipeline.
- Secrets in **Secrets Manager** with automatic rotation where supported (DB creds every 30d via Lambda).
Procedures:
- Disable default passwords; enforce TOTP seed protection; revoke authenticators on exit.
Evidence:
- Secrets Manager rotation config; rotation Lambda code stub; key rotation tickets.
Verification: Show last rotation dates; attempt to use expired secret (should fail).  
Role: Security Admin + DevOps  
Status: Implemented

# AU-2 — Event Logging
Family: Audit & Accountability
Intent: Define and collect security-relevant events.
Design:
- **CloudTrail** organization trail (all regions, all accounts if org’d), S3 bucket with KMS; log file validation on.
- **CloudWatch Logs** for API Gateway and Lambda; **VPC Flow Logs** for DB subnets.
- Application logs include user_id, clinic_id, request_id (no ePHI).
Procedures:
- Retention: 365 days (hot); archive to S3 Glacier after 365d.  
- Pseudonymize identifiers where possible; never log PHI.
Evidence:
- Audit Events Matrix; CloudTrail “ON” screenshot; sample masked log lines.
Verification: Trigger failed login → observe log within 5 min; confirm no PHI fields present.  
Role: Sec Analyst  
Status: Implemented

# AU-6 — Audit Review, Analysis, and Reporting
Family: Audit & Accountability
Intent: Review logs and generate alerts for suspicious activity.
Design:
- CloudWatch Metric Filters + Alarms (e.g., `ConsoleLogin` failure burst; >5 failed logins in 5 min).
- Weekly **Logs Insights** saved queries (privilege changes, large data exports).
Procedures:
- Daily triage of alarms; weekly trend review; monthly metrics sent to leadership: MTTD/MTTR, alert counts, false positives.
Evidence:
- Alarm JSON (`artifacts/configs/alarms/FailedLogins.json`), saved query screenshots, triage runbook.
Verification: Simulate burst of failed logins → alarm → incident ticket created automatically.  
Role: Sec Analyst  
Status: Implemented

# CM-2 — Baseline Configuration
Family: Configuration Management
Intent: Establish and document standard, approved configurations.
Design:
- Documented baseline in `artifacts/baseline/Baseline-Config.md` (RDS params, KMS policies, CloudTrail on, S3 block-public).
- Terraform planned for future; currently change tickets + manual change log.
Procedures:
- Any drift requires RFC ticket; security review for changes impacting crypto/logging/network.
Evidence:
- Baseline doc; screenshot of RDS parameter group; S3 Block Public Access settings.
Verification: Monthly drift check against baseline (Config snapshots); sample 5 settings.  
Role: DevOps + Sec Admin  
Status: Partial (IaC planned)

# CM-6 — Configuration Settings
Family: Configuration Management
Intent: Enforce secure parameter values.
Design (selected settings):
- Cognito session timeout **15 min**; password policy as IA-5.  
- RDS: `log_connections=1`, `log_disconnections=1`, `log_min_duration_statement=250ms`.  
- S3 buckets: **Block Public Access = ON**, default encryption KMS.  
- CloudFront: TLS 1.2 min; HSTS max-age 1 year; WAF rules enabled.
Procedures:
- Document settings → peer review → apply in console → record in change log.
Evidence:
- Screenshots of each setting; exported RDS parameter group; WAF rule list.
Verification: Inspect settings; attempt HTTP (non-TLS) to CloudFront (expect redirect/deny).  
Role: DevOps  
Status: Implemented

# CP-9 — System Backup
Family: Contingency Planning
Intent: Ensure data backup and restoration.
Design:
- RDS automated backups (35-day retention) + **weekly cross-region snapshot** to us-west-2 (KMS replica).  
- App artifacts (static UI) versioned in S3; configuration backups weekly.
Procedures:
- Quarterly restore test to isolated dev VPC; run integrity checks (row counts, hash totals).
Evidence:
- Backup schedule screenshot; restore test notes (`artifacts/backups/Restore-2025-07.md`); SQL verification script.
Verification: Perform restore; app connects; integrity checks pass; record RTO/RPO achieved.  
Role: DBA + DevOps  
Status: Implemented

# IR-4 — Incident Handling
Family: Incident Response
Intent: Standardize detect→contain→eradicate→recover→lessons learned.
Design:
- Severity matrix SEV1–SEV3; playbooks for credential compromise, data exfil indicator, DB corruption.  
- “War room” Slack channel #ir-active; ticket auto-created by CloudWatch alarm via SNS + webhook.
Procedures:
- First responder checklist; evidence collection steps; comms templates.
Evidence:
- IR Plan, playbooks, contact list (redacted); last tabletop notes.
Verification: Quarterly tabletop; measure MTTA/MTTR; update playbooks.  
Role: IR Lead  
Status: Implemented

# IR-6 — Incident Reporting
Family: Incident Response
Intent: Timely reporting to leadership and affected customers/partners.
Design:
- SEV1: internal leadership within 1 hour; customers within contractual SLAs; legal review required.  
- Single mailbox `security@<org>` + on-call rotation.
Procedures:
- Use pre-approved templates; track disclosures in ticket; store in `artifacts/ir/Reports/`.
Evidence:
- Redacted example notification; timestamped ticket trail.  
Verification: During tabletop, complete mock notifications and review for completeness.  
Role: IR Lead + Legal  
Status: Implemented

# SA-9 — External System Services
Family: System & Services Acquisition
Intent: Manage risk from third-party services (billing processor).
Design:
- Data flow limited to **tokenized** payment references; no card data stored.  
- Contract requires: encryption in transit/at rest, breach notice ≤72h, right to audit, data location (US), sub-processor list.
Procedures:
- Annual security questionnaire review; SOC 2 / ISO certs collected; track open risks in vendor register.
Evidence:
- Vendor risk review (`artifacts/vendor/Billing-VRR-2025.pdf`), DPA, data flow diagram.
Verification: Check attestations are current; verify TLS settings for API (Qualys/ssllabs screenshot).  
Role: GRC Analyst  
Status: Implemented

# SC-13 — Cryptographic Protection
Family: System & Communications Protection
Intent: Protect data at rest and in transit using approved crypto.
Design:
- **TLS 1.2+** everywhere (CloudFront, API GW).  
- KMS CMKs for RDS, S3, CloudTrail, Secrets; key rotation annually; least-priv key policies.  
- DB connections require TLS; enforce via parameter group and app connection string.
Procedures:
- Review cipher suites quarterly; rotate certs via ACM renewal; approve any key policy change.
Evidence:
- KMS key policies (`artifacts/configs/kms/`); ACM cert details; app connection string with `sslmode=require`.
Verification: Attempt non-TLS DB connection (should fail); confirm CloudFront minimum TLS set to 1.2 or 1.3.  
Role: Sec Admin  
Status: Implemented

# SI-2 — Flaw Remediation
Family: System & Information Integrity
Intent: Discover and remediate vulnerabilities.
Design:
- CI (GitHub Actions) runs `npm audit --production` + SCA (e.g., CodeQL) on PRs; fail on Critical.  
- Patch SLAs: Critical=7d, High=30d, Medium=90d.  
- RDS minor versions applied in maintenance window monthly.
Procedures:
- Weekly vuln review; exceptions documented with compensating controls.
Evidence:
- CI run screenshot; vulnerability report; changelog showing fixes.  
Verification: Review past 30 days of findings; sample 3 criticals → verify closed ≤7d.  
Role: DevOps + Sec Analyst  
Status: Implemented
