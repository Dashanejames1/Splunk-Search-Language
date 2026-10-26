# Splunk-Search-Language


**Author:** Dashane James
**Lab Environment:** VMware Workstation | Kali Linux | Metasploitable 2
**Purpose:** Demonstrate core SPL commands that cybersecurity analysts use to parse log files, aggregate events by IP and user, and build detection queries for brute force and authentication-based threats.
**Status:** 🔵 Completed

---

## 📋 Overview

This repository documents hands-on practice with Splunk's Search Processing Language using real log data ingested from the previous lab. I ran a series of SPL queries that form the foundation of day-to-day SOC analyst work: grouping events by sourcetype to understand what data is in the index, extracting and aggregating IP addresses to identify high-volume sources, filtering for authentication failures, and building a scheduled alert that fires when failed login counts exceed a threshold. The dataset at this stage was limited to Nmap scan output, which confirmed that the queries ran correctly and that any missing results were due to data gaps, not syntax errors. The same queries produce full detection results once authentication logs are flowing into the SIEM.

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

- **Splunk Enterprise (Free):** Log aggregation and search platform. All queries in this lab ran against data indexed in the previous repository.
- **SPL (Search Processing Language):** Splunk's query language for searching, filtering, aggregating, and transforming indexed event data.

---

## 🔬 Tasks / Assessments Performed

### 1. Key SPL Commands

For this task I ran four core SPL queries against the indexed data to practice the search syntax analysts use most often in production environments.

**Commands used:**

```
index=main | stats count by sourcetype
```

<img width="857" height="208" alt="Screenshot 2026-09-22 141927" src="https://github.com/user-attachments/assets/dd3e6303-f448-44a1-8a37-e951d0ae7256" />

```
index=main "failed" | table _time, user, src_ip
```

<img width="857" height="185" alt="Screenshot 2026-09-22 144302" src="https://github.com/user-attachments/assets/dd571619-8b99-41f8-9f52-74a25d58e719" />

```
index=main | stats count by src_ip | sort -count
```

<img width="858" height="179" alt="Screenshot 2026-09-22 215458" src="https://github.com/user-attachments/assets/8526c807-4d82-4449-b4ad-98b6b70d15d5" />

```
index=main EventCode=4625 | stats count by Account_Name | where count > 10
```

<img width="854" height="185" alt="Screenshot 2026-09-22 220202" src="https://github.com/user-attachments/assets/e183ded9-f0f9-4ad2-9725-0bdec4c905d6" />

**Output:**

`index=main | stats count by sourcetype` groups all indexed events by their assigned sourcetype and displays a count table. The result showed one sourcetype called 'save1' with 2 events, which is the sourcetype Splunk automatically assigned to the nmap_services.txt file during upload. In a production SOC this search would show multiple sourcetypes across firewall logs, Windows events, syslog, and IDS alerts, giving analysts a quick view of which log sources are actively feeding data into the SIEM.

`index=main "failed" | table _time, user, src_ip` searches for the keyword "failed" and returns results formatted as a table with timestamp, user, and source IP columns. This returned no results because the current Nmap dataset contains no login events. In a real environment with authentication logs ingested, this search surfaces failed login attempts with enough context to identify the source and targeted account immediately.

`index=main | stats count by src_ip | sort -count` counts events per IP address, sorted from highest to lowest. Splunk found the 2 events from the Nmap file but could not extract a structured src_ip field from plain Nmap text output, so the stats table returned empty. This is a data format issue, not a syntax issue. The same query against TCPDump or firewall log data returns a populated table showing exactly which IPs generated the most traffic.

`index=main EventCode=4625 | stats count by Account_Name | where count > 10` is a Windows failed login detection query. EventCode 4625 is the Windows Security Event Log code for a failed authentication attempt. The search returned 0 events because no Windows event logs had been ingested yet. The logic is sound: group failed logins by account name and surface only accounts with more than 10 failures, which eliminates noise and focuses the output on accounts showing brute force patterns. This is one of the most common detection queries run in a Windows-heavy SOC environment.

**Findings:**

The queries all executed without syntax errors. Missing results were expected given the dataset. The `table _time, user, src_ip` construct shows how SPL structures output into named columns for readability. The `where count > 10` filter demonstrates threshold-based detection, which is the same logic used in alert configurations to reduce false positives.

### 2. Find Top 10 Source IPs by Event Count

For this task I ran the `index=main | stats count by src_ip | sort -count | head 10` search to find the top 10 source IPs by event count. Becoming fluent with IP aggregation searches is important because high-volume source IPs are one of the first indicators a SOC analyst looks at during a triage investigation.

**Command used:**

```
index=main | stats count by src_ip | sort -count | head 10
```

<img width="854" height="181" alt="Screenshot 2026-09-23 173758" src="https://github.com/user-attachments/assets/9dbd4b5f-768b-4438-87d9-279306f1ce63" />

**Output:**

This search returned no results because the Nmap text output did not contain structured src_ip fields that Splunk could extract. The query syntax is correct. In a real SOC environment with TCPDump captures, firewall logs, or IDS alerts ingested, this search returns a sorted list showing which IPs generated the most events. A single IP generating thousands of events in a short window is a strong indicator of scanning, brute force, or denial-of-service activity and warrants immediate investigation.

### 3. Filter Events to a Specific Time Range Using the Time Picker

Splunk's time range picker sits in the top-right corner of the search bar and offers preset windows (Last 15 minutes, Last 60 minutes, Last 24 hours, All time) along with custom date and time range options. The same range can also be applied directly in SPL using the `earliest` and `latest` modifiers.

**Commands used:**

```
# Last 15 minutes
index=main earliest=-15m latest=now

# Custom time window (example)
index=main earliest="09/17/2026:14:00:00" latest="09/17/2026:16:00:00"
```

**Output:**

Since the lab index contained only 2 events uploaded during a single session, switching between different time ranges confirmed that both events fell within the same upload window. Splunk normalized the timestamps from the Nmap output file and indexed them with their original scan time, not the upload time. This distinction matters in real investigations: when logs are uploaded from a file rather than streamed in real time, the event timestamps reflect when the activity actually occurred, which is what you want when reconstructing an incident timeline.

Time filtering is used constantly in SOC work. Narrowing a search to the 15-minute window around an alert fires first helps reduce noise, while expanding to 24 hours or longer helps establish whether activity was isolated or part of a longer pattern.

### 4. Create an Alert That Fires When Failed Login Count Exceeds 5

For this task I configured a scheduled Splunk alert based on the Windows failed login detection query. Alerts convert a detection query into automated monitoring: instead of running the search manually, Splunk runs it on a schedule and notifies analysts when the trigger condition is met.

**Detection query:**

```
index=main EventCode=4625 | stats count by Account_Name | where count > 5
```

**Alert configuration:**
- Name: Failed Login Threshold Alert
- Alert type: Scheduled (hourly, the maximum frequency available in Splunk Free)
- Trigger condition: Number of Results is greater than 0
- Action: Add to Triggered Alerts

**Output:**

The alert was saved and confirmed active in the Alerts section of Splunk. The query searches for Windows Event ID 4625 (failed login), groups failures by account name, and surfaces only accounts that have failed more than 5 times within the scheduled window. Since no Windows event logs were in the index yet, the alert had nothing to trigger on during this lab, but the configuration is production-ready.

**Limitation:** Splunk Free supports only hourly scheduled alerts. A real brute force attack can compromise credentials in seconds, as demonstrated in the next repository. In a production SOC running Splunk Enterprise, this alert would be configured to execute every 60 seconds. The hourly limitation is specific to the free tier and is addressed in the Generating-Attacks-and-Defending-in-Splunk lab.

This detection pattern is one of the most critical in a Windows environment. EventCode 4625 fires on every failed authentication attempt, making it the primary data source for brute force detection. Any account accumulating more than 5 failed logins within a short window is a strong indicator of a password spraying or brute force attack in progress.

---

## 📊 Key Findings Summary

| SPL Query | Purpose | Result |
|---|---|---|
| `index=main \| stats count by sourcetype` | Verify data ingestion and identify log source types | 1 sourcetype ('save1'), 2 events from nmap_services.txt |
| `index=main "failed" \| table _time, user, src_ip` | Surface failed authentication events with context | No results; no authentication logs in current index |
| `index=main \| stats count by src_ip \| sort -count \| head 10` | Identify top IP addresses by event volume | No results; Nmap text output lacks structured src_ip fields |
| `index=main EventCode=4625 \| stats count by Account_Name \| where count > 10` | Detect brute force by account-level failure count | No results; no Windows Event Logs ingested yet |
| Failed Login Threshold Alert (scheduled) | Automate detection of credential-based attacks | Alert configured successfully; awaiting real authentication log data |

---

## 🗺️ MITRE ATT&CK Mapping

The detection logic built in this lab is designed to identify the following adversary techniques once the appropriate log sources are ingested:

| Detection Query | ATT&CK Tactic Targeted | Technique ID | Technique Name |
|---|---|---|---|
| `stats count by src_ip \| sort -count` against network logs | Discovery | T1046 | Network Service Discovery |
| `EventCode=4625 \| stats count by Account_Name \| where count > 5` | Credential Access | T1110.001 | Brute Force: Password Guessing |
| `index=main "failed"` against SSH auth logs | Credential Access | T1110.001 | Brute Force: Password Guessing |

---

## 🛡️ Defensive Recommendations

1. **Ingest Windows Event Logs to activate EventCode 4625 detection:** The failed login query and alert built in this lab have no data to work with until Windows Event Log forwarding is configured. In a production environment this is one of the highest-priority log sources to connect to the SIEM.

2. **Upgrade to real-time alerting for critical detection rules:** The Splunk Free hourly scheduling limitation means a brute force attack could execute and succeed within the window between alert runs. Upgrading to a production SIEM with per-minute or real-time alerting is the correct approach for a live environment.

3. **Pair IP aggregation alerts with a baseline:** The `stats count by src_ip` query is most useful when you know what normal traffic levels look like. Establishing a baseline of expected event volumes per IP makes it possible to set meaningful thresholds rather than arbitrary ones.

4. **Use the `table` command in alert outputs:** Returning raw event counts is less useful than structured output showing the affected accounts, source IPs, and timestamps. Formatting alert results as a table reduces the time it takes an analyst to assess the scope of an incident.

---

## 🔑 Technical Notes

**Core SPL syntax patterns:**

```
# Basic search (all events in main index)
index=main

# Keyword search with table output
index=main "keyword" | table _time, field1, field2

# Aggregate events by a field, sorted by count
index=main | stats count by <field> | sort -count

# Filter aggregated results by count threshold
index=main | stats count by <field> | where count > <threshold>

# Add a row limit to aggregated results
index=main | stats count by <field> | sort -count | head 10

# Extract structured fields from unstructured text using regex
index=main | rex field=_raw "(?P<fieldname>pattern)"

# Time-bounded search using SPL modifiers
index=main earliest=-1h latest=now
```

**Creating a scheduled alert:**
1. Run the detection query in the search bar
2. Save As > Alert
3. Set Alert type to Scheduled and choose the frequency
4. Set Trigger Conditions (e.g., Number of Results is greater than 0)
5. Add an action (Add to Triggered Alerts, Send Email, etc.)

**Splunk Free limitation:** The free tier caps scheduled alert frequency at once per hour. Any detection scenario requiring faster response (brute force, active exploitation) needs Splunk Enterprise or an alternative SIEM with real-time alerting.

---

## 📌 About This Project

This repository sits at the core of the SIEM, log analysis, and incident response phase. The ability to write effective SPL is what separates a SOC analyst who can use a SIEM from one who can build detection logic with it. The previous repository set up Splunk and confirmed data ingestion. This one establishes the query patterns. The next one, Generating-Attacks-and-Defending-in-Splunk, is where these same queries run against real attack traffic and return results that actually mean something.

The detection gap in this lab, running well-formed queries against a dataset that lacks the right log sources, is intentional. Understanding why a query returns no results and knowing what log sources would make it work is exactly the diagnostic skill a SOC analyst needs when investigating why an alert did not fire.

**Related repositories (SIEM, log analysis, and incident response phase):**
- `Splunk-Free-Installation-and-Ingest-First-Logs`: Installed Splunk Enterprise and ingested the first log files, establishing the SIEM foundation this lab builds on
- `Generating-Attacks-and-Defending-in-Splunk`: Generated live attack traffic (port scan and SSH brute force), ingested it into Splunk, and built automated detection alerts for both attack types
- `Install-and-Understand-ELK-Stack`: Installed the ELK Stack on Kali Linux and documented the full Elasticsearch, Logstash, and Kibana pipeline as a second SIEM platform
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
