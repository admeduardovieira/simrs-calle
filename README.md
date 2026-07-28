# 🏥 SIMRS × CALL-E

**Automated Patient Appointment Reminders for Indonesian Hospitals**

> 🏆 Hackathon submission for [CALL-E: Your Code Is Calling](https://call-e.devpost.com/) — $10,000 prize pool

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![PR #37](https://img.shields.io/badge/PR_CALE-E-37-brightgreen)](https://github.com/CALLE-AI/awesome-phone-call-agents/pull/37)
[![Python 3.10+](https://img.shields.io/badge/Python-3.10+-yellow.svg)](https://python.org)

---

## What Is This?

A healthcare-focused AI voice agent that **automatically calls hospital patients** to remind them about upcoming appointments. The patient can confirm, reschedule, or cancel — all through natural conversation in Bahasa Indonesia.

The system writes outcomes back to the hospital's **SIMRS** (Hospital Management System) and optionally generates **SatuSehat FHIR** resources for Indonesia's national health data exchange.

## 🎯 The Problem

Indonesian hospitals lose **15-30% of outpatient appointments** to no-shows. Manual reminder calls by admin staff are:
- ⏰ Time-consuming (200+ calls/day per clinic)
- 💰 Expensive (staff cost)
- ❌ Error-prone (missed calls, wrong numbers)
- 📋 Not tracked (no writeback to EHR)

## 💡 Our Solution

```
┌────────────────┐     ┌────────────────┐     ┌──────────────┐
│   SIMRS / EHR  │────▶│  SIMRS × CALL-E│────▶│  CALL-E AI   │
│   (appointments│◀────│  Batch Runner  │◀────│  Voice Call   │
│    database)   │     │  (Python)      │     │  (Indonesian) │
└────────────────┘     └────────────────┘     └──────────────┘
         │
         ▼
┌────────────────┐
│  SatuSehat     │
│  FHIR Server   │
│  (Kemenkes)    │
└────────────────┘
```

**Flow:**
1. Read tomorrow's appointments from SIMRS
2. For each patient, CALL-E places a natural voice call
3. Patient confirms / reschedules / cancels in real-time
4. Outcome is classified and written back to SIMRS
5. FHIR Encounter resource generated for SatuSehat

## 🚀 Quick Start

### Prerequisites
- Python 3.10+
- [CALL-E CLI](https://github.com/CALLE-AI/call-e-integrations) (`calle`)
- CALL-E account (free 20 calls at [heycall-e.com](https://heycall-e.com))

### Install

```bash
git clone https://github.com/sigit4715/simrs-calle.git
cd simrs-calle
```

### Run (Dry Run — No Calls)

```bash
python3 scripts/batch-reminder.py \
  --appointments scripts/example_appointments.json \
  --dry-run
```

### Run (Live — Uses CALL-E Credits)

```bash
calle auth login
python3 scripts/batch-reminder.py \
  --appointments scripts/example_appointments.json
```

### Run (With SIMRS Writeback)

```bash
python3 scripts/batch-reminder.py \
  --appointments scripts/example_appointments.json \
  --simrs-url http://simrs.local/api
```

## 📁 Project Structure

```
simrs-calle/
├── skill/                          # CALL-E Agent Skill
│   ├── SKILL.md                    # Skill specification
│   └── references/
│       ├── safety.md               # Patient consent, privacy, emergency rules
│       ├── examples.md             # 5 usage examples (CLI, Python, MCP, FHIR)
│       ├── satusehat-fhir.md       # FHIR resource schemas
│       ├── client-adapters.md      # Multi-client adapter guide
│       ├── calle-cli-bootstrap.md  # CLI detection and auth flow
│       └── runtime-prompt.md       # Call goal templates
├── scripts/
│   ├── batch-reminder.py           # Batch runner (zero dependencies)
│   └── example_appointments.json   # Sample appointment data
├── docs/
│   ├── architecture.md             # System architecture
│   ├── satusehat-setup.md          # SatuSehat integration guide
│   └── demo-video-script.md        # Demo video outline
├── LICENSE                         # MIT License
└── README.md                       # This file
```

## 🛡️ Safety Features

| Feature | Description |
|---|---|
| **Patient Consent** | Only calls patients who opted in to reminders |
| **No Medical Advice** | Never discusses diagnoses, medications, or treatment |
| **Emergency Escalation** | Detects emergencies, directs to 112 / IGD |
| **Privacy Masking** | Phone numbers masked in all logs and reports |
| **Calling Hours** | Only calls 08:00–20:00 patient local time |
| **Retry Policy** | Max 2 retries per appointment, 2h apart |
| **Data Retention** | Transcripts purged after 30 days |

## 🏥 SatuSehat FHIR Integration

Generates HL7 FHIR R4 resources compatible with Indonesia's SatuSehat platform:

- **Appointment** — tracks status (booked → arrived / cancelled)
- **Encounter** — links patient, practitioner, and appointment
- **Communication** — logs the reminder attempt for audit

See [docs/satusehat-setup.md](docs/satusehat-setup.md) for full integration guide.

## 🎬 Demo

### Video Script (3 minutes)

| Time | Scene |
|---|---|
| 0:00-0:20 | **Problem** — Hospital admin making manual reminder calls |
| 0:20-0:40 | **Solution** — Run `batch-reminder.py` with appointments file |
| 0:40-1:30 | **Live Call** — CALL-E calls patient, confirms appointment |
| 1:30-2:00 | **Outcome** — Parse result, show CONFIRMED status |
| 2:00-2:30 | **Writeback** — SIMRS updated, FHIR Encounter created |
| 2:30-3:00 | **Impact** — 15-30% no-show reduction estimate |

## 📊 Expected Impact

| Metric | Current | With SIMRS × CALL-E |
|---|---|---|
| No-show rate | 15-30% | 5-10% |
| Staff time on calls | 4 hrs/day | 0 hrs/day |
| Call success rate | 60% | 85%+ |
| Data tracking | Manual | Automated |

## 🏆 Hackathon Submission

- **Competition:** [CALL-E: Your Code Is Calling](https://call-e.devpost.com/)
- **PR to community repo:** [#37](https://github.com/CALLE-AI/awesome-phone-call-agents/pull/37)
- **Deadline:** September 14, 2026

## 📝 License

MIT License — see [LICENSE](LICENSE)

## 👤 Author

**Sigit Putra Prabowo** — [GitHub](https://github.com/sigit4715)

Built for Indonesian healthcare 🇮🇩
