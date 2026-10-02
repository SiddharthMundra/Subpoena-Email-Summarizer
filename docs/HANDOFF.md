# Project Handoff Guide

Start here if you are taking ownership of **Subpoena Email Summarizer**.

## What this project is

A Windows desktop app (`Email_Extractor.py`) that:

1. Recursively scans a folder for Outlook `.msg` files
2. Extracts metadata and body text
3. Generates short AI summaries via GitHub Models (`openai/gpt-4o-mini`)
4. Shows results in a live Tkinter table
5. Exports to Excel (`.xlsx`) or the clipboard

Primary audience: people reviewing large email dumps (e.g. subpoena / legal production review) who need structured summaries instead of opening every message.

## Repository layout

| Path | Purpose |
| --- | --- |
| `Email_Extractor.py` | Entire application (UI + pipeline + export) |
| `README.md` | Project overview and quick reference |
| `requirements.txt` | Python dependencies |
| `docs/HANDOFF.md` | This file — takeover checklist |
| `docs/SETUP.md` | Install, configure, run, package |
| `docs/ARCHITECTURE.md` | Code map, data flow, state files |
| `docs/OPERATOR_GUIDE.md` | How end users run a batch |
| `pipeline_state.json` | Created at runtime — resume folder/index/status |
| `pipeline_debug.log` | Created at runtime — processing log |

There is no separate backend, database, or web UI. Everything lives in one Python script.

## Day-one checklist (do these first)

### 1. Rotate the API token (security — required)

A GitHub Personal Access Token is currently **hardcoded** in `Email_Extractor.py` near the top (`GITHUB_TOKEN = "..."`).

Before you use or redistribute this project:

1. Revoke the existing token in GitHub (Settings → Developer settings → Personal access tokens).
2. Create a new token with access to GitHub Models.
3. Prefer loading it from an environment variable instead of pasting it into source:

```python
GITHUB_TOKEN = os.environ.get("GITHUB_TOKEN", "").strip()
```

4. Never commit a live token to git.

The README notes the previous token was expected to expire around **August 21, 2026** — treat that date as a reminder only; rotate immediately on handoff.

### 2. Confirm you can run locally

Follow [SETUP.md](SETUP.md). Minimum smoke test:

```bash
pip install -r requirements.txt
python Email_Extractor.py
```

Pick a small folder of `.msg` files, set **Limit total items** to `2` or `3`, click **START**, then export once to Excel.

### 3. Read the architecture and operator docs

- [ARCHITECTURE.md](ARCHITECTURE.md) — how the pipeline, skips, rate limits, and exports work
- [OPERATOR_GUIDE.md](OPERATOR_GUIDE.md) — what to tell non-developer users

## Ownership responsibilities

You will typically own:

- Keeping the GitHub Models token valid and rate-limit aware
- Adjusting skip rules / summary prompt if review quality needs change
- Helping operators resume after API quota hits
- Rebuilding the PyInstaller executable when source changes
- Deciding whether to keep the single-file design or split modules later

## Known issues / tech debt (important)

These are real behaviors in the current source — plan for them:

1. **Hardcoded secrets** — token in source (see above).
2. **Duplicate UI widget construction** — near the bottom of `Email_Extractor.py`, the Settings controls (`Start Index`, `Limit`, help `?` button) are built twice. The second block overwrites the Python variables; leftover widgets can still appear in the window. Safe cleanup: remove the first duplicate block and keep one Settings section.
3. **API-limit state save bug** — when a rate limit is hit inside `run_pipeline`, the code saves `status="api_limit_hit"` and then immediately saves `status="completed"` in the same branch. Resume still works via the Start Index UI fields, but `pipeline_state.json` status may be wrong. Fix by removing the premature `"completed"` save in the limit-hit path.
4. **Tkinter updates from a worker thread** — the pipeline runs in a daemon thread and updates widgets directly. This usually “works” on Windows but is not thread-safe Tk practice. A future hardening step is to marshal UI updates onto the main thread (`root.after(...)`).
5. **Fixed sleep between emails** — `time.sleep(4.1)` after each file slows large batches on purpose (quota friendliness). Change carefully; faster loops hit GitHub Models limits sooner.
6. **Filename de-dupe only** — files with the same basename in different subfolders are treated as duplicates (`processed_filenames`). Nested trees with colliding names will skip later copies.

## Suggested first improvements (optional)

Priority order if you have time after handoff:

1. Move token to environment / `.env` (gitignored)
2. Fix the rate-limit state save and remove duplicate UI widgets
3. Add a `requirements.txt` pin review / Python version note in CI if the team wants it
4. Split `Email_Extractor.py` into modules (`pipeline.py`, `ui.py`, `export.py`, `ai.py`) once changes become frequent

## Support context

| Topic | Detail |
| --- | --- |
| OS target | Windows 10 / 11 |
| Python | 3.10+ |
| Model | `openai/gpt-4o-mini` via `https://models.github.ai/inference` |
| Input | Outlook `.msg` only (not `.eml` / PST directly) |
| Output | Live table + `.xlsx` + clipboard TSV |

## Where to go next

1. [SETUP.md](SETUP.md) — get it running
2. [ARCHITECTURE.md](ARCHITECTURE.md) — understand the code
3. [OPERATOR_GUIDE.md](OPERATOR_GUIDE.md) — train the person who clicks START
