# Invoice Automation — PDF to Email & Slack

An n8n workflow that reads pending invoices from Google Sheets, generates a filled PDF
invoice from a Google Docs template, and delivers it automatically via Gmail and Slack —
then marks the row as sent. Built as a client-deliverable product under **MBK AI Automation**.

## What it does

1. Runs on a schedule (every 6 hours) and reads rows from a Google Sheet where `Status = Pending`
2. Loops through each pending row one at a time
3. Calculates subtotal, tax, and total from quantity / unit price / tax %
4. Copies a Google Docs invoice template and fills in the placeholders (`{{invoice_id}}`, `{{company}}`, `{{total}}`, etc.) via the Docs API
5. Exports the filled document as a PDF
6. Sets the PDF to "anyone with the link can view" on Google Drive
7. Emails the PDF to the client via Gmail
8. Sends the same PDF as a Slack DM
9. Updates the Google Sheet row status to `Invoice Sent`

## Architecture

```
Schedule Trigger (every 6h)
        │
Google Sheets - Read Row (Status = Pending)
        │
Loop Over Items
        │
IF - Status Pending? ──(false)──> No Op - Skip
        │(true)
Code - Calculate Totals (subtotal / tax / total)
        │
Set - Prepare Invoice Data
        │
Google Drive - Copy Template
        │
Code - Build Docs API Payload (maps fields → {{placeholders}})
        │
HTTP Request - Fill Placeholders (Google Docs batchUpdate API)
        │
Google Drive - Export as PDF
        │
        ├──> Google Drive - Set Sharing (public read link)
        ├──> Gmail - Send Invoice (PDF attached)
        └──> Slack - DM + Attach PDF
                    │
                  Merge
                    │
        Google Sheets - Update Row (Status = Invoice Sent)
                    │
              back to Loop Over Items
```

> Add a screenshot of the actual n8n canvas here: `screenshots/canvas.png`

## Stack

- **n8n** — orchestration
- **Google Sheets** — invoice data source / status tracking
- **Google Docs API** — template placeholder filling (`batchUpdate`)
- **Google Drive** — template copy, PDF export, sharing
- **Gmail** — invoice delivery via email
- **Slack** — invoice delivery via DM

## Sheet structure expected

The Google Sheet needs these columns: `Invoice ID`, `Invoice Date`, `Company`, `Email`,
`Slack ID`, `Address`, `Item`, `Qty`, `Unit Price`, `Tax %`, `Due Date`, `Status`.

See `sheet-template-structure.md` for the full column reference and the Google Docs
template placeholder format.

## Setup

1. Import `workflow.json` into your n8n instance
2. Create/connect credentials for: Google Sheets, Google Drive, Google Docs, Gmail, Slack
3. Replace these placeholders in the workflow with your own values:
   - `YOUR_GOOGLE_SHEET_ID` → your Google Sheet's document ID
   - `YOUR_SHEET_GID` → your specific sheet tab's gid
   - `YOUR_DRIVE_TEMPLATE_FILE_ID` → your Google Docs invoice template's file ID
4. Build a Google Docs invoice template with `{{invoice_id}}`, `{{invoice_date}}`, `{{company}}`,
   `{{address}}`, `{{item_line}}`, `{{due_date}}`, `{{subtotal}}`, `{{tax}}`, `{{total}}` placeholders
5. Activate the workflow

## Files

- `workflow.json` — sanitized n8n export (all credential IDs, sheet IDs, and file IDs replaced with placeholders)
- `sheet-template-structure.md` — required Google Sheet columns and Docs template placeholders
- `.env.example` — reference list of values you'll need to configure
- `screenshots/` — canvas view and sample output (add your own)

## Notes

This is a sanitized, portfolio version of a live client workflow. All document IDs, file IDs,
and credential references have been replaced with placeholders — no real client or business
data is included.
