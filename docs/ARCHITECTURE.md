# Architecture

How `Email_Extractor.py` is structured and how data moves through a run.

## High-level flow

```
Source folder (*.msg)
        │
        ▼
  Path.rglob("*.msg")  → ordered file list
        │
        ▼
  Slice by Start Index + Limit
        │
        ▼
  For each file:
      extract_msg parse → metadata + body
      clean / format contacts
      get_ai_summary()  → skip rules OR GitHub Models API
      append row to buffer + Treeview
      sleep 4.1s
        │
        ▼
  Export: Excel and/or clipboard
```

## Single-file module map

Everything is in `Email_Extractor.py`. Logical sections:

| Section | Functions / symbols | Role |
| --- | --- | --- |
| Logging & state | `STATE_FILE`, `save_pipeline_state`, `load_pipeline_state` | Persist resume info to `pipeline_state.json` |
| AI client | `GITHUB_TOKEN`, `client`, `get_ai_summary` | GitHub Models summarization + skip heuristics |
| Text helpers | `clean_text`, `format_contact` | Sanitize bodies/names for display and Excel |
| Pipeline | `run_pipeline` | Scan, parse, summarize, update UI, handle quota |
| Table UX | `handle_row_click`, `toggle_all_rows`, `invert_row_selection`, `open_full_body_window`, `select_completed_summaries_only` | Selection + full-body viewer |
| Export | `save_to_excel_file`, `copy_selection_to_clipboard`, export menus | `.xlsx` and clipboard |
| App shell | `start_thread`, `browse_folder`, widget construction, `root.mainloop()` | Tkinter UI |

## Runtime globals

| Name | Meaning |
| --- | --- |
| `extracted_data_buffer` | List of row lists ready for Excel/clipboard |
| `full_body_cache` | `filename → raw body` for double-click viewer |
| `processed_filenames` | Set of basenames already handled this run |
| `api_limit_hit` | Once true, further API calls are skipped / loop may stop |
| `pipeline_running` | Guards quit confirmation |
| `tree_item_ids` | Treeview item ids for selection helpers |
| `tokens_used_this_minute` / `minute_window_start` | Soft local token budget display (150k/min) |

## AI summarization rules (`get_ai_summary`)

Order of decisions:

1. Empty body → `"Empty Body"` (0 tokens)
2. Subject looks like auto-reply / read receipt / accepted-declined → truncated body, no API
3. Body looks like a calendar invite (`when:` + `where:`) → canned omit message, no API
4. Short body (`< 300` chars **or** `< 40` words) → use cleaned body as “summary”, no API
5. Quick-reply phrase match (`ok`, `thanks`, `fyi`, etc.) → body as-is, no API
6. `force_skip_api` → placeholder skip string
7. Otherwise call GitHub Models with a strict plain-text prompt (body truncated to 3500 chars, `max_tokens=150`, `temperature=0`)

Rate limit detection:

- `openai.RateLimitError`, or
- status `429` / message containing quota language

Returns sentinel `"API_LIMIT_TRIGGERED"` so `run_pipeline` can stop and set resume index.

## Pipeline details (`run_pipeline`)

1. Clears buffers, tree, and flags.
2. Validates token, folder, start index, limit.
3. Collects all `*.msg` under the folder recursively.
4. Processes from `start_idx` (1-based) for up to `limit` files.
5. Per file:
   - Skip if basename already seen
   - Parse with `extract_msg.Message`
   - Prefer plain `body`; else strip HTML via BeautifulSoup
   - Summarize
   - On API limit: record index, update UI fields, break
   - Else append row and refresh progress labels
6. Always sleeps **4.1 seconds** between files.
7. Re-enables START / EXPORT when finished or stopped.

### Index math

- Files are ordered as returned by `Path.rglob`.
- UI **Start Index** `1` means the first file in that list.
- On rate limit at overall index `N`, Start Index is set to `N` so the next run retries that file.

## Persistence: `pipeline_state.json`

Written next to the working directory when the app runs.

```json
{
  "next_start_idx": 1,
  "folder_path": "C:\\path\\to\\msgs",
  "status": "completed"
}
```

| Field | Use |
| --- | --- |
| `folder_path` | Restored into the Source Folder entry on launch; updated on Browse |
| `next_start_idx` | Restored into Start Index |
| `status` | Intended values: `in_progress`, `api_limit_hit`, `completed` (see known bug in HANDOFF) |

Also created: `pipeline_debug.log` (overwritten each launch, `filemode='w'`).

## Excel export behavior

Headers:

`FileName | Subject | Sender | Recipient | CC | Attachments | Date | Time | Summary`

Rules:

- Creates workbook or opens existing path chosen in Save dialog
- If a filename already exists and its Summary looks successful, that row is left alone
- If existing summary looks skipped/failed, the row is overwritten
- If `api_limit_hit`, appends a red NOTICE banner row
- PermissionError → “close Excel and retry”

## Threading model

```
Main thread: Tk event loop
Daemon thread: run_pipeline(...)
```

START disables buttons, starts the daemon thread, and returns. The worker updates labels, progress bar, and Treeview directly.

## External dependencies

| Package | Why |
| --- | --- |
| `extract_msg` | Read Outlook `.msg` |
| `beautifulsoup4` | HTML body → text |
| `openai` | Chat Completions client pointed at GitHub Models |
| `openpyxl` | Styled Excel write |
| `pandas` | Imported (legacy / available); Excel path uses openpyxl directly |
| `tkinter` | Stdlib GUI (must be available in the Python install) |

## Packaging

Optional standalone Windows build via PyInstaller (see SETUP.md). The distributed app is still the same single-file logic; packaging does not add a service layer.
