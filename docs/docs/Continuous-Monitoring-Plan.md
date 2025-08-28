# Continuous Monitoring Plan — MediTrack

## Roles
- Security Analyst: log triage, saved queries, metrics.
- DevOps: patching, backups, config drift.
- DBA: backups/restores.
- IR Lead: exercises, lessons learned.

## Cadence
- Daily: triage high/critical alarms; ticket creation and closure tracking.
- Weekly: dependency/vuln report review; failed-login trend review.
- Monthly: admin access review; confirm CloudTrail coverage; drift check vs baseline.
- Quarterly: restore test; IR tabletop; key policy review; WAF rules refresh.
- Annually: policy refresh; vendor re-assessment (SA-9).

## Metrics (reported monthly)
- MTTD / MTTR (median)
- % Critical vulns fixed ≤7d; High ≤30d
- % Leavers disabled ≤24h
- # of data export attempts; % with MFA
- Restore success rate; Avg restore time vs RTO
