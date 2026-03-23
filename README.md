# rss2zotero

Polls RSS/Atom feeds, uses Claude to filter and summarise new items, pushes them to a Zotero group library, and writes a dated Markdown report. Built at the SUNY Polytechnic Institute AIX Center.

---

## What it does

Each run:

1. Fetches all configured RSS/Atom feeds
2. Drops items already seen in previous runs (deduplication via `seen_items.json`)
3. Applies a keyword pre-filter (optional)
4. Sends new items to Claude for a bullet-point digest
5. Pushes each new item to a Zotero group library as a `webpage` record
6. Writes a dated Markdown report to the output directory (`./-f/` by default)

---

## Requirements

- Python 3.10+
- An [Anthropic API key](https://console.anthropic.com/)
- A [Zotero Web API key](https://www.zotero.org/settings/keys) with write access to a group library

Install dependencies:

```bash
pip install -r requirements.txt
```

---

## Setup

```bash
cp config.example.yaml config.yaml
cp .env.example .env
```

Edit `.env`:

```
ANTHROPIC_API_KEY=sk-ant-...
ZOTERO_API_KEY=your_zotero_key
```

Edit `config.yaml` — minimum changes:

- Add your feed URLs under `feeds:`
- Set `zotero.group_id` to your numeric Zotero group ID (found in the group URL: `zotero.org/groups/<id>/`)
- Set `claude.filter_keywords` to topics you care about, or leave empty (`[]`) to pass everything through

---

## Usage

Test run — no API calls, shows what would be processed:

```bash
python rss2zotero.py --dry-run --show-new
```

Live run:

```bash
python rss2zotero.py
```

With a non-default config:

```bash
python rss2zotero.py --config /path/to/my-config.yaml
```

---

## AIX Center example configuration

This is the configuration used at the SUNY Poly AIX Center to monitor AI literacy and higher education feeds:

```yaml
feeds:
  - name: "Google Alerts — AI literacy"
    url: "https://www.google.com/alerts/feeds/XXXXXXX/XXXXXXX"
    category: "alerts"

  - name: "Inside Higher Ed"
    url: "https://www.insidehighered.com/rss.xml"
    category: "news"

  - name: "EdSurge"
    url: "https://www.edsurge.com/feed"
    category: "news"

zotero:
  group_id: "YOUR_ZOTERO_GROUP_ID"
  item_type: "webpage"
  tag: "rss2zotero"

claude:
  model: "claude-opus-4-6"
  max_tokens: 1500
  filter_keywords:
    - "AI literacy"
    - "artificial intelligence"
    - "higher education"
    - "generative AI"
    - "LLM"

report:
  max_bullets: 15
  min_bullets: 3

state:
  seen_file: "./-f/seen_items.json"
  max_age_days: 30
```

---

## Output

Each run produces two files in the output directory (`./-f/` by default):

- `ai_literacy_YYYY-MM-DD.md` — Claude's bullet-point digest
- `ai_literacy_YYYY-MM-DD_items.json` — raw feed items for that run
- `rss2zotero.log` — running log

---

## Automating with cron

To run daily at 7am:

```
0 7 * * * cd /path/to/rss2zotero && python rss2zotero.py >> ./-f/cron.log 2>&1
```

---

## Optional: email delivery

Add an `email:` block to `config.yaml` and set `SMTP_PASSWORD` in `.env`. See `config.example.yaml` for the full structure.

---

## License

MIT — see `LICENSE`.
