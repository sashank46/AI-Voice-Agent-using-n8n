# AI Dental/Medical Appointment Reminder Agent

An automated outbound voice reminder system built with **n8n**, **Retell AI**, and **Google Calendar**. The system watches upcoming appointments on a calendar, and an AI voice agent (named **Anne**) automatically calls patients ahead of time to remind them of their appointment, confirm they'll still attend, and transfer them to a human staff member if they need to reschedule or cancel.

---

## 🎯 What It Does

1. **Watches Google Calendar** for appointments coming up (e.g. in the next 12 hours).
2. **Extracts structured appointment data** (name, email, phone, reason, start/end time) from the calendar event using an LLM.
3. **Triggers an outbound call** through Retell AI, passing the appointment details as dynamic variables.
4. The **AI voice agent ("Anne")** calls the patient, confirms their identity, reminds them of the appointment time and reason, and asks if the time still works.
5. If the patient wants to **reschedule/cancel**, the call is **transferred to a human staff member**.
6. If the patient **isn't available**, the agent either leaves a message with whoever answers or ends the call politely.

This is a **reminder/confirmation** workflow (outbound), not a booking workflow — the appointment already exists on the calendar; the AI agent's job is to call and confirm it.

---

## 🧩 Accounts Needed

| Service | Purpose |
|---|---|
| **[n8n](https://n8n.io)** | Workflow orchestration — watches the calendar, runs the LLM extraction step, and calls the Retell AI API |
| **[Retell AI](https://www.retellai.com)** | Powers the AI voice agent that places the outbound call and talks to the patient |
| **Google Calendar** | Source of truth for upcoming appointments |

---

## 🏗️ Workflow Architecture

```
Google Calendar Trigger
   (event starting within next 12 hours)
            │
            ▼
   n8n: LLM Extraction Node
   (turns calendar event into structured JSON:
    name, email, phone_number, reason, start_time, end_time)
            │
            ▼
   n8n: HTTP Request Node
   (calls Retell AI "create-phone-call" API,
    passes extracted fields as dynamic variables)
            │
            ▼
   Retell AI Voice Agent "Anne" places the call
            │
            ▼
   ┌─────────────────────────────────────────────┐
   │  1. Identify recipient                       │
   │  2. Introduce herself & purpose of the call   │
   │  3. Provide appointment details               │
   │  4. Confirm if the time still works           │
   │        ├─ Confirmed → politely end call       │
   │        ├─ Wants to reschedule/cancel          │
   │        │      → transfer_call to staff        │
   │        └─ Unclear → clarify, then confirm      │
   │             or transfer                       │
   └─────────────────────────────────────────────┘
```

---

## ⚙️ n8n Setup

### 1. Google Calendar Trigger
Watches for events starting within a set window.

**Expression used to define the "upcoming" window:**
```
{{ $now.plus(12, 'hours') }}
```
This checks for appointments starting within the next 12 hours, so reminder calls go out roughly half a day in advance.

### 2. LLM Node — Extract Structured Appointment Data

**System Message:**
```
You are an assistant. Generate structured JSON with the following fields:
- name: full name of the recipient
- phone_number: string
- reason: reason for the appointment
- start_time: ISO 8601 datetime (Format is date-time)
- end_time: ISO 8601 datetime (Format is date-time)
- email

Output ONLY in this exact JSON format. No comments, no extra text.
```

**JSON Schema:**
```json
{
  "type": "object",
  "properties": {
    "output": {
      "type": "object",
      "properties": {
        "name": { "type": "string" },
        "email": { "type": "string", "format": "email" },
        "phone_number": { "type": "string" },
        "reason": { "type": "string" },
        "start_time": { "type": "string", "format": "date-time" },
        "end_time": { "type": "string", "format": "date-time" }
      },
      "required": [
        "name",
        "email",
        "phone_number",
        "reason",
        "start_time",
        "end_time"
      ]
    }
  },
  "required": ["output"]
}
```

### 3. HTTP Request Node — Trigger the Retell AI Call

**Endpoint:**
```
POST https://api.retellai.com/v2/create-phone-call
```

**Headers:**
| Key | Value |
|---|---|
| `Authorization` | `Bearer <your_api_key>` |
| `Content-Type` | `application/json` |

**Body:**
```json
{
  "from_number": "<your connected phone number>",
  "to_number": "{{ $json.output.phone_number }}",
  "retell_llm_dynamic_variables": {
    "name": "{{ $json.output.name }}",
    "phone_number": "{{ $json.output.phone_number }}",
    "reason": "{{ $json.output.reason }}",
    "start_time": "{{ $json.output.start_time }}",
    "end_time": "{{ $json.output.end_time }}"
  },
  "override_agent_id": "<your agent id>"
}
```

---

## 🗣️ Retell AI Voice Agent — "Anne"

### Identity
Anne is an AI-powered voice assistant for Smile Dental Clinic. Her role is to remind patients about upcoming appointments — friendly, professional, and clear. She **does not** reschedule or cancel appointments herself, but she **can transfer** the call to a human staff member on request.

### Style Guardrails
- **Concise** — clear, to-the-point responses
- **Engaging** — conversational, natural tone
- **Proactive** — guides the conversation, offers next steps
- **Clarifying** — asks polite follow-ups when something is unclear
- **Non-repetitive** — rephrases instead of repeating the same line

### Conversation Flow

**1. Identify the Recipient**
> "Hi, am I speaking with {{name}}?"
- **Yes** → proceed to Introduction
- **No** → ask if {{name}} is available or if a message can be left
  - Message can be conveyed → continue script, prefaced with "Great, please let {{name}} know the following..."
  - Not available / no message possible → "No problem, I'll try reaching out later. Thank you and have a great day!" → `end_call`

**2. Introduction**
> "This is Anne calling from Smile Dental Clinic. I'm reaching out to remind you about your upcoming dental appointment."

**3. Provide Appointment Details**
> "Your appointment for {{reason}} is scheduled between {{start_time}} and {{end_time}}."

**4. Confirm Availability**
> "Does that time still work for you?"
- **Confirmed** → "Great! We look forward to seeing you then. Is there anything else I can help you with today?" → end call
- **Wants to reschedule/cancel** → "No problem. I'll transfer you to someone at our clinic who can assist with that." → `transfer_call`
- **Unsure/unclear** → re-confirm details once more; if still unsure or wants changes → `transfer_call`

### Functions Used
| Function | When It's Used |
|---|---|
| `transfer_call` | Patient wants to reschedule or cancel the appointment |
| `end_call` | Patient/contact unavailable with no message to leave, or conversation completed |

### Notes
- Always convert ISO timestamps into natural spoken language before speaking them.
  - e.g. `2025-04-12T15:30:00` → *"Three thirty PM on Saturday, April 12th"*
- Never read raw timestamps, seconds, or timezone codes aloud.

---

## 📁 Suggested Project Structure

```
ai-reminder-agent/
├── README.md
├── workflow.json
├── n8n/
│   └── reminder_workflow.json      # exported n8n workflow
├── retell/
│   └── anne_agent_prompt.md        # agent prompt/config
└── config/
    └── settings.example.json       # API keys, phone numbers (never commit real keys)
```

---

## 🔒 Notes on Sensitive Data

- Patient name, phone number, email, and appointment reason are sensitive — handle in line with applicable healthcare privacy regulations (e.g., HIPAA, GDPR, or local equivalents).
- Never commit real API keys (Retell AI, n8n credentials) to version control — use environment variables or a secrets manager.
- Restrict access to the n8n workflow and Retell AI dashboard to authorized staff only.

---

