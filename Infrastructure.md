# Infrastructure Penetration Testing Methodology

A comprehensive checklist for conducting internal and external infrastructure security assessments. Approach each engagement systematically — thorough enumeration will always surface more attack surface than rushing to exploitation.

---

## Pre-Engagement & Scoping

* [ ] Confirm scope — IP ranges, hostnames, excluded systems, and testing windows
* [ ] Clarify rules of engagement (DoS permitted? Phishing in scope? Credential stuffing?)
* [ ] Confirm emergency contact details and escalation procedure
* [ ] Identify whether assessment is black box, grey box, or white box
* [ ] Confirm whether credentials or VPN access are provided (internal assessment)
* [ ] Agree on reporting format and delivery timeline
* [ ] Set up attack infrastructure and logging (log all commands and timestamps)

---

## External Reconnaissance (OSINT)

* [ ] Enumerate subdomains (subfinder, amass, dnsx, crt.sh)
* [ ] Perform reverse DNS lookups across identified IP ranges
* [ ] Identify ASN and IP ownership (BGP.he.net, whois)
* [ ] Check DNS records (A, MX, TXT, SPF, DMARC, DKIM, CNAME, NS, PTR)
* [ ] Identify email provider and check for email spoofing via SPF/DMARC misconfiguration
* [ ] Search Shodan, Censys, and FOFA for exposed services on in-scope IPs
* [ ] Search for exposed credentials in breach databases (dehashed, haveibeenpwned)
* [ ] Search GitHub, GitLab, and Pastebin for leaked credentials and internal references
* [ ] Review LinkedIn and job postings for technology stack and internal tooling leakage
* [ ] Check for zone transfer vulnerability (dig axfr)
* [ ] Identify VPN, RDP, Citrix, and remote access portals
* [ ] Check for exposed cloud storage (S3, Azure Blob, GCS)

---

## Network Scanning & Enumeration

* [ ] Perform host discovery across all in-scope ranges (nmap -sn, fping)
* [ ] Perform full port scan on all live hosts (nmap -p- or masscan)
* [ ] Perform service version detection and OS fingerprinting (nmap -sV -sC -O)
* [ ] Perform UDP scanning on key ports (nmap -sU — 53, 67, 69, 123, 161, 500)
* [ ] Identify all running services and map to CVE exposure
* [ ] Check for network segmentation gaps — can you reach OOB segments?
* [ ] Capture and review broadcast traffic (Responder in analyse mode)
* [ ] Check for LLMNR, NBT-NS, and mDNS activity on the network
* [ ] Identify domain controllers, file servers, mail servers, and key infrastructure

---

## Service Enumeration

### FTP (21)
* [ ] Test for anonymous FTP access
* [ ] Check FTP banner for version information
* [ ] Test for writable directories
* [ ] Check for sensitive files accessible via anonymous login
* [ ] Test for FTP bounce attack potential

### SSH (22)
* [ ] Check SSH version and identify CVEs (especially older OpenSSH versions)
* [ ] Test for weak or default credentials
* [ ] Check supported authentication methods (ssh -v)
* [ ] Test for username enumeration (OpenSSH < 7.7 timing attack)
* [ ] Check for exposed private keys on web servers or shares
* [ ] Test for SSH key reuse across hosts

### Telnet (23)
* [ ] Test for default credentials
* [ ] Capture credentials — telnet transmits in plaintext

### SMTP (25 / 587 / 465)
* [ ] Check for open relay misconfiguration
* [ ] Enumerate valid users via VRFY, EXPN, and RCPT TO
* [ ] Check for email spoofing potential (no SPF/DMARC enforcement)
* [ ] Check SMTP banner for version information

### DNS (53)
* [ ] Test for zone transfer (dig axfr @target domain.com)
* [ ] Test for DNS cache poisoning potential
* [ ] Check for wildcard DNS records
* [ ] Enumerate subdomains via DNS brute force

### HTTP/HTTPS (80 / 443 / 8080 / 8443)
* [ ] Refer to Web Application Methodology (Web.md) for full coverage
* [ ] Check for default web server pages and admin interfaces
* [ ] Identify web application framework and version
* [ ] Check for common management interfaces (Tomcat manager, WebLogic console, JBoss)

### SMB (445 / 139)
* [ ] Check for null session access (smbclient -N, crackmapexec)
* [ ] Enumerate shares and check for read/write access
* [ ] Check for sensitive files in accessible shares (passwords, configs, backups)
* [ ] Check SMB signing status — required or not enforced?
* [ ] Test for EternalBlue (MS17-010) on unpatched hosts
* [ ] Test for PrintNightmare (CVE-2021-34527) on print servers
* [ ] Check for SMBGhost (CVE-2020-0796) on Windows 10 / Server 2019 hosts
* [ ] Enumerate users, groups, and password policy via RPC (rpcclient, enum4linux-ng)

### LDAP (389 / 636)
* [ ] Test for anonymous LDAP bind
* [ ] Enumerate domain users, groups, and computers via LDAP
* [ ] Check for sensitive attributes exposed (description fields, passwords)
* [ ] Use ldapsearch or windapsearch for enumeration

### RDP (3389)
* [ ] Test for default or weak credentials (hydra, crowbar)
* [ ] Check for BlueKeep (CVE-2019-0708) on older Windows hosts
* [ ] Check for NLA enforcement — if disabled, test for credential capture
* [ ] Check if RDP is exposed externally unnecessarily

### WinRM (5985 / 5986)
* [ ] Test for default or weak credentials (evil-winrm, crackmapexec)
* [ ] Check if WinRM access leads to a privileged session

### SNMP (161 UDP)
* [ ] Test for default community strings (public, private, community)
* [ ] Enumerate system information, running processes, installed software, and network interfaces via SNMP
* [ ] Test SNMPv1/v2 for community string brute force (onesixtyone, hydra)
* [ ] Check for write access via community string

### MSSQL (1433)
* [ ] Test for default sa credentials
* [ ] Test for Windows authentication brute force
* [ ] Check for xp_cmdshell enabled
* [ ] Enumerate linked servers
* [ ] Test for UNC path injection (xp_dirtree for hash capture)

### MySQL (3306)
* [ ] Test for default root credentials with no password
* [ ] Test for remote root login permitted
* [ ] Enumerate databases, tables, and sensitive data

### NFS (2049)
* [ ] Enumerate NFS exports (showmount -e)
* [ ] Check for no_root_squash on exports
* [ ] Check for world-readable or writable shares
* [ ] Mount and enumerate accessible NFS shares

### Redis (6379)
* [ ] Test for unauthenticated access
* [ ] Test for weak or default password
* [ ] Attempt to write SSH keys via Redis CONFIG SET
* [ ] Check for SSRF vector to reach internal Redis

---

## Vulnerability Assessment

* [ ] Run authenticated vulnerability scan (Nessus, OpenVAS, Qualys)
* [ ] Run unauthenticated scan and compare results
* [ ] Identify missing patches and map to public exploits (CVE, Exploit-DB)
* [ ] Identify end-of-life operating systems (Windows Server 2008, Windows 7, etc.)
* [ ] Check for known critical CVEs on identified service versions
* [ ] Verify vulnerability findings manually before exploitation

---

## Credential Attacks

* [ ] Perform password spraying against identified services (SSH, SMB, WinRM, RDP, OWA, VPN)
* [ ] Use organisation-specific password patterns (Company1!, Season+Year)
* [ ] Monitor for account lockout thresholds before spraying
* [ ] Test for credential reuse across services with any captured credentials
* [ ] Perform hash cracking on any captured NTLM hashes (hashcat, john)
* [ ] Test for Kerberoasting if domain access is available (see AD methodology)
* [ ] Check for cleartext credentials in configuration files, scripts, and shares

---

## Network Attacks

* [ ] Run Responder to capture NTLMv2 hashes via LLMNR/NBT-NS/mDNS poisoning
* [ ] Attempt NTLM relay with ntlmrelayx when SMB signing is not enforced
* [ ] Test for IPv6 DNS takeover via mitm6
* [ ] Perform ARP spoofing in permissive network environments (bettercap)
* [ ] Test for rogue DHCP server potential
* [ ] Attempt NTLM relay to LDAP for user or computer account creation
* [ ] Test for ADCS relay attacks (ESC8) if Certificate Services is present

---

## Linux Post-Exploitation & Privilege Escalation

* [ ] Identify current user context and groups (id, whoami, groups)
* [ ] Check sudo permissions (sudo -l)
* [ ] Look for SUID/SGID binaries (find / -perm -4000 2>/dev/null)
* [ ] Check for world-writable files and directories
* [ ] Check for sensitive files readable by current user (/etc/shadow, SSH keys, .bash_history)
* [ ] Check cron jobs for writable scripts or paths (crontab -l, /etc/cron*)
* [ ] Check running processes for interesting services (ps aux)
* [ ] Check for writable paths in root-owned cron scripts
* [ ] Enumerate installed software and check versions for local exploits
* [ ] Check for capabilities set on binaries (getcap -r / 2>/dev/null)
* [ ] Check NFS mounts with no_root_squash
* [ ] Check Docker socket exposure (/var/run/docker.sock)
* [ ] Check for container escape indicators (/.dockerenv, cgroup inspection)
* [ ] Review environment variables for credentials (env, printenv)
* [ ] Check for passwords in configuration files and scripts
* [ ] Run LinPEAS for automated enumeration
* [ ] Check kernel version for local kernel exploits (uname -r)
* [ ] Check for writable /etc/passwd (add root-level user)

---

## Windows Post-Exploitation & Privilege Escalation

* [ ] Identify current user context and privileges (whoami /all)
* [ ] Check for SeImpersonatePrivilege / SeAssignPrimaryTokenPrivilege (Potato attacks)
* [ ] Check for SeBackupPrivilege / SeRestorePrivilege (SAM/SYSTEM dump)
* [ ] Check for SeDebugPrivilege (process injection)
* [ ] Enumerate local users, groups, and administrators (net user, net localgroup)
* [ ] Check installed software and versions for local exploits
* [ ] Check running services for unquoted service paths
* [ ] Check service binary permissions for writable executables
* [ ] Check registry autoruns for writable paths (autoruns, reg query)
* [ ] Check scheduled tasks for writable script paths
* [ ] Check AlwaysInstallElevated registry key
* [ ] Check for cleartext credentials in registry (Autologon, WinSCP, PuTTY)
* [ ] Check for cleartext credentials in common file locations (unattend.xml, sysprep.xml, web.config)
* [ ] Search for passwords in files (findstr /si password *.txt *.xml *.ini *.config)
* [ ] Dump SAM/SYSTEM for local hash extraction (reg save, impacket-secretsdump)
* [ ] Run WinPEAS for automated enumeration
* [ ] Check for DLL hijacking opportunities in service binary paths
* [ ] Check for token impersonation opportunities
* [ ] Check Windows version for unpatched local privilege escalation CVEs

---

## Lateral Movement

* [ ] Test captured credentials against all other in-scope hosts (crackmapexec)
* [ ] Test for pass-the-hash with captured NTLM hashes (crackmapexec, impacket)
* [ ] Test for pass-the-ticket with captured Kerberos tickets
* [ ] Use psexec, smbexec, wmiexec, or winrm for remote execution
* [ ] Check for reused local administrator passwords across hosts
* [ ] Identify and pivot to additional network segments (chisel, ligolo-ng)
* [ ] Check for credentials stored in memory with Mimikatz (if AV permits)
* [ ] Dump LSASS for credential extraction (procdump, task manager, comsvcs.dll)
* [ ] Search for credentials in accessible network shares

---

## Pivoting & Tunnelling

* [ ] Identify network segments reachable from compromised hosts
* [ ] Set up SOCKS proxy via chisel or ligolo-ng for traffic routing
* [ ] Tunnel tools through proxy (proxychains configuration)
* [ ] Perform port forwarding for specific service access (ssh -L, chisel)
* [ ] Enumerate newly accessible segments from pivot point
* [ ] Document network topology as access expands

---

## Reporting & Evidence Collection

* [ ] Log all commands, timestamps, and outputs throughout the engagement
* [ ] Capture screenshots of all significant findings and exploitation steps
* [ ] Collect HTTP request/response pairs for web-based findings
* [ ] Document full attack chains with clear step-by-step reproduction
* [ ] Assess and document business impact for each finding
* [ ] Assign CVSS scores (CVSSv4 preferred)
* [ ] Provide actionable, prioritised remediation recommendations
* [ ] Include an executive summary covering key risks and overall posture
