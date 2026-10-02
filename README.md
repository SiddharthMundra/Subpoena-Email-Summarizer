# Subpoena Email Summarizer

Windows desktop app that scans folders of Outlook `.msg` files, extracts metadata, generates AI summaries (GitHub Models / `gpt-4o-mini`), and exports results to Excel or the clipboard.

## Documentation (start here for handoff)

| Doc | Audience | Contents |
| --- | --- | --- |
| [docs/HANDOFF.md](docs/HANDOFF.md) | New technical owner | Takeover checklist, security, known issues |
| [docs/SETUP.md](docs/SETUP.md) | Developers | Install, token config, run, PyInstaller build |
| [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) | Developers | Code map, pipeline, state files, export rules |
| [docs/OPERATOR_GUIDE.md](docs/OPERATOR_GUIDE.md) | End users | How to run batches, resume after rate limits, export |

## Quick start

```bash
pip install -r requirements.txt
python Email_Extractor.py
```

Configure a GitHub Models-capable token before processing real mail. **Do not commit tokens.** See [docs/SETUP.md](docs/SETUP.md) and [docs/HANDOFF.md](docs/HANDOFF.md).

## Features

- Recursive `.msg` discovery and metadata extraction
- AI summaries with automatic skips for short mail, auto-replies, and calendar invites
- Resume support via Start Index / Limit and `pipeline_state.json`
- Background processing thread with live progress table
- Excel export (styled, de-duplicating by filename) and clipboard TSV copy

## Prerequisites

- Windows 10 / 11
- Python 3.10+
- Network access to `https://models.github.ai/inference`
- Valid GitHub Personal Access Token for Models

## UI summary

1. **Browse** to a source folder of `.msg` files  
2. Set **Start Index** (default `1`) and optional **Limit**  
3. Click **START**  
4. Review the feed table (double-click a row for full body)  
5. **EXPORT LOG DATA** → Excel and/or clipboard  

Full operator steps: [docs/OPERATOR_GUIDE.md](docs/OPERATOR_GUIDE.md).

## Project layout

- `Email_Extractor.py` — application source (UI + pipeline)
- `requirements.txt` — Python dependencies
- `docs/` — handoff and reference documentation
- `pipeline_state.json` / `pipeline_debug.log` — created at runtime

## Packaging

```bash
pyinstaller --noconfirm --onedir --windowed --name "Email_Extractor" Email_Extractor.py
```

Output under `dist/Email_Extractor/`. Details in [docs/SETUP.md](docs/SETUP.md).

## Troubleshooting (short)

| Issue | Action |
| --- | --- |
| Missing / invalid token | Configure `GITHUB_TOKEN` (see SETUP) |
| Excel won’t save | Close the `.xlsx` in Excel and retry |
| API rate limit | Wait for reset; app updates Start Index for resume |
| Need logs | Check `pipeline_debug.log` |

More detail: [docs/SETUP.md](docs/SETUP.md) and [docs/HANDOFF.md](docs/HANDOFF.md).
