# AI LinkedIn Content Automation

An n8n workflow that runs daily at 9 AM, pulls a pending topic from a Google Sheets queue, uses Firecrawl to research it, generates two distinct LinkedIn and X post drafts with Groq, creates AI images for each via Cloudflare Workers AI, sends both draft options to Telegram for review, and then publishes the approved post directly to LinkedIn and forwards the X copy to Telegram for manual posting. All topics, drafts, and publishing records are tracked in Google Sheets.

## Features

- Triggers on a daily 9 AM schedule or manually via a Manual Start node.
- Reads the next pending topic from a Google Sheets topic queue.
- Marks the topic as "researching" in the sheet before processing begins.
- Uses Firecrawl to search and scrape relevant web content for the topic.
- Cleans and normalises the research output for the AI prompt.
- Uses Groq to generate two distinct draft angles (A and B) for both LinkedIn and X simultaneously.
- Validates both drafts: checks that subjects and messages are present and within limits.
- Generates a preview image for each draft angle using Cloudflare Workers AI.
- Sends both draft images and post texts to Telegram as inline previews.
- Sends a Telegram message with A / B choice buttons so you can pick which draft to publish.
- Saves both draft texts, images, and the topic status to Google Sheets.
- Listens for Telegram callback button presses (A, B, skip, or regenerate).
- Validates the callback and finds the corresponding sheet row.
- Routes the decision: publish chosen draft, regenerate new drafts, or mark topic as skipped.
- Downloads the selected preview image from Telegram.
- Publishes the chosen post with its image directly to LinkedIn.
- Saves the LinkedIn post URL, published timestamp, and chosen angle to Google Sheets.
- Sends the X copy and the LinkedIn post link to Telegram for manual posting.
- Notifies via Telegram if regeneration is requested and re-queues the topic.
- Marks skipped topics in the sheet and sends a Telegram skip confirmation.

## Architecture

```text
Flow 1 — Research and draft (Schedule trigger: daily 9 AM, or Manual start)
  Daily 9AM / Manual start
    -> Config (set Sheet ID, Telegram chat ID, Cloudflare account ID)
    -> Read topic queue (Google Sheets: find next Pending topic)
    -> Pick one topic
    -> Mark researching (Google Sheets: update status)
    -> Firecrawl search (HTTP Request: research topic)
    -> Clean research (normalise scraped content)
    -> Groq two angles (HTTP Request: generate Draft A and B for LinkedIn + X)
    -> Validate two drafts
    -> [In parallel for A and B]
       Cloudflare image A/B (HTTP Request: generate image)
         -> Binary image A/B (convert to binary)
         -> Preview image A/B (Telegram: send image)
         -> Preview text A/B (Telegram: send draft text)
    -> Save drafts (Google Sheets: save both drafts + status = Pending Review)
    -> Choose A or B (Telegram: send choice buttons)

Flow 2 — Approval and publishing (Telegram trigger: button press)
  Telegram choices (callback_query)
    -> Validate callback
    -> Callback config (set sheet/chat IDs)
    -> Find review row (Google Sheets: find matching Pending Review row)
    -> Check pending choice (validate row state)
    -> Route decision (Switch: Publish A / Publish B / Regenerate / Skip)

      [Publish A or B]
        -> Reserve chosen row (Google Sheets: lock row)
        -> Prepare selected post (extract text + image for chosen angle)
        -> Get a file (Telegram: download selected image)
        -> Publish LinkedIn (LinkedIn node: post with image)
        -> Build publishing record
        -> Save LinkedIn success (Google Sheets: URL, timestamp, angle)
        -> Send X copy and LinkedIn link (Telegram)

      [Regenerate]
        -> Queue fresh drafts (Google Sheets: reset status to Pending)
        -> Notify regenerate (Telegram)

      [Skip]
        -> Mark skipped (Google Sheets: update status)
        -> Notify skipped (Telegram)
```

## Google Sheets structure

Create a spreadsheet. The workflow uses the sheet ID set in the **Config** node.

**Topic queue tab** — suggested columns:

```text
topic | status | research_summary | draft_a_linkedin | draft_a_x | draft_b_linkedin | draft_b_x | image_a_file_id | image_b_file_id | chosen_angle | linkedin_url | published_at | skipped_at
```

| Status value | Meaning |
| --- | --- |
| Pending | Ready to be picked up by the daily run |
| Researching | Currently being processed |
| Pending Review | Drafts sent to Telegram; awaiting your choice |
| Published | Post successfully published to LinkedIn |
| Skipped | Manually skipped via Telegram |

## Requirements

- An n8n Cloud or self-hosted n8n instance.
- A Google account with Google Sheets OAuth2 access.
- A Firecrawl API key (HTTP Header Auth).
- A Groq API key (HTTP Header Auth).
- A Cloudflare account with Workers AI enabled and an API token.
- A Telegram bot token and your Telegram chat ID.
- A LinkedIn account connected via n8n's LinkedIn OAuth2 credential.

## Setup

1. Import the workflow JSON into n8n.
2. Open the **Config** node and set your Google Sheet ID, Telegram chat ID, and Cloudflare account ID.
3. Connect your Google Sheets OAuth2 credential to the **Read topic queue**, **Mark researching**, **Save drafts**, **Find review row**, **Reserve chosen row**, **Save LinkedIn success**, **Queue fresh drafts**, and **Mark skipped** nodes.
4. Connect your Firecrawl HTTP Header Auth credential to the **Firecrawl search** node (`Authorization: Bearer YOUR_FIRECRAWL_KEY`).
5. Connect your Groq HTTP Header Auth credential to the **Groq two angles** node (`Authorization: Bearer YOUR_GROQ_KEY`).
6. Connect your Cloudflare API credential to the **Cloudflare image A** and **Cloudflare image B** nodes.
7. Connect your Telegram credential to the **Preview image A**, **Preview text A**, **Preview image B**, **Preview text B**, **Choose A or B**, **Telegram choices**, **Send X copy and LinkedIn link**, **Notify regenerate**, and **Notify skipped** nodes.
8. Open the **Telegram choices** trigger and ensure the webhook is active.
9. Connect your LinkedIn OAuth2 credential to the **Publish LinkedIn** node and select your personal profile.
10. Add at least one row to your topic queue sheet with `status = Pending`.
11. Test with Manual start before activating the schedule.
12. Activate the workflow.

## Common customizations

- Edit the Groq prompt in **Groq two angles** to change post length, tone, hashtag count, or emoji rules.
- Swap the Cloudflare image model or change the image prompt style to match your brand aesthetic.
- Add a third draft angle (C) by duplicating the A/B parallel branches.
- Extend the topic queue with additional columns such as target audience, content pillar, or source URL.
- Add a Slack notification alongside the Telegram approval flow for team workflows.

## Security and privacy notes

- Topic content and research data are sent to the Firecrawl and Groq APIs; ensure your usage policies permit this.
- Never store API keys inside workflow nodes; always use n8n credentials.
- Rotate your Telegram bot token if it is ever exposed.
- The LinkedIn OAuth2 token grants posting access to your profile; store it securely and revoke it if compromised.

## Repository-safe export

This version contains no saved Google Sheets, Firecrawl, Groq, Cloudflare, Telegram, or LinkedIn credentials; no private spreadsheet IDs, Telegram chat IDs, Cloudflare account IDs, or LinkedIn profile IDs; and no n8n instance metadata or generated webhook IDs. Users must provide their own configuration after importing it.
