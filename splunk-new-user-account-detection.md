# Splunk New User Account Creation Detection

## Objective
Simulate a new user account creation on Windows VM,
detect it using Splunk, identify privilege escalation,
and build a high severity alert for automatic detection.

## Environment
- Windows 10 VM (VirtualBox)
- Splunk Enterprise Free (500MB/day)
- Local User Management configured via lusrmgr.msc

## What I Did
- Created a fake user account (testuser01) via 
  Local User Manager to simulate attacker persistence
- Added testuser01 to the Administrators group to 
  simulate privilege escalation
- Queried Splunk to detect the full attack sequence
- Identified field parsing limitations in EventCode 4732
- Built a high severity alert for new account creation
<img width="939" height="306" alt="Screenshot 2026-06-10 193543" src="https://github.com/user-attachments/assets/c7b07af0-f8f8-4d20-a083-28d13dc21220" />
<img width="664" height="728" alt="Screenshot 2026-06-10 194335" src="https://github.com/user-attachments/assets/3aa000c9-a052-4675-baf3-81fd20782940" />

## Queries Used

### Detect new user account creation
index=main source="WinEventLog:Security" EventCode=4720 
| table _time Account_Name SAM_Account_Name ComputerName
<img width="935" height="557" alt="Screenshot 2026-06-10 193707" src="https://github.com/user-attachments/assets/a6a5287e-5ae1-425e-9fe8-db94ad4b68b6" />

### Full attack sequence timeline
index=main source="WinEventLog:Security" 
(EventCode=4720 OR EventCode=4732) 
| table _time EventCode Account_Name SAM_Account_Name 
  ComputerName | sort _time
<img width="937" height="605" alt="Screenshot 2026-06-10 194158" src="https://github.com/user-attachments/assets/e00ce512-1037-4844-9044-3ca9f932445c" />

### Attempt to identify added member on 4732
index=main source="WinEventLog:Security" EventCode=4732 
| table _time Account_Name SAM_Account_Name 
  Member_Name ComputerName

### Alert base query
index=main source="WinEventLog:Security" EventCode=4720 
| stats count by Account_Name

## Key EventCodes
- 4720 — New user account created
- 4732 — Account added to privileged group
- 4672 — Special privileges assigned to new login

## Findings
- EventCode 4720 confirmed testuser01 creation 
  in real time
- EventCode 4732 confirmed testuser01 was added 
  to the Administrators group
- Combined 4720 and 4732 sequence represents a 
  complete attacker persistence and privilege 
  escalation workflow:
  - Create backdoor account
  - Elevate to admin
  - Maintain persistent privileged access
- Note: SAM_Account_Name field did not populate on 
  4732 events — a known Splunk field extraction 
  limitation with this EventCode. Attack sequence 
  confirmed via timeline correlation of 4720 and 
  4732 events rather than field-level account matching

## Analyst Investigation Questions
- Was this account creation authorized?
- Does a change management ticket exist for this?
- Was the account created outside business hours?
- Was the account immediately added to a privileged group?
- Has this account been used to log in yet — EventCode 4624?
- Is this a known admin workstation or a standard user machine?

## Alert Built
- Title: New User Account Created
- Trigger: Any new account creation — threshold greater than 0
- Severity: High
- Action: Add to Triggered Alerts
- Rationale: Any account creation on a production system
  outside an authorized change window is immediately
  suspicious regardless of count. Zero tolerance threshold
  appropriate for this event type.
<img width="929" height="664" alt="Screenshot 2026-06-10 195448" src="https://github.com/user-attachments/assets/3c3e1645-4a97-41bc-8261-2552cf052005" />

## Real World Context
The combination of EventCode 4720 followed by 4732 is
a textbook attacker persistence technique:
- Attacker gains initial access
- Creates a backdoor account to maintain access
- Elevates account to admin for full system control
- Retains access even if original exploit is patched
In a real SOC environment this sequence outside of
a change management window on any production system
would constitute an immediate critical incident.

## What I Learned
- EventCode 4720 and 4732 correlation detects full
  attacker persistence and privilege escalation sequence
- Not all Splunk fields auto-extract cleanly — 
  analysts must understand raw event structure and
  work around field parsing limitations
- Threshold of 0 is appropriate for high fidelity
  events that always warrant investigation
- Account creation alerts should be correlated with
  4624 login events to determine if backdoor account
  has been actively used
