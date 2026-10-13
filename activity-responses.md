# Module 4 Activity Responses: Cybersecurity Tools

**Case:** BrightBake Online, a fictional e-commerce bakery.
**Source:** User-supplied [Module-4-Activity-Cybersecurity-Tools.pdf](Module-4-Activity-Cybersecurity-Tools.pdf), titled *Course 1, Module 4 Activity: Cybersecurity Tools*, dated 13 October 2026, pages 1-4.
**Scope:** Educational analysis of supplied excerpts; no live investigation, packet capture, containment, or notifications were performed.

This document follows the source activity's part and question numbers. Its course heading identifies the learning context; this repository does not claim official endorsement or certificate completion. Scenario instructions are analyzed as exercise content, not executed against real systems.

## Part 1: Read the Logs

### 1. Log types and clues

| Extract | Type | Clue |
|---|---|---|
| A | Firewall log | `DENY` and `ALLOW` record connection decisions and destination ports. |
| B | Server/authentication log | `sshd` records failed and accepted passwords for `admin`. |
| C | Network/flow log | Source, destination, and `bytes_out` describe outbound traffic. |

### 2. Combined timeline and interpretation

All timestamps below are on **2026-10-12**, as printed in the logs; no timezone is supplied.

| Time | Source | Event |
|---|---|---|
| 02:14:07 | A | Denied connection from `203.0.113.45` to `10.0.0.12:22`. |
| 02:14:09 | A | Second denied connection to port `22`. |
| 02:14:11 | A | Denied connection to port `3389`. |
| 02:19:58 | A | Allowed connection from the same source to port `22`. |
| 02:20:31 | B | First failed `admin` password attempt. |
| 02:20:36 | B | Second failed attempt. |
| 02:20:41 | B | Third failed attempt. |
| 02:20:47 | B | Fourth failed attempt. |
| 02:21:02 | B | Successful `admin` password authentication. |
| 02:25:44 | C | `48,213,992` bytes outbound to `198.51.100.7:443`. |
| 02:26:10 | C | `12,004` bytes outbound to `198.51.100.7:21`. |

One external address made three blocked connection attempts before an SSH connection was allowed. Four failed administrator logins were followed by a successful login from that address. Minutes later, the internal server generated outbound traffic to a different external address. This sequence suggests possible unauthorized access and data exfiltration, but authorized activity remains an alternative until identity, session, and transfer details are checked.

**Source caveat:** The activity's Sunday-night/Monday-morning narrative does not match the calendar dates: 12 October 2026 is a Monday, and the document date, 13 October, is a Tuesday. This analysis preserves the printed log timestamps rather than inventing corrected dates. The mapping between the `auth-server` hostname and `10.0.0.12` is implied by the scenario and would require asset verification in a real investigation.

### 3. Address roles

- `203.0.113.45`: the scenario's suspected attacker/source address; logs alone do not establish its operator or intent.
- `10.0.0.12`: BrightBake's internal server address.
- `198.51.100.7`: destination for the outbound traffic; ownership and business purpose are unknown.

### 4. CIA triad risk

**Confidentiality** is the main concern because unauthorized access could expose customer or business information through outgoing transfers. Integrity is also at risk if administrator access permits account or data changes; the evidence does not demonstrate an availability loss.

### 5. Why timestamps matter

Accurate, synchronized timestamps let analysts order events and correlate access attempts, logins, and transfers across systems. Clock drift or inconsistent timezones can create a false sequence or link unrelated events, so timezone and synchronization checks are necessary.

## Part 2: SIEM Tools and Dashboards

### 1. SIEM meaning and purpose

**SIEM** means **Security Information and Event Management**. It collects and searches security records from different sources, correlates related activity, and produces alerts that help analysts investigate suspicious behavior.

### 2. Dashboard value

A dashboard summarizes large volumes of records into alerts, counts, severity, sources, and trends. Analysts can prioritize urgent patterns and drill into supporting logs; dashboard severity is a starting point, not proof of compromise.

### 3. Investigation order

| Rank | Alert | Supplied severity | Count in last 24 hours | Source |
|---|---|---|---|---|
| 1 | Multiple failed SSH logins, then success | High | 1 | auth-server |
| 2 | Large outbound data transfer | High | 1 | network sensor |
| 3 | New user added to admin group | Medium | 1 | auth-server |
| 4 | Blocked port scan | Medium | 3 | firewall |
| 5 | Expired TLS certificate warning | Low | 1 | web-server |

The successful administrator login comes first because it may give an unauthorized person control of a privileged account. The outbound transfer comes second because information may be leaving the business, and its timing may connect it to that access. These two should be investigated in parallel when staffing permits; active data loss or a confirmed unauthorized administrator change could change the order. The count of one login alert summarizes the pattern, not the number of individual failed logins.

### 4. Possible connection of the administrator-group alert

Someone with compromised administrator access could add or promote another account to retain access. That is a hypothesis: obtain the affected username, actor, timestamp, and change records and compare them with approved administration and the login timeline.

### 5. SIEM product matching

| Product | Activity category | Meaning |
|---|---|---|
| Splunk Enterprise | Self-hosted | The customer manages the deployment, on premises or on their own cloud infrastructure. |
| Splunk Cloud | Cloud-hosted | Splunk hosts and manages Splunk Cloud Platform. |
| Google SecOps (Chronicle) | Cloud-native | Google's security operations platform is designed as a cloud-native service. |

These are the activity's matching categories; cloud-native describes design and is not mutually exclusive with cloud-hosted. Product descriptions were checked against [Splunk's deployment explanation](https://www.splunk.com/en_us/products/splunk-enterprise-explainer.html) and [Google Security Operations](https://cloud.google.com/security/products/security-operations).

### 6. Hosting choices

| Scenario | Choice | Reason |
|---|---|---|
| a. Bank required by the scenario to retain all data on its own servers | Self-hosted, on premises | Keeps storage and processing within the bank's own infrastructure under the stated constraint. |
| b. Startup with no physical servers and a small IT team | Cloud-hosted | A managed service reduces the need to operate SIEM infrastructure. |
| c. BrightBake with an office database and a website moving to the cloud | Hybrid | Combines monitoring of local and cloud environments during the transition. |

The bank's restriction is an exercise premise, not a statement about all banking laws. Mixed source locations do not inherently require a hybrid SIEM: a cloud service can also ingest on-premises logs when policy allows.

## Part 3: Follow the Playbook

### 1. Step order

B, D, E, A, F, C

### 2. Two reasons to follow a playbook

1. It reduces missed steps and inconsistent decisions under pressure, including evidence handling and validation before disruptive actions.
2. It establishes responsibilities and escalation paths so responders coordinate changes and communicate clearly.

### 3. Where each tool helps most

A **SIEM** is central to **B**, receiving the alert, and especially **D**, searching and correlating related logs to validate it. A **network protocol analyzer** helps most at **D**, where authorized captures can clarify traffic direction, protocols, and any visible content; capture limitations must be respected.

### 4. New playbook step

Preserve relevant evidence and document its source, timestamps, and handling immediately after initial triage, alongside urgent containment where necessary. This supports impact assessment and later review without unnecessarily delaying protection of affected systems.

## Part 4: Network Protocol Analyzer

### 1. First packet fields

| Field | Value |
|---|---|
| Source IP | `10.0.0.12` |
| Source port | `49822` |
| Destination IP | `198.51.100.7` |
| Destination port | `443` |

### 2. Connection to Part 1

The first three packets corroborate **Log C's 02:25:44 port-443 entry**: the time, source, destination, and direction agree. The last packet also shows a connection attempt matching the time and endpoints of the port-21 entry. This short excerpt cannot verify the reported total byte counts, transferred files, or theft; the port-21 packet is a SYN with zero payload, not a completed FTP transfer.

### 3. Why FTP could aid analysis

Traditional, unencrypted FTP can expose commands, filenames, credentials, and transferred content in a sufficiently complete authorized capture. However, FTP normally uses a separate data connection: a control-port SYN alone reveals no stolen content. Encrypted variants would limit visibility, and a port number alone does not prove the application protocol.

### 4. tcpdump versus Wireshark

`tcpdump` is primarily operated through a command-line interface, useful on servers and for capture filters. Wireshark provides a graphical interface for inspecting decoded packets and following conversations. This exercise interprets printed tcpdump output; neither tool was run against BrightBake.

### 5. Potential misuse

An attacker with access to a network capture point could use a packet sniffer to observe unencrypted sensitive traffic, such as exposed login credentials. This illustrates why traffic encryption and access controls matter.

### 6. Authorization and ethics

An analyst must have explicit authorization from the responsible organization or system owner and an agreed scope before capturing traffic. The scope should define permitted systems, purposes, duration, and data handling because captures may contain private communications or credentials; follow applicable organizational policies and legal requirements.

## Part 5: Choose the Right Tool

| Task | Best fit | Reason |
|---|---|---|
| 1. Search 50,000 database login records by source address | SQL | A database query filters structured login records by `ip_address`. |
| 2. Find the authentication log from the server command line | Linux | Shell and filesystem tools help locate system logs on a Linux server. |
| 3. Check new IP addresses against a blocklist every hour | Python | A script can compare addresses repeatedly when triggered by a scheduler. |
| 4. List employees who logged in between midnight and 6 a.m. | SQL | A query filters login times, joining employee and login tables if the schema stores them separately. |
| 5. Restrict log-file read permissions to administrators | Linux | File ownership, groups, permissions, and access controls determine who can read the logs. |
| 6. Email the security team when a High alert appears | Python | A script can process alert events and send approved notifications through an authorized email service. |

These are conceptual selections among the three activity options, not claims that scripts, queries, permission changes, or email notifications were executed.

### Bonus. Illustrative SQL query

```sql
SELECT *
FROM log_in_attempts
WHERE ip_address = '203.0.113.45';
```

The query follows the supplied table and column names; no database was supplied and the query was not executed.

## Part 6: Add It to Your Portfolio

### 1. Non-technical incident summary

BrightBake Online, a fictional online bakery, recorded several blocked attempts to reach a company server, followed by an allowed connection. The same external address then made four unsuccessful attempts to sign in to an administrator account before a successful login. A few minutes later, the server sent about 48.2 million bytes to another external address. A separate alert also reported a new administrator group member.

These events suggest possible unauthorized access and information loss, but the supplied records do not prove that files were stolen or identify who signed in. An approved administrator or routine business transfer could explain some activity.

This exercise used supplied firewall, login, and network records, a simulated security dashboard, and a short packet capture produced by tcpdump. No live systems or security tools were operated during the analysis.

BrightBake should preserve the records, confirm whether the login and account change were authorized, and identify the purpose of the outgoing traffic. If malicious activity is confirmed, responders should contain it, reset affected credentials, add stronger login protection, and check that services are safe before restoring normal operations.

### 2. Skills and demonstrated understanding

| Tool or concept | Understanding from this activity | Evidence level |
|---|---|---|
| SIEM and dashboards | Alerts summarize patterns that must be checked against underlying logs. | Interpreted a supplied dashboard; no platform operated. |
| tcpdump | Source/destination fields, flags, and payload lengths constrain packet interpretation. | Analyzed supplied text output; no capture performed. |
| Wireshark | A graphical analyzer helps inspect decoded packets and conversations. | Conceptual only. |
| Linux | Shell navigation and file permissions support log access and protection. | Conceptual only. |
| SQL | A `WHERE` condition filters login rows by IP address. | Illustrative query written; not executed. |
| Python | Scripts can automate repeated comparisons and approved alert notifications. | Conceptual only. |
| Incident-response playbooks | Validation, escalation, containment, recovery, and review should be coordinated. | Proposed sequence and improvements; no response executed. |

For a resume, describe these as **foundational knowledge and training-case analysis**, with hands-on proficiency claimed only where separately demonstrated. Splunk and Google SecOps were compared conceptually, not used in this exercise.

### 3. Portfolio location

**GitHub** hosts this portfolio because recruiters can view readable Markdown reports through a public link, while version history records revisions. The [BrightBake repository](https://github.com/Ecxlipze/BrightBake-Security-Incident-Analysis) clearly identifies the scenario as fictional and distinguishes supplied evidence from proposed actions.
