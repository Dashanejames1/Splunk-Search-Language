# Splunk-Search-Language


**Author:** Dashane James
**Lab Environment:** VMware Workstation | Kali Linux | Metasploitable 2
**Purpose:** Install Splunk Enterprise Free on Kali Linux, ingest log output files from previous labs, and run basic SPL searches to confirm data is indexed and searchable.
**Status:** 🔵 Completed

---

## 📋 Overview

This repository documents the initial Splunk setup that opens the SIEM, log analysis, and incident response phase. I downloaded Splunk Enterprise Free directly from splunk.com using wget, installed it with dpkg, and brought up the web interface at http://localhost:8000. From there I ingested the Nmap scan output saved from an earlier lab and ran basic SPL searches to confirm the data was indexed correctly. The final step was building a simple column chart visualization from the event count data, which demonstrates the core SIEM workflow: ingest log data, query it, and surface it as a visual for analysis.

---

## 🧪 Lab Environment

| Component | Details |
|---|---|
| Hypervisor | VMware Workstation (Host-Only Network) |
| Analyst Machine | Kali Linux 2026.1, `192.168.79.129` |
| Target Machine | Metasploitable 2, `192.168.79.130` |
| Network Type | Host-Only (isolated, no internet exposure) |
| Host OS | Windows 11, ASUS Vivobook 14 |

> ⚠️ **Note:** All activity was performed in a controlled, isolated lab environment against deliberately vulnerable machines. No unauthorized access to live networks was performed.

---

## 🛠️ Tools Used

- **Splunk Enterprise (Free):** Log aggregation, indexing, and search platform. Installed via wget .deb package directly from splunk.com. The free tier supports all core functionality used in this lab, with a 500MB daily ingest limit and hourly alert scheduling.
- **wget / dpkg:** Used to download the Splunk .deb package from the Splunk website and install it on Kali Linux.
- **Nmap output file (nmap_services.txt):** Log data from a previous lab, used as the first data source ingested into Splunk.

---

## 🔬 Tasks / Assessments Performed

### 1. Download and Install Splunk Enterprise Free

I copied the wget download link for the .deb Linux version of Splunk Enterprise Free from the Splunk website, pasted it into the terminal, and ran the download. Once the file was on disk, I installed it with dpkg and started Splunk with the license accepted.

```bash
# Install the downloaded .deb package
sudo dpkg -i splunk-*.deb

# Start Splunk and accept the license on first run
sudo /opt/splunk/bin/splunk start --accept-license --run-as-root
```

<img width="671" height="365" alt="Screenshot 2026-09-17 141131" src="https://github.com/user-attachments/assets/ddaeb829-a42e-4d13-883b-6f07d9f21f87" />

<img width="327" height="224" alt="Screenshot 2026-09-17 141205" src="https://github.com/user-attachments/assets/0225e8f9-621a-4000-90de-2af60a4a0942" />

**Output:** Splunk downloaded successfully via wget and installed without errors. The `splunk start` command accepted the license and initialized the instance, establishing a local connection accessible at http://127.0.0.1:8000.

### 2. Access Splunk at http://localhost:8000

<img width="325" height="253" alt="Screenshot 2026-09-17 222727" src="https://github.com/user-attachments/assets/44a0f2ea-7d60-4087-b040-d5f9437ef2fd" />

<img width="329" height="257" alt="Screenshot 2026-09-17 222555" src="https://github.com/user-attachments/assets/80443ee6-dbee-4d9e-9ae3-014e6c586a0b" />

<img width="851" height="373" alt="Screenshot 2026-09-17 150533" src="https://github.com/user-attachments/assets/6e8ca33a-757f-4bd2-8bc4-a1d97c03ad4d" />

**Output:** Splunk's web interface loaded successfully at http://127.0.0.1:8000. The login page, Splunk home dashboard, and search interface were all accessible without any additional configuration. This confirmed the installation was complete and the service was running correctly.

### 3. Ingest Previous Nmap Output Files

I clicked the Add Data tab on the Splunk home page and uploaded nmap_services.txt, the Nmap scan output file saved during a previous lab. Splunk automatically assigned it a sourcetype and indexed the contents.

<img width="650" height="293" alt="Screenshot 2026-09-17 154617" src="https://github.com/user-attachments/assets/2972b744-7343-4102-8989-e835ae6b89f3" />

<img width="805" height="338" alt="Screenshot 2026-09-17 154915" src="https://github.com/user-attachments/assets/0b22ec2a-403b-454a-a1eb-96a292fb446b" />

**Output:** Splunk indexed nmap_services.txt and returned 2 events. The first event showed the scan summary (1 IP address scanned, host up), and the second showed the full scan initiation including the exact command used, the target IP, and the open ports with their service versions. This confirmed Splunk successfully parsed and indexed the Nmap output, making the scan data searchable within the SIEM.

### 4. Run a Basic Search: index=main

```
index=main
```

<img width="853" height="322" alt="Screenshot 2026-09-17 155300" src="https://github.com/user-attachments/assets/991c14a3-cee1-403e-8158-4f5f304f38fd" />

**Output:** Running `index=main` returned all indexed events. Since only one file had been ingested at this point, it returned the same 2 events from nmap_services.txt. This search is the starting point for any Splunk investigation: "show me everything in the main index." In a production SOC environment with dozens of log sources, this same search would return thousands of events across firewalls, endpoint logs, authentication systems, and network devices.

### 5. Create a Simple Dashboard Showing Event Counts

```
index=main | stats count by source
```

<img width="854" height="336" alt="Screenshot 2026-09-17 155416" src="https://github.com/user-attachments/assets/87e5a452-65ff-4859-b395-7af330bd7e90" />

**Output:** Running the search and switching to the Visualization tab rendered the results as a column chart. The chart showed one data source (nmap_services.txt) with 2 events, confirming the data was ingested and displayed correctly. This is a core SIEM dashboard function: in a real SOC, this view would show event volume across multiple log sources simultaneously, and a source suddenly dropping to zero events would indicate either a system going offline or a log forwarding failure, both of which warrant investigation.

---

## 📊 Key Findings Summary

| Finding | Details |
|---|---|
| Splunk installation successful | Installed via dpkg from splunk.com .deb; web interface accessible at http://localhost:8000 |
| Nmap scan data ingested successfully | nmap_services.txt indexed as sourcetype 'save1' with 2 events |
| SPL search confirmed data is searchable | `index=main` returned both events with correct timestamps and content |
| Dashboard visualization functional | `stats count by source` rendered as a column chart, confirming the SIEM's visual reporting capability |

---

## 🗺️ MITRE ATT&CK Mapping

The activity in this lab was defensive setup. The data ingested and the SPL queries run correspond to the following adversary techniques this SIEM configuration would detect:

| SPL Query / Log Source | ATT&CK Tactic Targeted | Technique ID | Technique Name |
|---|---|---|---|
| Nmap scan output ingested for analysis | Discovery | T1046 | Network Service Discovery |
| `index=main "failed"` (authentication failure search) | Credential Access | T1110.001 | Brute Force: Password Guessing |
| `EventCode=4625` (Windows failed login detection) | Credential Access | T1110.001 | Brute Force: Password Guessing |
| `stats count by src_ip` (high-volume source IP detection) | Discovery | T1046 | Network Service Discovery |

---

## 🛡️ Defensive Recommendations

1. **Ingest authentication logs, not just scan output:** nmap_services.txt was a useful starting point, but SSH authentication logs, Windows Event Logs, and firewall logs are where the real attack patterns live. Adding these sources transforms Splunk from a search tool into a working detection platform.

2. **Configure real-time alerting in production:** Splunk Free supports only hourly scheduled alerts. In an actual SOC environment, critical detection rules for brute force or privilege escalation need to run on a 1-5 minute cycle, not a 60-minute one.

3. **Set a data retention policy early:** Splunk's default behavior is to retain all indexed data indefinitely. Defining retention limits at initial setup prevents disk consumption from creeping up silently and potentially taking the SIEM offline during an active incident.

4. **Forward logs from monitored hosts:** A SIEM can only detect what it receives. Setting up log forwarding from both Kali and Metasploitable into Splunk gives the authentication-based SPL queries in the next lab actual data to work with.

---

## 🔑 Technical Notes

```bash
# Install Splunk Enterprise Free (.deb for Linux)
sudo dpkg -i splunk-*.deb

# Start Splunk on first launch (accepts the license, runs as root in lab)
sudo /opt/splunk/bin/splunk start --accept-license --run-as-root

# Access Splunk in a browser
http://localhost:8000
# or
http://127.0.0.1:8000

# Restart Splunk after configuration changes
sudo /opt/splunk/bin/splunk restart
```

**Core SPL searches used in this lab:**

```
# Return all events in the main index
index=main

# Count events by data source
index=main | stats count by source

# Count events by sourcetype
index=main | stats count by sourcetype

# Search for failed authentication events
index=main "failed" | table _time, user, src_ip

# Windows failed login detection (EventCode 4625)
index=main EventCode=4625 | stats count by Account_Name | where count > 10

# Top 10 source IPs by event count
index=main | stats count by src_ip | sort -count | head 10
```

---

## 📌 About This Project

This repository opens the SIEM, log analysis, and incident response phase of the portfolio. The first two weeks established what the network looks like and documented the full attack chain from the attacker's perspective. Week 3 builds the detection side: the platforms a SOC analyst uses to monitor, search, and alert on exactly those techniques.

Splunk is the starting point because it simplifies the initial setup and puts the focus where it belongs, on log ingestion, query syntax, and detection logic. The data ingested here and the SPL commands practiced carry forward through the next repository, where the same Splunk instance is used to detect live attack traffic. From there the phase expands to a second SIEM platform, a network intrusion detection system, and a full incident response lifecycle.

**Related repositories (SIEM, log analysis, and incident response phase):**
- `Splunk-Search-Language`: Core SPL commands used by SOC analysts to parse log data, aggregate events by IP, and detect authentication-based threats
- `Generating-Attacks-and-Defending-in-Splunk`: Generated live attack traffic (port scan and SSH brute force), ingested it into Splunk, and built automated detection alerts for both attack types
- `Install-and-Understand-ELK-Stack`: Installed the ELK Stack on Kali Linux and documented the full Elasticsearch, Logstash, and Kibana data pipeline as a second SIEM platform
- `Install-Snort-IDS-and-Write-Detection-Rules`: Installed Snort 3, wrote a custom ICMP detection rule, and confirmed it fired on live traffic from Metasploitable
- `Incident-Response-Simulation`: Walked through all seven NIST SP 800-61 phases using the SSH brute force attack as the scenario, from preparation through lessons learned
- `Incident-Response-Report`: Formal incident report for the SSH brute force event, covering timeline, impact assessment, containment, root cause, and recommended controls

---

## 👤 Author

**Dashane James**
Senior Field Service Technician → Cybersecurity Analyst
📍 Yonkers, NY
🎓 B.S. Information Technology, SUNY Canton
🏆 CompTIA Security+
🔗 [GitHub](https://github.com/Dashanejames1) | [Zero Trust Cyber Security Brand](https://www.instagram.com/zerotrust_cybersecurity)

---

*This repository is part of an active portfolio demonstrating hands-on cybersecurity skills. All lab work performed in isolated environments for educational purposes.*
