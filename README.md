# n8n Automation Workflows

This repository contains n8n workflows that I have designed and built for practical AI automation and business use cases. Each project combines n8n with external platforms and APIs to create a complete, reusable automation system.

The repository currently includes fifteen workflows.

## 1. AI Restaurant Voice Assistant

An automation backend for an ElevenLabs Conversational AI restaurant agent. It handles table availability, new reservations, reservation modifications and cancellations, pickup or delivery orders, and human callback requests.

The workflow uses Google Sheets to store operational records and Gmail to notify restaurant staff. Five n8n webhook endpoints allow the ElevenLabs agent to perform these actions during live customer calls and return natural-language results to the caller.

**Technologies:** n8n, ElevenLabs Conversational AI, Google Sheets, Gmail, webhooks, and JavaScript.

[View the workflow](./restaurant-voice-assistant%20-%20ElevenLabs/)

## 2. Website RAG Customer Support Chatbot

A retrieval-augmented customer support system that converts website content into a searchable AI knowledge base. Firecrawl discovers pages, n8n cleans and chunks the extracted content, Google Gemini generates embeddings, and Pinecone stores the vectors for semantic retrieval.

A Gemini-powered support agent searches the Pinecone knowledge base before answering questions about services, pricing, policies, FAQs, contact details, and other indexed website information.

**Technologies:** n8n, Firecrawl, Google Gemini, Pinecone, RAG, vector embeddings, and JavaScript.

[View the workflow](./website-rag-customer-support/)

## 3. Real Estate Voice Agent

An automation backend for a Retell Conversational AI real estate agent. It handles property searches by location, type, bedrooms, and budget; new listing submissions pending office review; property viewing bookings; detailed property lookups; and human escalation requests.

The workflow uses Google Sheets to store listings, viewings, and escalations, and Gmail to notify estate agents. Five webhook endpoints allow the Retell agent to perform these actions during live calls.

**Technologies:** n8n, Retell Conversational AI, Google Sheets, Gmail, webhooks, and JavaScript.

[View the workflow](./real-estate-voice-agent-retell/)

## 4. Hospital Appointment Voice Agent

An automation backend for a Vapi Conversational AI hospital agent. It handles new appointment bookings with auto-generated IDs, doctor and slot availability checks, appointment modifications and cancellations, and human callback requests.

The workflow uses Google Sheets to store appointment records and Gmail to send confirmation and notification emails to patients and staff. Four webhook endpoints allow the Vapi agent to perform these actions during live calls.

**Technologies:** n8n, Vapi Conversational AI, Google Sheets, Gmail, webhooks, and JavaScript.

[View the workflow](./hospital-appointment-voice-agent-vapi/)

## 5. AI Business Inbox Manager

An email triage workflow that watches a Gmail inbox, uses Groq to classify each incoming message by department, type, and priority, applies a structured set of Gmail labels, sends a Telegram alert for urgent emails, and creates a draft reply when a response is warranted. No email is ever sent automatically.

**Technologies:** n8n, Gmail, Groq, Telegram, OAuth2, and JavaScript.

[View the workflow](./ai-business-inbox-manager/)

## 6. AI Lead Qualifier & Outreach

A full-funnel lead management workflow that receives inbound leads via a webhook, scores them 1–10 using OpenAI, saves the result to a Google Sheets CRM, sends a category-appropriate outreach email, and posts a Slack notification for hot leads. A daily follow-up job re-contacts hot and warm leads that have not yet replied.

**Technologies:** n8n, OpenAI, Google Sheets, Gmail, Slack, webhooks, and JavaScript.

[View the workflow](./ai-lead-qualifier-outreach/)

## 7. AI Invoice Processing with Gemini, Slack Approvals & Google Sheets

An n8n workflow that watches a Gmail inbox and a Google Drive folder for incoming invoices and receipts, uses Google Gemini to extract structured data from PDFs and images, validates totals, deduplicates by invoice number, classifies the expense category, routes invoices for human approval in Slack when they exceed $500, and logs every result to a Google Sheets ledger. An error trigger posts a Slack alert if the workflow itself fails.

**Technologies:** n8n, Google Gemini, Google Sheets, Google Drive, Gmail, Slack, and JavaScript.

[View the workflow](./ai-invoice%20processing-gemini-slack-approvals-sheets/)

## 8. AI Invoice Reminder & Payment Tracker

An n8n workflow that runs a daily check on a Google Sheets invoice ledger, uses Groq to generate personalised payment reminder emails, sends them via SMTP, and logs every action to an activity log tab. A second branch accepts incoming payment notifications via a webhook and marks invoices as paid in the sheet. A daily HTML summary report is emailed to the finance team.

**Technologies:** n8n, Groq, Google Sheets, SMTP, webhooks, and JavaScript.

[View the workflow](./ai-invoice-reminder-payment-tracker-groq-sheets/)

## 9. Enquiry Capture & Response Automation

An n8n workflow that accepts website contact form submissions via a webhook, normalises field names from any major WordPress form plugin, filters out spam and incomplete submissions, assigns a regional owner by postcode, scores the enquiry by priority, saves the lead to a Google Sheets tracker before any email is attempted, sends the customer a branded acknowledgement with three qualification questions, and notifies the assigned owner by email. The website receives a clean JSON response for every outcome.

**Technologies:** n8n, Google Sheets, Gmail, webhooks, and JavaScript.

[View the workflow](./enquiry-capture-response-automation/)

## 10. AI Cold Outreach with Telegram Approval

An n8n workflow that monitors a Google Sheets prospect list, uses Groq to write a personalised cold email for each new row, saves a Gmail draft or sends immediately based on a per-row flag, and polls the inbox every two minutes for replies. Groq classifies each reply and drafts a response; a Telegram notification delivers the draft with an approve button so you can send it without leaving Telegram.

**Technologies:** n8n, Groq, Google Sheets, Gmail, Telegram, and JavaScript.

[View the workflow](./ai-cold-outreach-automation/)

## 11. Local Business Lead Machine — AI Lead Generation & Outreach

An n8n workflow that scrapes Google Maps via Apify for local businesses, audits each website with HTTP requests and Firecrawl, scores every lead 0–100 using a rule-based engine (no AI), and uses OpenAI to write personalised pitches for hot and warm leads. All leads are saved to a Google Sheets CRM with a run summary posted to Slack. A daily weekday job creates Gmail outreach drafts for approved leads up to a configurable cap.

**Technologies:** n8n, Apify, Firecrawl, OpenAI, Google Sheets, Gmail, Slack, and JavaScript.

[View the workflow](./ai-lead-generation-and-outreach-with-apify-maps/)

## 12. AI LinkedIn Content Automation

An n8n workflow that runs daily, picks a pending topic from a Google Sheets queue, researches it with Firecrawl, generates two distinct LinkedIn and X post drafts with Groq, creates an AI preview image for each via Cloudflare Workers AI, and sends both options to Telegram. Clicking an approve button publishes the chosen post with its image directly to LinkedIn and forwards the X copy to Telegram for manual posting.

**Technologies:** n8n, Firecrawl, Groq, Cloudflare Workers AI, Google Sheets, Telegram, LinkedIn, and JavaScript.

[View the workflow](./ai_linkedin_content_automation.json/)

## 13. Dental Appointment Voice Agent

An n8n backend for a Vapi dental receptionist that checks dentist availability, books appointments, handles cancellations and rescheduling, and records clinic callback requests. Google Sheets stores appointments and callbacks, while Gmail alerts clinic staff.

**Technologies:** n8n, Vapi Conversational AI, Google Sheets, Gmail, webhooks, and JavaScript.

[View the workflow](./dental-appointment-voice-agent-vapi/)

## 14. Home Services Booking Voice Agent

An n8n backend for a Vapi home-services receptionist that checks booking availability, schedules service visits, cancels or reschedules bookings, and records callback requests. Google Sheets stores operational records and Gmail notifies staff.

**Technologies:** n8n, Vapi Conversational AI, Google Sheets, Gmail, webhooks, and JavaScript.

[View the workflow](./home-services-booking-voice-agent-vapi/)

## 15. Travel Agency Voice Agent

An n8n backend for a Vapi travel agent that searches packages, creates bookings, updates or cancels existing bookings, and captures consultant callback requests. Google Sheets stores packages, bookings, leads, and callbacks, while Gmail sends confirmations and alerts.

**Technologies:** n8n, Vapi Conversational AI, Google Sheets, Gmail, webhooks, and JavaScript.

[View the workflow](./travel-agency-voice-agent-vapi/)

## Repository structure

Each workflow has its own folder containing:

- An importable n8n `workflow.json` file.
- A dedicated README with architecture, requirements, configuration, and setup instructions.
- Sanitized placeholders instead of private credentials or account-specific values.

## Using the workflows

1. Open the folder for the workflow you want to use.
2. Read its setup guide and prepare the required external services.
3. Import `workflow.json` into n8n.
4. Connect your own credentials and replace the configuration placeholders.
5. Test the workflow with non-production data before activating it.

## Security

No API keys, OAuth credentials, personal email addresses, private spreadsheet IDs, n8n instance IDs, or production webhook IDs are included in this repository. Users must provide their own credentials and secure all public endpoints before production use.

## Disclaimer

These workflows are provided as reusable examples and starting points. Review the logic, permissions, privacy requirements, API usage, and service costs before using them in a production environment.
