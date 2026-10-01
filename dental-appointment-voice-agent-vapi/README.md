# Dental Appointment Voice Agent

An n8n backend for a Vapi Conversational AI dental receptionist. It exposes four webhook tools that let callers check dentist availability, book appointments, cancel or reschedule existing appointments, and request a callback from clinic staff. Google Sheets stores appointment and callback records, while Gmail sends staff notifications.

## Live demo

[Open the KikoAI live demo](https://kikoai-agent.vercel.app)

## Features

- Checks appointment availability by treatment type, dentist, date, and time.
- Returns verified alternative slots when the requested time is unavailable.
- Books caller-confirmed appointments and generates a short `DEN-####` reference.
- Cancels or reschedules appointments after matching the appointment ID and phone number.
- Records callback requests for insurance, clinical questions, complaints, and other follow-up.
- Sends Gmail notifications to clinic staff.
- Returns structured responses for Vapi's function-call flow.

## Process flow

```text
Caller
  -> Vapi Conversational AI agent
  -> Matching Vapi tool call
  -> n8n production webhook
  -> Validation and scheduling logic
  -> Google Sheets and/or Gmail
  -> JSON response
  -> Vapi speaks the result
```

## Vapi tool mapping

| Tool | Method | n8n webhook path | Purpose |
| --- | --- | --- | --- |
| `check_dental_availability` | POST | `/dental_check_availability` | Checks a requested slot and returns alternatives when needed. |
| `book_dental_appointment` | POST | `/dental_book_appointment` | Creates a caller-confirmed appointment and alerts clinic staff. |
| `manage_dental_appointment` | POST | `/dental_manage_appointment` | Cancels or reschedules an appointment. |
| `request_dental_callback` | POST | `/dental_request_callback` | Records a request for clinic follow-up. |

## Requirements

- An n8n Cloud or self-hosted n8n instance with public HTTPS webhook access.
- A Vapi account with an assistant and tools configured.
- A Google account with Google Sheets and Gmail access.
- One Google spreadsheet containing the tabs described below.

## Google Sheets structure

Create the following tabs with these exact first-row headers.

### `Appointments`

```text
appointment_id | patient_name | phone_number | patient_type | appointment_type | dentist_name | appointment_date | appointment_time | visit_reason | insurance_provider | status | conversation_id | created_at
```

### `Callbacks`

```text
patient_name | phone_number | reason | conversation_id | created_at | status
```

## Setup

1. Import [`Dental-Appointment-Voice-Agent-N8N-Vapi.json`](./Dental-Appointment-Voice-Agent-N8N-Vapi.json) into n8n.
2. Connect your Google Sheets and Gmail credentials to the relevant nodes.
3. Select your spreadsheet and the correct `Appointments` or `Callbacks` tab in every Google Sheets node.
4. Configure the append and update mappings using the headers listed above.
5. Set the clinic notification recipient in all four Gmail nodes.
6. Replace `YOUR_CLINIC_NAME` and `YOUR_AGENT_NAME` in messages.
7. Replace `YOUR_DENTIST_1` and `YOUR_DENTIST_2` in both scheduling nodes, and update their lowercase values in the validation nodes.
8. Populate `CLINIC_SLOTS` in both scheduling nodes with valid times in `HH:mm` format.
9. Populate `BUSINESS_DAYS` in `Find Verified Alternative Slots`, where Sunday is `0` and Saturday is `6`.
10. Activate the workflow and connect the four production webhook URLs to matching Vapi tools.
11. Test availability, booking, appointment management, and callback paths before taking live calls.

## Supported appointment values

The workflow accepts `new_patient` or `existing_patient` for `patient_type`. Supported `appointment_type` values are:

```text
new_patient_exam | routine_checkup | cleaning | tooth_pain_exam | cosmetic_consultation | follow_up | pediatric_visit
```

Booking and appointment changes require `caller_confirmed` to be `true`. Rescheduling also requires `new_date` and `new_time`.

## Common customizations

- Add clinic opening-hours, holidays, and provider-specific schedules.
- Add patient email or SMS confirmations and appointment reminders.
- Extend the appointment types and duration rules.
- Add insurance verification or intake-form automation.
- Replace Google Sheets with a practice-management system or database.

## Production and privacy notes

- Protect public webhooks with authentication shared between Vapi and n8n.
- Restrict access to the spreadsheet and connected credentials.
- Collect only the patient information required for scheduling.
- Define retention and deletion policies for patient and call data.
- Confirm applicable healthcare and privacy requirements before production use.

## Repository-safe export

This template contains no saved credentials, private spreadsheet selection, clinic notification address, pinned execution data, or n8n instance metadata. It requires configuration after import.
