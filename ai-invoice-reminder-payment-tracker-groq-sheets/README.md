# AI Invoice Reminder & Payment Tracker

An n8n workflow that runs a daily check on a Google Sheets invoice ledger, uses Groq to generate personalised payment reminder emails, sends them via SMTP, and logs every action to an activity log tab. A second branch accepts incoming payment notifications via a webhook and marks invoices as paid in the sheet. A daily HTML summary report is emailed to the finance team.

## Features

- Runs a scheduled daily check at 09:00 to find unpaid and overdue invoices.
- Smart reminder timing prevents over-contacting: each reminder type enforces a minimum gap since the last reminder.
- Five reminder stages: upcoming reminder, due today, first reminder, second reminder, and final notice.
- Uses Groq to write a professional, context-aware email body for each invoice.
- Sends HTML reminder emails with a styled invoice details table and a Pay Invoice button.
- Updates the invoice row in Google Sheets with the latest reminder date, count, and type after each send.
- Appends a full activity log entry to a separate Activity Log tab for audit purposes.
- Sends an HTML daily summary report to the finance team email showing total reminders sent, outstanding amounts, and a per-client breakdown.
- Accepts a POST webhook at `/invoice-paid` to mark any invoice as paid instantly and record the payment date, amount, and method.

## Architecture

```text
Flow 1 — Daily reminder (Schedule trigger at 09:00)
  Schedule Daily Check
    -> Fetch Pending Invoices (Google Sheets)
    -> Calculate Reminder Logic (filter paid, calculate days overdue, assign reminder type)
    -> Prepare AI Prompt
    -> Generate Email with Groq
    -> Format Email (HTML with invoice table + Pay Invoice button)
    -> Send Email Reminder (SMTP)
    -> Update Reminder Status (Google Sheets)
    -> Create Activity Log entry
    -> Save to Activity Log (Google Sheets)
    -> Generate Daily Summary
    -> Send Summary to Finance Team (SMTP)

Flow 2 — Payment received (Webhook trigger)
  Webhook: Payment Received (POST /invoice-paid)
    -> Update Payment Status (Google Sheets: mark as paid, record date/amount/method)
    -> Respond OK
```

## Reminder stages

| Stage | Trigger | Urgency |
| --- | --- | --- |
| Upcoming reminder | Due in 1–3 days, no reminder sent yet | Low |
| Due today | Due today, last reminder > 1 day ago | Medium |
| First reminder | 1–14 days overdue, last reminder > 7 days ago | Medium |
| Second reminder | 15–30 days overdue, last reminder > 5 days ago | High |
| Final notice | > 30 days overdue, last reminder > 3 days ago | Critical |

## Google Sheets structure

Create a spreadsheet with two tabs.

**Invoices tab** (`Invoices`):

```text
invoice_id | client_name | client_email | invoice_number | invoice_amount | currency | issue_date | due_date | payment_status | payment_date | payment_amount | payment_method | last_reminder_sent | reminder_count | last_reminder_type
```

**Activity Log tab** (`Activity Log`):

```text
timestamp | invoice_id | invoice_number | client_name | client_email | amount | currency | reminder_type | urgency_level | days_overdue | status
```

## Webhook payload — mark invoice as paid

Send a `POST` request to the webhook URL with a JSON body.

```json
{
  "invoice_number": "INV-2025-001",
  "amount": 1500.00,
  "payment_method": "bank_transfer"
}
```

## Requirements

- An n8n Cloud or self-hosted n8n instance.
- A Google account with Google Sheets OAuth2 access.
- A Groq API key.
- An SMTP credential (e.g., Gmail SMTP, SendGrid, or Mailgun) for sending emails.

## Setup

1. Import the workflow JSON into n8n.
2. Connect your Google Sheets OAuth2 credential to the **Fetch Pending Invoices**, **Update Reminder Status**, **Save to Activity Log**, and **Update Payment Status** nodes.
3. Connect your Groq credential to the **Generate Email with Groq** HTTP Request node.
4. Connect your SMTP credential to the **Send Email Reminder** and **Send Summary to Finance Team** nodes.
5. Open the **Fetch Pending Invoices** and **Update Reminder Status** nodes and select your Google Sheet and the `Invoices` tab.
6. Open the **Save to Activity Log** node and select the `Activity Log` tab.
7. Replace `your-finance@example.com` in the **Send Email Reminder** and **Send Summary to Finance Team** nodes with your real finance email address.
8. Replace `https://your-payment-page.example.com/` in the **Format Email** node with your actual payment page URL.
9. Adjust the daily schedule time in **Schedule Daily Check** if needed (default: 09:22 UTC).
10. Test with a non-production invoice row before activating the workflow.
11. Activate the workflow.

## Common customizations

- Edit the reminder timing thresholds (days overdue, days since last reminder) in **Calculate Reminder Logic**.
- Adjust the Groq prompt in **Prepare AI Prompt** to change tone, signature, or instructions.
- Add a Slack or Teams notification for critical urgency invoices.
- Extend the **Update Payment Status** webhook to trigger downstream actions such as generating a receipt or updating a CRM.
- Add a second sheet tab for clients with customized payment terms.

## Security and privacy notes

- Client email addresses and invoice amounts are sent to the Groq API; ensure your Groq usage policy permits this.
- Never store API keys or personal data inside workflow nodes; always use n8n credentials.
- Secure the `/invoice-paid` webhook endpoint before production use; consider adding a secret header for verification.
- Ensure your reminder emails comply with applicable data protection and communication regulations.

## Repository-safe export

This version contains no saved Google Sheets, Groq, or SMTP credentials; no private spreadsheet IDs, email addresses, or payment page URLs; and no n8n instance metadata or generated webhook IDs. Users must provide their own configuration after importing it.
