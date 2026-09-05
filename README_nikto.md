# Task 7 — Vulnerability Scanning with Nikto

## Objective
Use Nikto to perform an automated vulnerability scan on a web server, analyse the results, and document identified security issues with recommended remediation steps.

## Tech Stack / Tools
- **Nikto v2.6.0** — web vulnerability scanner
- **Kali Linux** (VirtualBox VM) — attacker/scanning environment
- **DVWA (Damn Vulnerable Web Application)** — intentionally vulnerable test target, run locally via Podman (Docker-compatible container engine)

---

## 1. Installing Nikto

Kali Linux does not ship Nikto by default in this build, so it was installed manually:

```bash
sudo apt update
sudo apt install nikto -y
```

Verify the install:

```bash
nikto -Version
```

Output confirmed: **Nikto v2.6.0**

---

## 2. Setting Up the Target (DVWA)

DVWA was run locally as a container using **Podman** (Kali's Docker-compatible container engine — installed via `sudo apt install podman -y`):

```bash
sudo podman run --rm -it -p 80:80 docker.io/vulnerables/web-dvwa
```

This starts Apache and MySQL inside the container, serving DVWA on `http://127.0.0.1/`.

Setup steps:
1. Visited `http://127.0.0.1/setup.php` and clicked **Create / Reset Database**
2. Logged in with default credentials: `admin` / `password`
3. Set **DVWA Security** level to **Low**, to surface realistic, documentable findings

---

## 3. Running the Scans

**Basic scan:**
```bash
nikto -h http://127.0.0.1/
```

**Scan with output saved to file:**
```bash
nikto -h http://127.0.0.1/ | tee nikto_scan_results.txt
```

**SSL scan:** Not applicable — this DVWA instance is served over plain HTTP only (no TLS configured on the container), so the `-ssl` flag was not run against it. In a production or HTTPS-enabled deployment, this would be run as:
```bash
nikto -h [target] -ssl
```

---

## 4. Findings

| # | Finding | Severity |
|---|---|---|
| 1 | Session cookie (PHPSESSID) missing `HttpOnly` flag | Medium |
| 2 | Missing security headers (CSP, Permissions-Policy, Referrer-Policy, X-Content-Type-Options, Strict-Transport-Security) | Low |
| 3 | Outdated Apache version (2.4.25) | High |
| 4 | `/config/` directory indexing enabled | High |
| 5 | `/config/` configuration information may be available remotely | High |
| 6 | `/docs/` directory indexing enabled | Medium |
| 7 | `/icons/README` — Apache default file found | Informational |
| 8 | `/login.php` — admin login page/section found | Informational |

### Detailed breakdown

**1. Session cookie missing HttpOnly flag (Medium)**
Without `HttpOnly`, client-side JavaScript can read the session cookie. Combined with any XSS flaw, this lets an attacker steal a session and hijack a logged-in user.
*Fix:* Set `session.cookie_httponly = 1` in `php.ini`, or `ini_set('session.cookie_httponly', 1)` before `session_start()`.

**2. Missing security headers (Low individually, Medium in aggregate)**
No CSP makes any XSS bug easier to exploit; no X-Content-Type-Options allows MIME-sniffing of uploaded files; no Referrer-Policy can leak sensitive URLs to third parties; no Strict-Transport-Security leaves HTTPS deployments open to downgrade attacks; no Permissions-Policy leaves browser features unrestricted for embedded content.
*Fix:* Add via Apache `mod_headers`:
```apache
Header set X-Content-Type-Options "nosniff"
Header set Content-Security-Policy "default-src 'self'"
Header set Referrer-Policy "no-referrer-when-downgrade"
```

**3. Outdated Apache version (High)**
Apache/2.4.25 is well behind current stable releases and likely carries known, publicly documented CVEs.
*Fix:* Upgrade to the latest stable Apache release and maintain a regular patch schedule.

**4 & 5. `/config/` directory indexing and exposure (High)**
Config directories often contain credentials, DB connection strings, or app secrets. Indexing allows browsing and direct download of these files.
*Fix:* Disable listing with `Options -Indexes`; move config files outside the public web root or add explicit `Deny` rules.

**6. `/docs/` directory indexing (Medium)**
Reveals internal file structure and documentation, aiding an attacker's reconnaissance.
*Fix:* Disable indexing with `Options -Indexes`.

**7. Apache default file present (Informational)**
Confirms a stock, un-hardened Apache install — low risk alone but useful recon for an attacker.
*Fix:* Remove or block unused default Apache files.

**8. Admin login page found (Informational)**
Not a vulnerability itself, but identifies a target for brute-force/credential-stuffing attempts.
*Fix:* Pair with account lockout / rate limiting; avoid predictable admin panel paths in production.

---

## 5. What Nikto Does

Nikto is an open-source web server vulnerability scanner. It sends a large number of HTTP requests against a target web server to check for:
- Outdated server software and known vulnerable versions
- Dangerous or default files and CGI scripts
- Missing security headers and cookie flags
- Directory indexing and configuration exposure
- Common misconfigurations

## Nikto's Limitations

- **It is a noisy scanner.** Nikto sends a high volume of requests in quick succession and makes no attempt to be stealthy. This makes it easily detected by IDS/IPS systems and clearly visible in server logs — it's built for authorized testing, not covert reconnaissance.
- It relies on **signature and pattern matching** (known bad files, version banners), so it can produce false positives and won't catch custom application logic flaws or complex authentication/business-logic vulnerabilities. It's a first-pass scanner, not a substitute for a full penetration test.

## Nikto vs Nmap

- **Nmap** is a network/port scanner — it discovers live hosts, open ports, and running services across a network. It answers *"what's alive, and what's listening?"*
- **Nikto** is a web application vulnerability scanner — it only targets HTTP/HTTPS services and digs into a *specific* web server for known-bad files, misconfigurations, outdated banners, and missing security controls. It answers *"what's wrong with this one web app?"*

In practice, Nmap is typically run first to discover web services, then Nikto is pointed at those services for deeper HTTP-level analysis.

---

## Screenshots
See `/screenshots` — includes DVWA setup, Nikto scan in progress, and completed scan output.
