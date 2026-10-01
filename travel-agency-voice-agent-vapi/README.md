# Travel Agency Voice Agent

An n8n backend for a Vapi Conversational AI travel agent. Callers can search travel packages by destination and budget, book a package, cancel or update an existing booking, or request a consultant callback. Google Sheets stores packages, bookings, leads, and callback requests, while Gmail sends customer confirmations and staff alerts.

## Live demo

[Open the KikoAI live demo](https://kikoai-agent.vercel.app)

## Features

- Searches active travel packages by destination, budget, group size, style, and duration.
- Returns up to three matching packages and an optional upsell.
- Creates caller-confirmed bookings using package prices from the spreadsheet.
- Recalculates totals when the travel date or group size changes.
- Cancels bookings after matching the booking ID and caller phone number.
- Records leads and consultant callback requests for human follow-up.
- Emails booking confirmations to customers and callback alerts to staff.

## Process flow

```text
Caller
  -> Vapi Conversational AI agent
  -> Matching Vapi tool call
  -> n8n production webhook
  -> Validation and travel-package logic
  -> Google Sheets and/or Gmail
  -> JSON response
  -> Vapi speaks the result
```

## Vapi tool mapping

| Tool | Method | n8n webhook path | Purpose |
| --- | --- | --- | --- |
| `search_packages` | POST | `/travel_search_packages` | Finds active packages within the caller's requirements. |
| `book_package` | POST | `/travel_book_package` | Creates a confirmed booking and emails the customer. |
| `request_consultant_call` | POST | `/travel_request_consultant_call` | Stores a lead and callback request, then alerts staff. |
| `manage_booking` | POST | `/travel_manage_booking` | Cancels a booking or changes its date or group size. |

## Requirements

- An n8n Cloud or self-hosted n8n instance with public HTTPS webhook access.
- A Vapi account with an assistant and phone number.
- A Google account with Google Sheets and Gmail access.
- One Google spreadsheet containing the four tabs described below.

## Google Sheets structure

Create these exact tab names and first-row headers.

### `Packages`

```text
package_id | destination | title | duration_days | price_per_person | category | inclusions | best_season | visa_note | addons | active
```

### `Bookings`

```text
booking_id | name | phone_number | email | package_id | package_title | destination | travel_date | group_size | price_per_person | total_price | status | conversation_id | created_at | updated_at
```

### `Leads`

```text
lead_id | name | phone_number | email | destination | budget | group_size | package_id | notes | status | conversation_id | created_at
```

### `Consultant_Calls`

```text
request_id | lead_id | name | phone_number | reason | preferred_time | status | created_at
```

## Setup

1. Import [`Travel agency voice agent with Vapi, Google Sheets and Gmail.json`](./Travel%20agency%20voice%20agent%20with%20Vapi,%20Google%20Sheets%20and%20Gmail.json) into n8n.
2. Create the four spreadsheet tabs and add several active package rows for testing.
3. Connect Google Sheets OAuth2 and Gmail OAuth2 credentials to every relevant node.
4. Replace `YOUR_GOOGLE_SHEET_ID` in all Google Sheets nodes.
5. Replace `YOUR_STAFF_EMAIL@example.com` in both Gmail nodes.
6. Activate the workflow and copy the four production webhook URLs.
7. Create matching POST tools in Vapi and configure their JSON request bodies.
8. Test package search, booking, callback, cancellation, and update paths before taking live calls.

## Expected request fields

- `search_packages`: requires `destination`, total `budget`, and `group_size`; optionally accepts `travel_style` and `duration_days`.
- `book_package`: requires `name`, `phone_number`, `email`, `package_id`, `travel_date`, `group_size`, and `caller_confirmed: true`.
- `request_consultant_call`: requires `name`, `phone_number`, and `preferred_time`; other travel and contact details are optional.
- `manage_booking`: requires `action`, `booking_id`, `phone_number`, and `caller_confirmed: true`; updates also require `new_travel_date` and/or `new_group_size`.

## Operational notes

- Package prices always come from the `Packages` tab, not from caller input.
- The workflow does not process payments or check live seat inventory.
- New callback leads use `unconfirmed` status and consultant calls use `pending` status.
- Staff should update completed callback requests in the `Consultant_Calls` tab.

## Common customizations

- Connect a live inventory, airline, hotel, or tour-provider API.
- Add deposits and payment links after booking.
- Add SMS or WhatsApp confirmations and travel reminders.
- Route leads to a CRM or assign them to specific consultants.
- Add destination, visa, season, or accessibility filters.

## Production and privacy notes

- Protect all public webhooks with shared-secret or header authentication.
- Restrict access to customer contact details and travel plans.
- Validate pricing, availability, cancellation rules, and taxes before collecting payment.
- Add consent and retention policies appropriate to your jurisdiction.

## Repository-safe export

This template contains no saved credentials, private spreadsheet ID, staff email address, pinned execution data, or n8n instance metadata. Users must provide their own configuration after import.
