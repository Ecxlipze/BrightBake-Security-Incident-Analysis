# BrightBake Security Incident Analysis

**Project type:** Blue Team / SOC analyst practice case study  
**Environment:** Fictional organization and training logs  
**Focus:** Log correlation, suspicious authentication, possible data exfiltration, triage, and incident response

## Project overview

This project analyzes a fictional incident at **BrightBake**, an online business. A series of blocked inbound connections was followed by an allowed SSH connection, failed administrator logins, a successful administrator login, and outbound network traffic to an external address. The pattern warrants urgent investigation, but the provided evidence **does not conclusively prove** who logged in or whether customer data was stolen.

The goal is to demonstrate how an entry-level security analyst can correlate alerts and logs, reconstruct a timeline, prioritize investigation, and propose proportionate response actions.

## Key findings

- `203.0.113.45` attempted connections to `10.0.0.12` on SSH (`22`) and RDP (`3389`); three attempts were blocked.
- A connection to SSH port `22` from the same source was later allowed.
- The authentication server recorded **four failed SSH password attempts** for `admin`, followed by **one successful password authentication** from `203.0.113.45`.
- Approximately **48.2 million bytes** were reported outbound from `10.0.0.12` to `198.51.100.7:443`, plus **12,004 bytes** to port `21`. These are concerning outbound-flow records, not proof of stolen files.
- An additional dashboard alert reported a **new user added to the admin group**. The supplied evidence does not identify the user, timestamp, or actor, so it requires verification.

## Incident sequence

1. **02:14:07–02:14:11:** Three inbound firewall denies.
2. **02:19:58:** Firewall allows an SSH connection.
3. **02:20:31–02:20:47:** Four failed logins to the `admin` account.
4. **02:21:02:** Successful SSH password login to `admin`.
5. **02:25:44:** Large outbound data volume recorded to external port `443`.
6. **02:26:10:** Additional outbound traffic reported to external port `21`; packet capture shows a TCP SYN to that port, which alone does not establish an FTP transfer.

All times reflect the supplied training logs; no timezone is provided.

## Skills demonstrated

- Identifying **firewall**, **server/authentication**, and **network** logs
- Correlating events across multiple sources in a SIEM-style investigation
- Reading basic `tcpdump` output: source/destination IP, port, TCP flags, and payload length
- Prioritizing alerts and distinguishing **indicators** from **confirmed conclusions**
- Recommending containment, evidence preservation, recovery, and lessons learned
- Mapping the principal risk to the **confidentiality** element of the CIA triad

## Tools and concepts

| Tool/concept | Scope of this project |
|---|---|
| SIEM dashboard | Interpreted a provided alert summary and correlated source logs |
| `tcpdump` | Analyzed a supplied packet-capture excerpt; no live capture was performed |
| Firewalls / SSH / TCP | Interpreted connection decisions, logins, ports, and packet flags |
| Wireshark | Discussed as a possible GUI-based follow-up tool; **not used hands-on** |
| Linux, SQL, Python | Related learning topics; **not used to execute this investigation** |

## Repository contents

- [`incident-report.md`](incident-report.md): Management summary, assessment, and response status
- [`log-analysis.md`](log-analysis.md): Evidence table, unified timeline, and technical interpretation
- [`recommendations.md`](recommendations.md): Prioritized response steps, additional investigations, and prevention

## Important limitations

This is a **fictional educational case study**. The logs were provided as part of an exercise. I did not access BrightBake systems, run a SIEM, capture live traffic, or perform real incident response. IP addresses are documentation/example ranges, not identified real attackers. Several elements are hypotheses pending verification.

## What I would investigate next

Correlate the admin-group membership change with account audit logs, identify the owner and purpose of `198.51.100.7`, review SSH session commands and file access, and examine outbound HTTPS/FTP session metadata and any legitimately retained packet captures.
