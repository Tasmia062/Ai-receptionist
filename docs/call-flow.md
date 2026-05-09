# Call Flow

## Business Hours Flow

Caller dials main line
Hours/day check (Mon–Fri, 9am–5pm ET)
Ava greets caller
Intent identified

    ├── Named Agent Request
    │     → Ask for caller name
    │     → Set transfer_target_number + transfer_target_label
    │     → "One moment while I check if they're available"
    │     → Cold Transfer via SIP REFER
    │     → On failure → Transfer Failure Recovery
    │           → Retry / Callback / Appointment / Voicemail
    │
    ├── General Human Request
    │     → Ask reason for call
    │     → Route to Jason (primary) or Debbie (secondary)
    │     → Cold Transfer via SIP REFER
    │
    ├── Insurance Education Request
    │     → Clarify what they want to know
    │     → Route to Medicare / Life / Supplemental education node
    │     → After answering → offer agent transfer or appointment
    │
    ├── Compliance Escalation
    │     → Decline to advise
    │     → Offer agent transfer or appointment
    │
    ├── Appointment / Callback Request
    │     → Offer: Schedule now / Send link / Callback
    │     → Schedule now → Zapier workflow trigger
    │     → Send link → GHL calendar link via text or email
    │     → Callback → Collect details → GHL callback workflow
    │
    └── DNC Request
    → Confirm removal
    → End call immediately

## After Hours Flow

Caller dials main line
Hours/day check fails (weekend or outside 9am–5pm ET)
Ava informs caller of office hours
Offer: Appointment or Voicemail

    ├── Appointment → Appointment / Callback Intake
    └── Voicemail
    → Named agent voicemail → transfer to agent GHL number
    → General voicemail → transfer to Debbie's GHL number

## Post-Call Automation Flow
1. Call ends
2. Retell fires post-call webhook to n8n
3. n8n parses payload → Extracts: caller name, phone, state, product interest,
urgency, outcome, call summary, recording URL
4. Validate payload (name + phone required) → Invalid → log for manual review
5. Qualify lead → Qualified: has product interest + non-disqualified outcome → Disqualified: wrong number, spam, hangup, no product interest
6. GHL Contact upsert (all callers)
7. If Qualified: → Search for existing opportunity (deduplication) → If none exists → create opportunity in pipeline → Attach full call note with recording URL
8. If Disqualified: → Attach call note with recording URL for audit trail

## Agent Directory

    |  Agent |           Role            |     Transfer Method     |
    |--------|---------------------------|-------------------------|
    | Jason  |       Primary agent       | Cold Transfer SIP REFER |
    | Debbie |      Secondary agent      | Cold Transfer SIP REFER |
    | Debbie | Customer service fallback | Cold Transfer SIP REFER |

## Transfer Failure Handling

If a named agent transfer fails:
- If caller asked for Jason specifically → offer callback, appointment, or voicemail
- If general routing to Jason fails → auto-retry with Debbie
- If Debbie also fails → offer callback or appointment
