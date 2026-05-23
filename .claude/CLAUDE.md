# hire-me-pls — routing rules

These are project-level instructions for Claude when working inside a `hire-me-pls` repo.

## Skill router

Match user intent to the right skill before doing anything else:

| User says... | Skill |
| --- | --- |
| "set up hire-me-pls" / "first run" / "I just cloned this" | `job-search-setup` |
| "find jobs in [X]" / "Run [Country]" / "search [city]" | `job-discovery` → `job-search-pipeline` |
| "Run Remote" / "search remote" / "find remote roles" / "run the workflow remote" | `job-search-pipeline` (`Run Remote` — routes to the Remote sheet) |
| "tailor a CV for [company]" / "render a CV" / "Run CV only" | `role-diagnosis` (gate) → `cv-tailor` |
| "diagnose this role" / "what is this team actually hiring to fix" | `role-diagnosis` |
| "write a cover letter for [company]" | `cover-letter` (after diagnosis) |
| "prep me for the [company] interview" / "Run Interview Prep" | `interview-prep` |
| "refresh the story bank" / "Run Story Bank Refresh" | `story-bank` |
| "blacklist [company]" / "Run Blacklist: add/remove" | `job-discovery` (blacklist sub-action) |
| Any `Run [shortcut]` command | `job-search-pipeline` (it owns the DSL) |

## First-run check

On the first turn of any session in this repo, check for `config.yaml` at the repo root.
- If it does not exist → suggest running `job-search-setup` before any other skill.
- If it exists → load it. Subsequent skills read region, branches, voice references, output formats from there.

## Opinionation policy

Default is `warn-once-then-comply`. Hard gates (diagnosis-first; voice reference for cover letters; max experience slots; append-only Excel) emit a one-time explanation on the first bypass per session, then comply silently. Track which gates have been warned about in session memory.

Strict mode (`opinionation: strict` in `config.yaml`) reverts to the original behavior: refuses to proceed when a gate is bypassed.

## File location conventions

All session outputs live under `data/sessions/[dd.mm]/[Country or City]/`. The repo's `data/` folder is gitignored. Do not write user data outside `data/`.

The job log lives at `data/job-log/Job Listings.xlsx`. It is **append-only**. Never overwrite. Always back up to `data/job-log/Backup/Job Listings — [DD.MM.YY HH.MM].xlsx` before any write. See `job-discovery/references/append-only-safety.md`.

## Voice and tone

When drafting any candidate-facing copy (CV summary, cover letter, interview prep, LinkedIn nudge), read the candidate's voice reference files from the paths in `config.yaml > voice_references` before writing the first sentence. If no voice references exist for the document type, prompt the user to add one — do not draft from memory of "what good looks like."

## Failure recovery

When a skill cannot complete (Apify timeout, Excel locked, template missing, etc.), do not silently fall back. Stop, log to `data/Session Notes.txt` in the format defined in `job-search-pipeline/references/failure-recovery.md`, and notify the user with the exact error.

## What this file is not

It is not the full workflow. The behavior of each skill lives in its `SKILL.md` and `references/` files. This file is the router and the project-level conventions only.
