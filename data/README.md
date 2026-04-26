# `data/` — snapshot only as of 2026-04-26

**These JSON files are no longer authoritative.**

Before 2026-04-26 the three Nourish HTML apps (Meal Planner, Grocery, Settings)
read and wrote directly to the files in this directory via the GitHub
Contents API. As of the cutover commit, all three apps now talk to a
private FastAPI service running on Sunil's NUC (`nutrition-mcp`), which
holds the canonical state in SQLite.

The files here are a **point-in-time snapshot** taken just before the cutover.
They are kept so that:

1. The pre-cutover Git history (and the `pre-nuc-cutover-20260426` tag) can
   be `git revert`-ed if a hard rollback is ever needed. Reverting the cutover
   commit restores the apps to their previous behaviour, and they will find
   these snapshot files where they expect them.
2. Anyone reading the repo cold can see what the data shapes used to look
   like, even though the live data is no longer here.

**Do not edit these files** to make a change to your live data — edits here
will not propagate anywhere. Use the apps (which now write to the NUC).

## Future of this directory

A nightly NUC → GitHub backup job is on the followups list (`docs/04-followups.md`
in the `nutrition-mcp` repo). Once that ships, these files will be refreshed
automatically as a real backup snapshot. Until then, they are frozen at the
2026-04-26 cutover state.

## What lives in SQLite now

| File here | Replaced by table on NUC |
|---|---|
| `config.json` | `config` (key/value rows) |
| `custom_meals.json` | `meals` rows where `source='custom'` |
| `grocery_current.json` | `grocery_items` (current week) |
| `history.json` | computed on demand from `week_plans` + `meals` |
| `pantry.json` | `pantry` (table created but pre-cutover empty) |
| `week_*.json` | `week_plans` rows joined with `meals` |
