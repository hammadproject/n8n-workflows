# Local Business Lead Machine — AI Lead Generation & Outreach

An n8n workflow that scrapes Google Maps via Apify for local businesses matching a search query, audits each website for digital gaps, scores every lead using a rule-based 0–100 opportunity score, uses OpenAI to write a personalised pitch for hot and warm leads, saves all results to a Google Sheets CRM, posts a run summary to Slack, and creates Gmail outreach drafts for approved leads on a daily schedule.

## Features

- Launched via an n8n form: enter business type, city, country code, max results, and your service offering.
- Scrapes Google Maps using the Apify `compass/crawler-google-places` actor to find matching local businesses.
- Normalises and deduplicates results before any processing.
- Posts a Slack alert if no results are found for a search.
- Fetches each business website with an HTTP request to detect live/dead status.
- Uses Firecrawl to deep-scrape websites that need richer data (contact forms, booking widgets, chat, mobile-friendliness, copyright year).
- Scores each lead 0–100 using a rule-based engine (no AI):
  - No website: +45 (or +40 for social-only)
  - No online booking: +20
  - No live chat or after-hours widget: +15
  - Not mobile-friendly: +10
  - No contact form: +5
  - Outdated copyright year: +10
  - Unclaimed Google Business Profile: +10
  - 4.2+ rating with 50+ reviews: +15
  - No phone and no email: −20
- Buckets leads into Hot (60+), Warm (35–59), or Cold (<35).
- Skips AI pitch generation for cold leads to save API credits.
- Uses OpenAI to write a personalised email subject, body, and call opener for hot and warm leads in configurable batches with rate-limit pauses.
- Saves all leads with scores, gaps, and pitches to a Google Sheets Leads tab.
- Logs a run summary (total, hot, warm, cold, emails found) to a separate Runs tab.
- Posts a formatted run summary to Slack.
- Runs a daily outreach job at 10 AM on weekdays: reads leads with status `Approved`, filters those with an email address, caps at 20 drafts per day, creates a Gmail draft for each, marks the row as Drafted, and posts a Slack summary when done.
- An Error Trigger posts a Slack alert if any part of the workflow fails.

## Architecture

```text
Flow 1 — Lead search (n8n Form trigger)
  Start Lead Search (form: business type, city, country, max results, service)
    -> Build Apify Input
    -> Apify - Scrape Google Maps (HTTP Request to Apify API)
    -> Normalize & Dedupe
    -> Got Results?
      -> [No]  Slack - No Results
      -> [Yes] Fetch Website (HTTP Request per business)
               -> Detect Website Signals
               -> Need Firecrawl?
                 -> [Yes] Firecrawl Scrape -> Extract Firecrawl Data
                 -> [No]  (skip)
               -> Merge Website Results
               -> Opportunity Score (rule-based 0-100, Hot/Warm/Cold)
               -> Needs Pitch?
                 -> [Yes] AI Batch Loop
                          -> Write Pitch (AI) with OpenAI
                          -> Pitch JSON Parser
                          -> Attach Pitch
                          -> Rate Limit Pause
                 -> [No]  Skip Pitch (Cold)
               -> Merge All Leads
               -> Prepare Sheet Row
               -> Save Leads to Sheet (Google Sheets: Leads tab)
               -> Run Summary
               -> Log Run (Google Sheets: Runs tab)
               -> Slack - Run Summary

Flow 2 — Daily outreach (Schedule trigger: weekdays at 10 AM)
  Weekdays 10 AM
    -> Config
    -> Get Approved Leads (Google Sheets: status = Approved)
    -> Has Email? (filter leads with email address)
    -> Daily Cap (20)
    -> Loop Emails
       -> Create Gmail Draft
       -> Mark as Drafted (Google Sheets)
       -> Short Pause
    -> Slack - Outreach Done

Flow 3 — Error handling
  On Workflow Error -> Slack - Error Alert
```

## Scoring reference

| Signal | Points |
| --- | --- |
| No website | +45 |
| Social page only (no real site) | +40 |
| No online booking | +20 |
| No live chat / after-hours widget | +15 |
| Outdated copyright year | +10 |
| Not mobile-friendly | +10 |
| Unclaimed Google Business Profile | +10 |
| 4.2+ rating with 50+ reviews | +15 |
| No contact form | +5 |
| No phone AND no email | −20 |
| **Hot** | **60+** |
| **Warm** | **35–59** |
| **Cold** | **<35** |

## Google Sheets structure

Create a spreadsheet with two tabs.

**Leads tab** (`Leads`):

```text
place_id | business_name | category | address | city | phone | website | rating | reviews_count | maps_url | email | social_url | whatsapp_link | unclaimed_profile | has_website | has_booking | has_chat_widget | has_contact_form | mobile_friendly | copyright_year | opportunity_score | priority | top_gaps | email_subject | email_body | call_opener | status | search | scraped_at | sent_at
```

**Runs tab** (`Runs`):

```text
run_date | search | city | total | hot | warm | cold | emails_found
```

## Requirements

- An n8n Cloud or self-hosted n8n instance.
- An Apify account and API token (free tier includes $5 credit; actor: `compass/crawler-google-places`).
- A Firecrawl API key (free tier available).
- OpenAI access via n8n Gateway credits (no separate API key needed by default) or your own OpenAI API key.
- A Google account with Google Sheets OAuth2 access and Gmail OAuth2 access.
- A Slack workspace with a bot app that has `chat:write` scope.

## Setup

1. Import the workflow JSON into n8n.
2. Open the **Apify - Scrape Google Maps** node and set the HTTP Header Auth credential with `Authorization: Bearer YOUR_APIFY_TOKEN`.
3. Open the **Firecrawl Scrape** node and set its HTTP Header Auth credential with `Authorization: Bearer YOUR_FIRECRAWL_KEY`.
4. Connect your OpenAI credential (or use n8n Gateway) to the **OpenAI Chat Model** node.
5. Connect your Google Sheets OAuth2 credential to the **Save Leads to Sheet**, **Log Run**, and **Get Approved Leads** and **Mark as Drafted** nodes.
6. Connect your Gmail OAuth2 credential to the **Create Gmail Draft** node.
7. Connect your Slack credential to the **Slack - No Results**, **Slack - Run Summary**, **Slack - Outreach Done**, and **Slack - Error Alert** nodes.
8. Open all Slack nodes and select your alerts channel.
9. Open all Google Sheets nodes and select your spreadsheet and the correct tab (`Leads` or `Runs`). Set `cellFormat` to `RAW` to prevent phone numbers being interpreted as formulas.
10. In workflow **Settings**, set **Error Workflow** to this workflow so the **On Workflow Error** node fires on failure.
11. Start with **Max Results = 20** to conserve Apify credits before running larger searches.
12. After any edit, toggle the workflow off and back on to publish the updated version.
13. Activate the workflow.

## Common customizations

- Edit the service options in the **Start Lead Search** form to match your actual services.
- Adjust the scoring thresholds and point values in **Opportunity Score** to reflect your ideal customer profile.
- Change the daily outreach cap in **Daily Cap** (default: 20).
- Edit the OpenAI prompt in **Write Pitch (AI)** to match your brand voice and service.
- Add additional columns to the Leads tab and populate them in **Prepare Sheet Row**.

## Security and privacy notes

- Business contact details and website content are sent to the OpenAI and Firecrawl APIs; ensure your usage policies permit this.
- Never store API keys inside workflow nodes; always use n8n credentials.
- Gmail drafts are never auto-sent; review each draft before sending.
- Secure your n8n form URL if you do not want public access to the lead search trigger.

## Repository-safe export

This version contains no saved Apify, Firecrawl, OpenAI, Google Sheets, Gmail, or Slack credentials; no private spreadsheet IDs, Slack channel IDs, or n8n instance metadata; and no generated webhook IDs. Users must provide their own configuration after importing it.
