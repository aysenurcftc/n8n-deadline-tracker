# Legal Document Intelligence & Deadline Tracker (n8n)


Watches a Google Drive folder for new legal PDFs, uses an AI model (Groq) to extract critical dates (hearing, objection deadline, appeal deadline, contract renewal), logs them to Google Sheets, creates Calendar events for confident dates, and notifies you on Telegram — including a 1-day-before reminder.

![workflow-1](https://github.com/aysenurcftc/n8n-deadline-tracker/blob/main/workflows/workflow-1.jpg)
![workflow-2](https://github.com/aysenurcftc/n8n-deadline-tracker/blob/main/workflows/workflow-2.jpg)
## Workflows

| File | Trigger | What it does |
|---|---|---|
| `1-document-intake-and-extraction.json` | New file in a Drive folder | Downloads PDF → extracts text → AI extraction (doc type, parties, summary, critical dates) → writes rows to Sheets → creates Calendar events for confident dates → sends a Telegram summary for every document → sends a Telegram alert on AI/parsing failure |
| `2-daily-reminder-check.json` | Daily schedule | Reads the same Sheet → finds dates due tomorrow that haven't been reminded → sends Telegram reminder → marks the row as sent |

## Stack

n8n (self-hosted) · Google Drive · Google Sheets · Google Calendar · Groq (`openai/gpt-oss-120b`) · Telegram Bot API

## Setup

1. Import both JSON files into n8n.
2. Replace the placeholders in each file / node:
   - `YOUR_DRIVE_FOLDER_ID`, `YOUR_GOOGLE_SHEET_ID`, `your-calendar-email@gmail.com`
   - `YOUR_*_CREDENTIAL_ID` → pick/create your own credentials in n8n for Google Drive, Sheets, Calendar, Groq, and Telegram
   - `REPLACE_WITH_YOUR_CHAT_ID` in both `Config` nodes → your Telegram chat_id
3. Create the "Dates" Google Sheet with columns: `id, file_name, date_type, computed_date, confidence, needs_human_review, meaning, calendar_event_id, reminder_1_sent`
4. Activate workflow 2 (schedule trigger only runs when active).

## Design notes

- **Dedup:** workflow 1 uses n8n's "Remove Items Seen Before" to avoid reprocessing the same Drive file, and `appendOrUpdate` in Sheets keyed by a generated `fileId_index` so re-runs update rather than duplicate rows.
- **Confidence gating:** only dates with a `computed_date` and `confidence` above a threshold (default 0.5, set in `Config`) get a Calendar event. Lower-confidence dates still land in the sheet, flagged `needs_human_review`, for manual review.
- **Every document gets a Telegram summary**, independent of whether dates were found — so nothing is silently skipped.



