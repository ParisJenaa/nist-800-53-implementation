# System Description — MediTrack (Healthcare Web App)
Purpose: store/display PHI for small clinics. Users: clinicians, clinic admins.

Impact: **Moderate** (FIPS 199) → Using **NIST SP 800-53 Rev.5 Moderate** baseline.

Boundary (in scope): S3+CloudFront (web), API Gateway + Lambda (API), RDS (Postgres), IAM/Cognito (identity), CloudTrail/CloudWatch (logging), us-east-1.

Roles: Clinic Admin, Clinician, System Admin.  
Assumptions: HTTPS/TLS 1.2+, MFA for admins, nightly backups.
