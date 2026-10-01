# Home Services Booking Voice Agent

An n8n backend for a Vapi Conversational AI home-services receptionist. It handles service availability checks, new bookings, cancellations and rescheduling, and staff callback requests. Google Sheets stores bookings and callbacks, while Gmail notifies the service team about customer activity.

## Live demo

[Open the KikoAI live demo](https://kikoai-agent.vercel.app)

## Features

- Checks technician or service-category availability for a requested date and time.
- Creates caller-confirmed service bookings with generated booking IDs.
- Cancels or reschedules bookings after matching the booking ID and customer phone number.
- Stores service addresses, issue descriptions, and optional technician preferences.
- Records callback requests and emails staff for follow-up.
- Returns structured JSON responses suitable for Vapi tools.

## Process flow

```text
Caller
  -> Vapi Conversational AI agent
  -> Matching Vapi tool call
  -> n8n production webhook
  -> Validation and booking logic
  -> Google Sheets and/or Gmail
  -> JSON response
  -> Vapi speaks the result
```

## Vapi tool mapping

| Tool | Method | n8n webhook path | Purpose |
| --- | --- | --- | --- |
| `check_booking_availability` | POST | `/home_services_check_availability` | Checks whether a service slot is available. |
| `book_service` | POST | `/home_services_book_service` | Creates a confirmed service booking and alerts staff. |
| `manage_booking` | POST | `/home_services_manage_booking` | Cancels or reschedules an existing booking. |
| `request_human_callback` | POST | `/home_services_request_callback` | Logs a callback request and notifies staff. |

## Requirements

- An n8n Cloud or self-hosted n8n instance with public HTTPS webhook access.
- A Vapi account with an assistant and tools configured.
- A Google account with Google Sheets and Gmail access.
- One Google spreadsheet containing the tabs described below.

## Google Sheets structure

Create the following tabs with these exact first-row headers.

### `Bookings`

```text
booking_id | customer_name | phone_number | service_address | service_category | technician_name | booking_type | booking_date | booking_time | issue_description | status | conversation_id | created_at
```

### `Callbacks`

```text
customer_name | phone_number | reason | conversation_id | created_at | status
```

## Setup

1. Import [`Home Services Booking Voice Agent - Vapi Template.json`](./Home%20Services%20Booking%20Voice%20Agent%20-%20Vapi%20Template.json) into n8n.
2. Connect your Google Sheets and Gmail credentials to every relevant node.
3. Select your spreadsheet in all Google Sheets nodes and verify the `Bookings` and `Callbacks` tabs.
4. Confirm that append and update mappings match the headers listed above.
5. Enter the staff notification address in each Gmail node.
6. Activate the workflow and copy its four production webhook URLs.
7. Create matching POST tools in Vapi and send JSON request bodies using the expected field names.
8. Test booking, availability, cancellation, rescheduling, and callback behavior before activation.

## Expected request fields

- Availability requires `service_category`, `date`, and `time`; `technician_name` is optional.
- Booking requires `customer_name`, `phone_number`, `service_address`, `service_category`, `date`, `time`, and `caller_confirmed: true`.
- Booking management requires `action`, `booking_id`, `phone_number`, and `caller_confirmed: true`.
- The management `action` must be `cancel` or `reschedule`; rescheduling also requires `new_date` and `new_time`.
- Callback requests can include the customer name, phone number, reason, and Vapi conversation ID.

## Common customizations

- Define business hours, technician schedules, travel zones, and blackout dates.
- Add service durations, pricing, deposits, or emergency surcharges.
- Send customer confirmations through email, SMS, or WhatsApp.
- Connect bookings to Google Calendar or a field-service platform.
- Replace Google Sheets with Airtable, a CRM, or a database.

## Production and privacy notes

- Add authentication to every public webhook and configure the same secret in Vapi.
- Restrict access to customer addresses, phone numbers, and service details.
- Validate coverage areas before accepting a booking.
- Add collision protection if multiple calls may reserve the same slot concurrently.
- Test staff notifications and failure handling before going live.

## Repository-safe export

This template contains no saved credentials, private spreadsheet selection, staff email address, pinned execution data, or n8n instance metadata. It requires configuration after import.
