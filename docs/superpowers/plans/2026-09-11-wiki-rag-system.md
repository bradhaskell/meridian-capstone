# Project Wiki RAG System Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a Claude Code skill that maintains a persistent, cross-linked Markdown knowledge base (`wiki/`) over this repo's own project documentation, queryable via `/wiki`.

**Architecture:** A single skill definition (`.claude/skills/wiki/SKILL.md`) implements four subcommands — `init`, `ingest`, `query <question>`, `lint` — dispatched by the agent reading and following that file's instructions when invoked. No scripts, no API calls, no vector DB: the agent itself reads `raw/`/`docs/` sources and writes/maintains Markdown pages under `wiki/`.

**Tech Stack:** Markdown only. No new dependencies, no API keys.

**Spec:** `docs/superpowers/specs/2026-09-11-wiki-rag-system-design.md`

**Status:** All 6 tasks complete. Final whole-branch review found 5 Important + 6 Minor findings; one fix wave addressed all 5 Important (guard-scope gaps, log format, SKILL.md documentation drift) plus the content-accuracy Minors; re-review confirmed all addressed, two low-severity residuals parked. Merged to `main` directly (no feature branch was used, by explicit choice).

## Global Constraints

- No API calls, no embeddings, no vector database, no `.env`/API keys — the agent invoking the skill is the entire retrieval+generation mechanism.
- NDA guard applies during `ingest`: skip any file (without reading its content) whose extension isn't `.md`/`.txt`, or whose path/filename (case-insensitive) contains `extract`, `pos`, `loyalty`, `labor`, `transaction`, `payroll`, `employee`, or `customer`. Every skip is logged, never silent.
- `raw/` and `docs/` are read-only inputs — `ingest` never modifies them.
- `wiki/` content is committed to git (not gitignored) — it's a persistent, compounding artifact per the spec.
- `query` never guesses when the wiki doesn't cover a question — it says so plainly.
- `lint` only reports findings; it edits nothing without the user confirming first.
- Filing a query's synthesis into `wiki/analyses/` requires explicit user confirmation — never automatic.

---

### Task 1: Write the wiki skill definition

**Files:**
- Create: `.claude/skills/wiki/SKILL.md`

**Interfaces:**
- Produces: a skill named `wiki`, invoked as `Skill({skill: "wiki", args: "<subcommand> [question]"})` where subcommand is `init`, `ingest`, `query`, or `lint`. All later tasks consume this.

- [x] **Step 1: Write the skill file**

```markdown
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
  - Never ingest a file the NDA guard flags (see this skill's
    `ingest` section) — data classification is documented in
    `docs/data-handling-checklist.md`.
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

For every `*.md` and `*.txt` file under `raw/` and `docs/`
(recursively), in this order:

1. **NDA guard.** Skip the file — without opening or reading its
   content — if:
   - its extension is anything other than `.md` or `.txt`, or
   - its path or filename, lowercased, contains any of: `extract`,
     `pos`, `loyalty`, `labor`, `transaction`, `payroll`, `employee`,
     `customer`

   For a skipped file, append to `log.md`:
   `## [DATE] skip | <path> — matched guard rule: <rule text>`
   and do not create or touch any wiki page for it.

2. **Already-ingested check.** If `log.md` already has an `ingest`
   line naming this exact path and the file's content hasn't changed
   since, skip it silently (no new log line).

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
   `## [DATE] ingest | <Title>`

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
  its own in `entities/` or `concepts/`
- Any page that appears to contain row-level personal data — names
  paired with contact info, individual schedules, or individual
  transaction/loyalty records — as a backstop in case something
  restricted got in despite the `ingest` guard

If the user asks you to fix any finding, make the edit and append a
`## [DATE] lint-fix | <what changed>` line to `log.md`.
```

- [x] **Step 2: Verify the file**

Read `.claude/skills/wiki/SKILL.md` back and confirm:
- Frontmatter has `name: wiki` and a `description:` line
- All four subcommands (`init`, `ingest`, `query`, `lint`) have their own section
- The NDA guard keyword list matches the Global Constraints above exactly
- `query` and `lint` both handle being run before `wiki/` exists (tell the user to run `init` first, don't fail confusingly)

**Done looks like:** the file exists with all four subcommand sections and the exact guard list.
**How to check:** open `.claude/skills/wiki/SKILL.md` and read it top to bottom against the checklist in Step 2.

**Review notes:**
- What I asked the AI to do: write the complete `SKILL.md` skill definition (init/ingest/query/lint subcommands, the NDA guard keyword list) verbatim to the plan's template.
- How I checked its work: read the saved file back line by line against the Step 2 checklist — frontmatter present, all four subcommand sections present, guard list matching the Global Constraints exactly, `query`/`lint` both handling a missing `wiki/`.
- What I accepted/changed/rejected, and why: accepted as written. Every checklist item matched on the first pass — no gaps found, nothing to send back.

- [x] **Step 3: Commit**

```bash
git add .claude/skills/wiki/SKILL.md
git commit -m "feat: add wiki skill definition (init/ingest/query/lint)"
```

---

### Task 2: Scaffold the wiki with `/wiki init`

**Files:**
- Create (by running the skill): `wiki/index.md`, `wiki/log.md`, `wiki/conventions.md`, `wiki/overview.md`, `wiki/sources/.gitkeep`, `wiki/entities/.gitkeep`, `wiki/concepts/.gitkeep`, `wiki/analyses/.gitkeep`

**Interfaces:**
- Consumes: the `init` section of `.claude/skills/wiki/SKILL.md` from Task 1.
- Produces: the `wiki/` directory structure that Tasks 3–6 read and write into.

- [x] **Step 1: Run init**

Invoke `Skill({skill: "wiki", args: "init"})`. If the harness reports the skill isn't recognized in this session (it may not appear in the skill listing established at session start), open `.claude/skills/wiki/SKILL.md` directly, read its `init` section, and carry out those exact steps by hand — the resulting files must be identical either way.

- [x] **Step 2: Verify the structure**

List `wiki/` and confirm every file/folder from the Files section above exists, and that `index.md`, `log.md`, `conventions.md`, `overview.md` each match the templates in `SKILL.md`'s `init` section exactly (substituting today's date in `log.md`).

**Done looks like:** `wiki/` contains `index.md`, `log.md`, `conventions.md`, `overview.md`, and four empty subfolders each holding a `.gitkeep`, with content matching the `init` templates.
**How to check:** run a directory listing of `wiki/` and read each of the four top-level files.

- [x] **Step 3: Commit**

```bash
git add wiki/
git commit -m "chore: scaffold wiki/ structure via /wiki init"
```

---

### Task 3: Ingest the existing project docs

**Files:**
- Create (by running the skill): `wiki/sources/client-brief.md`, `wiki/sources/data-handling-checklist.md`, plus whatever `wiki/entities/*.md` and `wiki/concepts/*.md` pages the ingest cascade warrants (at minimum, expect pages for Dana Okafor and Meridian Markets, since both are named repeatedly in the client brief)
- Modify (by running the skill): `wiki/index.md`, `wiki/log.md`, `wiki/overview.md`

**Interfaces:**
- Consumes: the `ingest` section of `SKILL.md`, and the `wiki/` skeleton from Task 2.
- Produces: populated wiki content that Tasks 4–6 query, lint, and extend.

- [x] **Step 1: Run ingest**

Invoke `Skill({skill: "wiki", args: "ingest"})`, with the same manual fallback as Task 2 Step 1 if the skill isn't recognized in-session. This should process `raw/client-brief.md` and `docs/data-handling-checklist.md` (the only two `.md`/`.txt` files under `raw/`/`docs/` right now).

- [x] **Step 2: Verify source pages**

Read `wiki/sources/client-brief.md` and `wiki/sources/data-handling-checklist.md`. Confirm each:
- Cites the correct raw path (`raw/client-brief.md` / `docs/data-handling-checklist.md`)
- Has an accurate summary and key points — spot-check against the actual source file, don't just trust it looks plausible

- [x] **Step 3: Verify cascade, index, log, and overview**

- Confirm any `entities/`/`concepts/` pages created are warranted (they name something the client brief or checklist actually discusses) and each has a `## Mentioned in` link back to the right source page.
- Confirm `wiki/index.md` lists both source pages under `## Sources`, plus any entity/concept pages under their headings, each with an accurate one-line summary.
- Confirm `wiki/log.md` has two new `## [DATE] ingest |` lines.
- Confirm `wiki/overview.md` is no longer the "not yet populated" stub — it should now describe Meridian Markets and this project in a few sentences.

**Done looks like:** two accurate source pages exist, `index.md`/`log.md`/`overview.md` all reflect them, and any cascade pages are real and correctly cross-linked.
**How to check:** read every file touched and compare its claims against `raw/client-brief.md` and `docs/data-handling-checklist.md` directly.

**Review notes:**
- What I asked the AI to do: run `/wiki ingest` on `raw/client-brief.md` and `docs/data-handling-checklist.md`, producing source pages, cascade entity/concept pages, and updated `index.md`/`log.md`/`overview.md`.
- How I checked its work: read every generated page and traced individual claims back to the exact source sentence — e.g. the $78M revenue and ~620-employee figures in `wiki/sources/client-brief.md` were checked word-for-word against `raw/client-brief.md`'s "About us" section, and the Restricted/Conditional/Safe classification table in `wiki/sources/data-handling-checklist.md` was checked against `docs/data-handling-checklist.md`'s actual table.
- What I accepted/changed/rejected, and why: initially rejected as incomplete. The agent correctly skipped ingesting this project's own `docs/superpowers/` plan/spec files, but never updated `SKILL.md` to document that exclusion — leaving the rule un-codified for the next run. Sent back with the finding; accepted once `SKILL.md` was amended with an explicit exclusion bullet and a re-review confirmed it.

- [x] **Step 4: Commit**

```bash
git add wiki/
git commit -m "feat: ingest client brief and data handling checklist into wiki"
```

---

### Task 4: Verify the NDA guard

**Files:**
- Create then delete (scratch, never committed): `raw/pos_extract_test.md`
- Modify (by running the skill): `wiki/log.md`

**Interfaces:**
- Consumes: the `ingest` section of `SKILL.md` (guard logic) and the populated wiki from Task 3.
- Produces: a log entry proving the guard fires; no other lasting artifact.

- [x] **Step 1: Create a guard-triggering scratch file**

Write `raw/pos_extract_test.md` with placeholder content, e.g.:

```markdown
# Test file

This file exists only to verify the wiki ingest guard skips it. Not
real data.
```

- [x] **Step 2: Run ingest again**

Invoke `Skill({skill: "wiki", args: "ingest"})` (or the manual fallback).

- [x] **Step 3: Verify the skip**

Confirm:
- `wiki/log.md` has a new `## [DATE] skip |` line naming `raw/pos_extract_test.md` and citing the matched rule (its filename contains both `pos` and `extract`)
- No file under `wiki/sources/`, `wiki/entities/`, or `wiki/concepts/` references this file
- `wiki/index.md` has no entry for it

- [x] **Step 4: Delete the scratch file**

```bash
rm "raw/pos_extract_test.md"
```

**Done looks like:** the guard skipped the file and logged why, and no wiki page was created from it.
**How to check:** read the new `log.md` line, and grep `wiki/` for `pos_extract_test` — it should appear nowhere except that one log line.

**Review notes:**
- What I asked the AI to do: create a scratch file with a restricted-looking filename (`raw/pos_extract_test.md`), re-run `/wiki ingest`, confirm the NDA guard skipped it and logged why, then delete the scratch file.
- How I checked its work: read the new `wiki/log.md` line myself and grepped the whole `wiki/` folder for the scratch filename to confirm it appeared nowhere except that one log line — i.e. the file's content was never actually read into any wiki page.
- What I accepted/changed/rejected, and why: accepted. The guard fired exactly as designed, the skip was logged with the correct matched rule (`extract`/`pos`), and no wiki page referenced the restricted file — nothing to change.

- [x] **Step 5: Commit**

```bash
git add wiki/log.md
git commit -m "test: verify NDA ingest guard skips a restricted-looking filename"
```

(The scratch file itself is never staged — it was deleted in Step 4.)

---

### Task 5: Verify querying

**Files:** none (read-only verification)

**Interfaces:**
- Consumes: the `query` section of `SKILL.md` and the populated wiki from Task 3.

- [x] **Step 1: Ask an in-scope question**

Invoke `Skill({skill: "wiki", args: "query What data is restricted from AI tools?"})`.
**Expected:** an answer naming loyalty and labor data as restricted, citing `wiki/sources/data-handling-checklist.md` (or a concept page derived from it).

- [x] **Step 2: Ask another in-scope question**

Invoke `Skill({skill: "wiki", args: "query What is Meridian's annual revenue?"})`.
**Expected:** an answer citing roughly $78M, citing `wiki/sources/client-brief.md`.

- [x] **Step 3: Ask an out-of-scope question**

Invoke `Skill({skill: "wiki", args: "query What is the capital of France?"})`.
**Expected:** the answer states this isn't covered by the wiki, and does not answer from outside knowledge.

**Done looks like:** both real questions get correct, cited answers; the nonsense question gets an honest "not covered" response.
**How to check:** read all three responses and confirm the citations point to files that actually contain the claimed information.

No commit — this task is read-only (skip filing anything to `wiki/analyses/` for this verification pass).

---

### Task 6: Verify linting

**Files:** none, unless a real finding is fixed

**Interfaces:**
- Consumes: the `lint` section of `SKILL.md` and the full wiki state after Tasks 3–4.

- [x] **Step 1: Run lint**

Invoke `Skill({skill: "wiki", args: "lint"})`.

- [x] **Step 2: Review the report**

Given the wiki only has two real sources at this point, expect zero or very few findings. Confirm any finding reported is real (e.g. a genuine orphan page), not a false positive.

- [x] **Step 3: Fix only if something real is found**

If a finding is real, ask for confirmation, apply the fix, and append a `## [DATE] lint-fix |` line to `wiki/log.md`.

**Done looks like:** lint runs without error and produces an accurate report (few or no findings expected at this size).
**How to check:** read the lint output; for any finding, verify it by opening the pages named and confirming the problem is real.

**Result:** lint ran clean (no Critical/Important findings); one Minor observation (a "loyalty program" cascade page could be defensible) was reviewed and parked rather than fixed — `SKILL.md`'s "create only what's warranted" already covers the call. No fix needed, so Step 4 did not apply.

- [x] **Step 4: Commit (only if Step 3 made changes)** — N/A, no fix was made in Step 3.
