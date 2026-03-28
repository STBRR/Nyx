# Active Directory Penetration Testing Methodology

A comprehensive checklist for Active Directory assessments. AD environments are complex and interconnected — thorough enumeration before exploitation will always reveal more paths than rushing blindly.

> 💡 **Mentor Note:** AD testing is all about chaining small misconfigurations into a full domain compromise. Rarely will you find a single magic vulnerability — instead you'll find a series of stepping stones. Always ask yourself: *"What can I do with what I have right now?"*

---

## Pre-Engagement

* [ ] Confirm scope — which domains, forests, and trusts are in scope
* [ ] Confirm whether credentials are provided (grey box) or not (black box from network access)
* [ ] Confirm whether the assessment includes physical/phishing or is purely network-based
* [ ] Identify domain name, DC hostnames, and DNS server from DHCP or network traffic
* [ ] Set DNS to point at the Domain Controller for resolution

---

## Unauthenticated Enumeration (No Credentials)

* [ ] Identify domain name via broadcast traffic, NBNS, or LDAP
* [ ] Run Responder in analyse mode to passively observe traffic (no poisoning yet)
* [ ] Check for null session access on SMB (smbclient -N, crackmapexec)
* [ ] Attempt anonymous LDAP bind and enumerate base DN
* [ ] Enumerate RPC endpoints anonymously (rpcclient -U "" -N)
* [ ] Check for open shares accessible without credentials
* [ ] Run enum4linux-ng for anonymous enumeration
* [ ] Check for AS-REP Roastable accounts with no pre-auth required (no creds needed)
* [ ] Run mitm6 to capture IPv6 DNS requests and relay to LDAP/SMB
* [ ] Run Responder to capture NTLMv2 hashes (LLMNR/NBT-NS/mDNS poisoning)
* [ ] Check if SMB signing is enforced across the domain (crackmapexec --gen-relay-list)
* [ ] Attempt NTLM relay with ntlmrelayx to hosts without SMB signing
* [ ] Attempt NTLM relay to LDAP for creating new machine accounts or reading hashes
* [ ] Check for Pre2k computer accounts (computers with default passwords — machine$)
* [ ] Identify web interfaces (OWA, ADFS, Azure AD Connect, Intranet) for password spraying

---

## Password Attacks

* [ ] Enumerate valid usernames before spraying (LDAP, Kerbrute userenum, OSINT)
* [ ] Identify password policy before spraying (lockout threshold, observation window)
* [ ] Perform password spraying with organisation-specific patterns (Kerbrute, crackmapexec)
* [ ] Common spray candidates: Season+Year!, Company+Year!, Welcome1!, Password1!
* [ ] Spray against OWA, ADFS, VPN portals if in scope
* [ ] Test for credential reuse across services once first credential is captured
* [ ] Brute force individual accounts only if no lockout policy is confirmed

---

## Authenticated Enumeration (With Credentials)

> 💡 **Mentor Note:** The moment you have any domain credential — even a low-privileged user — the entire attack surface of the domain opens up. LDAP allows any authenticated user to enumerate almost everything by default. Always start here before attempting any exploitation.

* [ ] Confirm credential validity across services (crackmapexec smb, ldap, winrm)
* [ ] Enumerate domain users, groups, computers, and OUs (bloodhound, ldapsearch, windapsearch)
* [ ] Collect BloodHound data (SharpHound or bloodhound-python)
* [ ] Review BloodHound for shortest path to Domain Admin
* [ ] Review BloodHound for AS-REP Roastable, Kerberoastable, and DCSync rights
* [ ] Enumerate user description fields for stored passwords (ldapsearch, RSAT)
* [ ] Enumerate password policy (min length, lockout, complexity)
* [ ] Enumerate fine-grained password policies (PSOs) and who they apply to
* [ ] Enumerate domain trusts (Get-DomainTrust, nltest /domain_trusts)
* [ ] Enumerate Group Policy Objects (GPOs) — check for password storage or interesting configs
* [ ] Enumerate LAPS deployment — which OUs have LAPS, can current user read LAPS passwords?
* [ ] Enumerate AD Certificate Services (ADCS) — is a CA present? (certutil, certipy)
* [ ] Identify AdminSDHolder protected accounts
* [ ] Enumerate privileged groups: Domain Admins, Enterprise Admins, Schema Admins, Backup Operators, Account Operators, Server Operators, Print Operators, DNSAdmins
* [ ] Check for nested group memberships that grant unexpected privilege
* [ ] Identify service accounts (SPN set) for Kerberoasting
* [ ] Identify accounts with no pre-auth required for AS-REP Roasting

---

## Kerberos Attacks

### AS-REP Roasting
* [ ] Identify accounts with "Do not require Kerberos pre-authentication" set
* [ ] Request AS-REP hashes for identified accounts (GetNPUsers.py, Rubeus)
* [ ] Crack captured hashes offline (hashcat mode 18200)

> 💡 **Mentor Note:** AS-REP Roasting works because when pre-authentication is disabled, you can ask the DC for an encrypted ticket for any user without proving who you are first. The DC hands back a blob encrypted with the user's password hash — you then crack it offline. This is why pre-auth should always be enabled.

### Kerberoasting
* [ ] Identify all accounts with SPNs set (GetUserSPNs.py, Rubeus kerberoast)
* [ ] Request TGS tickets for identified service accounts
* [ ] Crack captured hashes offline (hashcat mode 13100)
* [ ] Prioritise high-value targets: accounts in privileged groups, accounts with AdminCount=1
* [ ] Attempt targeted Kerberoasting if you have write access to an account's SPN attribute

> 💡 **Mentor Note:** Kerberoasting abuses the fact that any domain user can request a service ticket for any SPN, and that ticket is encrypted with the service account's password hash. Service accounts often have weak, non-rotating passwords — making them prime cracking targets. Always check what groups the service account is in before cracking, to prioritise effort.

### Pass-the-Ticket
* [ ] Export Kerberos tickets from memory (Rubeus dump, Mimikatz sekurlsa::tickets)
* [ ] Import and use captured tickets (Rubeus ptt, Mimikatz kerberos::ptt)
* [ ] Attempt overpass-the-hash — convert NTLM hash to a Kerberos TGT (Rubeus asktgt)

### Golden / Silver Tickets
* [ ] If KRBTGT hash is captured — forge a Golden Ticket (Mimikatz, ticketer.py)
* [ ] If service account hash is captured — forge a Silver Ticket for that service
* [ ] Test Golden Ticket persistence after password change (valid for ticket lifetime)

---

## ACL / ACE Abuse

> 💡 **Mentor Note:** ACL abuse is one of the most common and overlooked escalation paths in AD. BloodHound will show you these edges visually — look for: GenericAll, GenericWrite, WriteOwner, WriteDACL, ForceChangePassword, AddMember, and AllExtendedRights. Each of these gives you a different kind of leverage over the target object.

* [ ] Review BloodHound for abusable ACEs on users, groups, computers, and GPOs
* [ ] Check for GenericAll on a user — reset their password or add SPN for Kerberoasting
* [ ] Check for GenericWrite on a user — add SPN (targeted Kerberoast) or modify logon script
* [ ] Check for ForceChangePassword on a user — reset password without knowing current one
* [ ] Check for GenericAll/GenericWrite on a group — add yourself as a member
* [ ] Check for GenericAll on a computer — perform Resource-Based Constrained Delegation (RBCD)
* [ ] Check for WriteOwner — take ownership of the object, then grant yourself full control
* [ ] Check for WriteDACL — write a new ACE granting yourself GenericAll
* [ ] Check for DCSync rights (GetChanges + GetChangesAll on the domain object) — dump all hashes
* [ ] Check for AllExtendedRights on a user — can force password change or read LAPS

---

## Delegation Attacks

> 💡 **Mentor Note:** Delegation is a mechanism that allows a service to impersonate a user to access other services on their behalf. There are three types — Unconstrained, Constrained, and Resource-Based Constrained Delegation (RBCD). Each has different attack paths. Understanding *why* they exist (to allow multi-tier applications to act on a user's behalf) helps understand why they're dangerous when misconfigured.

### Unconstrained Delegation
* [ ] Identify computers and users with Unconstrained Delegation set (BloodHound, ADSearch)
* [ ] Compromise the system with Unconstrained Delegation
* [ ] Use the Printer Bug (SpoolSample) or PetitPotam to coerce a DC to authenticate
* [ ] Capture the DC TGT from memory and use for DCSync or lateral movement
* [ ] Alternatively — wait for privileged users to authenticate naturally

### Constrained Delegation
* [ ] Identify accounts with Constrained Delegation set and note the allowed services
* [ ] If you control the account — request a TGS for the allowed service (s4u2proxy)
* [ ] If protocol transition is enabled (TrustedToAuthForDelegation) — use s4u2self first to impersonate any user, then s4u2proxy
* [ ] Use Rubeus s4u or getST.py for ticket requests

### Resource-Based Constrained Delegation (RBCD)
* [ ] Identify if you have GenericAll/GenericWrite/WriteProperty on a computer object
* [ ] Create a new machine account (MachineAccountQuota > 0) or use an existing controlled computer
* [ ] Write the controlled computer's SID to the target computer's msDS-AllowedToActOnBehalfOfOtherIdentity attribute
* [ ] Use s4u2self + s4u2proxy to impersonate any user (e.g. Administrator) to the target
* [ ] Use the resulting ticket to access the target as Administrator

---

## Active Directory Certificate Services (ADCS)

> 💡 **Mentor Note:** ADCS is a goldmine for privilege escalation and persistence. Many organisations deploy it without fully understanding the attack surface. Certipy is the go-to tool here — run it as soon as you confirm ADCS is present. ESC1 is the most commonly found and exploitable misconfiguration.

* [ ] Identify if ADCS is deployed (certutil, certipy find, BloodHound)
* [ ] Enumerate certificate templates and their permissions (certipy find --vulnerable)
* [ ] Check for ESC1 — template allows requestor to supply arbitrary Subject Alternative Name (SAN) + low-priv user can enrol
* [ ] Check for ESC2 — Any Purpose EKU or no EKU, can be used as a smart card cert
* [ ] Check for ESC3 — Certificate Request Agent template abuse
* [ ] Check for ESC4 — low-priv user has write access over a template (modify it to introduce ESC1)
* [ ] Check for ESC6 — CA has EDITF_ATTRIBUTESUBJECTALTNAME2 flag set (arbitrary SAN on any template)
* [ ] Check for ESC7 — low-priv user has ManageCA or ManageCertificates rights
* [ ] Check for ESC8 — NTLM relay to AD CS HTTP enrolment endpoint (no signing required)
* [ ] Exploit ESC1 — request cert with DA UPN as SAN, use cert to get DA TGT (PKINIT)
* [ ] Use captured certificate to request a TGT (certipy auth)
* [ ] Use certificate for persistence — valid for template lifetime regardless of password changes

---

## Lateral Movement

* [ ] Validate credentials across all domain-joined hosts (crackmapexec smb)
* [ ] Identify hosts where current user is local admin (crackmapexec --local-auth, BloodHound)
* [ ] Use wmiexec, smbexec, psexec, or evil-winrm for remote execution
* [ ] Perform pass-the-hash with NTLM hashes (crackmapexec, impacket)
* [ ] Perform pass-the-ticket with Kerberos tickets (Rubeus, impacket)
* [ ] Dump LSASS on compromised hosts for additional credentials (procdump, comsvcs.dll)
* [ ] Use Mimikatz sekurlsa::logonpasswords on hosts where AV permits
* [ ] Check for cached credentials on servers (Domain Admin logged in recently?)
* [ ] Identify servers with interesting roles — CA, Exchange, file servers, backup servers
* [ ] Enumerate admin shares and writable paths on lateral targets

---

## Domain Privilege Escalation

* [ ] Attempt DCSync if account has replication rights (secretsdump.py, Mimikatz lsadump::dcsync)
* [ ] Dump NTDS.dit if DC is compromised (secretsdump.py, vssadmin shadow copy)
* [ ] Check DNSAdmins group membership — dll injection via DNS service restart
* [ ] Check for writable GPOs applied to privileged OUs or DCs
* [ ] Check for AdminSDHolder misconfiguration — write ACE that propagates to protected accounts
* [ ] Abuse Backup Operators privilege — access DC filesystem, dump SAM/SYSTEM/NTDS
* [ ] Check Account Operators group — can modify non-protected users and add to groups
* [ ] Check Server Operators group — can manage DC services, potential code execution
* [ ] Check Print Operators group — can load kernel drivers on DCs (SeLoadDriverPrivilege)

---

## Domain Persistence

> 💡 **Mentor Note:** Persistence is important on real engagements but be cautious in assessments — always confirm with the client what persistence mechanisms are permitted and clean up anything you deploy.

* [ ] Create Golden Ticket using KRBTGT hash (valid for 10 years by default)
* [ ] Create Diamond Ticket (harder to detect than Golden Ticket)
* [ ] Add DCSync rights to a low-privilege user via ACL abuse
* [ ] Create a shadow credential (add KeyCredentialLink to a high-value account)
* [ ] Register a rogue certificate template for persistent certificate-based auth
* [ ] Create a new machine account with constrained delegation to the DC
* [ ] Add a backdoor admin account to a privileged group
* [ ] Modify AdminSDHolder DACL for persistent ACE propagation to protected groups

---

## Forest & Trust Attacks

* [ ] Enumerate all domain trusts (BloodHound, Get-DomainTrust)
* [ ] Identify trust direction (one-way, bidirectional) and transitivity
* [ ] Check for SID history abuse across trusts
* [ ] Attempt trust ticket forgery if parent-child trust exists (Extra SIDs attack)
* [ ] Enumerate foreign group memberships across trusts
* [ ] Check for shared accounts or credentials that work across trusted domains
* [ ] Identify Azure AD Connect sync accounts for potential credential abuse

---

## Key Tools Reference

| Tool | Purpose |
|------|---------|
| BloodHound + SharpHound/bloodhound-python | AD enumeration and attack path visualisation |
| crackmapexec / netexec | Multi-protocol credential testing and enumeration |
| impacket suite | Kerberos attacks, relay, secretsdump, lateral movement |
| Rubeus | Kerberos ticket manipulation, roasting, delegation abuse |
| Certipy | ADCS enumeration and exploitation |
| Responder | LLMNR/NBT-NS/mDNS poisoning and hash capture |
| mitm6 | IPv6 DNS spoofing and NTLM relay |
| Mimikatz | Credential dumping, ticket manipulation, Golden Ticket |
| Kerbrute | Username enumeration and password spraying via Kerberos |
| ldapsearch / windapsearch | LDAP enumeration |
| enum4linux-ng | SMB/RPC enumeration |
| chisel / ligolo-ng | Pivoting and tunnelling |
| PowerView / SharpView | AD enumeration from Windows |

---

## Common Attack Chains

> 💡 **Mentor Note:** These are the chains worth understanding end-to-end — knowing how the pieces connect is what separates a good AD tester from someone just running tools.

* **Unauthenticated → Initial Access:** Responder hash capture → crack offline → domain user
* **Unauthenticated → Initial Access:** mitm6 + ntlmrelayx → LDAP relay → create machine account → RBCD → local admin on target
* **Low-priv user → DA:** Kerberoast service account → crack → reuse credentials → lateral move → DA
* **Low-priv user → DA:** BloodHound path → ACL abuse chain → DCSync
* **Low-priv user → DA:** ADCS ESC1 → request cert with DA SAN → PKINIT TGT → DA
* **Unconstrained Delegation → DA:** Coerce DC auth (PetitPotam/PrinterBug) → capture DC TGT → DCSync
* **GenericAll on Computer → DA:** RBCD attack → impersonate Administrator on target → lateral to DA
