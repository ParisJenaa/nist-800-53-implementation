# Access Control Policy (ACP)

## Principles
- Least privilege and need-to-know.
- MFA required for all IAM users; Cognito MFA for privileged app actions.
- No shared accounts; break-glass admin controlled and audited.

## User Lifecycle (AC-2)
- Joiner: HR ticket + manager approval → Create user (Cognito or IAM) → Assign role.
- Mover: Update role in RBAC; adjust groups within 24h.
- Leaver: Disable within 24h; delete after 30d; revoke tokens/keys immediately.

## RBAC (AC-3)
- Roles: Clinician, Clinic Admin, ReadOnly (app); DevOps, SecurityAdmin (cloud).
- Clinic scoping enforced at API and DB layers.

## Remote Access (AC-17)
- Console/API access only from corporate VPN IP ranges.
- MFA mandatory; break-glass exceptions require IR Lead approval.

## Authenticator Management (IA-5)
- Password policy: min 14 chars; deny top 10k passwords.
- Secrets in Secrets Manager; rotation 30d for DB, 90d for CI keys.

## Logging (AU-2)
- CloudTrail (all regions); CloudWatch for app/API; VPC Flow Logs for DB subnets.
- Retention 365 days; no PHI in logs.

## Reviews (AU-6)
- Daily alert triage; monthly metrics report.
- Quarterly access review; semi-annual RBAC review.
