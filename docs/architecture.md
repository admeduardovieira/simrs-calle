# System Architecture — SIMRS × CALL-E

## Overview

SIMRS × CALL-E is a healthcare appointment reminder system that combines CALL-E's AI voice agent platform with Indonesian hospital management systems (SIMRS) and the national health data exchange (SatuSehat).

## Components

### 1. Appointment Source

**Options:**
- **SIMRS REST API** — query upcoming appointments via HTTP
- **CSV/JSON Export** — manual file upload from scheduling system
- **SatuSehat FHIR** — query FHIR Appointment resources

### 2. Batch Reminder Engine (`scripts/batch-reminder.py`)

Python script (zero external dependencies) that orchestrates the reminder workflow:

```
Load Appointments → Validate → For Each: Plan Call → Run Call → Parse Result → Writeback
```

**Key responsibilities:**
- Input validation (E.164 phone, required fields)
- CALL-E CLI interaction (plan → run → status)
- Outcome classification (regex pattern matching)
- SIMRS writeback via REST API
- Batch summary reporting

### 3. CALL-E Voice Agent

Places outbound calls via the CALL-E platform. Each call:
- Speaks Bahasa Indonesia
- Greets patient by name
- Reads appointment details (doctor, poli, date, time)
- Asks for confirmation
- Handles reschedule requests
- Handles cancellations
- Escalates emergencies to 112

### 4. SIMRS Writeback

Updates appointment status in the hospital's EHR:
- CONFIRMED → mark patient as confirmed
- RESCHEDULED → cancel old + create new appointment
- CANCELLED → mark as cancelled with reason
- PENDING_RETRY → flag for retry
- CONTACT_ERROR → flag for staff follow-up

### 5. SatuSehat FHIR Layer (Optional)

Generates FHIR R4 resources for Indonesia's national health data exchange:
- **Appointment** resource with status update
- **Encounter** resource for pre-confirmed visits
- **Communication** resource for audit trail

## Data Flow Diagram

```
                    ┌──────────────────────┐
                    │   Hospital Staff     │
                    │   (initiates batch)  │
                    └──────────┬───────────┘
                               │
                               ▼
                    ┌──────────────────────┐
                    │  batch-reminder.py   │
                    │  ┌─────────────────┐ │
                    │  │ Load Appointments│ │
                    │  │ (JSON/CSV/API)  │ │
                    │  └────────┬────────┘ │
                    │           │          │
                    │  ┌────────▼────────┐ │
                    │  │ Validate Fields │ │
                    │  │ (E.164, etc.)   │ │
                    │  └────────┬────────┘ │
                    │           │          │
                    │  ┌────────▼────────┐ │
                    │  │ For Each Patient│ │
                    │  │ ┌─────────────┐ │ │
                    │  │ │ calle plan  │ │ │
                    │  │ │ calle run   │ │ │
                    │  │ │ Parse Result│ │ │
                    │  │ └─────────────┘ │ │
                    │  └────────┬────────┘ │
                    └───────────┼──────────┘
                               │
                    ┌──────────▼───────────┐
                    │    CALL-E Platform    │
                    │  ┌─────────────────┐ │
                    │  │ AI Voice Agent  │ │
                    │  │ (Bahasa Indo)   │ │
                    │  │ Calls Patient   │ │
                    │  └─────────────────┘ │
                    └──────────┬───────────┘
                               │
              ┌────────────────┼────────────────┐
              │                │                │
              ▼                ▼                ▼
     ┌──────────────┐ ┌──────────────┐ ┌──────────────┐
     │    SIMRS     │ │   SatuSehat  │ │  Call Report │
     │  (EHR/EPR)  │ │  (FHIR API)  │ │  (JSON/CSV)  │
     └──────────────┘ └──────────────┘ └──────────────┘
```

## Security Considerations

- Patient phone numbers never stored in CALL-E metadata
- CALL-E auth tokens cached locally, not transmitted
- SIMRS writeback uses hospital's internal network
- SatuSehat uses OAuth 2.0 client credentials
- All call transcripts retained for 30 days max
- Patient consent verified before each batch run
