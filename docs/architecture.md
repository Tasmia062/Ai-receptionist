# Architecture Overview

## System Diagram

              Inbound Call
                   ↓
            Twilio (via GHL)
                   ↓
Retell AI – Conversation Flow Agent (Ava)
                   ↓
┌─────────────────────────────────────┐
│         Intent Routing              │
├─────────────────────────────────────┤
│ Named Agent Request                 │
│   → Cold Transfer (SIP REFER)       │
│   → GHL Agent Number                │
│                                     │
│ General Human Request               │
│   → Cold Transfer (SIP REFER)       │
│   → GHL Agent Number                │
│                                     │
│ Appointment / Callback              │
│   → GHL Webhook / Zapier            │
│                                     │
│ Insurance Education                 │
│   → Handled in-call by Ava          │
│                                     │
│ After Hours                         │
│   → Voicemail Transfer              │
│   → Appointment Offer               │
│                                     │
│ DNC Request                         │
│   → Confirmation + End Call         │
└─────────────────────────────────────┘
                   ↓
              Call Ends
                   ↓
        Retell Post-Call Webhook
                   ↓
              n8n Workflow
                   ↓
┌─────────────────────────────────────┐
│       Parse + Validate Payload      │
│       Qualify Lead                  │
│       GHL Contact Upsert            │
│       Duplicate Opportunity Check   │
│       Pipeline Entry (Qualified)    │
│       Call Note + Recording URL     │
│       Contact Log (Disqualified)    │
└─────────────────────────────────────┘

## Components

**Retell AI**
Hosts the Conversation Flow Agent (Ava) and Transfer Agent. Manages call logic, node transitions, dynamic variables, tool calls, and post-call webhooks.

**GoHighLevel (GHL)**
Acts as the telephony layer via Twilio-backed phone numbers. Also serves as the CRM — contacts, pipelines, notes, calendars, and workflows all live here.

**Twilio**
Underlying carrier for all GHL phone numbers. Inbound calls arrive via Twilio SIP and are forwarded to Retell. Outbound transfer legs use SIP REFER.

**n8n**
Automation layer that receives Retell post-call webhooks, parses call data, qualifies leads, and creates/updates records in GHL via API.

**Zapier**
Handles the schedule-now workflow trigger when a caller opts to book an appointment during the call.

## Key Technical Decisions
________________________________________________________________________________________________________________________
|                Decision                |                                   Reason                                    |
|----------------------------------------|-----------------------------------------------------------------------------|
| Cold Transfer via SIP REFER            | Agentic Warm Transfer caused SIP 488 TLS errors with GHL Twilio numbers     |
| Dynamic variables for transfer routing | Allows flexible agent directory without hardcoded destinations              |
| n8n post-call webhook                  | Decouples CRM logging from live call—no latency impact on caller experience |
| Opportunity deduplication check        | Prevents duplicate pipeline entries for repeat callers                      |
| Disqualified contact logging           | Maintains full compliance audit trail regardless of lead quality            |
|________________________________________|_____________________________________________________________________________|
