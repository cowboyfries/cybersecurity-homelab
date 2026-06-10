# Splunk Account Lockout Detection

## Objective
Simulate an account lockout scenario on Windows VM,
detect it using Splunk, and build a high severity
alert for automatic detection.

## Environment
- Windows 10 VM (VirtualBox)
- Splunk Enterprise Free (500MB/day)
- Account Lockout Policy configured via secpol.msc

## Lockout Policy Configuration
- Account lockout threshold: 5 invalid attempts
- Account lockout duration: 2 minutes
- Reset account lockout counter after: 2 minutes
<img width="711" height="269" alt="Screenshot 2026-06-09 210239" src="https://github.com/user-attachments/assets/1bfcb64b-35a8-4749-b319-ed5be79d59f5" />

## What I Did
- Configured Windows account lockout policy via 
  Local Security Policy (secpol.msc)
- Deliberately entered wrong password 5 times to 
  trigger account lockout
- Queried Splunk to identify the full attack timeline
- Built a high severity alert for lockout detection
<img width="921" height="366" alt="Screenshot 2026-06-09 205821" src="https://github.com/user-attachments/assets/d13b049a-a847-4d50-81cd-937b25afdfc7" />

## Queries Used

### Detect account lockout events
index=main source="WinEventLog:Security" EventCode=4740 
| table _time Account_Name ComputerName

### Full incident timeline
index=main source="WinEventLog:Security" 
(EventCode=4625 OR EventCode=4740) 
| table _time EventCode Account_Name ComputerName 
| sort _time
<img width="909" height="846" alt="Screenshot 2026-06-09 205211" src="https://github.com/user-attachments/assets/d07cebc4-12d9-4913-8ab8-4ed7d9082954" />

### Lockout count by account
index=main source="WinEventLog:Security" EventCode=4740 
| stats count by Account_Name

## Key EventCodes
- 4625 — Failed login attempt
- 4740 — Account locked out
- 4624 — Successful login

## Findings
- Observed multiple 4625 events leading up to 4740
- Account lockout confirmed after 5 failed attempts
- Full timeline showed complete attack sequence:
  failed attempts → lockout → successful login after unlock
- In a real environment this sequence on a domain 
  account outside business hours would constitute 
  an immediate incident requiring investigation

## Analyst Investigation Questions
- Was this the legitimate user who forgot their password?
- Did any successful logins occur from unexpected locations?
- Has this account been targeted previously?
- Was the lockout intentional — denial of service against
  a specific user account?

## Alert Built
- Title: Account Lockout Detected
- Trigger: Any lockout event — threshold greater than 0
- Severity: High
- Action: Add to Triggered Alerts
- Rationale: Unlike brute force detection, even a single
  lockout event warrants investigation — no noise 
  threshold required

## What I Learned
- EventCode 4740 only generates if a lockout policy 
  is configured — default Windows has none set
- Account lockout and brute force are related but 
  distinct detections — lockout confirms the threshold
  was reached, brute force catches attempts before it
- Sorting by _time in SPL reveals the incident 
  narrative chronologically
- High severity alerts should be reserved for events
  that always warrant analyst attention regardless of count
