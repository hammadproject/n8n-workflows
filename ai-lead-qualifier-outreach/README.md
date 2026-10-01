# AI Lead Qualifier & Outreach

An n8n workflow that receives inbound leads via a webhook, uses OpenAI to score and categorize them, saves the result to a Google Sheets CRM, sends a tailored outreach email, and posts a Slack notification. A second branch runs on a daily schedule to follow up with Hot and Warm leads that have not yet received a response.

## Features

- Accepts leads via a POST webhook from any web form or CRM.
- Validates and normalizes lead data including email format, domain, and website URL.
- Checks for duplicate leads by email before processing.
- Fetches and reads the company website for additional qualification context.
- Scores each lead 1–10 with Hot, Warm, and Cold categories using OpenAI.
- Saves or updates leads in a Google Sheets CRM with a full qualification record.
- Sends category-specific outreach emails: booking link for Hot, discovery questions for Warm, and a nurture email for Cold.
- Posts a Slack alert for Hot leads and logs duplicates and dedup failures.
- Runs a daily follow-up job that re-contacts Hot and Warm leads that have not replied.
- Skips follow-up for leads that have already replied in Gmail.
- Treats lead messages and website content as untrusted data in the AI prompt.

## Architecture

```text
Flow 1 — New lead (Webhook trigger)
  Webhook - New Lead (POST /lead-capture)
    -> Normalize Lead Data
    -> IF Valid Email?
      -> Check Duplicate Lead (Google Sheets)
        -> [Duplicate] Slack Note (Duplicate) -> stop
        -> [New] IF Has Website?
          -> [Yes] Fetch Company Website -> Extract Website Text
          -> [No]  No Website Context
          -> AI Qualify Lead (OpenAI)
          -> Parse AI Output
          -> Upsert CRM (Google Sheets)
          -> Route by category (Hot / Warm / Cold)
            -> [Hot]  Slack alert + booking email
            -> [Warm] Qualifying questions email
            -> [Cold] Nurture email

Flow 2 — Daily follow-up (Schedule trigger at 09:00)
  Daily 9am Trigger
    -> Get All Leads (Google Sheets)
    -> Filter: Hot (≥1 day) or Warm (≥3 days), not yet followed up
    -> Check Gmail for existing replies
    -> Send follow-up email
    -> Update Lead Record (followed_up = true)
```

## Lead scoring

| Score | Category | Action |
| --- | --- | --- |
| 8–10 | 🔥 Hot | Slack alert + immediate booking email + follow-up after 1 day |
| 5–7 | 🌤 Warm | Discovery questions email + follow-up after 3 days |
| 1–4 | 🧊 Cold | Polite nurture email, no follow-up scheduled |

## Webhook payload

Send a `POST` request to the webhook URL with a JSON body. Only `email` is required; the other fields improve the AI score.

```json
{
  "name": "Jane Doe",
  "email": "jane@acme.com",
  "company": "Acme Inc.",
  "message": "We want to automate our invoicing process.",
  "budget": "$2,000",
  "website": "acme.com"
}
```

## Requirements

- An n8n Cloud or self-hosted n8n instance.
- A Google account with Google Sheets and Gmail OAuth2 access.
- A Slack workspace with an OAuth2 app connection.
- An OpenAI API key.

## Google Sheets structure

Create a spreadsheet with a tab named `Leads` and these first-row headers.

```text
name | email | company | message | budget | website | score | category | reasoning | suggested_reply_opener | timestamp | followed_up | followup_days | followup_sent_at
```

## Setup

1. Import [`workflow.json`](./workflow.json) into n8n.
2. Connect your Google Sheets OAuth2 credential to all Google Sheets nodes.
3. Connect your Gmail OAuth2 credential to all Gmail nodes.
4. Connect your Slack OAuth2 credential to all Slack nodes.
5. Connect your OpenAI credential to the **OpenAI Chat Model - Lead Qualification** node.
6. Select your Google Sheet and the `Leads` tab in the **Check Duplicate Lead**, **Upsert CRM**, **Get All Leads**, and **Update Lead Record** nodes.
7. Select your Slack channel in all three Slack nodes.
8. Replace the Calendly booking link placeholder in the **Booking Email (Hot)** and **Send Follow-up Email** Gmail nodes.
9. Update the agency name and signature in the Gmail email nodes.
10. Update the `User-Agent` header in **Fetch Company Website** with your own contact URL.
11. Test with a POST request to the webhook Test URL, then activate the workflow.

## Common customizations

- Edit the qualification prompt in **AI Qualify Lead** to reflect your ideal customer profile.
- Change the scoring thresholds and follow-up delays in **Parse AI Output**.
- Adjust the follow-up schedule in the **Daily 9am Trigger** node.
- Customize outreach email copy in the Gmail nodes.
- Swap Slack for Teams or WhatsApp, or Gmail for Outlook.
- Add CRM integrations such as HubSpot, Pipedrive, or Notion.

## Security and privacy notes

- Lead messages and company website content are treated as untrusted data in the AI prompt.
- Private and local network URLs are never fetched.
- Never store API keys or personal data inside workflow nodes; always use n8n credentials.
- Ensure your lead capture form complies with applicable data collection and consent regulations.

## Repository-safe export

This version contains no saved Google, Gmail, Slack, or OpenAI credentials; no private spreadsheet IDs or Calendly links; and no n8n instance metadata or generated webhook IDs. Users must provide their own configuration after importing it.
