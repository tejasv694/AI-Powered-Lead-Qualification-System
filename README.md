# 🤖 AI Lead Qualifier — n8n Workflow

> Automatically score, route, and respond to inbound B2B leads using GPT-4o and the BANT framework. Zero manual triage.

---

## What It Does

This n8n workflow triggers on every inbound lead form submission and:

1. **Normalizes** the raw POST payload into a clean lead object (handles different field names from any form)
2. **Scores** the lead using GPT-4o mini with BANT criteria (Budget, Authority, Need, Timeline)
3. **Routes** the lead — qualified vs. unqualified — into separate Google Sheets tabs
4. **Sends** a personalized auto-reply email based on lead tier
5. **Returns** a `200 OK` JSON response to the originating form with a unique reference ID

---

## Architecture

![AI Lead Qualifier Architecture](./architecture.png)

---

## How It Works

### Nodes Overview

| Node | Type | Purpose |
|---|---|---|
| `Webhook – Receive Lead` | Webhook | Accepts POST at `/lead-qualifier` |
| `Normalize Lead Data` | Code (JS) | Standardizes field names from any form shape |
| `GPT-4o mini – BANT Scoring` | OpenAI | Scores lead, returns structured JSON |
| `Parse AI Response` | Code (JS) | Merges AI output with lead data |
| `Qualified?` | IF | Routes on `qualified: true/false` |
| `Google Sheets – Qualified` | Google Sheets | Appends to "Qualified Leads" tab |
| `Google Sheets – Unqualified` | Google Sheets | Appends to "Unqualified Leads" tab |
| `Email – Qualified Reply` | Email (SMTP) | Sends hot-lead email with Calendly link |
| `Email – Nurture Reply` | Email (SMTP) | Sends nurture-track email |
| `Webhook Response` | Respond to Webhook | Returns `200` JSON with reference ID |

---

## AI Scoring Output Schema

The GPT-4o node is prompted to return strict JSON at temperature 0:

```json
{
  "qualified": true,
  "score": 85,
  "tier": "Hot",
  "reason": "Decision-maker with defined budget, clear use case, and near-term timeline.",
  "strengths": ["C-suite authority", "Defined budget range", "Specific use case"],
  "weaknesses": ["Mid-market company size"],
  "recommended_action": "Book a discovery call within 24 hours",
  "follow_up_priority": "Immediate"
}
```

**Tier rules:**
- `80–100` → Hot
- `60–79` → Warm
- `40–59` → Cold Nurture
- `0–39` → Cold / Disqualify

**Follow-up priority values:** `Immediate` · `Within 24h` · `Within 1 week` · `Nurture`

---

## Expected Webhook POST Body

```json
{
  "name": "TJ MK",
  "email": "user@example.com",
  "company": "TJ Corp",
  "role": "COO",
  "budget": "$10,000 – $25,000",
  "employees": "11–50 employees",
  "timeline": "Next quarter",
  "use_case": "Automate lead qualification and CRM workflows",
  "message": "Additional context here",
  "source": "direct"
}
```

The `Normalize Lead Data` node maps common field name aliases automatically — `full_name`, `email_address`, `company_size`, `job_title`, etc. are all handled.

---

## Webhook Response

On success, the form receives:

```json
{
  "success": true,
  "message": "Your inquiry has been received. We'll be in touch very shortly.",
  "reference": "user@example.com-1775746305379"
}
```

---

## Tech Stack

- **n8n** — workflow automation engine
- **OpenAI GPT-4o mini** — AI scoring via chat completions
- **Google Sheets API** — lead CRM storage
- **SMTP / Gmail** — transactional email delivery

---

## Request Access

The full n8n workflow file is available upon request — not published here to prevent direct production cloning without setup context.

To request it, open a [GitHub Issue](../../issues) with the title **"Workflow Request"** or reach out directly via [LinkedIn](https://linkedin.com/in/yourprofile).

---

## License

MIT — architecture, documentation, and approach are free to reference and adapt.
