# Ai-receptionist
AI-powered inbound call handler built on Retell AI, Twilio, GHL, and n8n

## Overview
Ava is an AI-powered inbound call handler built on Retell AI. 
She answers calls, qualifies callers, transfers to named agents, 
books appointments, and logs every call into GHL automatically.

## Stack
- **Voice AI:** Retell AI (Conversation Flow Agent)
- **Telephony:** Twilio via GoHighLevel
- **CRM:** GoHighLevel (GHL)
- **Automation:** n8n
- **Scheduling:** Zapier + GHL Calendar

## Features
- Named agent transfer via Cold Transfer SIP REFER
- After-hours handling with voicemail routing
- Appointment booking and callback collection
- DNC self-service compliance handling
- Automatic lead qualification and GHL pipeline entry
- Full call notes with recording URL attached to every contact

## Architecture

<img width="1510" height="528" alt="image" src="https://github.com/user-attachments/assets/ba68ffb5-f3e9-44db-a4e5-87cc645cc43a" />


See [docs/architecture.md](docs/architecture.md)

## Call Flow
See [docs/call-flow.md](docs/call-flow.md)
