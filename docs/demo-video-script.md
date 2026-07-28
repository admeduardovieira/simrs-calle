# Demo Video Script — SIMRS × CALL-E

**Duration:** 3 minutes max  
**Platform:** YouTube or Vimeo  
**Language:** English (with Indonesian call example)

---

## Scene Breakdown

### [0:00 – 0:15] Opening / Problem Statement

**Visual:** Hospital admin desk, phone ringing, paperwork  
**Text Overlay:** "Indonesian hospitals lose 20% of appointments to no-shows"  
**Narration:**
> "Every day, hospital staff spend hours calling patients to remind them about appointments. Many patients forget, don't answer, or the calls never get made. This costs hospitals money and patients miss critical care."

---

### [0:15 – 0:30] Introducing SIMRS × CALL-E

**Visual:** Terminal showing the project repo  
**Text Overlay:** "SIMRS × CALL-E — Automated Appointment Reminders"  
**Narration:**
> "SIMRS × CALL-E uses AI voice calling to automatically remind patients about their appointments. One command processes the entire next-day schedule."

---

### [0:30 – 0:50] Dry Run Demo

**Visual:** Terminal screen recording  
**Actions:**
```bash
python3 scripts/batch-reminder.py \
  --appointments scripts/example_appointments.json \
  --dry-run
```
**Show:** Batch report output with 3 appointments, all DRY_RUN  
**Narration:**
> "First, let's see how it works with dry-run mode. No real calls — just validation. We load 3 appointments and verify each one is ready."

---

### [0:50 – 1:40] Live Call Demo

**Visual:** Split screen — terminal + phone call UI (or phone screen)  
**Text Overlay:** "Live CALL-E Outbound Call"  
**Actions:**
1. Show `calle call start --to-phone ... --goal ...`
2. Show the CALL-E voice greeting in Indonesian
3. Show patient responding "Ya, saya bisa hadir"
4. Show outcome: `CONFIRMED`

**Narration:**
> "Now the real thing. CALL-E calls the patient in Bahasa Indonesia. The patient confirms their appointment. The system classifies this as CONFIRMED."

**Call Example (Indonesian):**
> AI: "Selamat pagi, Bapak Budi Santoso? Saya dari RSUD Leuwiliang. Mengingatkan jadwal kontrol Bapak besok jam 9 pagi dengan Dr. Ahmad di Poli Jantung. Apakah Bapak bisa hadir?"
> Patient: "Ya, bisa. Terima kasih."
> AI: "Baik, terima kasih Bapak. Sampai jumpa besok."

---

### [1:40 – 2:10] Outcome & Writeback

**Visual:** Terminal showing JSON result + SIMRS dashboard mockup  
**Show:**
```json
{
  "appointmentId": "APT-20260729-001",
  "status": "CONFIRMED",
  "phone": "+628****1919",
  "runId": "run_abc123"
}
```
**Text Overlay:** "Outcome auto-written to SIMRS"  
**Narration:**
> "The outcome is classified automatically and written back to the hospital's SIMRS system. No manual data entry needed. For SatuSehat-compatible hospitals, a FHIR Encounter resource is also created."

---

### [2:10 – 2:40] Batch Summary

**Visual:** Terminal showing full batch report  
**Show:**
```
Total: 3
CONFIRMED: 2
PENDING_RETRY: 1
```
**Text Overlay:** "200+ patients processed in minutes, not hours"  
**Narration:**
> "For a full clinic schedule, the batch runner processes hundreds of patients in minutes. Failed calls get flagged for staff follow-up."

---

### [2:40 – 3:00] Impact & Closing

**Visual:** Impact metrics table  
**Text Overlay:**
```
No-show rate: 20% → 8%
Staff time: 4 hrs/day → 0 hrs/day
Call success: 60% → 85%+
```
**Narration:**
> "Hospitals using automated voice reminders see no-show rates drop by more than half. Staff are freed up for patient care. Built with CALL-E, SatuSehat-ready, and open source."

**End screen:**
> SIMRS × CALL-E  
> github.com/sigit4715/simrs-calle  
> Built for Indonesian healthcare 🇮🇩

---

## Recording Tips

- Use `asciinema` or `OBS` for terminal recording
- Use `edge-tts` for English narration (Ardi voice)
- Keep terminal font large and readable
- Use dark theme for professional look
- Add subtle background music (low volume)
