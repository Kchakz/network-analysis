# network-analysis


# 🔐 Network Security Correlation Engine

## Overview

This project is a **network security correlation engine** designed to ingest parsed network events, apply multiple independent detectors, and correlate their outputs into higher-confidence **security incidents**.

Rather than relying on a single alert type, the engine combines signals such as authentication abuse, suspicious port activity, and beaconing behavior to identify coordinated or sustained malicious activity across time.

The project was built as a **portfolio-grade system**, emphasizing:

* clear detection logic
* explainable correlation rules
* extensibility
* realistic incident modeling

---

## Key Goals

* Detect suspicious activity using **multiple independent detectors**
* Correlate alerts across **time windows and shared network context**
* Reduce noise by promoting only **multi-signal incidents**
* Produce structured incidents suitable for SOC-style analysis

---

## High-Level Architecture

```
Raw Network Logs
        ↓
     Parser
        ↓
  Normalized Events
        ↓
+-------------------+
|   Detectors       |
|-------------------|
| SSH Detector      |
| Port-Time Detector|
| Beacon Detector   |
+-------------------+
        ↓
     Alerts
        ↓
Correlation Engine
        ↓
   Incidents
```

---

## Event Model

All detectors operate on a shared, normalized event structure, typically including:

* `timestamp`
* `src_ip`
* `dst_ip`
* `src_port`
* `dst_port`
* protocol / metadata

This allows detectors to remain independent while still producing alerts that can be meaningfully correlated.

---

## Detectors

Each detector focuses on a **single behavioral signal**:

### SSH Detector

Identifies suspicious SSH activity such as:

* repeated authentication attempts
* abnormal connection timing patterns

### Port-Time Detector

Flags unexpected port usage occurring:

* outside expected time windows
* on ports considered low-risk under normal conditions

### Beacon Detector

Detects potential command-and-control behavior by identifying:

* regular, low-variance communication intervals
* consistent destination patterns

Each detector outputs **alerts**, not incidents.

---

## Alert Structure

Detector alerts share a common schema, including:

* `src_ip`
* `dst_ip`
* `start_time`
* `end_time`
* `detector`
* `confidence`

This consistency enables downstream correlation without detector-specific logic.

---

## Correlation Engine (Design)

The correlation engine is responsible for turning multiple related alerts into a **single incident**.

### Core Correlation Principles

1. **Contextual Grouping**

   * Alerts are grouped by `(src_ip, dst_ip)`
   * This ensures only logically related activity is considered together

2. **Time-Based Sliding Window**

   * Alerts are sorted chronologically
   * A sliding window (e.g. 10 minutes) is applied to find overlapping or near-overlapping alerts

3. **Incident Creation**

   * An incident is created when multiple alerts from different detectors occur within the window
   * Incident start and end times are dynamically expanded as new alerts are added

4. **Window Advancement**

   * If a new alert falls outside the current incident window (plus tolerance), the incident is closed
   * Correlation then resumes from the next alert

---

## Incident Model

An incident aggregates multiple alerts and includes:

* `src_ip`
* `dst_ip`
* `start_time`
* `end_time`
* `detectors_triggered`
* `confidence`
* `severity`

This mirrors how real-world SOC tools summarize activity rather than flooding analysts with raw alerts.

---

## Confidence & Severity

* **Confidence** increases with:

  * number of detectors involved
  * detector-specific confidence values
* **Severity** can be derived from:

  * detector combinations
  * duration of activity
  * persistence over time

These values are intentionally transparent and tunable.

---

## Design Philosophy

* **Explainability over black-box ML**
* **Composable detectors**
* **Explicit correlation logic**
* **Security-analyst-friendly output**

The engine is designed to be extended with additional detectors without rewriting correlation logic.

---

## AI Usage Statement

This project was developed with **AI-assisted support** used in a professional and intentional manner.

AI was used to:

* discuss architectural approaches to detection and correlation
* validate assumptions about time-based correlation strategies
* refine detector heuristics and edge-case handling
* review and improve clarity of documentation

All final design decisions, implementation details, and system integration were performed and validated by the author.
AI assistance functioned as a **design reviewer and reasoning partner**, similar to peer review or rubber-duck debugging.

This mirrors modern software development practices where AI tools are used to enhance productivity, reasoning quality, and documentation clarity.

---

## Future Improvements

* Additional detectors (DNS tunneling, HTTP anomalies)
* Persistence-aware scoring across multiple incidents
* Visualization of incidents over time
* Export to SIEM-compatible formats

---

## Why This Project Matters

This project demonstrates:

* applied security thinking
* event-driven system design
* correlation logic beyond simple rule-matching
* realistic incident modeling

It reflects how detection engineering and SOC tooling work in real environments, not just academic examples.

---

## Correlation Flow (ASCII Diagrams)

### 1. Alert Generation Flow

This shows how raw data becomes detector alerts before correlation even begins.

```
Raw Logs
   |
   v
+------------------+
|  Log Parser      |
|------------------|
| Normalize fields |
| Extract timing   |
| Extract IPs      |
+------------------+
   |
   v
Normalized Events
   |
   v
+----------------------------+
|        Detectors           |
|----------------------------|
| SSH Detector               |
| Port-Time Detector         |
| Beacon Detector            |
+----------------------------+
   |
   v
Detector Alerts
```

Each detector operates **independently** and emits alerts without awareness of other detectors.

---

### 2. Correlation Engine – Grouping Phase

Alerts are grouped by shared network context to avoid unrelated correlations.

```
All Alerts
   |
   v
Group by (src_ip, dst_ip)
   |
   +----------------------+
   |                      |
   v                      v
Group A               Group B
(10.0.0.5 → 8.8.8.8)  (10.0.0.7 → 1.1.1.1)
```

Only alerts within the **same group** are ever considered for correlation.

---

### 3. Correlation Engine – Sliding Window

Within each group, alerts are sorted and evaluated over time.

```
Time →
[E1]----[E2]--------[E3]----[E4]

<------ 600s window ------>

Window contents:
[E1, E2, E3]
```

Rules:

* Alerts must fall within the time window
* Multiple detectors must be represented
* Single-alert windows are ignored

---

### 4. Incident Creation Logic

An incident begins when correlation conditions are met.

```
E1 (SSH)        E2 (Port-Time)
|---------------|
        |
        v
   Create Incident
   st = E1.start
   et = E2.end
   detectors = {SSH, Port-Time}
```

The incident’s time range is **derived from alert timing**, not fixed.

---

### 5. Incident Extension (Sliding Window Amendment)

As new alerts arrive, the incident may be extended.

```
Current Incident:
[--------------------]

New Alert (E3)
              [----]

If:
E3.start <= incident.end + tolerance

Then:
incident.end = E3.end
incident.detectors += {Beacon}
```

This allows sustained or evolving behavior to remain part of the same incident.

---

### 6. Incident Closure & Window Advancement

If no alerts fall within the tolerance window:

```
Incident End
[--------------------]

Next Alert
                      [----]

Result:
✔ Close incident
→ Start new correlation window
```

This prevents unrelated activity from being merged incorrectly.

---

## Correlation Summary Diagram

```
Alerts
  |
  v
Group by src/dst
  |
  v
Sort by time
  |
  v
Sliding Window
  |
  +--> Single detector → ignore
  |
  +--> Multiple detectors
          |
          v
      Create / Extend Incident
          |
          v
      Close when window breaks
```

---

## Why This Design Works

* Prevents alert flooding
* Preserves temporal causality
* Encourages multi-signal confidence
* Matches real SOC correlation logic

## Correlation Engine – Design Evolution

### Initial Approach
The first version of the correlation engine was designed to correlate alerts from two detectors: SSH brute-force activity and suspicious port usage. Correlation was performed using explicit pairwise logic, matching alerts with the same source and destination IPs and overlapping time windows.

This approach was intentionally simple and effective for a two-detector system, allowing clear reasoning about confidence scoring and alert independence.

### Limitation Discovered
After introducing a third detector (beaconing activity), the original design no longer scaled cleanly. Pairwise correlation logic would have required detector-specific nesting and special cases, making the system harder to extend and maintain.

This exposed a core limitation:
- Correlation logic was detector-aware instead of alert-aware.

### Revised Approach
The correlation engine was redesigned to operate on generic alert objects rather than detector pairs. Alerts are now:
1. Aggregated from all detectors
2. Grouped by source and destination IP
3. Sorted chronologically
4. Evaluated using a sliding time window

If multiple alerts from different detectors appear within the same window, they are merged into a single incident.

This design allows:
- Arbitrary numbers of detectors
- Incremental incident growth
- Detector-agnostic correlation logic

## Redesign Process

### Early Correlation Concept
![Initial correlation sketch](docs/images/Correlation_Logic_V1.png)

This sketch represents my initial approach to alert grouping and time-based correlation.

### Revised Correlation Flow
![Amended correlation sketch](docs/images/Correlation_Logic_V2.png)
![Ammendment](docs/images/Correlation_Logic_V2a.png)

After implementing detectors, I refined the design to support sliding windows and incident extension.
