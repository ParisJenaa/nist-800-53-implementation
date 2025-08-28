# Access Control Policy (ACP)
Scope: All MediTrack systems and data.
Principles: Least privilege, need-to-know, MFA for admins.

Provisioning (AC-2): HR ticket → approval → account creation → 24h deprovision on exit.  
RBAC (AC-3): Admin, Clinician, ReadOnly; keep RBAC mapping in repo.  
Remote Access (AC-17): Admins use VPN + MFA; no shared accounts.  
Passwords/Keys (IA-5): 12+ chars; rotate keys 90 days; no hard-coded secrets.  
Logging (AU-2): CloudTrail & CloudWatch enabled; retention 365 days.  
Reviews: Quarterly access reviews and emergency access audits.

