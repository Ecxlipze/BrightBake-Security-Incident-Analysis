# Incident Report: Suspicious Login and Possible Data Exfiltration

**Case:** BrightBake (fictional training scenario)  
**Classification:** Suspected unauthorized access / possible data exfiltration  
**Priority:** High, pending confirmation  
**Status:** Analysis of supplied evidence only; response actions not executed

## Executive summary (non-technical)

BrightBake Online, a fictional online bakery, recorded several blocked attempts to reach a company server, followed by an allowed connection. The same external address then made four unsuccessful attempts to sign in to an administrator account before a successful login. A few minutes later, the server sent about 48.2 million bytes to another external address. A separate alert also reported a new administrator group member.

These events suggest possible unauthorized access and information loss, but the supplied records do not prove that files were stolen or identify who signed in. An approved administrator or routine business transfer could explain some activity.

This exercise used supplied firewall, login, and network records, a simulated security dashboard, and a short packet capture produced by tcpdump. No live systems or security tools were operated during the analysis.

BrightBake should preserve the records, confirm whether the login and account change were authorized, and identify the purpose of the outgoing traffic. If malicious activity is confirmed, responders should contain it, reset affected credentials, add stronger login protection, and check that services are safe before restoring normal operations.

## Scope and sources

Source: the user-supplied four-page `Module-4-Activity-Cybersecurity-Tools.pdf`, titled *Course 1, Module 4 Activity: Cybersecurity Tools* and dated 13 October 2026.

Evidence supplied for the exercise:

1. **Log A:** Firewall connection decisions.
2. **Log B:** SSH authentication messages.
3. **Log C:** Outbound network-flow summaries.
4. **SIEM-style dashboard:** Five alert categories and relative severity.
5. **`tcpdump` excerpt:** Selected packets showing traffic to ports `443` and `21`.

No original full log files, server access, packet capture file, or user identity records were made available.

## Findings

| Finding | Assessment | Confidence |
|---|---|---|
| Repeated connections from `203.0.113.45` | Suspicious external access attempts | High |
| Successful `admin` SSH password login after failures | A login occurred; authorization unknown | High |
| Outbound traffic to `198.51.100.7` | Large transfer indicated by flow log; content unknown | High for recorded volume; low for theft conclusion |
| New admin-group user | Possible persistence, but not yet connected to incident | Low |

## Business risk

**Primary CIA-triad concern: confidentiality.** If the administrator login was unauthorized and outbound traffic carried internal data, sensitive information may have been exposed. **Integrity** could also be at risk because an administrator account may change accounts, permissions, and systems. The available logs do not demonstrate service disruption.

## Response status

This portfolio exercise documents **recommended** actions only. No IP addresses were blocked, accounts disabled, credentials reset, or evidence collected from real systems.

## Conclusion

Treat the pattern as a high-priority suspected incident, verify it with additional evidence, and follow an approved incident response playbook. See [`log-analysis.md`](log-analysis.md) for the evidence and [`recommendations.md`](recommendations.md) for next steps.
