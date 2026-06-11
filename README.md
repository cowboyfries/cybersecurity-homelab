# Cybersecurity Home Lab

## About Me
I am a career changer transitioning from 12 years as 
a Critical Care ICU Nurse (CCRN) into cybersecurity. 
My clinical background built skills that translate 
directly into security operations — triage under 
pressure, pattern recognition, disciplined 
documentation, and calm decision making when 
the stakes are high.

CompTIA Security+ certified (May 2026), actively 
building hands-on skills toward a SOC Analyst role 
with a long term focus on Incident Response and 
Cloud Security.

---

## Purpose
This repository documents my home lab investigations,
simulations, and detections as I build practical 
cybersecurity skills beyond certification study.
Every entry reflects real hands-on work — not just 
theoretical knowledge.

---

## Lab Environment
- **Host OS:** Windows (VirtualBox)
- **Windows VM:** Windows 10 — Splunk Enterprise, 
  log analysis, attack simulation
- **Kali Linux VM:** Offensive tooling, 
  network scanning, attack simulation
- **SIEM:** Splunk Enterprise Free (500MB/day)

---

## Certifications
- CompTIA Security+ SY0-701 — May 2026

## Currently Working On
- TryHackMe SOC Level 1 Path
- Expanding Splunk detection scenarios
- Kali Linux to Splunk attack/detect integration

---

## Lab Write-Ups

| # | Title | EventCodes | Techniques |
|---|---|---|---|
| 1 | [Brute Force Login Detection](splunk-windows-log-analysis.md) | 4625, 4624 | Failed login correlation, alert threshold |
| 2 | [Account Lockout Detection](splunk-account-lockout-detection.md) | 4625, 4740, 4624 | Lockout policy, full incident timeline |
| 3 | [New User Account Creation Detection](splunk-new-user-account-detection.md) | 4720, 4732 | Persistence, privilege escalation |

---

## Skills Demonstrated
- SIEM deployment and configuration (Splunk Enterprise)
- Windows Security Event Log analysis
- SPL (Splunk Processing Language) query writing
- Attack simulation and detection correlation
- Alert building with appropriate severity and thresholds
- Incident timeline reconstruction
- Documentation and analytical write-up

---

## Roadmap
- [ ] TryHackMe SOC Level 1 completion
- [ ] Wireshark network traffic analysis lab
- [ ] Kali Linux Nmap scan detection in Splunk
- [ ] Process creation monitoring — EventCode 4688
- [ ] CySA+ certification

---

## Connect
- LinkedIn: www.linkedin.com/in/michael-santa-cruz-2720183b4
- Email: mike.santacruz@proton.me
