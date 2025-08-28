# AC-2 — Account Management
Family: Access Control  
Intent: Only the right people have accounts; remove them when they leave.  
Design: Cognito for app users; IAM+MFA for admins; Joiner/Mover/Leaver via HR tickets.  
Procedures: Create/update/disable with approvals; delete after 30 days.  
Evidence: Access Control Policy; Cognito screenshot; IAM policy JSON.  
Verification: Quarterly roster vs accounts review.  
Role: Security Admin  
Status: Implemented

# AU-2 — Event Logging
Family: Audit & Accountability  
Intent: Capture security-relevant events for investigations.  
Design: CloudTrail (all regions), CloudWatch for API/Lambda, RDS audit logs; 365-day retention.  
Procedures: Centralize logs; alerts on failed login bursts.  
Evidence: Policy logging section; CloudTrail ON screenshot; sample log JSON.  
Verification: Simulate failed login → see alert within 5 mins.  
Role: Security Analyst  
Status: Implemented

# SI-2 — Flaw Remediation
Family: System & Information Integrity  
Intent: Find and fix vulnerabilities promptly.  
Design: Weekly dependency scans; monthly RDS patching; Critical=7 days, High=30 days.  
Procedures: CI fails on Critical; auto-create ticket; monthly patch window.  
Evidence: CI failure screenshot; rollback steps in Contingency Plan.  
Verification: Review last scan; check SLA met.  
Role: DevOps Lead  
Status: Partial

