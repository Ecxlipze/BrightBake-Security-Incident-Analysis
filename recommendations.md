# Response Recommendations and Lessons Learned

**Scenario:** BrightBake suspected unauthorized administrator login and possible data exfiltration.  
**Note:** Actions below are proposals for a fictional exercise; none have been carried out.

## 1. Proposed playbook order

**B → D → E → A → F → C**

| Step | Action | Practical application |
|---|---|---|
| B | Receive and acknowledge SIEM alert | Open and assign an incident ticket |
| D | Verify using related logs | Correlate firewall, SSH, network, and packet evidence; rule out known legitimate activity |
| E | Escalate | Notify the incident response team and manager through approved channels |
| A | Contain | In coordination with responders, block malicious access and disable confirmed compromised accounts |
| F | Recover | Reset affected credentials, enable MFA, remove unauthorized access, validate systems, then restore normal operations |
| C | Review | Document timeline, impact, response, and prevention measures |

**Additional critical step:** Preserve relevant evidence with timestamps and access controls **as early as practical**, without delaying necessary urgent containment.

## 2. Immediate investigation and containment

1. **Confirm the login:** Check SSH session records, authorized maintenance windows, user identity, VPN records, and asset ownership.
2. **Preserve evidence:** Secure copies of the source logs, relevant audit trails, timestamps, network metadata, and available packet captures. Document handling and who accessed them.
3. **Validate the admin-group alert:** Identify the created/promoted user, actor, timestamp, and changes made; remove unauthorized access following the approved process.
4. **Investigate outbound data:** Check the destination's business purpose, transfer volume baseline, proxy/firewall logs, file-access logs, process telemetry, and authorized backup or deployment jobs.
5. **Contain confirmed malicious activity:** Under incident-response approval, disable or isolate affected accounts/hosts, block malicious indicators where appropriate, and watch for alternative access paths.
6. **Protect business operations:** Coordinate changes with service owners, especially if administrator account disablement could interrupt orders or critical services.

## 3. Recovery and prevention

- Reset compromised passwords and revoke relevant sessions, tokens, and credentials.
- Require **multi-factor authentication (MFA)** for administrator and remote access wherever supported.
- Limit SSH exposure with VPN or allowlisted management networks; restrict unneeded RDP access.
- Apply least privilege; review administrator-group membership and remove unnecessary permissions.
- Configure alerts for repeated failed logins followed by success, unusual privilege changes, and abnormal outbound volumes.
- Maintain reliable time synchronization and consistent timestamp/timezone conventions across logs.
- Establish network egress controls and baselines for approved outbound destinations.
- Apply system updates and review authentication and monitoring configurations.

## 4. Determine whether data was exposed

Before concluding there was a breach, responders should identify: which accounts and services were accessed, what data the server could reach, which files were read, whether transfers were legitimate, and whether records suggest actual loss. If sensitive data exposure is confirmed, involve the organization's legal/privacy response personnel to assess applicable reporting obligations.

## 5. Lessons learned and new playbook step

**New playbook step:** *Preserve and document evidence immediately after initial triage, in parallel with urgent containment when needed.*

Why: Evidence supports accurate attribution of actions, impact assessment, recovery decisions, and post-incident improvements. Avoid changing or deleting records unnecessarily during the investigation.

## 6. Tool selection for follow-up work

| Tool | Appropriate use |
|---|---|
| SIEM | Correlate log sources, view alerts, and construct a timeline |
| Network protocol analyzer (`tcpdump`, Wireshark) | Examine authorized packet captures for protocol details and traffic behavior |
| Linux shell | Inspect relevant system logs and process evidence on authorized systems |
| SQL | Query login-event tables in an approved security data store |
| Python | Automate repeatable log parsing and analysis after validation |

### Example SQL query (illustrative only)

If a training table named `log_in_attempts` contains `ip_address`, one possible lookup is:

```sql
SELECT *
FROM log_in_attempts
WHERE ip_address = '203.0.113.45';
```

This query was **not executed** in this case study.

## 7. Completion checklist

- [ ] Validate whether the SSH login was authorized
- [ ] Verify the admin-group change
- [ ] Determine whether outbound traffic was expected
- [ ] Preserve and correlate additional evidence
- [ ] Contain any confirmed compromise
- [ ] Confirm recovery and monitor for recurrence
- [ ] Complete the incident report and update the playbook

**Portfolio disclaimer:** Fictional training case; no real organization was investigated and no live security controls were changed.
