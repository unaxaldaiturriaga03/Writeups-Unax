✅ FTP Service Vulnerability Analysis – Metasploitable2
1. Overview

During the security assessment of the Metasploitable2 target system, an active FTP service was identified and analyzed for potential misconfigurations and weaknesses. The main objective was to enumerate the service, validate authentication mechanisms, and determine whether the configuration could expose sensitive data or lead to further compromise.

2. Service Enumeration
🔹 Port Scanning

An initial network scan revealed that port 21/TCP was open:

nmap -sV -p 21 <TARGET-IP>


Result:

21/tcp open  ftp     vsFTPd 2.3.4


The service was confirmed to be running vsFTPd 2.3.4, a version historically associated with known vulnerabilities, making it a high-value target for further inspection.

3. Authentication Testing
🔹 Anonymous Login Attempt

Based on common FTP misconfigurations, an authentication test was performed using the default anonymous credentials:

ftp <TARGET-IP>
Name: anonymous
Password: anonymous


Result:

230 Login successful.


✅ Anonymous authentication was enabled, indicating a misconfigured access control policy.

4. Post-Authentication Enumeration

Once authenticated, a directory listing was requested:

ftp> ls


Result:
The server allowed read-only access but did not expose any files or directories of immediate value. No write permissions were granted, as confirmed by the following failed upload attempt:

ftp> put test.txt
553 Could not create file.


This verifies that the anonymous user is restricted to read-only privileges.

5. Vulnerability Analysis
✅ Identified Issue

Anonymous FTP Access Enabled

Description:
The FTP server permits unauthenticated users to log in and interact with the service. While access is limited to read-only mode and no sensitive content was exposed during testing, the configuration still violates security best practices.

🔥 Risk Factors

Increases attack surface for automated scanners and brute-force tools.

Could expose sensitive files if content is added to the FTP root in the future.

The service version (vsFTPd 2.3.4) is historically linked to known exploits, compounding risk.

✅ Impact Assessment
Factor	Level
Confidentiality	Low (no sensitive data exposed)
Integrity	Low (no write permissions)
Availability	None
Overall Risk	Low, but security posture is weakened

Although the current impact is limited, anonymous access on legacy FTP services is considered a policy and configuration weakness, and could become a pivot point in a real-world attack chain.

6. Recommendations

✔️ Disable anonymous access in the vsFTPd configuration:

anonymous_enable=NO


✔️ Restrict access to authenticated users only.

✔️ Implement stronger access controls, including:

Enforced authentication

User isolation (chroot)

Logging and monitoring

✔️ Consider migrating to more secure protocols (e.g., SFTP or FTPS).

✔️ Given the outdated version, update or decommission the service if not required.

7. Conclusion

The FTP service on Metasploitable2 is misconfigured, allowing anonymous authentication. Although no sensitive data or write privileges were identified, the service increases exposure and contradicts security best practices. In a real environment, this weakness could be leveraged alongside other vulnerabilities to escalate access or extract sensitive information.

Proper remediation should focus on eliminating anonymous access, updating the service, and transitioning to secure communication protocols.
