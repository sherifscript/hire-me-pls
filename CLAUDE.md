# hire-me-pls — project context for Claude Code

## What this repo is

A Claude Code plugin (`v1.2.0`) implementing a diagnosis-first job search system. Eight skills cover the full pipeline: discover → diagnose → tailor → cover → interview prep → story bank. Installable via `claude plugin install`.

Plugin manifest: `.claude-plugin/plugin.json`. Marketplace definition: `.claude-plugin/marketplace.json`.

## Repo layout

```
.claude-plugin/         plugin manifest + marketplace
skills/                 8 skill folders (each has SKILL.md + references/ + scripts/ where needed)
  cover-letter/
  cv-tailor/            the most complex skill — docxtpl render pipeline, audit, bold helper
  interview-prep/
  job-discovery/
  job-search-pipeline/  the orchestrator; owns the skill router and shortcut DSL
  job-search-setup/     first-run wizard
  role-diagnosis/
  story-bank/
shared/                 example YAMLs (committed), scripts (text_to_docx.py), conventions.md
templates/              .docx template sets (OPUS is the primary; others are stubs)
tests/                  pytest suite — test_autoescape.py, test_bold_runs.py (5 tests)
data/                   gitignored — all user-generated artifacts (career.md, job log, sessions/)
settings.json           plugin-level permissions (python *, pip *)
requirements.txt        docxtpl, python-docx, docxcompose, openpyxl, PyYAML, pytest
```

## Key design rules

**Path conventions**
- Cross-skill references inside `skills/` use relative paths: `../sibling-skill/references/foo.md`
- References to `shared/` or `templates/` from inside skills use `${CLAUDE_PLUGIN_ROOT}/shared/...`
- Root YAML files (`config.yaml`, `branches.yaml`, `connectors.yaml`, `regional-headers.yaml`) are gitignored; `shared/*.example.yaml` are the committed templates

**cv-tailor render pipeline** (non-negotiable rules, hard-won from past failures)
- `autoescape=True` is mandatory on every `tpl.render()` call — omitting it silently strips ampersands
- Call `convert_content_map(cm)` (in `skills/cv-tailor/scripts/md_to_richtext.py`) immediately before render — converts `**phrase**` markers to docxtpl RichText bold runs
- `run_full_audit()` (in `skills/cv-tailor/scripts/audit.py`) runs after every render; a failed audit means the CV is not shipped
- The template file must not be open in Word when rendering — docxtpl will silently write a corrupt file

**Skill router** (lives in `skills/job-search-pipeline/SKILL.md`)
- `job-search-pipeline` is the single entry point for multi-step tasks and all `Run [...]` shortcut commands
- `role-diagnosis` is a hard gate before `cv-tailor` — no Diagnosis.md means no CV (except `Run CV only`)
- Opinionation mode defaults to `warn-once-then-comply`; `opinionation: strict` in config.yaml makes gates hard refuses

**`~~` placeholder convention**
- Tool-agnostic public plugin: third-party product names in body text are wrapped as `~~job board (Indeed)`, `~~web scraper (Apify)` etc.
- `CONNECTORS.md` at repo root documents the full placeholder system
- Never apply `~~` to SKILL.md frontmatter `name` or `description` fields, YAML keys, or Python strings

## Testing

```
python -m pytest tests/ -v          # must be green before any PR
claude plugin validate .             # must pass with no warnings
```

`tests/conftest.py` adds `skills/cv-tailor/scripts` to `sys.path`.

## Data folder structure (gitignored)

```
data/
  career.md                         candidate's full career file
  Blacklist.txt                     companies to skip in discovery
  Session Notes.txt                 running anomaly log
  voice/                            tone/voice reference files
  job-log/Job Listings.xlsx         append-only job log (openpyxl)
  sessions/[dd.mm]/[Country]/       per-day, per-geography output
    Diagnosis - [Company] - [Title].md
    CV - [Company] - [Title].docx
    Cover Letter - [Company] - [Title].docx
    LinkedIn Messages.txt
```

## Branch / PR workflow

Main branch is `main`. Feature work goes on named branches, PR to `main`. CI runs on push (`.github/workflows/ci.yml`): SKILL.md frontmatter lint, pytest.
