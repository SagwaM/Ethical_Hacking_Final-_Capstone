# Final Capstone Report

This repository contains detailed write-ups and solutions for four security challenges focusing on vulnerability discovery, exploitation, and mitigation techniques, including SQL Injection, Web Server Vulnerabilities, SMB Share Exploitation, and Network Traffic Analysis. **@ParoCyber Guided**.

# Table of Contents

- [Challenge 1: SQL Injection](#challenge-1-sql-injection)
- [Challenge 2: Web Server Vulnerabilities](#challenge-2-web-server-vulnerabilities)
- [Challenge 3: Exploit Open SMB Server Shares](#challenge-3-exploit-open-smb-server-shares)
- [Challenge 4: Analyze a PCAP File](#challenge-4-analyze-a-pcap-file)
- [Remediation and Best Practices](#remediation-and-best-practices)


# Challenge 1: SQL Injection
## Objective

Discover user account information by exploiting SQL Injection on a vulnerable web application, crack the password of a specific user (Bob Smith), and access a protected file on a remote server.

## Steps & Approach

**1. Preliminary Setup:**

- Accessed the web app at http://10.5.5.12.

- Logged in with credentials admin / password.

- Set DVWA security level to low to ease exploitation.

**2. User Credential Retrieval:**

- Identified the vulnerable input form on the login or user ID page allowing SQL injection.

- Used an SQL Injection payload to retrieve usernames and password hashes from the users table:
```
1' OR 1=1 UNION SELECT user, password FROM users #
```

- Extracted Bob Smith’s username (`smithy`) and his password hash.

**3. Password Cracking:**

- Used an online hash cracking tool (e.g., CrackStation) to crack Bob’s password hash.

- Successfully cracked the password as `password`.

**4. File Access on Remote Server:**

- SSH’d into the remote server at `192.168.0.10` using the credentials:
```
Username: smithy
Password: password
```

- Located the file `my_password.txt` in the user’s home directory.

- Opened the file and retrieved the challenge code:
```
8748wf8j
```

**5. SQL Injection Remediation Recommendations:**

- Use prepared statements (parameterized queries).

- Validate and sanitize all user inputs.

- Employ stored procedures.

- Enforce least privilege on database users.

- Deploy Web Application Firewalls (WAFs).

# Challenge 2: Web Server Vulnerabilities
## Objective

Identify web server directory misconfigurations allowing directory listing, find directories with exposed files, and retrieve the Challenge 2 flag.

## Steps & Approach

**1. Preliminary Setup:**

- Logged into the server at `http://10.5.5.12` using `admin` / `password`.

- Set security level to low.

**2. Reconnaissance:**

- Used Gobuster to perform directory enumeration:
```
gobuster dir -u http://10.5.5.12 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

- Found directories with HTTP 301 redirects:

- `/docs/`

- `/external/`

- `/config/`

- `/vulnerabilities/`

**3. Flag Discovery:**

- Browsed `/config/` directory.

- Found the file `db_form.html` containing the Challenge 2 flag:
```
CHALLENGE2{aWe-4975}
```

**4. Remediation Recommendations:**

- Disable directory listing via web server configuration (`Options -Indexes` for Apache, `autoindex off`; for Nginx).

- Restrict access to sensitive directories with proper permissions and `.htaccess` rules.

# Challenge 3: Exploit Open SMB Server Shares
## Objective

Discover unsecured SMB shares in the network, identify those accessible without authentication, and extract the Challenge 3 flag file.

## Steps & Approach

**1. SMB Target Discovery:**

- Scanned `10.5.5.0/24` for SMB servers using Nmap on ports 139 and 445.

- Identified SMB service on host `10.5.5.14`.

**2. SMB Share Enumeration:**

-Used `smbclient` to enumerate shares:
```
smbclient -L //10.5.5.14 -N
```

- Found shares:

  - `homes`

  - `workfiles`

  - `print$`

  - `IPC$`

- Confirmed anonymous access to `print$`.

**3. File Retrieval:**

- Connected to `print$` share anonymously:
```
smbclient //10.5.5.14/print$ -N
```

- Navigated to `OTHER` directory.

- Located the file `sxij42.txt`.

- Downloaded and read the file locally to obtain the flag.

**4 Remediation Recommendations:**

- Disable anonymous SMB access.

- Configure strong share permissions.

- Limit SMB access with firewalls and authentication.

# Challenge 4: Analyze a PCAP File to Find Information
## Objective

Analyze network traffic captured in a PCAP file to extract the target’s IP address, identify directories accessed, and find the Challenge 4 flag file.

## Steps & Approach

**1. PCAP File Analysis:**

- Opened `/home/kali/Downloads/SA.pcap` in Wireshark.

- Inspected HTTP traffic, focusing on GET requests and DNS queries.

- Determined the target IP address from the destination IP in HTTP traffic.

- Extracted directories from HTTP GET requests.

**2. Flag File Retrieval:**

- Visited the URLs uncovered in the PCAP using a browser.

- Located the Challenge 4 flag file at:
```
http://10.5.5.1/database-offline.php
```

- Read the flag content, e.g.:
```
CHALLENGE4{}
```

**3. Remediation Recommendations:**

- Use HTTPS (TLS) to encrypt web traffic.

- Employ VPNs or encrypted tunnels for secure communication.

## 🛡️ Remediation and Best Practices Summary

| Vulnerability Type            | Recommended Mitigations                                                                 |
|------------------------------|------------------------------------------------------------------------------------------|
| **SQL Injection**             | Use prepared statements, input validation, stored procedures, least privilege, WAFs     |
| **Directory Listing**         | Disable directory listing, restrict directory access with permissions and `.htaccess`   |
| **SMB Open Shares**           | Disable anonymous access, enforce share permissions, firewall SMB ports                  |
| **Clear-text Network Traffic**| Use HTTPS/TLS, VPNs, encrypted tunnels                                                    |


# Conclusion

These challenges demonstrated practical skills in:

- Identifying and exploiting web and network vulnerabilities.

- Using industry-standard tools like Wireshark, Gobuster, smbclient, and SQL injection techniques.

- Understanding the importance of security best practices and mitigation strategies.

This documentation can serve as a reference for learning common attack vectors and how to protect against them.
