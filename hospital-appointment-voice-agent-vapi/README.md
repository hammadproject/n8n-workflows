# Hospital Appointment Voice Agent

An n8n backend for a Vapi Conversational AI hospital agent. It exposes four webhook tools that let the voice agent book appointments, check doctor and slot availability, manage existing appointments, and request a human callback. Records are stored in Google Sheets and appointment notifications are sent through Gmail.

## Features

- Books new patient appointments with auto-generated appointment IDs.
- Checks doctor and department availability against existing appointments before confirming.
- Modifies and cancels existing appointments by appointment ID and phone number.
- Logs human callback requests and notifies hospital staff.
- Sends formatted Gmail confirmation emails to patients and staff.
- Returns structured JSON responses formatted for Vapi's function-call protocol.
- Includes detailed sticky notes for setup and maintenance.

## Process flow

```text
Caller
  -> Vapi Conversational AI agent
  -> Matching Vapi function call
  -> n8n production webhook
  -> Validation and business logic
  -> Google Sheets and/or Gmail
  -> JSON response
  -> Vapi speaks the result
```

## Vapi tool mapping

| Tool | Method | n8n webhook path | Purpose |
| --- | --- | --- | --- |
| `book_appointment` | POST | `/hospital_book_appointment` | Creates a new appointment, saves it to the sheet, and emails the patient. |
| `check_availability` | POST | `/hospital_check_availability` | Checks whether a doctor and slot are free on the requested date and time. |
| `manage_appointment` | POST | `/hospital_manage_appointment` | Modifies or cancels an appointment by ID and phone number. |
| `request_callback` | POST | `/hospital_request_callback` | Logs an escalation and alerts hospital staff. |

## Requirements

- An n8n Cloud or self-hosted n8n instance with public HTTPS webhook access.
- A Vapi account with a Conversational AI assistant and Function Calls configured.
- A Google account with Google Sheets and Gmail access.
- One Google spreadsheet containing the tab described below.

## Google Sheets structure

Create a spreadsheet with this exact tab name and first-row headers.

### `Appointments`

```text
appointment_id | patient_name | phone_number | department | doctor_name | appointment_type | appointment_date | appointment_time | reason_for_visit | status | conversation_id | created_at
```

## Setup

1. Import [`workflow.json`](./workflow.json) into n8n.
2. Replace `YOUR_GOOGLE_SHEET_ID` in every Google Sheets node with your spreadsheet ID.
3. Connect your Google Sheets OAuth2 credential to all Google Sheets nodes.
4. Replace the placeholder hospital/staff email in every Gmail node with your notification address.
5. Connect your Gmail OAuth2 credential to all Gmail nodes.
6. Activate the workflow so its production webhook URLs become available.
7. Create four Function Calls in Vapi and paste the matching production URL into each function.
8. Configure the request-body parameters in Vapi to match the fields expected by each webhook.
9. Test every path with sample data before accepting live calls.

## Appointment ID format

Appointment IDs are generated automatically using the pattern `APT-YYYYMMDDHHMMSS-<executionId>`. This ensures each booking has a unique, human-readable reference that the patient can use to manage their appointment later.

## Common customizations

- Add opening-hours or same-day booking validation.
- Extend the schema with insurance details, referral codes, or language preference.
- Customize Gmail confirmation and notification templates.
- Change Google Sheet tab names and field mappings.
- Add SMS reminders, calendar invites, or payment integrations.
- Replace Gmail with another notification channel.

## Production and privacy notes

- Secure each public webhook, for example with a shared secret header configured in both n8n and Vapi.
- Restrict access to the spreadsheet and Gmail credential.
- Define data-retention rules for patient names, phone numbers, and medical details.
- Confirm compliance with applicable health data privacy regulations in your jurisdiction before going live.
- Avoid logging webhook payloads beyond what is operationally required.

## Repository-safe export

This version contains no saved credentials, personal staff email, private spreadsheet URL/ID, n8n instance metadata, or generated webhook IDs. Users must provide their own configuration after importing it.
