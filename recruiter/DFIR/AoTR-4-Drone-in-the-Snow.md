# 🛩️ AoTR 4 – A Drone in the Snow — DFIR Case Study

**Platform:** Hack The Box (Sherlock – AoTR Series)  
**Category:** Drone / Flight Forensics  
**Difficulty:** Easy  
**Rank Achieved:** #95  
**HTB Verification:** https://labs.hackthebox.com/achievement/sherlock/847519/1099  

---

## 🧩 Incident Summary

During a winter blackout, a criminal group used a drone for surveillance
activities. The drone later crashed in the snow.

I was tasked with reconstructing the **entire drone operation**, including:
- mission planning behavior
- actual flight execution
- crash circumstances
- takeoff location of the operator

This case focused on **UAV telemetry forensics**, not traditional DFIR logs.

---

## 📦 Evidence Overview

The provided archive (`AOTR4.zip`) contained two key artifacts:

- **Mission Plan (`.plan`)**
  - JSON-based file used by QGroundControl
  - Defines planned waypoints, holds, and landing

- **Flight Log (`log.bin`)**
  - Binary MAVLink telemetry
  - Captures real flight behavior, GPS, altitude, speed, and crash event

---

## 🧭 Investigation Approach

### 1️⃣ Mission Plan Analysis
I analyzed the `.plan` file to understand **intended drone behavior**.

Findings:
- Total mission items: **49**
- Spline waypoints used for smooth flight
- A deliberate **loiter/hold** mid-mission
- Planned route passed key Budapest landmarks, including a Danube crossing

This indicated **controlled surveillance**, not random flight.

---

### 2️⃣ Flight Log Forensics
Using MAVLink tooling, I parsed the binary `log.bin` file.

Key findings:
- Total flight time: **~10 minutes**
- Maximum altitude: **377 meters**
- Highest ground speed: **10.24 m/s**
- Crash event clearly logged by the flight controller

The logs confirmed that the drone **did not follow the full planned mission**.

---

### 3️⃣ Crash & Impact Analysis
By correlating GPS and event logs:
- Exact crash timestamp was identified
- Impact speed confirmed a **hard ground collision**
- Crash coordinates were extracted and mapped

This ruled out signal loss or soft landing.

---

### 4️⃣ Operator Takeoff Location
Early GPS entries revealed the **takeoff coordinates**.

This location represents a strong candidate for:
- operator presence
- police raid or suspect identification

---

## 🧠 Key Takeaways

This case reinforced several important DFIR concepts:

- Forensics applies beyond servers and PCs
- Binary telemetry can fully reconstruct attacker behavior
- Planned intent vs real execution often differs
- Timeline reconstruction is critical, regardless of data source

---

## ✅ Outcome

I successfully:
- Reconstructed the full drone flight timeline
- Identified planned vs actual behavior
- Determined crash cause and location
- Extracted the operator’s takeoff point

This investigation expanded my DFIR skill set into **UAV and flight telemetry forensics**.

---

📌 *This document is a recruiter-focused case summary.*

---

## 📚 Full Technical Write-up (Public)

The complete step-by-step investigation includes:
- all terminal commands used
- MAVLink parsing techniques
- coordinate extraction methods
- full question-by-question analysis

🔗 **Full Technical Write-up:**  
https://github.com/shanithakuralg/Write-ups/blob/main/Hackthebox/Sherlock/Aotr4.md
