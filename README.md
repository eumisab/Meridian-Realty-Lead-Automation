# AI Lead Management System — Meridian Realty Group

An end-to-end AI-powered lead capture, categorization, and follow-up automation built with n8n, Google Gemini, Airtable, and Gmail.

---

## Overview

Real estate agencies lose deals not because of bad leads — but because of slow follow-up. This system ensures every inquiry gets an instant, personalized response and a structured follow-up sequence, without any manual effort from the team.

Built as a portfolio project simulating a real-world deployment for a small real estate agency.

---

## What It Does

- Captures leads from a web form (Tally)
- Uses AI to classify each lead as **Hot Lead**, **Warm Lead**, or **Just Browsing** based on their message
- Automatically creates a CRM record in Airtable with lead details and category
- Sends a personalized reply email written by AI, tailored to the lead's intent and message
- Follows up automatically at Day 3 and Day 7 if the lead hasn't responded

---

## Workflow Architecture

### Pipeline 1 — Lead Intake (Triggered by form submission)

```
Tally Form → Webhook → Parse Fields → Gemini (Categorize) → Airtable (Log Lead) → Gemini (Write Email) → Gmail (Send)
```

### Pipeline 2 — Follow-up Sequence (Triggered daily by schedule)

```
Schedule Trigger → Airtable (Search uncontacted leads) → If (leads exist?) → Switch (Day 3 or Day 7?) → Gemini (Write follow-up) → Gmail (Send) → Airtable (Update status)
```

---

## Tech Stack

| Tool | Role |
|------|------|
| n8n (self-hosted) | Workflow automation engine |
| Google Gemini 3.5 Flash | Lead categorization + email generation |
| Airtable | Lead CRM and status tracking |
| Tally | Lead capture form |
| Gmail | Automated email delivery |
| Railway | n8n hosting |

---

## AI Prompt Design

### Lead Categorization
Classifies each lead into one of three categories based on urgency signals in their message:
- **Hot Lead** — pre-approved, ASAP, ready to act
- **Warm Lead** — interested but no immediate urgency
- **Just Browsing** — early-stage, exploratory

### Personalized Email Generation
Writes a unique reply for each lead using:
- Their name and specific message content
- Category-aware tone and call to action
- Vague message detection — if key details like location or budget are missing, the AI asks one targeted follow-up question instead of guessing or giving a generic response
- Category-aware follow-up logic:
  - Hot Lead + vague → asks for location or move-in timeline
  - Warm Lead + vague → asks for property type or location
  - Just Browsing + vague → offers a general resource, no pressure

### Follow-up Emails
Two automated follow-ups with distinct tone per category and lead stage, designed to re-engage without feeling spammy. Final follow-up (Day 7) closes the sequence gracefully regardless of response.

---

## Key Features

- **Zero manual effort** — runs entirely on its own after initial setup
- **AI that reads context** — not template-fill-in-the-blank, each email is generated from the actual lead message
- **Resilient field parsing** — all Tally fields parsed by label name, not array index, so form reordering never breaks the workflow
- **Safe optional fields** — all non-required fields use null-safe expressions (`?.value || ""`) so missing data never crashes an execution
- **Full audit trail** — every lead, status change, and follow-up date is logged in Airtable

---

## Stress Testing

Tested across 5 message types:
- Clear Hot Lead ("pre-approved, need a house ASAP")
- Clear Just Browsing ("just curious, no rush")
- Minimal input ("looking")
- Special characters and line breaks
- Emoji or typos in message body

All categorized correctly with no execution errors.

---

## Files

| File | Description |
|------|-------------|
| `Meridian_Lead_Pipeline_Workflow.json` | Main intake workflow (form → categorize → log → email) |
| `Meridian_follow-up.json` | Daily follow-up sequence (Day 3 and Day 7) |

### How to import
1. In n8n, go to **Workflows → Import from File**
2. Import each JSON file separately
3. Reconnect credentials (Gemini API, Airtable token, Gmail OAuth)
4. Update the Airtable base/table IDs to match your own base
5. Activate both workflows

---

## What I Learned

- Designing multi-branch conditional workflows with Switch and If logic in n8n
- Integrating Google Gemini via LangChain nodes for both classification and generation tasks
- Writing prompts that handle ambiguous real-world inputs reliably
- Building resilient field-mapping expressions that don't break on optional or reordered form fields
- Structuring a follow-up sequence using Airtable as the source of truth for lead state

---

## About

Built by Eumi Sabueto — AI Automation Developer specializing in n8n, Make, Zapier, Airtable, with deep expertise in integrating LLMs like Gemini, Claude, and OpenAI.
