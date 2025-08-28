# Contingency (Backup & Recovery) Plan

## Strategy (CP-9)
- RDS automated backups (35d), daily snapshots; weekly cross-region copy (us-west-2).
- Test restore quarterly; verify row counts and checksums.

## Objectives
- RTO: 4 hours (API/UI usable).
- RPO: 24 hours (data loss ≤ 1 day).

## Procedures
1) Initiate restore to isolated dev VPC.
2) Point test app to restored DB; run integrity SQL script.
3) Document results, start/finish times, anomalies.

## Roles
- DBA owns backups and tests; DevOps coordinates app validation.

## Communications
- Notify stakeholders on failed tests; create remediation plan within 5 business days.
