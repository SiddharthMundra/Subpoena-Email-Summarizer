# Setup Guide

Install, configure, run, and optionally package the Email Summarizer.

## Requirements

- Windows 10 or 11 (64-bit) — primary target
- Python **3.10+** with `tkinter` included
- Outbound HTTPS to `https://models.github.ai/inference`
- A GitHub Personal Access Token that can call GitHub Models

## Install

From the project root:

```bash
python -m venv .venv
# Windows PowerShell:
.venv\Scripts\activate
# or Git Bash:
source .venv/Scripts/activate

pip install -r requirements.txt
```

Dependencies (also listed in `requirements.txt`):

- `openpyxl`
- `pandas`
- `extract-msg`
- `beautifulsoup4`
- `openai`
- `pyinstaller` (only needed if you build an `.exe`)

## Configure the API token

Open `Email_Extractor.py` and locate:

```python
GITHUB_TOKEN = "..."
```

**Handoff requirement:** revoke any token you inherited, create a new one, and do not leave production secrets in git.

Recommended pattern:

```python
GITHUB_TOKEN = os.environ.get("GITHUB_TOKEN", "").strip()
```

Then set the variable in your shell before launch:

```bash
# Git Bash / bash
export GITHUB_TOKEN="your_new_token"

# PowerShell
$env:GITHUB_TOKEN = "your_new_token"
```

If the token is empty or still a placeholder, START shows: **Configuration Error: Paste GitHub Token**.

## Run from source

```bash
python Email_Extractor.py
```

Working directory matters: `pipeline_state.json` and `pipeline_debug.log` are written relative to the current working directory.

### Quick smoke test

1. Place 2–3 sample `.msg` files in a test folder
2. Browse to that folder
3. Set **Limit total items** to `2`
4. Click **START**
5. Confirm rows appear and summaries fill in
6. **EXPORT LOG DATA → Excel Document Exports → Save Entire Set…**

## Operator UI fields

| Control | Meaning |
| --- | --- |
| Source Folder | Root directory to scan recursively for `.msg` |
| Start Index | 1-based position in the discovered file list to begin |
| Limit total items | Optional cap on how many files to process this run |

See [OPERATOR_GUIDE.md](OPERATOR_GUIDE.md) for the full workflow, including resume after rate limits.

## Build a Windows executable (optional)

```bash
python Email_Extractor.py   # confirm it works first

pyinstaller --noconfirm --onedir --windowed --name "Email_Extractor" Email_Extractor.py
```

Output: `dist/Email_Extractor/`

Notes:

- `--windowed` hides the console
- Recipients still need network access and a valid token inside the built app (or you must change the code to read env/config at runtime before shipping)
- Rebuild after any source change you want in the distributed build

## Troubleshooting

| Symptom | What to check |
| --- | --- |
| App won’t start / Tk errors | Confirm Python includes tkinter (`python -m tkinter`) |
| No emails found | Folder actually contains `.msg` (not only `.eml`/PST) |
| Configuration Error (token) | `GITHUB_TOKEN` empty or placeholder |
| API Limit Reached | Wait for quota reset; Start Index / Limit are updated for resume |
| Write Lock Error | Close the target `.xlsx` in Excel |
| Odd duplicate Settings controls | Known duplicate widget construction — see HANDOFF.md |
| Need failure details | Open `pipeline_debug.log` in the working directory |

## Related docs

- [HANDOFF.md](HANDOFF.md) — ownership checklist
- [ARCHITECTURE.md](ARCHITECTURE.md) — internals
- [OPERATOR_GUIDE.md](OPERATOR_GUIDE.md) — end-user workflow
