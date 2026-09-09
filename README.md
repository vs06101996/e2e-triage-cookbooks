# e2e-triage-cookbooks

Cookbook and JIRA-template source of truth for the E2E auto-triage pipeline
(`cursor_skills/skills/e2e-triage/` in cvo-misc). Moved here from Confluence
so a new agent-drafted cookbook goes through an actual reviewed pull request
before it's live — Confluence had no review gate at all, and this is the
whole point of the split: **code lives in cvo-misc, knowledge lives here**,
under separate RBAC.

- `cookbooks/*.md` — matched by `e2e-triage`/`e2e-cookbook-index` phases 1-2.
  YAML frontmatter (`id`, `jira`, `defect_type`, `test_aliases`,
  `failure_signatures`, `source_files`) + `## Identify` / `## Debug` /
  `## Validate` / `## Resolution` sections. New ones are drafted by
  `scripts/draft_cookbook.py` (in cvo-misc) from a confirmed phase-3
  investigation, opened as a PR here, and only become part of the live
  matching set once merged.
- `templates/*.jira` — NOC ticket wiki-markup templates
  (`jira-ticket.template.jira` for cookbook-matched tickets,
  `jira-investigation.template.jira` for code-RCA investigation tickets),
  filled by `jira_draft.py`.

This repo is read by the triage pipeline via a `git pull` before each run —
merging a PR here *is* the "make it available" step, the same way the
pipeline already treats a `git pull` of cvo-misc itself. Read access (clone,
pull) needs no token — this repo is public. A `GITHUB_TOKEN` is only needed
by `draft_cookbook.py`'s push + PR-creation step.

## The review gate is enforced, not just documented

`main` has branch protection requiring 1 approving review. GitHub itself
refuses self-approval on a PR ("Can not approve your own pull request") and
refuses to merge an unreviewed PR once protection is on
(`mergeStateStatus: BLOCKED`) — verified directly against this repo, not
assumed. On a single-maintainer repo like this one, that means a second
person (or a second GitHub identity) has to actually look at every new
cookbook before it can go live.
