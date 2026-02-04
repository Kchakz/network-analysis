# network-analysis

---

## **📘 network-analysis**

# Network Security Analysis & Correlation Engine

This repository contains a **network security analysis system** that parses network logs, generates detector alerts, and correlates them into **structured security incidents**. The system demonstrates applied detection engineering, temporal reasoning, and multi-signal incident construction — all designed to feel closer to real SOC workflows than simple script logic.

The correlation logic itself — while a key part — is one component in a full pipeline of log ingestion, normalization, detection, and incident generation.

---

## 🚀 Project Summary

Many security systems generate hundreds or thousands of *alerts*, but struggle to combine related events into meaningful *incidents*. This project tackles that problem by:

* Parsing structured network logs
* Applying **multiple independent detectors**
* Correlating the resulting alerts based on time, source/destination, and detector diversity
* Producing **incident summaries** that aggregate evidence over time

This workflow is closer to how real Security Operations Centers (SOCs) reason about ongoing intrusion activity.

---

## 🧠 Architecture Overview

```
Raw Network Logs
        ↓
     Parser
        ↓
  Normalized Events
        ↓
+----------------------+
|      Detectors       |
|----------------------|
| SSH Detector         |
| Port-Time Detector   |
| Beaconing Detector   |
+----------------------+
        ↓
     Alerts
        ↓
Correlation Engine
        ↓
   Incident Objects
        ↓
 Output: alerts.json
```

---

## 📌 What’s Included

### 📂 Parser

A log parser reads structured logs, discards comments/invalid lines, and normalizes fields like:

* timestamp
* src_ip / dst_ip
* ports
* byte counts
* description

This allows detectors to operate on a *uniform event representation*.

---

## 🛡️ Detectors

Each detector focuses on a specific **behavioral signal**:

### 🔐 SSH Detector

Flags potential SSH brute force when there are many failed connection attempts within a short timeframe.

### ⏰ Port-Time Detector

Identifies unusual port usage occurring outside “normal business hours” or on ports not in the safe list.

### 📡 Beaconing Detector

Detects periodic communication indicative of beaconing (e.g., C2 traffic) by checking regular timing and uniform packet sizes.

Each detector creates its own alert objects with:

* src_ip, dst_ip
* confidence score
* time interval

---

## 🔗 Correlation Engine (Design & Behavior)

The core idea behind correlation is to build *incidents* — collections of related alerts — instead of reporting standalone alerts.

### Core Principles

1. **Group by Flow Context**
   Alerts are grouped by `(src_ip, dst_ip)`.

2. **Temporal Reasoning**
   Within each group, alerts are sorted chronologically and evaluated using adaptable time logic (not fixed windows).

3. **Incident Growth**
   A new incident is created when alerts from multiple detectors fall close in time.
   Incident intervals expand as more alerts arrive.

4. **Adaptive Closure**
   When alerts fall outside a tolerance threshold, the active incident ends and a new one begins.

This avoids rigid sliding windows and supports a richer incident timeline.

---

## 📈 Incident Model

Each generated incident includes:

* Source and destination IP
* Start and end time
* Detectors involved
* Confidence (aggregated from detector scores)
* Severity (based on signal combination and persistence)

---

## 🧠 Design Philosophy

* **Explainability over black box** – No ML models; all logic is transparent
* **Composable detectors** – Add new detectors without rewriting correlation logic
* **Temporal continuity** – Incidents are defined by evolving timing, not fixed buckets
* **SOC-friendly output** – Structured JSON suitable for analysts or SIEM ingestion

---

## 🤖 AI Usage Statement

AI assistance was used as a **design reviewer and reasoning partner**, not as an autopilot code generator. AI support helped with:

* Architectural exploration and alternatives
* Temporal correlation logic validation
* Edge case reasoning and detector design discussion
* Improving documentation clarity

All final design decisions, implementation logic, and structure were conceived and authored by the project maintainer.

This mirrors modern engineering workflows where AI tools serve as *peer reviewers and sounding boards*.

---

## 🧾 Design Process & Sketches

Below are key stages of the design process that guided implementation:

### 📝 Early Correlation Concept

(*Insert your photo here: initial plan*)

> This sketch shows my first attempt at grouping and correlating events.

### 🔄 Refined Correlation Model

(*Insert your amended sketches here*)

> Revised logic supporting dynamic incident extension and tolerance-based grouping.

### 📊 Correlation Flow (ASCII diagrams)

```
Raw Logs
   |
Parser
   |
Normalized Events
   |
Detectors
   |
Alerts
   |
Group by (src, dst)
   |
Sort by time
   |
Sliding window logic
   |
Create / Extend Incidents
```

---

## 🛠 Getting Started

### 🧾 Requirements

* Python 3.x
* `ports.txt`
* `logs.txt`

### 🟢 Run

```bash
python3 main.py
```

This will produce an `alerts.json` file with all correlated incidents.

---

## 📦 Output

The output format is structured JSON:

```json
[
  {
    "type": "Correlated Alert",
    "src_ip": "10.0.0.5",
    "dst_ip": "192.168.1.10",
    "detectors": ["SSH Brute Force","Suspicious port usage","Beaconing"],
    "confidence": 0.84,
    "start_time": "2025-02-01T18:02:00Z",
    "end_time": "2025-02-01T18:15:32Z"
  }
]
```

---

## 📌 Future Improvements

Ideas for improvement include:

* More detectors (DNS anomalies, HTTP anomalies)
* Real-time streaming support
* SIEM-compatible exports
* UIs or visual timelines

---

## ⭐ Why This Project Matters

This project demonstrates:

* Applied detection engineering
* Temporal correlation logic
* Multi-signal incident modeling
* Realistic SOC-style reasoning

It shows **not just a script**, but a **thinking process** behind how real detection systems evolve.

---

## 📁 Files & Structure

* `parse.py` – Log parsing
* `detectors.py` – Modular detectors
* `correlator.py` – Engine that builds incidents
* `main.py` – Entry point
* `alerts.json` – Output

---
