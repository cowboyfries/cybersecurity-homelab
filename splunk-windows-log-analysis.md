# Splunk Windows Log Analysis — Failed Login Detection

## Objective
Configure Splunk Enterprise to ingest Windows Security 
logs, identify a brute force login pattern, and build 
a detection alert.

## Environment
- Windows 10 VM (VirtualBox)
- Splunk Enterprise Free (500MB/day)

## What I Did
- Installed Splunk Enterprise on Windows VM
- Configured Local Event Log Monitoring for Security, 
  System, and Application logs
- Ran SPL queries to identify login activity
- Simulated a brute force attack pattern
- Built a triggered alert for failed login threshold
<img width="934" height="685" alt="Screenshot 2026-06-09 185810" src="https://github.com/user-attachments/assets/f230f132-3e64-4570-a2d1-7cc6aeb8543e" />

## Queries Used

### View all security events by EventCode
index=main source="WinEventLog:Security" | stats count by EventCode

### Detect failed logins
index=main source="WinEventLog:Security" EventCode=4625 
| table _time Account_Name Logon_Type ComputerName
<img width="843" height="457" alt="Screenshot 2026-06-09 185737" src="https://github.com/user-attachments/assets/021b4a8f-3c6b-4e31-a0ec-ee53e48d9d69" />

### Brute force pattern — failed then successful login
index=main source="WinEventLog:Security" 
(EventCode=4625 OR EventCode=4624) 
| table _time EventCode Account_Name Logon_Type ComputerName
<img width="923" height="853" alt="Screenshot 2026-06-09 190841" src="https://github.com/user-attachments/assets/bbb58b39-8e89-4fed-9051-3c998f0bca61" />

## Key EventCodes
- 4624 — Successful login
- 4625 — Failed login  
- 4672 — Admin privileges assigned to login session

## Logon Types
- Type 2 — Interactive (physical keyboard login)
- Type 3 — Network login
- Type 5 — Service account login

## Findings
- Observed multiple 4625 events followed by 4624
- This pattern indicates a successful login after 
  repeated failures — consistent with a brute force attempt
- In a real environment this would require immediate 
  investigation of the account and source IP

## Alert Built
- Title: Failed Login Attempts Detected
- Trigger: More than 3 failed logins within one hour
- Action: Added to Triggered Alerts at Medium severity
- Alert confirmed firing after simulated failed logins

## What I Learned
- EventCode 4625 and 4624 correlation is a foundational 
  SOC detection pattern
- Service account logins (Type 5) are expected noise — 
  interactive service logins are the anomaly
- Splunk SPL pipe operator formats raw logs into 
  readable investigative tables
- Alert thresholds balance sensitivity vs false positives
