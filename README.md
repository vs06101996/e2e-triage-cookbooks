# e2e-triage-cookbooks

Cookbook and JIRA-template source of truth for the E2E auto-triage pipeline
(`cursor_skills/skills/e2e-triage/` in cvo-misc), moved here from Confluence
so a new agent-drafted cookbook goes through an actual reviewed pull request
before it's live — Confluence had no review gate at all.

- `cookbooks/*.md` — matched by `e2e-triage`/`e2e-cookbook-index` phases 1-2.
  New ones are drafted by `scripts/draft_cookbook.py` from a confirmed
  phase-3 investigation, opened as a PR here, and only become part of the
  live matching set once merged.
- `templates/*.jira` — NOC ticket wiki-markup templates, filled by
  `jira_draft.py`.

This repo is read by the triage pipeline via a `git pull` before each run —
merging a PR here *is* the "make it available" step, the same way the
pipeline already treats a `git pull` of cvo-misc itself.
