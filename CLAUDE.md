# CLAUDE.md

Project instructions for anyone (or any agent) working in this repo.

## What this project is

Meridian Markets capstone workspace: the client brief and data
handling checklist, plus a persistent knowledge wiki built from those
project docs and public research sources.

## How the wiki works

- Sources live in `raw/` and `docs/` (client brief, data handling
  checklist, public research articles).
- The `/wiki` skill (`.claude/skills/wiki/SKILL.md`) is the actual
  operational definition — read it for the full procedure. Summary:
  - `/wiki init` — scaffold `wiki/` (`index.md`, `log.md`,
    `conventions.md`, `overview.md`, and `sources/`, `entities/`,
    `concepts/`, `analyses/`)
  - `/wiki ingest` — summarize new `raw/`/`docs/` sources into
    `wiki/sources/`, cascade entity/concept pages, update
    `index.md`/`log.md`/`overview.md`. Skips (and logs) any file that
    looks like restricted Meridian data, or this project's own
    process docs under `docs/superpowers/**`.
  - `/wiki query <question>` — answers from the wiki only, with
    citations to the wiki pages used; says plainly when something
    isn't covered rather than guessing.
  - `/wiki lint` — audits the wiki for contradictions, orphan pages,
    and accidental restricted-data leakage.

## Data boundaries (see `docs/data-handling-checklist.md`)

Customer/loyalty and employee/labor data must never reach an AI tool,
in full or as excerpts. Raw POS transactions are conditional — safe
only once aggregated to store + week totals. Sales totals and store
attributes are safe to use as-is.

## Checking the wiki

Every wiki page cites its source. To verify a claim: open
`wiki/index.md`, follow the citation to the relevant
`wiki/sources|entities|concepts|analyses/*.md` page, then open the raw
source it names under `raw/` or `docs/` and confirm the claim actually
appears there.
