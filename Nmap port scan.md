# Nmap Port Scan Detection Using Windows Firewall Logs

## Objective
Use Kali Linux to run an Nmap port scan against a 
Windows VM and detect the scan activity using Windows 
Firewall logs ingested into Splunk.

## Environment
- Windows 10 VM (VirtualBox) — Target machine
- Kali Linux VM (VirtualBox) — Attack machine
- Splunk Enterprise Free (500MB/day)
- Both VMs configured on Host-Only network adapter
  to communicate in isolation

## Network Configuration
- Windows VM IP: 192.168.56.101
- Kali Linux VM IP: 192.168.56.102
- Network mode: Host-Only — VMs communicate with 
  each other but have no internet access during
  attack simulation

## What I Did

### Step 1 — Configured VM network settings
Changed both VMs from NAT to Host-Only adapter in
VirtualBox network settings so the machines could
communicate with each other in an isolated environment.
<img width="776" height="508" alt="Screenshot 2026-06-16 134532" src="https://github.com/user-attachments/assets/d73a9c12-9d9b-4d15-a843-0cc662b2b13c" />

### Step 2 — Verified connectivity
Enabled ICMP through Windows Firewall and confirmed
Kali could reach the Windows VM using ping:
- Kali ran: ping 192.168.56.101
- Windows VM responded successfully

### Step 3 — Ran initial Nmap scan
Ran a basic Nmap scan from Kali against the Windows VM:
nmap 192.168.56.101
Result: All 1000 ports filtered — Windows Firewall 
blocked everything and Nmap received no responses.
<img width="748" height="296" alt="Screenshot 2026-06-16 125901" src="https://github.com/user-attachments/assets/89233069-6e81-4edd-be99-d06080444c58" />

### Step 4 — Ran service detection scan
Temporarily disabled Windows Firewall to allow Nmap
to identify open ports and running services:
nmap -sV 192.168.56.101
<img width="589" height="800" alt="Screenshot 2026-06-16 130141" src="https://github.com/user-attachments/assets/05a92205-43c5-4817-8f4c-88d4c8d7e69a" />

Open ports discovered:
- 135/tcp — Microsoft Windows RPC
- 139/tcp — NetBIOS
- 445/tcp — SMB (Server Message Block)
- 5432/tcp — PostgreSQL (Splunk backend database)
- 8000/tcp — Splunk web interface
- 8089/tcp — Splunk management port
<img width="750" height="535" alt="Screenshot 2026-06-16 130341" src="https://github.com/user-attachments/assets/294c4af0-8ebb-4ee4-a315-9602af9ff883" />

### Step 5 — Configured Windows Firewall logging
Re-enabled Windows Firewall and configured it to
log both dropped packets and successful connections
via wf.msc → Windows Defender Firewall Properties
→ Customize Logging.
Log file location:
C:\Windows\System32\LogFiles\Firewall\pfirewall.log
<img width="370" height="366" alt="Screenshot 2026-06-16 134913" src="https://github.com/user-attachments/assets/2bf683f5-e75e-4546-adc1-270fcdcff5b7" />

### Step 6 — Added firewall log to Splunk
Added pfirewall.log as a monitored file input in
Splunk under Settings → Data Inputs → Files & 
Directories. Used Default Settings as sourcetype
since Windows Firewall log is not natively supported
by Splunk without additional add-ons.
<img width="588" height="194" alt="Screenshot 2026-06-16 135522" src="https://github.com/user-attachments/assets/3d80b26e-ca3b-4bcd-a7c5-79d155521f4a" />

### Step 7 — Re-ran Nmap scan with firewall enabled
Ran Nmap again from Kali with firewall active:
nmap -sV 192.168.56.101
Result: All ports now filtered — firewall blocking
scan traffic and logging the attempts.
<img width="744" height="301" alt="Screenshot 2026-06-16 132613" src="https://github.com/user-attachments/assets/90a2bc6e-4509-41da-b872-e3b95461c030" />

### Step 8 — Detected scan in Splunk
Searched Splunk for Kali's IP in the raw firewall log:
index=main source="*pfirewall*" "192.168.56.102"
<img width="601" height="656" alt="Screenshot 2026-06-16 133533" src="https://github.com/user-attachments/assets/24c24311-b082-4de1-b08c-1ec9904e89a4" />

## What I Found
Multiple DROP TCP entries appeared showing:
- Source IP: 192.168.56.102 (Kali — attacker)
- Destination IP: 192.168.56.101 (Windows — target)
- Multiple destination ports hit in rapid succession
- All connections blocked and logged by firewall

This is the signature pattern of a port scan —
a single source IP attempting connections across
many ports in a short time window.

## Challenges Encountered

### Field extraction limitation
Because Splunk's Default Settings sourcetype does
not automatically extract fields from Windows Firewall
logs, field-based searches like src_ip=192.168.56.102
returned no results. Had to search raw log text using
the IP address as a string instead.

Resolution: The proper fix is installing the Splunk
Add-on for Microsoft Windows which includes correct
field extractions for Windows Firewall logs. This is
identified as a next step for the lab.

### Initial scan returned no data
First Nmap scan with firewall enabled showed all ports
as filtered with no detection in Splunk because firewall
logging had not yet been configured. Logging must be
explicitly enabled — it is off by default in Windows.

## Key Concepts Learned

### What a port scan looks like in logs
A port scan generates a high volume of connection
attempts from a single source IP across many ports
in a very short time window. In firewall logs this
appears as repeated DROP TCP entries with the same
source IP and rapidly incrementing destination ports.

### Port 445 — SMB
One of the most significant findings from the open
port scan. SMB (Server Message Block) on port 445
has been exploited in major attacks including
WannaCry and NotPetya. An exposed SMB port is an
immediate finding in any real security assessment.

### Exposed database port
PostgreSQL running on port 5432 was visible during
the open port scan. This is Splunk's backend database.
In a real environment an exposed database port is a
critical security finding — databases should never
be directly accessible from the network.

### Firewall logging is not enabled by default
Windows Firewall does not log dropped or allowed
packets without explicit configuration. In a real
SOC environment this means a system with default
firewall settings produces no network-level log
evidence even when under active attack.

### Host-Only vs NAT network modes
NAT — VM has internet access but is isolated from
other VMs. Appropriate for downloading tools and
updates.
Host-Only — VMs can communicate with each other
but have no internet access. Appropriate for attack
simulations in an isolated lab environment.

## What I Would Do Next as an Analyst
- Investigate why SMB port 445 is exposed and
  whether it is necessary
- Determine if the PostgreSQL database port should
  be accessible from the network
- Install Splunk Add-on for Microsoft Windows to
  enable proper field extraction from firewall logs
- Install Sysmon to capture process-level and
  network connection details that Windows Event
  Logs and Firewall logs cannot provide
- Correlate port scan activity with subsequent
  login attempts or process execution

## Next Steps for This Lab
- Install Splunk Add-on for Microsoft Windows
- Install and configure Sysmon
- Complete TryHackMe SOC Level 1 path
- Document additional attack scenarios using
  Kali Linux against Windows VM

## Commit Message
Add Nmap port scan detection lab write-up
