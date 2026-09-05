# Full Network Security Assessment Report

**Prepared for:** Oasis Infobyte Internship — Task 10
**Assessment type:** Structured, end-to-end security assessment (Reconnaissance → Traffic Analysis → Web Vulnerability Scan)

---

## 1. Scope of Assessment

**In scope:**
- **Host:** `127.0.0.1` (localhost) — Apache web server hosting DVWA (Damn Vulnerable Web Application), inside an isolated Kali Linux VM lab
- **Service:** Port 80/tcp (HTTP)
- **Network traffic:** Live capture on the host system's Wi-Fi interface, used to demonstrate traffic-analysis methodology (background OS/application traffic — not traffic to/from the DVWA target specifically)

**Out of scope:**
- Any external, production, or third-party systems
- Any host other than the local lab target
- Any network not owned by, or explicitly authorized for, the assessor

**Time window:** Conducted across three lab sessions:
- Phase 1 (Nmap): 14 July 2026
- Phase 2 (Wireshark): Aug 2026
- Phase 3 (Nikto): 1–3 September 2026

**Authorization:** All testing was performed against a self-owned, isolated lab environment (local VM, local Wi-Fi interface). No external or third-party systems were scanned.

---

## 2. Phase 1 — Reconnaissance (Nmap)

**Command used:**
```
nmap -sV -O 127.0.0.1
```

**Results:**
```
Nmap scan report for localhost (127.0.0.1)
Host is up (0.0000020s latency).
Not shown: 999 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.68 (Debian)
```

- Only one host in scope (localhost); one open port found: **80/tcp (HTTP)**
- Service/version detection identified **Apache httpd 2.4.68 (Debian)**
- OS fingerprinting (`-O`) did not return an exact OS match — the TCP/IP stack fingerprint was captured but Nmap could not confirm a specific OS build with confidence

**Screenshot evidence:** `screenshots/nmap_scan.png`

---

## 3. Phase 2 — Traffic Analysis (Wireshark)

5+ minutes of live traffic was captured on the host's Wi-Fi interface and filtered by protocol (HTTP, DNS, TCP) to identify unencrypted or sensitive data in transit.

### HTTP traffic
Filtering on `http` showed cleartext requests, primarily:
- Certificate revocation checks (OCSP/CRL) — e.g. `GET /r4.crl`, `GET /gsr1.crl`
- Windows Update manifest lookups — e.g. `GET /msdownload/update/v3/static/trustedr/en/authrootstl.cab`

**Observation:** No login credentials, session tokens, or application-layer sensitive data were observed in cleartext during this capture window. The cleartext traffic present is background OS/certificate-infrastructure traffic, which is by design unauthenticated and non-sensitive — but it does confirm the host routinely sends *some* traffic over plain HTTP, which is worth noting for a full network hardening review.

### DNS traffic
Filtering on `dns` showed standard queries/responses, predominantly resolving Microsoft Teams/Skype infrastructure domains (`teams.live.com`, `pub-csm-jpea-02-t.trouter.skype.com`). DNS queries are inherently unencrypted (unless DoH/DoT is configured) and reveal which services a host communicates with — this is a passive information-disclosure point, though not a direct vulnerability.

### TCP traffic
Filtering on `tcp` showed the majority of application traffic riding over **TLS 1.2/1.3** (`Application Data`, `Client Hello`, `Server Hello, Change Cipher Spec`), confirming most real application traffic on this host is encrypted in transit.

### ARP traffic
Filtering on `arp` captured 18 ARP packets forming 9 clean request/reply pairs, all between the same two hosts on the local network:

```
Request: 192.168.130.57 asks "Who has 192.168.130.167?" (hwsrc 96:5a:75:01:29:2d)
Reply:   192.168.130.167 responds (hwsrc d8:b3:2f:a7:4f:f7)
```

**Observation:** Every reply for `192.168.130.167` consistently mapped to the same MAC address (`d8:b3:2f:a7:4f:f7`) across all 9 exchanges. This is normal ARP cache-refresh behavior between the assessed host and another device on the same LAN (most likely the gateway/router). **No evidence of ARP spoofing or poisoning was found** — a spoofing attack would typically show conflicting MAC addresses replying for the same IP, or unsolicited/gratuitous ARP replies with no matching request. Neither pattern was present in this capture.

**Screenshot evidence:** `screenshots/http_traffic.png`, `screenshots/dns_traffic.png`, `screenshots/tcp_traffic.png`, `screenshots/arp_traffic.png`

---

## 4. Phase 3 — Web Vulnerability Scan (Nikto)

Target: `http://127.0.0.1/` (DVWA, Apache/2.4.25 at time of scan — see note below on version discrepancy)

**Commands used:**
```
nikto -h http://127.0.0.1/
nikto -h http://127.0.0.1/ | tee nikto_scan_results.txt
```

**Key results:** (full output in `nikto_scan_results.txt`)
- Session cookie (PHPSESSID) missing `HttpOnly` flag
- Missing security headers: CSP, Permissions-Policy, Referrer-Policy, X-Content-Type-Options, Strict-Transport-Security
- Apache/2.4.25 flagged as outdated
- `/config/` — directory indexing enabled, configuration info potentially exposed
- `/docs/` — directory indexing enabled
- `/icons/README` — default Apache file present
- `/login.php` — admin login page discoverable
- `/.gitignore` — file exposed, revealing directory structure
- X-Frame-Options header flagged as deprecated (superseded by CSP `frame-ancestors`)

**Note on version discrepancy:** Nmap (Phase 1, July scan) reported Apache 2.4.68, while Nikto (Phase 3, September scan) reported Apache 2.4.25 on what is nominally the same lab host. This is because the DVWA container was torn down and recreated between sessions using a community Docker image pinned to an older Apache build — not a live downgrade of a single system. In a real assessment, this kind of drift is itself worth flagging: **environment/version consistency should be verified at the start of each engagement**, and reports should always state the exact scan date next to any version finding.

**Screenshot evidence:** `screenshots/dvwa_dashboard.png`, `screenshots/nikto_scan_progress.png`, `screenshots/nikto_scan_complete.png`

---

## 5. Findings Register

| Finding ID | Description | Severity | Affected Asset | Recommended Fix |
|---|---|---|---|---|
| NM-01 | HTTP (port 80) is the only web service exposed; no HTTPS/TLS in use | Medium | 127.0.0.1:80 | Configure HTTPS with a valid TLS certificate; redirect HTTP → HTTPS |
| NM-02 | Service version banner (Apache 2.4.x) disclosed via Nmap/Nikto probing | Low | 127.0.0.1:80 | Set `ServerTokens Prod` and `ServerSignature Off` in Apache config |
| WS-01 | Cleartext HTTP used for OCSP/CRL and update-manifest checks (background OS traffic) | Low | Host network interface | Prefer HTTPS-only update/revocation endpoints where the OS/vendor supports it; monitor for anomalous cleartext traffic |
| WS-02 | DNS queries are unencrypted, revealing which external services the host talks to | Informational | Host network interface | Consider DNS-over-HTTPS/TLS (DoH/DoT) where policy allows |
| WS-03 | ARP traffic reviewed for spoofing indicators — none found; consistent MAC-to-IP mapping observed | Informational | Host network interface | No action required; recommend periodic ARP monitoring (e.g. `arpwatch`) as a general best practice |
| NK-01 | Session cookie (PHPSESSID) missing `HttpOnly` flag | Medium | DVWA / 127.0.0.1 | Set `session.cookie_httponly = 1` in `php.ini` |
| NK-02 | Missing security headers (CSP, Permissions-Policy, Referrer-Policy, X-Content-Type-Options, HSTS) | Low | DVWA / 127.0.0.1 | Add headers via Apache `mod_headers` |
| NK-03 | Apache version outdated at time of scan | High | DVWA / 127.0.0.1 | Upgrade Apache to latest stable release; establish patch cadence |
| NK-04 | `/config/` directory indexing enabled | High | DVWA / 127.0.0.1 | Disable with `Options -Indexes`; move config outside web root |
| NK-05 | `/config/` configuration info may be exposed remotely | High | DVWA / 127.0.0.1 | Move config files outside public web root; add explicit `Deny` rules |
| NK-06 | `/docs/` directory indexing enabled | Medium | DVWA / 127.0.0.1 | Disable with `Options -Indexes` |
| NK-07 | Default Apache file present (`/icons/README`) | Informational | DVWA / 127.0.0.1 | Remove unused default Apache files |
| NK-08 | Admin login page discoverable (`/login.php`) | Informational | DVWA / 127.0.0.1 | Pair with account lockout / rate limiting |
| NK-09 | `/.gitignore` exposed, revealing directory structure | Low | DVWA / 127.0.0.1 | Exclude `.gitignore` and other repo metadata from the deployed web root |
| NK-10 | X-Frame-Options header deprecated (should migrate to CSP `frame-ancestors`) | Informational | DVWA / 127.0.0.1 | Replace with CSP `frame-ancestors` directive |
| ENV-01 | Apache version differed between Phase 1 (Nmap) and Phase 3 (Nikto) scans | Informational | Lab environment | Re-verify environment state at the start of each assessment session; timestamp all version findings |

**Severity methodology:** Ratings are qualitative (Critical/High/Medium/Low/Informational), informed by CVSS v3.1 severity bands — High findings correspond broadly to CVSS ≥7.0 (direct information/config exposure), Medium to CVSS 4.0–6.9, Low/Informational to CVSS <4.0 or non-exploitable disclosure.

---

## 6. Executive Summary

*(Non-technical — for management review)*

This assessment examined a single local test system across three angles: what's reachable on the network (Nmap), what data travels across the network in the clear (Wireshark), and what's wrong with the web application itself (Nikto).

**Overall risk posture: Medium-High.** No traffic-analysis phase turned up leaked passwords or sensitive personal data — that's a positive sign. However, the web application scan found **three High-severity issues**: an outdated web server version, and a configuration folder that can be browsed and potentially downloaded by anyone who reaches the site. These are the kind of gaps that let an attacker map out a system and potentially pull sensitive configuration files (which often contain credentials) without needing to "hack" anything — the server simply hands the information over.

The remaining issues (missing browser security protections, a session cookie that's slightly less locked-down than it should be, some leftover default files) are lower-risk on their own but add up, and are inexpensive to fix.

**Recommendation:** Prioritize closing the three High-severity items first — they represent the most direct path to a real compromise — then work through the Medium and Low items as routine hardening. None of the fixes require replacing the system, only reconfiguring it.

---

## 7. Remediation Roadmap

Ordered by priority (risk first), with rough effort estimates:

| Priority | Finding(s) | Fix | Effort |
|---|---|---|---|
| 1 | NK-03 (outdated Apache) | Upgrade Apache to current stable release | Hard |
| 2 | NK-04, NK-05 (`/config/` exposure) | Disable directory indexing, relocate config outside web root | Easy |
| 3 | NK-01 (cookie HttpOnly) | Set `session.cookie_httponly = 1` | Easy |
| 4 | NK-02 (missing headers) | Add security headers via `mod_headers` | Easy |
| 5 | NK-06 (`/docs/` indexing) | Disable directory indexing | Easy |
| 6 | NM-01 (no HTTPS) | Configure TLS cert, redirect HTTP → HTTPS | Medium |
| 7 | NK-09 (`.gitignore` exposed) | Exclude repo metadata from deployment | Easy |
| 8 | NM-02, NK-07, NK-10 (banner/default files/deprecated header) | `ServerTokens Prod`, remove default files, migrate to CSP `frame-ancestors` | Easy |
| 9 | NK-08 (login page discoverable) | Add rate limiting / account lockout | Medium |
| 10 | ENV-01 (version drift) | Standardize/document environment build process for future assessments | Easy |

---

## 8. Repository Contents

```
├── network_security_assessment.md   (this report)
├── nmap_scan_results.txt
├── nikto_scan_results.txt
├── wireshark_capture.pcap/          (dns.pcapng, http.pcapng, tcp.pcapng)
└── screenshots/
    ├── nmap_scan.png
    ├── http_traffic.png
    ├── dns_traffic.png
    ├── tcp_traffic.png
    ├── arp_traffic.png
    ├── dvwa_dashboard.png
    ├── nikto_scan_progress.png
    └── nikto_scan_complete.png
```
