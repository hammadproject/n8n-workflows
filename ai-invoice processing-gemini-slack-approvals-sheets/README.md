# AI Invoice Processing with Gemini, Slack Approvals & Google Sheets

An n8n workflow that watches a Gmail inbox and a Google Drive folder for incoming invoices and receipts, uses Google Gemini to extract structured data from PDFs and images, validates totals, deduplicates by invoice number, classifies the expense category, routes invoices for human approval in Slack when they exceed $500, and logs every result to a Google Sheets ledger. An error trigger posts a Slack alert if the workflow itself fails.

## Features

- Monitors Gmail every minute for unread emails with invoice or receipt attachments.
- Monitors a Google Drive folder for newly created invoice files.
- Accepts only PDF and image files; unsupported formats are skipped with a Slack notification.
- Uses Google Gemini to extract vendor name, invoice number, dates, line items, subtotal, tax, total, currency, and payment terms.
- Validates extracted data: missing required fields or totals that do not add up trigger a Slack review alert.
- Deduplicates by invoice number; repeat invoices are skipped with a Slack notification.
- Uses Gemini to classify each invoice into an expense category: Software, Utilities, Contractor, Office Supplies, Travel, or Other.
- Invoices up to $500 are logged automatically as Approved.
- Invoices over $500 are logged as Pending Review and sent to Slack with Approve and Reject buttons.
- Clicking Approve or Reject in Slack updates the ledger row to Approved or Rejected; rows with no response after three days are marked No Response.
- An Error Trigger posts a Slack alert if any part of the workflow fails.

## Architecture

```text
Intake (runs every minute)
  Gmail Trigger (unread emails with invoice/receipt attachments)
    -> Email Attachment Is PDF or Image?
      -> [Yes] Extract Invoice Data (Email) -> Tag Source Email
      -> [No]  Mark Unsupported Email As Read -> Notify Unsupported File Type (Slack)

  Google Drive Trigger (new files in watched folder)
    -> Download Drive File
    -> Drive File Is PDF or Image?
      -> [Yes] Extract Invoice Data (Drive) -> Tag Source Drive
      -> [No]  Notify Unsupported File Type (Slack)

AI extraction & validation
  Combine Extraction Results (Merge)
    -> Parse & Validate Extraction
    -> Extraction Valid?
      -> [No]  Notify Extraction Needs Review (Slack)
      -> [Yes] Check Duplicate Invoice (Google Sheets)

Duplicate check & categorisation
  Duplicate Found?
    -> [Yes] Notify Duplicate Invoice (Slack)
    -> [No]  Classify Expense Category (Gemini)
             -> Determine Approval Route

Approval routing & ledger
  Needs Approval? (total > $500)
    -> [Yes] Mark Pending Review -> Log Pending Invoice -> Send Approval to Slack (Approve/Reject)
                                                          -> Update row to Approved / Rejected
    -> [No]  Mark Auto Approved  -> Log Invoice To Ledger

Failure alerts
  On Workflow Error -> Slack alert
```

## Google Sheets structure

Create a spreadsheet with a tab named `invoices` and these first-row headers.

```text
vendor_name | invoice_number | invoice_date | due_date | subtotal | tax | total | currency | category | source | needsApproval | status
```

## Requirements

- An n8n Cloud or self-hosted n8n instance.
- A Gmail account with OAuth2 access.
- A Google Drive account with OAuth2 access.
- A Google Sheets account with OAuth2 access.
- A Google Gemini API key.
- A Slack workspace with a bot app that has the `chat:write` scope, Interactivity enabled, and a signing secret.

## Setup

1. Import the workflow JSON into n8n.
2. Connect your Gmail OAuth2 credential to the **Watch Invoice Emails** trigger and the **Mark Email As Read** and **Mark Unsupported Email As Read** nodes.
3. Connect your Google Drive OAuth2 credential to the **Watch Invoice Drive Folder** trigger and the **Download Drive File** node.
4. Connect your Google Gemini credential to the **Extract Invoice Data (Email)**, **Extract Invoice Data (Drive)**, and **Classify Expense Category** nodes.
5. Create a Google Sheet with an `invoices` tab using the headers listed above. Select it in all four Google Sheets nodes.
6. Create a Slack app with the `chat:write` scope. Enable Interactivity and set the Request URL to `https://<your-n8n-domain>/webhook-waiting-slack`. Add the bot token and signing secret to the Slack credential in n8n.
7. Connect the Slack credential to every Slack node and invite the bot to your target channel.
8. Select your Slack channel in every Slack node.
9. In workflow **Settings**, set **Error Workflow** to this workflow so the **On Workflow Error** node fires on failure.
10. Activate the workflow.

## Common customizations

- Change the `$500` approval threshold in **Determine Approval Route**.
- Edit the allowed expense categories in **Classify Expense Category**.
- Adjust the Gmail search query in **Watch Invoice Emails** to broaden or narrow the trigger.
- Add additional Gemini fields to the extraction prompt if your invoices contain extra data.

## Security and privacy notes

- Invoice content is passed to the Gemini API; ensure your Gemini usage policy permits this.
- No invoice content, credentials, or personal data should be stored inside workflow nodes; always use n8n credentials.
- The Slack signing secret must be kept confidential; rotate it if it is ever exposed.
- Review Google and Slack API usage limits and billing before running the workflow continuously.

## Repository-safe export

This version contains no saved Gmail, Google Drive, Google Sheets, Gemini, or Slack credentials; no private spreadsheet IDs, Google Drive folder IDs, or Slack channel IDs; and no n8n instance metadata or generated webhook IDs. Users must provide their own configuration after importing it.
