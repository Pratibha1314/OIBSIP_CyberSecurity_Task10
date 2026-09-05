# Task 1: Basic Network Scanning with Nmap

## 1. Nmap Installation Steps
To install Nmap on a Debian-based Linux system (such as Kali Linux or Ubuntu), run the following standard commands:
'sudo apt update'
'sudo apt install nmap -y'

## 2. What Nmap Is
Nmap (Network Mapper) is an open-source tool used for network discovery and scurity auditing. It sends specially crafted network packets to target hosts to discover open ports, active services , and operating system details.

## 3. Why Network Scanning Matters 
Network scanning maps out the available attack surface of a system. It helps security professionals identify exposed entry points, unauthorized open ports, and outdated software versions before an external attacker can exploit them.

## 4. Port Analysis & Security Assessment
Based on our scan results of the target IP (127.0.0.1), the following port details were uncovered:
* **Port Found**: 80/tcp
* **State**: Open
* **Service/Version**:Apache httpd 2.4.68 ((Debian))
* **Service Explanation**: Port 80 runs HTTP traffic, which handles unencrypted web page delivery to web browsers.
* **Security Risk**: Plain HTTP transmits all traffic and data in clear text. This exposes the system to data interception or sniffing attacks. To secure this service, it should be configured to use HTTPS on port 443 with an SSL/TLS certificate.

## 5. Ethical Use Guidelines
* Only perform network scans against assets you own or have explicit,formal written authorization to test.
* Never scan production networks, public infrastructure, or external websites without permission.
* Always execute testing inside a controlled, isolated laboratory virtual environment.
