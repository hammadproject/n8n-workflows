# AI Business Inbox Manager

An n8n workflow that watches a Gmail inbox, uses Groq to classify every incoming email by department, type, and priority, applies a matching set of Gmail labels, sends an instant Telegram alert for urgent messages, and optionally creates a reply draft. No email is ever sent automatically; every generated draft stays in Gmail Drafts for human review.

## Features

- Polls Gmail every minute for new messages.
- Classifies each email by department, type, and priority using Groq.
- Applies a structured set of AI-managed Gmail labels automatically.
- Adds an `AI/Needs Review` label when classification confidence falls below 0.75.
- Sends a real-time Telegram alert for urgent emails with a direct Gmail link.
- Creates a professional draft reply when a response is warranted.
- Treats all email content as untrusted data; the AI never acts on instructions found inside messages.
- Includes a one-time label setup branch to create all required Gmail labels.

## Architecture

```text
Label setup branch (run once)
  Manual Trigger
    -> Generate required label list
    -> Create Gmail labels

Main processing branch (runs every minute)
  Gmail Trigger
    -> Prepare and clean email body
    -> Classify and draft with Groq
    -> Validate and sanitize AI output
    -> Fetch Gmail label IDs
    -> Resolve label IDs for this email
    -> Apply AI Gmail labels
    -> [If urgent] Send Telegram alert
    -> [If draft needed] Create Gmail draft
```

## Gmail label taxonomy

The one-time setup creates the following labels.

| Label | Values |
| --- | --- |
| `AI/Department/` | Finance, Sales, Support, HR, Operations, General |
| `AI/Type/` | Invoice, Payment, Complaint, Inquiry, Meeting, Job Application, Other |
| `AI/Priority/` | Urgent, Normal |
| `AI/Needs Review` | Applied when confidence < 0.75 |

## Urgency criteria

An email is classified as Urgent only when the model detects a credible deadline within 24 hours, a security incident, a service outage, a legal or regulatory risk, a payment issue blocking operations, or a serious customer escalation. All other emails receive a Normal priority.

## Requirements

- An n8n Cloud or self-hosted n8n instance.
- A Gmail account with OAuth2 access.
- A Groq API key.
- A Telegram bot token and a numeric chat ID for urgent alerts.

## Setup

1. Import [`workflow.json`](./workflow.json) into n8n.
2. Connect your Gmail OAuth2 credential to every Gmail node and the Gmail Trigger.
3. Create a Groq Header Auth credential in n8n with header name `Authorization` and value `Bearer YOUR_GROQ_API_KEY`.
4. Connect the Groq credential to the **Classify and Draft with Groq** HTTP Request node.
5. Create a Telegram API credential using your bot token from `@BotFather`.
6. Connect the Telegram credential to the **Send Telegram Urgent Alert** node.
7. Replace `REPLACE_WITH_TELEGRAM_CHAT_ID` in **Send Telegram Urgent Alert** with your numeric chat ID.
8. Run the **Run Label Setup Once** manual trigger to create the `AI/…` Gmail label hierarchy.
9. Confirm the labels appear in Gmail before proceeding.
10. Test with a routine email, a time-sensitive email, and a reply-needed email.
11. Activate the workflow.

## How to get your Telegram chat ID

1. Open `@BotFather` in Telegram and run `/newbot` to create a new bot.
2. Open your new bot and press **Start**.
3. Ask `@get_id_bot` for your numeric Chat ID.
4. Paste the ID into the **Send Telegram Urgent Alert** node.

## Common customizations

- Edit the allowed categories and reply style in **Classify and Draft with Groq**.
- Change the confidence threshold (default 0.75) for the `AI/Needs Review` label in **Validate AI Output**.
- Adjust the Gmail polling interval in the Gmail Trigger node.
- Replace Telegram with Slack, Teams, or another alerting channel.
- Add downstream actions such as ticket creation, CRM logging, or automated routing.

## Security and privacy notes

- Email content is treated as untrusted data by the Groq prompt.
- The AI classifier never sends email, modifies workflow logic, or exposes credentials.
- Generated drafts must be reviewed and sent manually by a human.
- Keep Groq, Gmail, and Telegram credentials outside version control.
- Review Groq API usage limits and billing before running the workflow continuously.

## Repository-safe export

This version contains no saved Groq, Gmail, or Telegram credentials; no personal email address or Telegram chat ID; and no n8n instance metadata or generated webhook IDs. Users must provide their own configuration after importing it.
