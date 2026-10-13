# Log Analysis and Event Timeline

## 1. Log source identification

| Extract | Log type | Supporting clue |
|---|---|---|
| A | Firewall log | `DENY` and `ALLOW` for TCP connections and destination ports |
| B | Server/authentication log | `sshd: Failed password` and `Accepted password` |
| C | Network/flow log | `src`, `dst`, destination port, and `bytes_out` |

## 2. Evidence excerpts

### Log A: firewall

```text
2026-10-12 02:14:07 DENY TCP 203.0.113.45:51544 -> 10.0.0.12:22
2026-10-12 02:14:09 DENY TCP 203.0.113.45:51545 -> 10.0.0.12:22
2026-10-12 02:14:11 DENY TCP 203.0.113.45:51546 -> 10.0.0.12:3389
2026-10-12 02:19:58 ALLOW TCP 203.0.113.45:51600 -> 10.0.0.12:22
```

Interpretation: The source tried SSH (`22`) and RDP (`3389`). Three connections were denied; a later SSH connection was allowed. **ALLOW does not itself mean authentication succeeded.**

### Log B: SSH authentication

```text
2026-10-12 02:20:31 auth-server sshd: Failed password for admin from 203.0.113.45
2026-10-12 02:20:36 auth-server sshd: Failed password for admin from 203.0.113.45
2026-10-12 02:20:41 auth-server sshd: Failed password for admin from 203.0.113.45
2026-10-12 02:20:47 auth-server sshd: Failed password for admin from 203.0.113.45
2026-10-12 02:21:02 auth-server sshd: Accepted password for admin from 203.0.113.45
```

Interpretation: Four failures preceded a successful password authentication for `admin`. This could reflect password guessing or an authorized user retrying; corroboration is required.

### Log C: outbound network flows

```text
2026-10-12 02:25:44 src=10.0.0.12 dst=198.51.100.7:443 bytes_out=48,213,992
2026-10-12 02:26:10 src=10.0.0.12 dst=198.51.100.7:21 bytes_out=12,004
```

Interpretation: The records report approximately **48.2 MB (decimal)** outbound to HTTPS port `443` and **12.0 KB (decimal)** to port `21` (commonly associated with FTP). These counts do not reveal content, legitimacy, or whether data left the organization without approval.

### `tcpdump` excerpt

```text
02:25:44.101 IP 10.0.0.12.49822 > 198.51.100.7.443: Flags [P.], length 1448
02:25:44.102 IP 10.0.0.12.49822 > 198.51.100.7.443: Flags [P.], length 1448
02:25:44.103 IP 10.0.0.12.49822 > 198.51.100.7.443: Flags [P.], length 1448
02:26:10.004 IP 10.0.0.12.49830 > 198.51.100.7.21: Flags [S], length 0
```

The first three packets show TCP payloads of **1,448 bytes each**, from source port `49822` to destination port `443`. Here `[P.]` represents PSH + ACK. The final line is a TCP SYN (`[S]`) to port `21` with **zero TCP payload bytes**: it indicates a connection initiation attempt, **not evidence that an FTP file was transferred**. The short capture supports the direction, addresses, ports, and timing of Log C, but cannot independently verify its full reported byte totals.

## 3. Unified timeline

| Timestamp (2026-10-12) | Source | Observation | Significance |
|---|---|---|---|
| 02:14:07 | Firewall | DENY `203.0.113.45` → `10.0.0.12:22` | Blocked SSH attempt |
| 02:14:09 | Firewall | DENY same source → port `22` | Another blocked SSH attempt |
| 02:14:11 | Firewall | DENY same source → port `3389` | Blocked RDP attempt |
| 02:19:58 | Firewall | ALLOW same source → port `22` | SSH connection permitted |
| 02:20:31 | Authentication | Failed `admin` password | First observed failure |
| 02:20:36 | Authentication | Failed `admin` password | Second failure |
| 02:20:41 | Authentication | Failed `admin` password | Third failure |
| 02:20:47 | Authentication | Failed `admin` password | Fourth failure |
| 02:21:02 | Authentication | Accepted `admin` password | Login succeeded; identity/authorization unknown |
| 02:25:44 | Network flow | `48,213,992` bytes outbound to `198.51.100.7:443` | Possible unusual transfer |
| 02:25:44.101–.103 | `tcpdump` | Three `1448`-byte TCP payloads to port `443` | Confirms traffic direction/port |
| 02:26:10 | Network flow | `12,004` bytes outbound to `198.51.100.7:21` | Further suspicious outbound flow |
| 02:26:10.004 | `tcpdump` | SYN to port `21`, `length 0` | Connection attempt only |

**Timestamp caveat:** The log extracts use different levels of precision, and no time zones or clock synchronization details are supplied. Alignment is provisional until verified. The activity narrative says Sunday night, but the printed log date (2026-10-12) is a Monday; this report preserves the log date. The scenario implies that `auth-server` corresponds to `10.0.0.12`; confirm that mapping against asset records in a real investigation.

## 4. Address and port reference

| Value | Role in this case |
|---|---|
| `203.0.113.45` | Suspected external source of access attempts; **not conclusively an attacker** |
| `10.0.0.12` | BrightBake internal server; RFC 1918 private address |
| `198.51.100.7` | External destination for recorded outbound flows |
| `22` | SSH |
| `3389` | RDP |
| `443` | Commonly HTTPS |
| `21` | Commonly FTP control connection |

`203.0.113.45` and `198.51.100.7` are example/documentation addresses (TEST-NET ranges) used in this fictional case.

## 5. SIEM dashboard triage

| Investigation priority | Alert | Dashboard severity | Reason |
|---|---|---|---|
| 1 | Multiple failed SSH logins, then success | High | Possible compromised administrator account |
| 2 | Large outbound data transfer | High | Potential loss of sensitive data |
| 3 | New user added to admin group | Medium | Could indicate persistence; verify timestamp and actor urgently |
| 4 | Blocked port scan | Medium | Context for the access attempts |
| 5 | Expired TLS certificate warning | Low | Operational risk, but less directly linked to this incident |

Priority may change as new evidence emerges. The dashboard alert for a new admin-group user is **not supported by a detailed source log in the supplied materials**.

## 6. Working hypothesis and alternatives

**Working hypothesis:** Someone attempted access from `203.0.113.45`, successfully authenticated as an administrator, and then initiated outbound traffic that may reflect data exfiltration.

**Alternative explanations:** An authorized administrator mistyped a password, followed by legitimate backups, deployments, or uploads to a known business service. A scheduled job could also explain the traffic. The admin-group change could be unrelated.

## 7. Missing evidence

- Was the SSH login approved, and who controlled the source address?
- Which account was added to the admin group, by whom, and at what time?
- What application/process created the outbound connections?
- Does the external destination belong to an approved vendor or backup service?
- What data, if any, was accessed or transferred?
- Were clocks synchronized and are there additional relevant hosts?

See [`recommendations.md`](recommendations.md) for next investigation and response steps.
