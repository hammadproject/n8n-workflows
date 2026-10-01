# Real Estate Voice Agent

An n8n backend for a Retell Conversational AI real estate agent. It exposes five webhook tools that let the voice agent search property listings, log new listings for review, book property viewings, retrieve property details, and escalate to a human agent. Records are stored in Google Sheets and operational notifications are sent through Gmail.

## Features

- Searches listings by location, property type, number of bedrooms, budget, and listing type.
- Logs new property submissions as pending review and notifies the office for approval.
- Books property viewings after checking for slot conflicts.
- Reads full property details including price, size, bedrooms, and description.
- Logs unresolved requests for human follow-up and alerts the office.
- Sends formatted Gmail notifications to estate agents.
- Returns concise JSON responses that Retell can speak to callers.
- Includes detailed sticky notes for setup and maintenance.

## Process flow

```text
Caller
  -> Retell Conversational AI agent
  -> Matching Retell custom tool
  -> n8n production webhook
  -> Validation and business logic
  -> Google Sheets and/or Gmail
  -> JSON response
  -> Retell speaks the result
```

## Retell tool mapping

| Tool | Method | n8n webhook path | Purpose |
| --- | --- | --- | --- |
| `check_property_availability` | POST | `/check_property_availability` | Searches listings by criteria and returns matching properties. |
| `list_property` | POST | `/list_property` | Saves a new listing as pending review and emails the office. |
| `book_property_visit` | POST | `/book_property_visit` | Checks slot availability, logs the viewing, and emails the agent. |
| `get_property_details` | POST | `/get_property_details` | Reads and returns full details for a specific listing. |
| `request_human_callback` | POST | `/request_human_callback` | Logs an escalation and alerts the office. |

## Requirements

- An n8n Cloud or self-hosted n8n instance with public HTTPS webhook access.
- A Retell account with a Conversational AI agent and Custom Tools configured.
- A Google account with Google Sheets and Gmail access.
- One Google spreadsheet containing the tabs described below.

## Google Sheets structure

Create a spreadsheet with these exact tab names and first-row headers.

### `Listings`

```text
property_id | address | location | property_type | bedrooms | bathrooms | price | listing_type | size_sqft | description | status | agent_email | created_at
```

### `Viewings`

```text
viewing_id | property_id | customer_name | phone_number | viewing_date | viewing_time | agent_email | conversation_id | created_at
```

### `Escalations`

```text
customer_name | phone_number | reason | conversation_id | created_at
```

## Setup

1. Import [`workflow.json`](./workflow.json) into n8n.
2. Replace `YOUR_GOOGLE_SHEET_ID` in every Google Sheets node with your spreadsheet ID.
3. Connect your Google Sheets OAuth2 credential to all Google Sheets nodes.
4. Replace the placeholder office/agent email in every Gmail node with your notification address.
5. Connect your Gmail OAuth2 credential to all Gmail nodes.
6. Activate the workflow so its production webhook URLs become available.
7. Create five Custom Tools in Retell and paste the matching production URL into each tool.
8. Configure the request-body parameters in Retell to match the fields expected by each webhook.
9. Test every path with sample data before accepting live calls.

## Common customizations

- Add opening-hours validation or same-day booking rules.
- Extend the listing schema with additional fields such as garage, garden, or floor number.
- Customize agent responses and Gmail notification templates.
- Change Google Sheet tab names and field mappings.
- Add payment, CRM, calendar, or SMS integrations.
- Replace Gmail with another notification channel.

## Production and privacy notes

- Secure each public webhook, for example with a shared secret header configured in both n8n and Retell.
- Restrict access to the spreadsheet and Gmail credential.
- Define retention rules for names, phone numbers, and conversation IDs.
- Avoid logging webhook payloads longer than required.
- Confirm applicable consent, privacy, and call-recording requirements in your jurisdiction.

## Repository-safe export

This version contains no saved credentials, personal agent email, private spreadsheet URL/ID, n8n instance metadata, or generated webhook IDs. Users must provide their own configuration after importing it.
