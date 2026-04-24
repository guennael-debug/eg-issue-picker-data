# eg-issue-picker-data

Issue library consumed by the **Execution Gaps Issue Picker** at [execution-gaps.lovable.app](https://execution-gaps.lovable.app).

## What this is

A single data file (`issues.json`) containing metadata for every published issue of G's Execution Gaps newsletter. The Issue Picker's Lovable edge function fetches this JSON at request time and passes it to Claude Haiku as the matching library.

## Schema

`issues.json` is a JSON array. Each element:

| Field | Type | Notes |
|---|---|---|
| `issue_number` | integer | 1-based. No gaps expected. |
| `title` | string | Title as published on Substack. |
| `date` | string | ISO-8601 (`YYYY-MM-DD`). |
| `concepts` | string[] | Core concepts / principles the issue maps to. |
| `key_story` | string | Short label for the anchor story in the issue. |
| `url` | string | Canonical Substack URL. |
| `blurb` | string | 1-2 sentence "why this one" used by the matcher and shown on result cards. |

## Raw URL

```
https://raw.githubusercontent.com/guennael-debug/eg-issue-picker-data/main/issues.json
```

Served by GitHub's CDN. The Lovable edge function fetches this URL on every `/api/match` request.

## How it gets updated

Automatically, when a new issue ships. The pipeline:

1. G runs `/Gsubstack-continue ship [N]` to mark a Substack issue published.
2. The Ship on command flow in `~/.claude/skills/substack-draft.md` updates the wiki index at `~/OneDrive/Documents/Claude/Projects/wiki/wiki/execution-gaps-newsletter.md` (step 5).
3. Immediately after, step 5b runs `~/.claude/scripts/regen_issue_picker.py`, which parses the wiki, regenerates this `issues.json`, and pushes.
4. The next Issue Picker query hits the fresh library.

## Manual regen

```bash
python ~/.claude/scripts/regen_issue_picker.py
```

Idempotent: no changes → no commit.

Dry run: add `--dry-run` to preview the diff without writing or pushing.

## Source of truth

The **wiki** (`execution-gaps-newsletter.md`) is the authoritative source. `issues.json` is a derived artifact. Don't edit `issues.json` by hand unless you're also editing the wiki; the next regen will overwrite it.
