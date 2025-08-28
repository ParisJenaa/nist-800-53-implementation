# System Security Plan (SSP-lite) — MediTrack

## 1) System Purpose
MediTrack is a multi-tenant web app for small clinics to store, view, and update electronic Protected Health Information (ePHI). Primary users are Clinic Admins and Clinicians.

## 2) Data Types & Impact (FIPS 199)
- ePHI (patient demographics, diagnosis codes, visit notes) → **C: High, I: Moderate, A: Moderate**
- PII (clinic staff names/emails) → **C: Moderate**
Overall impact level targeted: **Moderate** → apply **NIST SP 800-53 Rev. 5 Moderate** baseline subset.

## 3) System Boundary (what’s in)
- **Web UI:** S3 (static) + CloudFront (TLS via ACM)
- **API:** Amazon API Gateway (REST) → AWS Lambda (Node.js 20)
- **Data:** Amazon RDS for PostgreSQL 15 (private subnets, no public endpoint)
- **Identity:** 
  - App users: Amazon Cognito (user pools)
  - Cloud admins: AWS IAM (least privilege, MFA)
- **Secrets & Crypto:** AWS Secrets Manager; AWS KMS CMKs for RDS, S3, Secrets, CloudTrail
- **Logging/Monitoring:** CloudTrail (all regions), CloudWatch Logs, VPC Flow Logs (DB subnets), CloudWatch Alarms
- **Networking:** One VPC, private subnets for RDS/Lambda; NAT GW for outbound updates; WAF on CloudFront
- **Region:** us-east-1

Out of scope: third-party billing processor (card payments), marketing site (separate account).

## 4) Trust Zones
- **Public Zone:** CloudFront distribution + S3 website.
- **App Zone (managed):** API Gateway + Lambda.
- **Data Zone (restricted):** RDS in private subnets; only Lambda security group can connect on 5432.
- **Admin Zone:** AWS Console/API access from corporate IP ranges / VPN only.

## 5) User Roles
- **Clinician:** read/write patient records for assigned clinic
- **Clinic Admin:** manage clinicians for their clinic; export limited reports
- **Security Admin (cloud):** manage IAM/Cognito, KMS, logging
- **DevOps:** CI/CD, patching, backups

## 6) Data Flow (high-level)
Browser → HTTPS (CloudFront) → API Gateway (JWT from Cognito) → Lambda (role-based authorization) → RDS (parameterized queries). Audit events → CloudWatch Logs. Admin actions → CloudTrail.

## 7) Key Security Assumptions
- All endpoints enforce **TLS 1.2+** (preferred 1.3).
- **MFA** mandatory for IAM users; Cognito MFA for privileged app functions.
- **No hard-coded secrets**; all credentials in Secrets Manager.
- **Backups nightly**; cross-region snapshot weekly; quarterly restore tests.

## 8) Inherited/Shared Controls (awareness)
AWS provides physical, environmental, and many platform controls. MediTrack implements application-layer, identity, data protection, and monitoring controls.

## 9) Attachments (artifacts/)
- Architecture diagram (`artifacts/diagrams/Architecture.png`)
- RBAC mapping (`artifacts/rbac/RBAC-Mapping.xlsx`)
- Logging matrix (`artifacts/logging/Audit-Events-Matrix.xlsx`)
- Sample IAM policies, alarms, runbooks
