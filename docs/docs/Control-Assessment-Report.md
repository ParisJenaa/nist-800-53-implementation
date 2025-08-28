# Security Assessment Report (SAR) — MediTrack

## AU-2 — Event Logging (Test)
Objective: Verify security events are captured.  
Method: Inspect + Test  
Steps: Confirm CloudTrail enabled; verify Lambda/API log groups; trigger failed login; confirm log + alert.  
Expected: Screenshots + log entries; alert in email/Slack.  
Results: Pass.

## AC-2 — Account Management (Test)
Objective: Accounts align with HR roster.  
Method: Inspect  
Steps: Export Cognito & IAM users; compare to HR roster; check last login; verify leaver disabled in 24h.  
Expected: CSVs + annotated screenshots.  
Results: Partial — one leaver at 48h; ticket opened.

