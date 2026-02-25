# HTB - Bashed (OSCP Preparation Write-up)

Platform: Hack The Box
Machine: Bashed
Difficulty: Easy
Goal: User + Root
Flags: Not included for security reasons.

--------------------------------------------------

1) Recon

Nmap Scan:
nmap -sC -sV -oN nmap.txt <TARGET_IP>

Findings:
- 80/tcp open http
- Apache 2.4.18 (Ubuntu)
- Web Title: Arrexel's Development Site

Since only HTTP was exposed, the focus shifted to web enumeration.

--------------------------------------------------

2) Web Enumeration

Directory brute forcing:
gobuster dir -u http://<TARGET_IP>/ -w /usr/share/seclists/Discovery/Web-Content/common.txt -x php,txt,html

Discovered:
- /dev directory

Inside /dev, a web-based command execution interface (phpbash) was identified.

--------------------------------------------------

3) Initial Access (www-data)

After confirming command execution via the web interface, a reverse shell was attempted.

Netcat attempts failed.
A Python-based reverse shell successfully established a connection.

Listener:
nc -lvnp <LPORT>

After receiving the shell:
whoami
id

Result:
- User: www-data

--------------------------------------------------

4) Privilege Escalation (www-data → scriptmanager)

Sudo privileges were checked:
sudo -l

Output:
User www-data may run the following commands on bashed:
    (scriptmanager : scriptmanager) NOPASSWD: ALL

This means www-data can execute commands as scriptmanager without a password.

User pivot:
sudo -u scriptmanager bash
id

Result:
- User: scriptmanager

--------------------------------------------------

5) Privilege Escalation (scriptmanager → root)

Files owned by scriptmanager were enumerated:
find / -user scriptmanager 2>/dev/null

Discovered:
- /scripts
- /scripts/test.py

Permissions check:
ls -la /scripts

The test.py file was writable by scriptmanager.

In the same directory, test.txt was owned by root.

stat /scripts/test.py
stat /scripts/test.txt

Timestamp analysis confirmed that test.py was being executed automatically by root.

This represents a classic privilege escalation scenario:

Writable script + higher privilege execution = privilege escalation

--------------------------------------------------

6) Root Access

By abusing the writable script executed by root, full root access was obtained.

Verification:
id

Output:
uid=0(root)

Root flag:
cat /root/root.txt

User flag:
cat /home/<user>/user.txt

--------------------------------------------------

Methodology Notes (OSCP Preparation)

- Web enumeration is critical.
- Always carefully analyze sudo -l output.
- Writable scripts executed by cron are common privilege escalation vectors.
- Timestamp analysis can confirm execution context.
- Kernel exploits should be the last resort.
- Think in privilege chains:
  Low User → Mid User → Root

--------------------------------------------------

Key Takeaway

Misconfigured sudo permissions and writable scripts executed by higher-privileged users can lead to full system compromise.

Successful exploitation depends on:
- Proper enumeration
- Understanding privilege boundaries
- Verifying execution context
