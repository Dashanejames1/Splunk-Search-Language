# Splunk-Search-Language


**Author:** Dashane James  
**Lab Environment:** VMware Workstation | Kali Linux | Metasploitable 2 
**Purpose:** Demonstrate core SPL commands that cyber security analysts often use to parse through files and data.10
**Status:** 🔵 Completed

---

## 📋 Overview

[]

---

## 🧪 Lab Environment

| Component | Details |
|---|---|
| Hypervisor | VMware Workstation (Host-Only Network) |
| Attacker Machine | Kali Linux 2026.1 — `192.168.79.129` |
| Target Machine | [Metasploitable 2] — `192.168.79.130` |
| Network Type | Host-Only (isolated, no internet exposure) |
| Host OS | Windows 11 — ASUS Vivobook 14 |

> ⚠️ **Note:** All activity was performed in a controlled, isolated lab environment against deliberately vulnerable machines. No unauthorized access to live networks was performed.

---

## 🛠️ Tools Used

- **Nmap** — Used for both standard scanning and running NSE (Nmap Scripting Engine) scripts to detect and, in one case, actively exploit vulnerabilities
- Splunk
- **NSE (Nmap Scripting Engine)** — Ran default (`-sC`), targeted (`ftp-anon`, `smb-vuln*`), and broad (`vuln`) script categories against the target
- **NVD (National Vulnerability Database)** — Referenced to research CVE details, CVSS scores, and attack vectors for confirmed vulnerabilities

---

---

## 🔬 Tasks / Assessments Performed

### 1. Key Spl Commands
For this task I 

# Commands used

index=main | stats count by source type

<img width="857" height="208" alt="Screenshot 2026-09-22 141927" src="https://github.com/user-attachments/assets/dd3e6303-f448-44a1-8a37-e951d0ae7256" />

index=main failed | table _time, user, src_ip

<img width="857" height="185" alt="Screenshot 2026-09-22 144302" src="https://github.com/user-attachments/assets/dd571619-8b99-41f8-9f52-74a25d58e719" />

index=main | stats count by src_ip | sort -count.

<img width="858" height="179" alt="Screenshot 2026-09-22 215458" src="https://github.com/user-attachments/assets/8526c807-4d82-4449-b4ad-98b6b70d15d5" />

index=main EventCode=4625 | stats count by Account_Name | where count > 10

<img width="854" height="185" alt="Screenshot 2026-09-22 220202" src="https://github.com/user-attachments/assets/e183ded9-f0f9-4ad2-9725-0bdec4c905d6" />


# Output

index=main | stats count by sourcetype

Running this groups all indexed events by their assigned sourcetype and displays the count as a table. The result shows one sourcetype called 'save1' with a count of 2 — this is the sourcetype Splunk automatically assigned to the nmap_services.txt file during upload. In a real SOC environment this search would show multiple sourcetypes such as firewall, windows_event, syslog, and IDS alerts, allowing analysts to quickly see which log sources are actively feeding data into the SIEM.

index=main "failed" | table_time, user, src_ip

searches for the word "failed" and displays results in a table. Won't return results with the saved Nmap data since it has no login events, but good to practice the syntax

index=main | stats count by src_ip | sort -count

counts events per IP, sorted highest to lowest. Results says 2 events but Statistics (0). Splunk found the 2 events from the indicated Nmap file, but couldn't extract a src_ip field from it because Nmap text output doesn't have that structured field, so the stats table returns empty.

index=main EventCode=4625 | stats count by Account_Name | where count > 10


this is a Windows failed login detection query. Needs Windows event logs to work.

returned 0 events because EventCode 4625 is a Windows Security Event log code that specifically indicates a failed login attempt — and no Windows event logs have been ingested into Splunk yet. The stats count by Account_Name portion would group the failed logins by username, and the where count > 10 filter would only surface accounts that have failed more than 10 times, eliminating noise and focusing only on suspicious activity.


In a real SOC environment this is one of the most important brute force detection queries an analyst runs — any account accumulating more than 10 failed logins in a short timeframe is a strong indicator of a password spraying or brute force attack in progress and would trigger an immediate investigation.

**Findings:**

what is table time, user, src ip command used for ?

what is index=main




### 2. Find top 10 source IP's by event count
For this task I ran the "index=main | stats count by src_ip | sort -count | head 10" search to be able to find the top 10 source Ip's by event count. The goal being to become familiarized with search language such as this to be able to efficiently use Splunk to parse through files and data.

# Splunk command used

index=main | stats count by src_ip | sort -count | head 10

<img width="854" height="181" alt="Screenshot 2026-09-23 173758" src="https://github.com/user-attachments/assets/9dbd4b5f-768b-4438-87d9-279306f1ce63" />


# Output

This search finds the top 10 IP addresses generating the most events, sorted highest to lowest. It returned no results because the current dataset does not contain a structured src_ip field. In a real SOC environment this query is used to identify the most active or suspicious IP addresses — a single IP generating thousands of events in a short window is a strong indicator of scanning, brute force, or DDoS activity



### 3. Filter events to a specific time range using the time picker
[] 


# Command used
[]

# Output


```



### 4. Create an alert that fires when failed login count exceeds 5 in 1 minute.

In this task, I

#Command used
[]

# Output





### 5. []


### Output



**Findings:** 

During this lab I practiced core SPL commands used in real SOC environments. While the current dataset (nmap_services.txt) was limited to port scan output, running these searches against it demonstrated how Splunk ingests, indexes, and queries data regardless of the source type.

The stats count by sourcetype and stats count by source commands confirmed the Nmap scan file was successfully indexed under sourcetype 'save1' with 2 events. These types of searches are used in production environments to monitor log source health — if a sourcetype suddenly drops to zero events, it signals a potential ingestion failure or a compromised system that stopped generating logs.

The authentication-based searches (failed, EventCode=4625, stats count by src_ip) returned no results due to the absence of Windows event logs or authentication data in the current index. However these queries represent three of the most critical detection patterns a SOC analyst runs daily — failed login detection, brute force identification, and high-volume source IP analysis. EventCode 4625 specifically is a Windows Security log event that fires on every failed login attempt, making it the primary indicator for credential-based attacks.

These searches will return meaningful results when real attack traffic is generated and ingested into Splunk, allowing full detection and analysis of potential threats.

---

## 📊 Key Findings Summary

| Port/Service | Tool Used | Risk Level | Notes |

| 21/tcp FTP | Nmap (`vuln`, `ftp-anon`) | 🔴 Critical | vsFTPd 2.3.4 backdoor (CVE-2011-2523) — live exploitation confirmed, root access achieved |
| 23/tcp Telnet | Nmap (`vuln`) | 🟠 High | Transmits credentials in plaintext, vulnerable to interception |
| 80/tcp HTTP | Nmap (`vuln`) | 🟠 High | SQL injection vulnerability detected |
| 80/tcp HTTP | Nmap (`vuln`) | 🟡 Medium | Susceptible to Slowloris denial-of-service attack |
| 445/tcp SMB | Nmap (`-sC`, `smb-vuln*`) | 🟡 Medium | Message signing disabled; guest authentication allowed |
| 22/tcp SSH | Nmap (`vuln`) | 🟡 Medium | Weak/outdated cryptographic configuration identified |
| 3306/tcp MySQL | Nmap (`vuln`) | 🟡 Medium | Database version and service info exposed |

**Risk Levels:** 🔴 Critical | 🟠 High | 🟡 Medium | 🟢 Low


---

## 🗺️ MITRE ATT&CK Mapping

| Action Performed | ATT&CK Tactic | Technique ID | Technique Name |
|---|---|---|---|
| Default and targeted script scanning (-sC, ftp-anon, smb-vuln*) | Reconnaissance | T1595 | Active Scanning |
| Anonymous FTP login confirmed | Initial Access | T1078 | Valid Accounts |
| vsFTPd backdoor exploitation (root shell via `id` command) | Execution | T1059 | Command and Scripting Interpreter |
| Root-level access confirmed post-exploitation | Privilege Escalation | T1068 | Exploitation for Privilege Escalation |

---

## 🛡️ Defensive Recommendations

Based on findings, the following remediations would be recommended in a real environment:

1. **vsFTPd 2.3.4 backdoor (Critical)** — Immediately upgrade or replace the FTP service; this version is known to contain an intentional backdoor and should never be used in production.
2. **Anonymous FTP login allowed** — Disable anonymous access entirely; require authenticated accounts for any FTP access.
3. **Telnet enabled (plaintext credentials)** — Disable Telnet and replace with SSH for all remote administration.
4. **SMB message signing disabled** — Enable SMB message signing to prevent tampering and man-in-the-middle attacks on SMB traffic.
5. **SQL injection on port 80** — Implement input validation/parameterized queries on the web application, and consider a WAF as an additional layer of defense.

---


---

## 🔑 Technical Notes

> # Note: not all NSE vulnerability scripts behave the same way — most only detect and report (true/false), while a small number (like ftp-vsftpd-backdoor) actually attempt live exploitation. Always check script documentation or output carefully rather than assuming uniform behavior.
>
> "Always add the -n flag to Nmap scans in this VMware environment to prevent DNS resolution hangs."]
> 
> -sP and -sT contradict eachother (Can't be used together because -sP means just do a ping/host-discovery sweep, skip ports entirely but -sT tells Nmap to do a full TCP connect port scan. These commands contradict eachother.)

```bash

# Any important commands or workarounds
[-sC]
[-Pn]

# Any important commands or workarounds

# Task 1: Run default scripts (-sC) against key ports
sudo nmap -sC -Pn --disable-arp-ping -n -p 21,22,80,445 192.168.79.130

# Task 2: Check FTP for anonymous login specifically
sudo nmap --script ftp-anon -Pn --disable-arp-ping -n -p 21 192.168.79.130

# Task 3: Run all SMB vulnerability scripts against port 445
sudo nmap --script smb-vuln* -Pn --disable-arp-ping -n -p 445 192.168.79.130

# Task 4: Run the full vuln script category across all open ports
sudo nmap --script vuln -Pn --disable-arp-ping -n 192.168.79.130

# Workaround: -p requires a value directly after it — omitting the port number
# (e.g. "-p 192.168.79.130") causes Nmap to misinterpret the target IP as a
# port specification. To scan all ports instead of a specific one, remove
# the -p flag entirely rather than leaving it empty.

# Workaround: always use -n in this lab to prevent DNS resolution hangs,
# since the isolated Host-Only network has no real DNS server.

# Note: --script <name>* (wildcard) runs every NSE script matching that
# name pattern — useful for running a whole category (e.g. smb-vuln*)
# in a single command instead of specifying each script individually.
```

---

## 📌 About This Project

This repository is important because here we have built up to the point of having live proof of the actual exploitation by an automated Nmap script for the CVE-2011-2523. In a past NVD task I researched this exact vulnerability with a CVSS score of 9.8 now this has come full circle and we found out how this exact vulnerability is exploited and root access was acheived. 


**Related repositories:**
- Nmap-Host-Discovery-and-Lab-Baseline — Established a baseline of the lab network using ping sweeps and host discovery, documenting the target environment before deeper scanning began
- Nmap-Scan-Types-SYN-vs-TCP-vs-UDP — Compared SYN, TCP connect, and UDP scan types against the target, examining differences in speed, stealth, and reliability
- Service-Version-Detection-and-OS-Fingerprinting — Identified exact software versions on Metasploitable and researched real CVEs tied to them, including the vsFTPd backdoor later exploited in this repository
- Wireshark-Capture-and-Analyze-Traffic — Captured and analyzed live packet traffic, including plaintext credential exposure over Telnet
- TCPDump-CLI-Packet-Capture — Used TCPDump from the command line to capture traffic (including a live Telnet session showing plaintext credential exposure), and demonstrated saving captures to a `.pcap` file for later analysis in Wireshark

---

## 👤 Author

**Dashane James**  
Senior Field Service Technician → Cybersecurity Analyst  
📍 Yonkers, NY  
🎓 B.S. Information Technology — SUNY Canton  
🏆 CompTIA Security+ | CySA+ (In Progress)  
🔗 [GitHub](https://github.com/Dashanejames1) | [Zero Trust Cyber Security Brand](https://www.instagram.com/zerotrust_cybersecurity)

---

*This repository is part of an active portfolio demonstrating hands-on cybersecurity skills. All lab work performed in isolated environments for educational purposes.*
