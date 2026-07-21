# AI Medical Appointment Agent

An AI-powered voice agent that answers hospital phone calls, books patient appointments, confirms bookings via email, and logs every appointment into a spreadsheet for doctors and staff to track.

---

## 📋 Overview

Patients call the hospital's phone number as usual. Instead of a human receptionist, an **AI voice agent** answers the call, understands the patient's request through natural conversation, checks doctor availability, books the appointment, and completes the loop by:

1. Sending the patient a **confirmation email**
2. Logging the appointment into a **spreadsheet / Excel file** for doctors and hospital staff

This removes manual scheduling work, reduces missed calls, and keeps a clean, always-up-to-date appointment record.

---

## 🎯 Key Features

- **📞 Automated Call Handling** — AI answers incoming hospital calls 24/7, no hold times.
- **🗣️ Natural Conversation** — Understands patient name, preferred doctor/department, date, and time through speech.
- **🩺 Doctor Availability Check** — Cross-checks requested slot against the doctor's live schedule.
- **📅 Appointment Booking** — Books the confirmed slot automatically in the scheduling system.
- **📧 Email Confirmation** — Sends the patient an automated confirmation email with appointment details.
- **📊 Spreadsheet Logging** — Writes every appointment (patient name, doctor, date, time, contact info, status) to an Excel/Google Sheet for hospital records.
- **🔁 Rescheduling & Cancellation** *(optional/future)* — Patients can call back to modify or cancel bookings.

---

## 🏗️ How It Works (High-Level Flow)

```
Patient Call
     │
     ▼
AI Voice Agent Answers
     │
     ▼
Collect Patient Details (name, phone, reason, preferred doctor/date/time)
     │
     ▼
Check Doctor Availability
     │
     ├── Slot Available ──► Book Appointment
     │                           │
     │                           ├──► Send Confirmation Email to Patient
     │                           └──► Log Appointment to Spreadsheet/Excel
     │
     └── Slot Unavailable ──► Offer Alternative Slots ──► (loop back)
```

A machine-readable version of this workflow is provided in **`workflow.json`**.

---

## 🧩 System Components

| Component | Purpose |
|---|---|
| **Voice/Telephony Layer** | Receives and handles the incoming call (e.g., speech-to-text, text-to-speech) |
| **AI Conversation Engine** | Understands patient intent, extracts appointment details, handles dialogue |
| **Scheduling/Calendar System** | Source of truth for doctor availability |
| **Appointment Booking Logic** | Validates and confirms the appointment slot |
| **Email Service** | Sends confirmation emails to patients |
| **Spreadsheet/Excel Integration** | Records appointment data for hospital staff and doctors |

---

## 🛠️ Suggested Tech Stack

> These are common choices — swap for whatever your project actually uses.

- **Voice/Call Handling:** Twilio Voice, Vonage, or similar telephony API
- **Speech-to-Text / Text-to-Speech:** Whisper, Google Speech-to-Text, ElevenLabs
- **AI/NLU Engine:** Claude API / LLM-based intent extraction
- **Backend:** Node.js / Python (FastAPI or Flask)
- **Database/Calendar:** Google Calendar API or custom database for doctor schedules
- **Email Service:** SMTP, SendGrid, or Gmail API
- **Spreadsheet Logging:** Google Sheets API or Excel (openpyxl / SheetJS)

---

## 📁 Suggested Project Structure

```
ai-medical-agent/
├── README.md
├── workflow.json
├── src/
│   ├── call_handler/         # Handles incoming calls
│   ├── nlu/                  # Extracts intent & appointment details
│   ├── scheduler/             # Checks/manages doctor availability
│   ├── booking/                # Appointment booking logic
│   ├── email_service/         # Sends confirmation emails
│   └── spreadsheet_logger/    # Writes appointments to Excel/Sheets
├── config/
│   └── settings.example.json
└── data/
    └── appointments.xlsx      # Appointment log (or Google Sheet link)
```

---

## 📊 Spreadsheet Log Format

Each booked appointment is recorded with these columns:

| Patient Name | Phone Number | Email | Doctor | Department | Appointment Date | Appointment Time | Reason for Visit | Status | Booked At |
|---|---|---|---|---|---|---|---|---|---|

---

## 📧 Email Confirmation Contents

The confirmation email typically includes:
- Patient name
- Doctor name & department
- Appointment date & time
- Hospital address/location
- Instructions (arrival time, documents to bring)
- Contact number for rescheduling/cancellation

---

## 🚀 Setup (Placeholder — customize per your implementation)

1. Clone the repository
2. Install dependencies
3. Add API keys/config in `config/settings.json` (telephony, email, spreadsheet/calendar credentials)
4. Connect your phone number to the voice agent webhook
5. Run the service
6. Test by calling the connected number

---

## 🔒 Notes on Sensitive Data

- Patient data (name, phone, health-related reason for visit) is sensitive information — store and transmit it securely (encryption in transit/at rest).
- Follow applicable healthcare data privacy regulations for your region (e.g., HIPAA, GDPR, or local equivalents) when handling patient information.
- Limit spreadsheet/email access to authorized hospital staff only.

---

## 🗺️ Roadmap Ideas

- [ ] Multi-language support for patients
- [ ] SMS confirmation in addition to email
- [ ] Rescheduling/cancellation via call-back
- [ ] Doctor-side dashboard instead of raw spreadsheet
- [ ] Waitlist handling when no slots are available
