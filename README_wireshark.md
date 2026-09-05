# Task 8: Network Traffic Capture and Analysis
## Installation Documentation

### Step-by-Step Process:
1. Downloaded the official Windows 64-bit installer executable from `wireshark.org`.
2. Launched the installer package and accepted the standard end-user license agreements.
3. Enabled the crucial installation checkbox for **Npcap**, which allows the system to sniff and capture live network packets.
4. Completed the wizard setup and verified the application shortcut appeared on the desktop.

### Required Permissions:
* **Administrator Privileges**: To capture live data, Wireshark must be explicitly run with **Administrator permissions** (Right-click -> Run as Administrator). 
* **Why it's required**: Standard user accounts are restricted from accessing raw hardware interfaces for security reasons. Administrator privileges unlock the network interface card (NIC) and put it into **Promiscuous Mode**, allowing Npcap to actively hook into the network driver and copy incoming/outgoing packets.

## Security Observations: HTTP vs. HTTPS

### Why Unencrypted HTTP Traffic is Dangerous
Unencrypted HTTP traffic sends all data over the network in plaintext (clear, readable text). This creates a massive security risk because anyone sitting on the same network pathway can use a packet sniffing tool (like Wireshark) to capture the packets. They can easily read private web requests, view visited pages, steal session cookies, or intercept sensitive information like usernames and passwords without needing a decryption key.

### How HTTPS Prevents Eavesdropping
HTTPS resolves this vulnerability by establishing a secure, encrypted tunnel using TLS (Transport Layer Security). Before any web data is sent, the browser and server securely trade cryptographic keys. All traffic traveling through this tunnel is completely scrambled into unreadable gibberish. Even if an attacker intercepts the data packets using a network sniffer, they will only see random characters that are mathematically impossible to read without the private secret key.

---

## Glossary of Networking Terms

* **Packet**: A tiny, individual bundle of data split up from a larger file so it can easily travel across a digital network.
* **Protocol**: A strict, universally agreed-upon set of communication rules that computers must follow to talk to each other cleanly.
* **Port**: A numbered virtual slot or doorway on a device that directs specific incoming internet traffic straight to the correct app or service.
* **Payload**: The actual core message, text, or data content being carried inside a packet, separate from its routing headers.
* **Handshake**: A quick automated greeting ritual where two connecting computers introduce themselves and safely agree on terms before swapping data.
