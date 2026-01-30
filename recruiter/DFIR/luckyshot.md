# 🕵️ LuckyShot — DFIR Case Study

**Platform:** Hack The Box (Sherlock)  
**Category:** Digital Forensics & Incident Response  
**Difficulty:** Easy  
**Rank Achieved:** #169  

---

## 🧩 Incident Summary

An IT Manager reported missing and altered files on his Linux system.  
Initial signs pointed toward unauthorized access and long-term persistence.

I was tasked with identifying **how the attacker entered**, **what they did**, and **how they maintained access**.

---

## 🚪 Initial Access — What Happened?

The first challenge was identifying *how* access was gained.

While reviewing `auth.log`, I noticed:
- Hundreds of failed SSH login attempts
- Followed by a successful login

**Conclusion:**  
The system was compromised via an **SSH brute-force attack** targeting the `administrator` account.

This immediately set the investigation timeline.

---

## 🕒 Attacker Activity — Rebuilding the Timeline

After access, the attacker:
- Verified privileges using `groups administrator`
- Downloaded **LaZagne**, a known credential-dumping tool
- Explored the system via shell commands

All of this was confirmed through careful analysis of `.bash_history`.

This stage was straightforward, but it confirmed **post-exploitation intent**.

---

## 📤 Data Exfiltration — A Clear Red Flag

While reviewing command history, I noticed repeated use of `scp`.

Further analysis showed:
- Sensitive files being transferred
- Destination IP: `192.168.161.198`

This confirmed **active data exfiltration** to an attacker-controlled machine.

---

## 🦠 Malware & Persistence — The Hardest Part

This was the most time-consuming phase.

I discovered:
- A malicious script named `sys_monitor.sh`
- A fake system service: `systemd-networkm.service`
- A malicious cron job that fetched and executed a remote payload automatically

The attacker used **legitimate-looking names** to hide persistence — a classic real-world technique.

Finding the **lowest-port listener** required checking:
- cron jobs
- systemd services
- startup scripts

Eventually, the root cause was traced to `.bashrc`.

This step alone took several hours and multiple failed assumptions.

---

## 👤 Long-Term Access — User Persistence

The attacker didn’t rely only on malware.

By re-checking `auth.log`, I found:
- A new user account named `Regev`
- Created shortly after initial compromise

This confirmed **defense evasion and long-term persistence planning**.

---

## 🧠 Key Takeaways

This investigation reinforced critical DFIR principles:

- Logs never lie — but they must be read patiently
- Attackers often reuse simple techniques with deceptive naming
- Persistence is layered, not singular
- Good DFIR work means correlating *multiple weak signals*

---

## ✅ Outcome

I successfully identified:
- Initial access vector
- Full attacker timeline
- Malware execution path
- Persistence mechanisms
- Data exfiltration method

This case strengthened my confidence in **independent DFIR investigations without relying on public write-ups**.

---

📌 *Full technical investigation with commands and hashes is available in the public GitHub write-up.*

---

## 📚 Full Technical Write-up (Public)

This document is a **high-level DFIR case summary** intended for recruiters.

If you're interested in the **complete step-by-step investigation**, including:
- exact commands used
- log analysis screenshots
- failed assumptions & corrections
- full attacker timeline reconstruction

you can read the full public write-up here:

🔗 **Public Technical Write-up:**  
[Hack The Box – Sherlock LuckyShot (Full DFIR Analysis)](../../Hackthebox/Sherlock/luckyshot.md)
