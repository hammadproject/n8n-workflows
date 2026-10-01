# AI Restaurant Voice Assistant

An n8n backend for an ElevenLabs Conversational AI restaurant agent. It exposes five webhook tools that let the voice agent check availability, create and manage reservations, place pickup or delivery orders, and request a human callback. Records are stored in Google Sheets and operational notifications are sent through Gmail.

## Features

- Checks availability against a configurable venue capacity.
- Creates, modifies, and cancels reservations.
- Records pickup and delivery orders.
- Logs unresolved requests for human follow-up.
- Sends formatted Gmail notifications to restaurant staff.
- Returns concise JSON responses that ElevenLabs can speak to callers.
- Includes detailed sticky notes for setup and maintenance.

## Process flow

```text
Caller
  -> ElevenLabs Conversational AI agent
  -> Matching ElevenLabs webhook tool
  -> n8n production webhook
  -> Validation and business logic
  -> Google Sheets and/or Gmail
  -> JSON response
  -> ElevenLabs speaks the result
```

## ElevenLabs tool mapping

| Tool | Method | n8n webhook path | Purpose |
| --- | --- | --- | --- |
| `make_reservation` | POST | `/restaurant_reservation` | Creates a reservation and confirms it. |
| `check_availability` | POST | `/check_availability` | Checks the remaining capacity for a date and time. |
| `place_order` | POST | `/restaurant_order` | Records a pickup or delivery order. |
| `manage_reservation` | POST | `/manage_reservation` | Modifies or cancels an existing reservation. |
| `request_human_callback` | POST | `/request_human_callback` | Logs an escalation and alerts the manager. |

## Requirements

- An n8n Cloud or self-hosted n8n instance with public HTTPS webhook access.
- An ElevenLabs account with a Conversational AI agent and Webhook/Server Tools.
- A Google account with Google Sheets and Gmail access.
- One Google spreadsheet containing the tabs described below.

## Google Sheets structure

Create a spreadsheet with these exact tab names and first-row headers.

### `Reservations`

```text
customer_name | phone_number | party_size | reservation_date | reservation_time | special_requests | conversation_id | created_at
```

### `orders`

```text
customer_name | phone_number | items | order_type | delivery_address | conversation_id | created_at
```

### `Escalations`

```text
customer_name | phone_number | reason | conversation_id | created_at
```

## Setup

1. Import [`workflow.json`](./workflow.json) into n8n.
2. Replace `YOUR_GOOGLE_SHEET_ID` in every Google Sheets node with your spreadsheet ID.
3. Connect your Google Sheets OAuth2 credential to all Google Sheets nodes.
4. Replace `manager@example.com` in every Gmail node with the staff notification address.
5. Connect your Gmail OAuth2 credential to all Gmail nodes.
6. Review the `CAPACITY = 40` value in the **Check Capacity** code node.
7. Activate the workflow so its production webhook URLs become available.
8. Create five Webhook/Server Tools in ElevenLabs and paste the matching production URL into each tool.
9. Configure the request-body parameters in ElevenLabs to match the fields expected by each webhook.
10. Test every path with sample data before accepting live calls.

## Common customizations

- Change restaurant capacity and booking-slot rules.
- Add opening-hours or same-day booking validation.
- Customize agent responses and Gmail templates.
- Change Google Sheet tab names and field mappings.
- Add payment, POS, CRM, SMS, or calendar integrations.
- Replace Gmail with another notification channel.

## Production and privacy notes

- Secure each public webhook, for example with a shared secret header configured in both n8n and ElevenLabs.
- Restrict access to the spreadsheet and Gmail credential.
- Define retention rules for names, phone numbers, addresses, and conversation IDs.
- Avoid logging webhook payloads longer than required.
- Confirm applicable consent, privacy, and call-recording requirements in your jurisdiction.

## Repository-safe export

This version contains no saved credentials, personal manager email, private spreadsheet URL/ID, n8n instance metadata, or generated webhook IDs. Users must provide their own configuration after importing it.

