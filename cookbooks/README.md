# Cookbooks (source of truth)

**POC editable source:** [Cookbooks — POC Source](https://netapp.atlassian.net/wiki/spaces/CLOUDMGR/pages/661008644/Cookbooks+POC+Source) on Confluence (child pages with markdown code blocks).

Local `*.md` files are fallback when Confluence is unavailable (`E2E_TRIAGE_SOURCE=local` or `auto` fallback).

The **LLM agent** reads cleansed build evidence plus cookbooks and decides match, RCA validation, skip vs ticket, and JIRA draft content.

## Runtime source modes

| `E2E_TRIAGE_SOURCE` | Behavior |
|---------------------|----------|
| `auto` (default) | Confluence first → local fallback |
| `confluence` | Confluence only (fail if unreachable) |
| `local` | Repo `cookbooks/*.md` only |

Verify: `PYTHONPATH=src python3 scripts/verify_confluence_sources.py`

## Behavior

- **Redundant match + confident validation** → JIRA draft from Confluence template
- **No match / low confidence** → skip; update Confluence cookbook if recurring

## Dedup (POC)

- **One triage outcome per build ID** — see `output/triage_state.db`
- Each new failed build may get its own ticket
- `run_triage.py --force` to re-process

## Format

YAML frontmatter + Identify / Debug / Validate / Resolution (in Confluence code block or local markdown).

## Promoting a confirmed investigation into a cookbook

When phase 3 (`e2e-code-rca`) produces a `create_investigation_ticket` and a
human confirms it's a real, recurring issue (not a one-off/flaky failure),
promote it so future occurrences get caught cheaply by phase 1/2 instead of
paying for a fresh code-RCA every time:

```bash
PYTHONPATH=src python3 scripts/draft_cookbook.py \
  --build-id <id> --defect-type "Automation Issue" --dry-run
# review the drafted markdown + dedup report, then:
PYTHONPATH=src python3 scripts/draft_cookbook.py \
  --build-id <id> --defect-type "Automation Issue" --jira NOC-XXXXX --confirm-recurring
```

This is **human-gated by design** — it's never run automatically. It
requires `run_triage.py` to have already recorded the investigation
(`output/investigations/<build_id>.json`), derives `test_aliases`/
`failure_signatures`/`source_files` deterministically from the stored RCA and
build evidence, and — this is the important part — checks the draft against
every currently-loaded cookbook (reusing `matcher.match_cookbooks`, the same
logic phase 1/2 use) before writing anything, to catch the case where this
"new" issue is actually a repeat of one already documented. See
`--dry-run`'s dedup report; `--force` overrides an overlap warning, an id
collision never can.

**The tool only writes a local `cookbooks/*.md` file** — this repo cannot
write to Confluence (`confluence.py` is read-only). Since production
defaults to reading Confluence first, a locally-written cookbook needs a
manual follow-up: paste it into a new `Cookbook: <title>` child page under
the Confluence parent above.
# gate test 1788938734
