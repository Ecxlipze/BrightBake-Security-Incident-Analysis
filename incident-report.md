# Incident Report: Suspicious Login and Possible Data Exfiltration

**Case:** BrightBake (fictional training scenario)  
**Classification:** Suspected unauthorized access / possible data exfiltration  
**Priority:** High, pending confirmation  
**Status:** Analysis of supplied evidence only; response actions not executed

## Executive summary (non-technical)

During a security review, BrightBake's logs showed an unusual series of attempts to access a company server. Several connections were blocked, but a later connection was allowed. The same external address then made four unsuccessful attempts to sign in to an administrator account before a successful login was recorded.

A few minutes later, the server was associated with a large amount of outgoing network traffic to an external address. This raises the possibility that someone gained unauthorized access and transferred company information. However, the available records do not confirm who signed in or whether any sensitive files were taken.

I reviewed the supplied firewall, login, and network records, compared alerts from a simulated security monitoring dashboard, and interpreted a short packet-capture excerpt. These sources helped establish the order of events and identify issues needing urgent attention.

BrightBake should preserve relevant evidence, verify whether the login was authorized, protect potentially affected accounts, and investigate the outgoing traffic. If the activity is confirmed as malicious, the incident response team should contain it, reset credentials, enable multi-factor authentication, and check for any additional unauthorized accounts. Management should receive updates and a final review of lessons learned.

## Scope and sources

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
