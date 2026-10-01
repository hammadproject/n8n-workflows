# AI Cold Outreach with Telegram Approval

An n8n workflow that monitors a Google Sheets prospect list, uses Groq to write a personalised cold email for each new row, saves a draft to Gmail, sends it automatically when flagged, and monitors the inbox every two minutes for replies. Groq classifies each reply, logs it to the sheet, and sends a Telegram notification with a ready-to-send response draft that you approve or reject from Telegram.

## Features

- Triggers on every new row added to a Google Sheets prospect list.
- Validates each prospect: requires a unique alphanumeric Prospect ID, a real email address, a company name, and a service offer before proceeding.
- Skips rows that already have a Status value set, preventing duplicate sends.
- Uses Groq to write a short, natural cold email under 110 words tailored to the prospect's company, relevant observation, service offer, proof point, and booking link.
- Validates the generated draft: rejects missing fields and emails over the length limit.
- Saves the outreach copy and status back to Google Sheets.
- If the row has `Auto Send` set to yes, sends the email immediately via Gmail and records the sent status.
- If `Auto Send` is no, saves a Gmail draft for manual review and records the draft status.
- Polls the Gmail inbox every two minutes for replies from active prospects.
- Uses Groq to classify each reply as interested, not interested, or a question, and drafts an appropriate response.
- Logs every reply and its classification to Google Sheets.
- Sends a Telegram notification for each reply with the full message and classification.
- Sends a Telegram message when a response draft is ready for review.
- Listens for Telegram button presses to approve a draft send.
- Sends the approved Gmail draft via the Gmail API and records the outcome in Google Sheets.
- Confirms the send in Telegram.

## Architecture

```text
Flow 1 — New prospect (Google Sheets trigger: row added)
  New prospect row
    -> Validate prospect (check ID, email, company, service; skip if Status set)
    -> Groq personalized outreach (write cold email JSON via Groq API)
    -> Check outreach draft (validate subject, message, length)
    -> Save outreach copy (Google Sheets: save draft text + status)
    -> Send approved by row? (is Auto Send = yes?)
      -> [Yes] Send first email (Gmail)
               -> Record sent email (Google Sheets)
      -> [No]  Save Gmail draft
               -> Record unsent draft (Google Sheets)

Flow 2 — Reply monitor (Schedule trigger: every 2 minutes)
  Check inbox every 2 minutes
    -> Read prospect rows (Google Sheets: fetch all active prospects)
    -> Select active prospects (filter by status)
    -> Search prospect replies (Gmail: search for unread replies per prospect)
    -> Choose unseen reply (pick first unseen reply)
    -> Groq classify and respond (classify reply intent, draft response)
    -> Validate reply classification (check output)
    -> Record prospect reply (Google Sheets: log reply + classification)
    -> Notify about reply (Telegram: send reply summary)
    -> Draft response appropriate? (is a response draft warranted?)
      -> [Yes] Create response draft (Gmail)
               -> Save response draft ID (Google Sheets)
               -> Telegram reply draft ready (send notification)
      -> [No]  (no action)

Flow 3 — Telegram approval (Telegram trigger: button press)
  Telegram approval buttons
    -> Read approval rows (Google Sheets: find matching draft)
    -> Authorize draft send (validate the approval)
    -> Send approved Gmail draft (Gmail API: send draft)
    -> Record approved reply sent (Google Sheets)
    -> Confirm send in Telegram
```

## Google Sheets structure

Create a spreadsheet with a single tab. Use these column headers.

```text
Prospect ID | Company | Contact Name | Email | Website | Service Offer | Relevant Observation | Proof Point | Booking Link | Auto Send | Status | Draft Subject | Draft Message | Reply | Reply Classification | Response Draft ID | Response Sent At
```

| Column | Purpose |
| --- | --- |
| Prospect ID | Unique alphanumeric ID (1–40 characters, letters, numbers, hyphens, underscores) |
| Auto Send | Set to `yes` or `true` to send immediately; leave blank to save as a Gmail draft |
| Status | Left blank on new rows; filled by the workflow (Draft for review / Ready to send / Sent / Reply received) |
| Relevant Observation | Optional context about a post or detail to reference naturally in the email |
| Proof Point | Optional social proof to include in the email |
| Booking Link | Optional link appended to the email |

## Requirements

- An n8n Cloud or self-hosted n8n instance.
- A Google account with Google Sheets OAuth2 access.
- A Gmail account with OAuth2 access.
- A Groq API key (configured as an HTTP Header Auth credential with `Authorization: Bearer YOUR_KEY`).
- A Telegram bot token and your Telegram chat ID.

## Setup

1. Import the workflow JSON into n8n.
2. Connect your Google Sheets OAuth2 credential to the **New prospect row**, **Save outreach copy**, **Record sent email**, **Record unsent draft**, **Read prospect rows**, **Record prospect reply**, **Save response draft ID**, **Read approval rows**, and **Record approved reply sent** nodes.
3. Connect your Gmail OAuth2 credential to the **Send first email**, **Save Gmail draft**, **Search prospect replies**, **Create response draft**, and **Send approved Gmail draft** nodes.
4. Connect your Groq HTTP Header Auth credential to the **Groq personalized outreach** and **Groq classify and respond** nodes.
5. Connect your Telegram credential to the **Notify about reply**, **Telegram reply draft ready**, **Telegram approval buttons**, and **Confirm send in Telegram** nodes.
6. Open every Google Sheets node and select your spreadsheet and the correct tab.
7. Open every Telegram node and set your target chat ID.
8. Test with a single prospect row with `Auto Send` blank before activating.
9. Activate the workflow.

## Common customizations

- Adjust the Groq prompt in **Groq personalized outreach** to change tone, length limit, or signature.
- Edit the reply classification categories in **Groq classify and respond**.
- Change the polling interval on **Check inbox every 2 minutes** to suit your reply volume.
- Add a Slack notification alongside the Telegram alert for team visibility.
- Extend **Validate prospect** to enforce additional required fields.

## Security and privacy notes

- Prospect names, emails, and observations are sent to the Groq API; ensure your Groq usage policy permits this.
- Never store API keys or personal data inside workflow nodes; always use n8n credentials.
- Rotate your Telegram bot token if it is ever exposed.
- Ensure your outreach emails comply with applicable anti-spam and data protection regulations (e.g., CAN-SPAM, GDPR).

## Repository-safe export

This version contains no saved Google Sheets, Gmail, Groq, or Telegram credentials; no private spreadsheet IDs, email addresses, or Telegram chat IDs; and no n8n instance metadata or generated webhook IDs. Users must provide their own configuration after importing it.
