# Enquiry Capture & Response Automation

An n8n workflow that accepts website contact form submissions via a webhook, normalises field names from any major WordPress form plugin, filters out spam and incomplete submissions, assigns a regional owner by postcode, scores the enquiry by priority, saves the lead to a Google Sheets tracker before any email is attempted, sends the customer a branded acknowledgement with three qualification questions, and notifies the assigned owner by email. The website receives a clean JSON response for every outcome.

## Features

- Accepts POST submissions from Contact Form 7, WPForms, Gravity Forms, Elementor, and Fluent Forms without any per-plugin configuration.
- Normalises all field names into a single clean enquiry object regardless of how each plugin labels them.
- Blocks spam via honeypot field detection, invalid email validation, and empty submission checks.
- Generates a short human-readable reference number for every enquiry (e.g. `CE-20260923-4821`).
- Routes enquiries to a regional owner based on UK postcode prefix, with a configurable fallback for unmatched postcodes.
- Scores each enquiry Low, Normal, High, or Urgent using rule-based logic: company supplied, phone supplied, message detail, urgent keywords, commercial scope, and postcode presence.
- Saves the enquiry row to Google Sheets before sending any email, so no lead is ever lost to a mail failure.
- Sends the customer a branded HTML acknowledgement email with their reference number and three qualification questions.
- Sends the assigned owner an internal email with the full enquiry details and priority.
- Supports three safety modes: Preview (log only, no email), Test (redirect all emails to a test inbox), and Live.
- Returns a predictable JSON result to the website with `accepted`, `preview`, or `rejected` status.

## Architecture

```text
1. Capture and standardise
  Webhook (POST /cleaning-enquiry)
    -> Configure Enquiry Pipeline (load all settings)
    -> Parse Form Fields (normalise field names, detect postcode, spam check)

2. Validation
  Check If Real Enquiry
    -> [Spam/incomplete] Handle Spam or Incomplete -> Build Website Response -> Send Webhook Response
    -> [Valid] Assign Enquiry Owner (postcode routing)

3. Route, score, and personalise
  Score Enquiry Priority
    -> Compose First Reply (build customer HTML email + owner HTML email)

4. Save to tracker
  Prepare Sheet Row
    -> Append Enquiry to Sheets (Google Sheets — written before any email)

5. Safety gate
  Check Sending Condition
    -> [Preview mode] Preview Mode: No Email Sent -> Build Website Response -> Send Webhook Response
    -> [Test/Live]    Send Reply to Customer (Gmail)
                        -> Verify Owner Email Exists
                          -> [Yes] Notify Owner via Email (Gmail)
                          -> [No]  No Owner Email Defined
                        -> Build Website Response -> Send Webhook Response
```

## Google Sheets structure

Create a spreadsheet with a tab named `Enquiries` and these first-row headers.

```text
Ref | Received | Stage | Owner | Owner email | Company | Contact | Email | Phone | Postcode | Area | Premises | What they said | Sq ft | Frequency | Access | Last touch | Chases sent | Next action | Notes
```

## Configuration

All settings live in the **Configure Enquiry Pipeline** Code node. Open it to change:

| Setting | Description |
| --- | --- |
| `TRACKER_SHEET_ID` | Google Sheet ID from the URL |
| `AREAS` | Array of regional owners with name, email, and postcode prefixes |
| `FALLBACK` | Owner used when no postcode rule matches |
| `COMPANY_NAME` | Shown in customer emails |
| `REPLY_TO` | Address for customer replies |
| `QUESTIONS` | The three qualification questions sent to the customer |
| `TEST_RUN` | Set to `false` for live mode |
| `TEST_EMAIL` | Redirect address for test mode |

## Requirements

- An n8n Cloud or self-hosted n8n instance.
- A Google account with Google Sheets OAuth2 access.
- A Gmail account with OAuth2 access.
- A website contact form that can POST to a webhook URL.

## Setup

1. Import the workflow JSON into n8n.
2. Connect your Google Sheets OAuth2 credential to the **Append Enquiry to Sheets** node.
3. Connect your Gmail OAuth2 credential to the **Send Reply to Customer** and **Notify Owner via Email** nodes.
4. Open **Configure Enquiry Pipeline** and set `TRACKER_SHEET_ID` to your Google Sheet ID.
5. Replace the sample `AREAS` entries with your real team members, their email addresses, and the postcode prefixes they cover.
6. Update `COMPANY_NAME`, `REPLY_TO`, `PHONE`, and `WEBSITE` with your company details.
7. Edit the `QUESTIONS` array to match the information you need from enquirers.
8. Set `TEST_EMAIL` to your own address and keep `TEST_RUN = true` for initial testing.
9. Submit one valid form, one form with a missing email, and one form with the honeypot field filled in to verify all three branches.
10. Once verified, set `TEST_RUN = false` in **Configure Enquiry Pipeline** to go live.
11. Activate the workflow and copy the production webhook URL into your website form.

## Priority scoring

| Score | Priority | Criteria |
| --- | --- | --- |
| ≥ 6 | Urgent | Combination of urgent keywords, commercial scope, detailed brief |
| ≥ 4 | High | Multiple positive signals present |
| ≥ 2 | Normal | Company or phone supplied, reasonable message |
| < 2 | Low | Minimal information, missing postcode |

## Common customizations

- Add or remove regional owners and their postcode prefixes in the `AREAS` array.
- Change the three qualification questions in `QUESTIONS`.
- Adjust the priority scoring thresholds in **Score Enquiry Priority**.
- Replace Gmail with an SMTP node if you use a different email provider.
- Add a Slack or Teams notification for Urgent priority enquiries.
- Build a follow-up chase branch triggered on a schedule to re-contact leads that have not replied.

## Security and privacy notes

- Customer email addresses and enquiry content are stored in Google Sheets; ensure your data handling complies with applicable privacy regulations.
- The webhook endpoint is public; consider adding IP allowlisting or a secret header check if your form supports it.
- Never store credentials or personal data inside workflow nodes; always use n8n credentials.
- Keep `TEST_RUN = true` and a real `TEST_EMAIL` until you have verified every branch.

## Repository-safe export

This version contains no saved Google Sheets or Gmail credentials; no private spreadsheet IDs, real email addresses, or company-specific data; and no n8n instance metadata or generated webhook IDs. Users must provide their own configuration after importing it.
