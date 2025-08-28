# Incident Response Plan (IRP)

## Severity
- SEV1: Confirmed ePHI exposure, widespread compromise.
- SEV2: Contained security event, potential exposure.
- SEV3: Suspicious activity; investigation needed.

## Roles
- IR Lead (primary), Deputy, Comms/Legal, DevOps On-Call, Security Analyst.

## Workflow (IR-4)
1) Detect (alarm/ticket) → Acknowledge (15m) → Contain (1h) → Eradicate → Recover → Lessons Learned (≤5 days).
2) Evidence handling: preserve logs, snapshot resources, maintain chain of custody.
3) Playbooks:
   - Credential compromise
   - Data exfil indicator
   - DB corruption / ransomware (admin endpoint)

## Reporting (IR-6)
- SEV1: leadership within 1h; customers & regulators per SLA/contract.
- Templates stored in `artifacts/ir/templates/`.

## Exercises
- Quarterly tabletop focusing on highest risks; action items tracked to closure.


