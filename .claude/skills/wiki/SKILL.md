---
name: wiki
description: Maintain and query this project's persistent knowledge wiki under wiki/, built by ingesting raw/ and docs/ sources. Subcommands: init, ingest, query <question>, lint.
---

# Project wiki

This skill maintains `wiki/`, a persistent, cross-linked Markdown
knowledge base built from this repo's own project documentation. It
is NOT a tool for analyzing Meridian's grocery data — see the NDA
guard below.

Parse `args` for the subcommand (the first word) and its remainder:

- `init` — scaffold the wiki structure
- `ingest` — process new/changed files from `raw/` and `docs/`
- `query <question>` — answer a question from the wiki
- `lint` — audit the wiki for consistency problems

If `args` is empty or doesn't match one of these, ask the user which
subcommand they meant.

## `init`

Create, if missing:

- `wiki/index.md` — start with:
  ```markdown
  # Wiki index

  See [Overview](overview.md) for a synthesis of the project.

  ## Sources

  ## Entities

  ## Concepts

  ## Analyses
  ```
- `wiki/log.md` — start with:
  ```markdown
  # Wiki log

  Append-only. Newest entries at the bottom. One line per action:
  `## [YYYY-MM-DD] <action> | <Title>` where action is `init`,
  `ingest`, `skip`, `lint-fix`, or `analysis`.

  ## [DATE] init | Wiki created
  ```
  (substitute today's actual date for DATE)
- `wiki/conventions.md` — start with:
  ```markdown
  # Conventions

  - One page per source in `sources/`, named by a kebab-case slug of
    its title (e.g. `client-brief.md`).
  - Every wiki page cites the raw file(s) it was built from.
  - `entities/` = people and organizations. `concepts/` = abstract
    ideas, terms, policies. `analyses/` = synthesized answers to
    complex questions, filed back only with the user's confirmation.
  - Never ingest a file the NDA guard flags (see the `ingest` section
    of `.claude/skills/wiki/SKILL.md`) — data classification is
    documented in `docs/data-handling-checklist.md`.
  - Files under `docs/superpowers/**` are excluded from ingest
    entirely — they're this wiki-building project's own process
    artifacts, not project knowledge (see `.claude/skills/wiki/SKILL.md`).
  - Keep `index.md` and `log.md` up to date with every change.
  ```
- `wiki/overview.md` — start with:
  ```markdown
  # Overview

  Not yet populated. Run `/wiki ingest` to build this from `raw/`
  and `docs/` sources.
  ```
- `wiki/sources/.gitkeep`, `wiki/entities/.gitkeep`,
  `wiki/concepts/.gitkeep`, `wiki/analyses/.gitkeep` — empty files,
  so the empty directories are tracked by git.

If any of these already exist, leave them alone — `init` never
overwrites existing content.

## `ingest`

Scan ALL files under `raw/` and `docs/` (recursively) — every file,
regardless of extension — excluding anything under
`docs/superpowers/**` (match this exclusion against both `/` and `\`
path separators, since this repo runs on Windows). Those files are
this wiki-building project's own process artifacts (specs/plans), not
project knowledge, so they are never enumerated in the first place and
never logged.

For each remaining file, in this order:

1. **NDA guard.** Skip the file — without opening or reading its
   content — if:
   - its extension is anything other than `.md` or `.txt`, or
   - its path or filename, lowercased, contains any of: `extract`,
     the standalone word `pos` (matched as a whole word or path
     segment, not as a substring of a longer word — so `proposal.md`
     or `position.md` do NOT match), `loyalty`, `labor`,
     `transaction`, `payroll`, `employee`, `customer`, `member`,
     `schedule`, `shift`, `timesheet`, `hours`, `roster`, `staff`,
     `pii`, or `personal`.

   For a skipped file, append to `log.md`:
   `## [DATE] skip | <path> — matched guard rule: <rule text>`
   and do not create or touch any wiki page for it. If a file is
   still guard-flagged on a later run, it will get another `skip` log
   line each time ingest runs — this repeat logging is expected and
   by design, not a bug.

2. **Already-ingested check.** If `log.md` already has an `ingest`
   line naming this exact path (the log format is
   `## [DATE] ingest | <path> | <Title>`) and the file's content
   hasn't changed since, skip it silently (no new log line).

3. **Summarize.** Read the file. Write or update
   `wiki/sources/<slug>.md`:
   ```markdown
   # <Title>

   Source: `<raw/or/docs path>`

   ## Summary

   <2-5 sentence summary>

   ## Key points

   - <point, each citing the section it came from if the source has headings>
   ```

4. **Cascade.** Identify people, organizations, and concepts worth
   their own page (e.g. a named client contact, a defined policy
   term). For each:
   - If `wiki/entities/<slug>.md` or `wiki/concepts/<slug>.md`
     doesn't exist, create it with a `# <Name>` heading, a one-line
     description, and a `## Mentioned in` list of source pages
     linking back to them.
   - If it exists, add this source to its `## Mentioned in` list if
     not already there.

   Don't force a fixed number of cascade pages — create only what's
   actually warranted by the source's content.

5. **Update the catalog.** In `wiki/index.md`, add or update a
   one-line entry (link + single-sentence summary) under the correct
   `## Sources` / `## Entities` / `## Concepts` heading.

6. **Update the overview.** If this source materially changes the
   project-level picture (e.g. the first real source, or a change to
   scope/timeline/participants), update `wiki/overview.md` to reflect
   it in a few sentences.

7. **Log it.** Append to `log.md`:
   `## [DATE] ingest | <path> | <Title>`

## `query <question>`

0. If `wiki/index.md` doesn't exist yet, tell the user to run
   `/wiki init` (and `/wiki ingest`) first — don't fail confusingly
   or fall back to reading raw sources directly.
1. Read `wiki/index.md` first. Identify which pages look relevant to
   the question — don't scan the whole wiki, and don't re-read raw
   sources directly.
2. Open only those specific pages under `sources/`, `entities/`,
   `concepts/`, `analyses/`.
3. Answer the question, citing the wiki page(s) used for each claim.
4. If nothing in the wiki covers the question, say so plainly. Do not
   guess or answer from outside knowledge of the raw sources.
5. If the answer required real synthesis across multiple pages, offer
   to file it as a new page in `wiki/analyses/`. Only create it if the
   user confirms; if they do, log it:
   `## [DATE] analysis | <Title>`

## `lint`

If `wiki/` doesn't exist yet, tell the user to run `/wiki init` first
rather than failing confusingly.

Read every page under `wiki/` and report, as a list (do not edit
anything without asking first):

- Direct contradictions between two pages
- A claim that looks superseded by a more recent source (check
  `log.md` order)
- Orphan pages: any page under `sources/`, `entities/`, `concepts/`,
  or `analyses/` with no link pointing to it from `index.md` or any
  other page
- A name/term referenced in at least two pages that has no page of
  its own in `entities/` or `concepts/` — unless its coverage in
  existing pages is already adequate; use judgment, don't force a
  page for the sake of the rule (this mirrors `ingest`'s own "create
  only what's warranted" principle)
- Any page that appears to contain row-level personal data — names
  paired with contact info, individual schedules, or individual
  transaction/loyalty records — as a backstop in case something
  restricted got in despite the `ingest` guard

If the user asks you to fix any finding, make the edit and append a
`## [DATE] lint-fix | <what changed>` line to `log.md`.
