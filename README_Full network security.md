# Task 10 — Full Network Security Assessment Report

## Objective
Conduct a structured, end-to-end security assessment of a local test network using multiple tools, and produce a professional security report suitable for presentation to a technical team or management.

## Steps Performed
1. **Reconnaissance (Nmap)** — Scanned the target host to discover open ports and running services. See `README_nmap.md` and `nmap_scan_results.txt`.
2. **Traffic Analysis (Wireshark)** — Captured live network traffic and filtered by HTTP, DNS, TCP, and ARP to check for unencrypted or sensitive data, and to rule out ARP spoofing. See `README_wireshark.md`.
3. **Web Vulnerability Scan (Nikto)** — Scanned the identified web server (DVWA) for known vulnerabilities and misconfigurations. See `README_nikto.md`.
4. **Reporting** — Consolidated all findings into a single findings register with severity ratings, an executive summary, and a prioritized remediation roadmap. See `network_security_assessment.md`.

## Tools Used
- **Nmap** — network/port scanning and service detection
- **Wireshark** — live traffic capture and protocol analysis
- **Nikto** — automated web vulnerability scanning
- **DVWA** (Damn Vulnerable Web Application) — test target for the web scan phase
- **Kali Linux** — assessment environment

## Outcome
The assessment identified an overall **Medium-High** risk posture. No sensitive credentials or personal data were found in transit during traffic analysis, and no ARP spoofing was detected. However, the web application scan surfaced **3 High-severity findings** (an outdated web server version and an exposed, browsable configuration directory), along with several Medium/Low findings (missing security headers, a session cookie missing the HttpOnly flag, directory indexing). A full findings register, executive summary, and prioritized remediation roadmap are documented in `network_security_assessment.md`.

## Files in This Repository
| File | Description |
|---|---|
| `network_security_assessment.md` | Full report: scope, findings register, executive summary, remediation roadmap |
| `README_nmap.md` | Nmap phase details |
| `README_wireshark.md` | Wireshark phase details |
| `README_nikto.md` | Nikto phase details |
| `nmap_scan_results.txt` | Raw Nmap output |
| `nikto_scan_results.txt` | Raw Nikto output |
| `ARP scanning 3.pcapng` | Raw ARP capture |
| `nmap.png`, `dvwa_dashboard.png`, `nikto_scan_progress.png`, `nikto_scan_complete.png`, `HTTP_Traffic.png`, `DNS_Traffic.png`, `TCP_Traffic.png`, `ARP_Traffic.png` | Screenshot evidence for each phase |
