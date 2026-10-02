# Operator Guide

Day-to-day instructions for running email extraction and export batches.

## What you need

- This application (Python script or packaged `Email_Extractor`)
- A folder of Outlook **`.msg`** files (subfolders are included automatically)
- Internet access (summaries call GitHub’s AI endpoint)
- A valid API token configured by your technical owner

## Basic run

1. Launch the app.
2. Click **Browse…** and select the folder that contains the emails.
3. Leave **Start Index** at `1` for a fresh folder.
4. Optionally set **Limit total items** (useful for test runs or quota budgeting).
5. Click **START**.
6. Wait while the status bar advances. The table fills row by row.
7. When finished, click **EXPORT LOG DATA** and choose Excel and/or clipboard export.

Click the **?** icon in Settings for the in-app short guide.

## Reading the live table

Columns: Select, FileName, Subject, Sender, Recipient, CC, Attachments, Date, Time, Summary.

| Action | How |
| --- | --- |
| Check / uncheck one row | Click the Select column (☐ / ☑) |
| Select all / clear all / invert | Buttons above the table |
| Read full email body | Double-click the row |

Some rows will show the original short text instead of an AI paraphrase. That is intentional for short messages, auto-replies, and calendar invites (saves API quota).

## Exporting

### Excel

**EXPORT LOG DATA → Excel Document Exports → Save Entire Set to Excel File (.xlsx)**

- Creates or updates a workbook you choose
- Columns: FileName, Subject, Sender, Recipient, CC, Attachments, Date, Time, Summary
- Re-exporting to the same file updates failed/skipped summaries without blindly duplicating successful rows
- If a run stopped on an API limit, a red NOTICE line may appear at the bottom

If save fails with a lock error, close the spreadsheet in Excel and try again.

### Clipboard

**EXPORT LOG DATA → Clipboard Operations**

- Copy checked rows, or the entire table
- Optional: include header names
- If nothing is checked, “copy checked” falls back to copying everything

### Selection filters

Under **Selection Rules and Filters**:

- Select only completed summaries (skips rows whose summary text looks skipped/errored)
- Select all / clear / invert

## When the API limit is hit

You will see a warning and Status text like “API Limit Hit” with an approximate reset time.

The app updates:

- **Start Index** → next email to process
- **Limit total items** → remaining count (if you had set a limit)

What to do:

1. Export whatever was completed so far (recommended).
2. Wait until the quota window resets (often about a day for hard caps; follow the message).
3. Click **START** again without changing the folder — processing continues from the saved index.

Do not reset Start Index to `1` unless you intentionally want to reprocess from the beginning.

## Tips for large folders

- Run a small **Limit** first to validate the folder and export path.
- Keep the machine awake; a large batch sleeps ~4 seconds between emails by design.
- Prefer stable folder paths; the last browsed folder is remembered across launches.
- Duplicate filenames in different subfolders: only the first basename is processed in a run.

## Common messages

| Message | Meaning |
| --- | --- |
| No email files found | No `.msg` under the selected tree |
| Process Complete | Finished the planned slice successfully |
| API Limit Reached | Stopped early; resume later with updated Start Index |
| Write Lock Error | Target Excel file is open elsewhere |
| Configuration Error | Folder missing or token not configured |

## Who to contact

- Token / install / build issues → technical owner (see [SETUP.md](SETUP.md) and [HANDOFF.md](HANDOFF.md))
- Summary quality (“too vague”, “includes names we don’t want”) → owner can adjust the prompt and skip rules in `get_ai_summary`
