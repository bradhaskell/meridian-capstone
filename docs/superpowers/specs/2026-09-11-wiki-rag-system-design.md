# Design: Project wiki RAG system

Date: 2026-09-11
Status: Approved design, pending spec review
Revision: regenerated to follow the agent-maintained wiki pattern from
https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f

## Purpose

A knowledge base over this repo's own project documentation (client
brief, data handling checklist, future notes/deliverables) — not over
Meridian's actual grocery data — so the team can ask things like "what
did Dana say about the timeline" and get an answer with a citation
back to the source doc.

This replaces the earlier embeddings/vector-search design entirely.
Instead of a script that calls Voyage AI + Claude APIs at query time,
this is an **agent-maintained wiki**: this coding agent (Claude Code)
incrementally reads raw sources and writes a persistent, cross-linked
Markdown knowledge base. Retrieval means the agent reads a catalog
page first, then drills into the relevant files — no embeddings, no
vector DB, no external API calls, no API keys.

Per the source pattern: raw sources are the ground truth, the wiki is
a compiled, compounding artifact built from them, and a schema file
governs how the agent maintains it. Obsidian/a file browser is the
IDE; the agent is the programmer; the wiki is the codebase.

## Non-goals

- Not a Q&A system over Meridian's sales/loyalty/labor data
- Not a script or service that calls an LLM API — no API keys, no
  embeddings, no vector search
- Not designed to run unattended — ingest/query/lint are commands a
  team member runs, with this agent doing the work interactively

## Layers

Reusing folders that already exist in this repo rather than inventing
new ones:

| Gist's layer | This repo |
|---|---|
| Raw sources (immutable, read-only) | `raw/` and `docs/` (existing project docs) |
| The wiki (agent-authored) | `wiki/` (new) |
| The schema | `.claude/skills/wiki/SKILL.md` (new) |

Using the skill file as the schema (instead of a separate top-level
`CLAUDE.md`) avoids having two rule files saying the same thing — the
skill definition is already the persistent, agent-facing instruction
set, and it's what actually runs when someone invokes `/wiki`.

## Commands

Implemented as a Claude Code skill, `.claude/skills/wiki/SKILL.md`,
invoked as `/wiki <subcommand>`:

- `/wiki init` — create the `wiki/` directory structure if it doesn't
  exist yet (`index.md`, `log.md`, `conventions.md`, `overview.md`,
  and empty `sources/`, `entities/`, `concepts/`, `analyses/` folders)
- `/wiki ingest` — process any file under `raw/` or `docs/` not yet
  reflected in `wiki/log.md`
- `/wiki query <question>` — answer a question from the wiki
- `/wiki lint` — audit the wiki for consistency problems

## `wiki/` structure

```
wiki/
  index.md          # catalog: every wiki page with a one-line summary, by category
  log.md             # append-only history: ## [YYYY-MM-DD] ingest | Title
  conventions.md     # maintenance rules and house style for this wiki
  overview.md        # entry-point synthesis of the project as a whole
  sources/           # one page per ingested raw/docs file
  entities/          # people and orgs: Dana Okafor, Marcus, Meridian Markets
  concepts/          # abstract ideas: NDA terms, data classification, timeline
  analyses/          # synthesized answers worth keeping permanently
```

## Ingest workflow

For each candidate file under `raw/` or `docs/`:

1. **NDA guard** (runs before the file's content is ever read into
   context): skip the file, and append a `## [DATE] skipped | <path>`
   line to `log.md` naming the matched rule, if:
   - its extension is not `.md` or `.txt`, or
   - its path or filename (case-insensitive) contains any of:
     `extract`, `pos`, `loyalty`, `labor`, `transaction`, `payroll`,
     `employee`, `customer`

   This matters more here than in a generic wiki: this project's
   `raw/` folder is exactly where Meridian's actual data extract will
   eventually land once IT delivers it, and per
   [`data-handling-checklist.md`](../../data-handling-checklist.md)
   that data must never reach an AI tool — including this agent. The
   guard is a heuristic backstop; it never substitutes for keeping
   restricted extracts out of `raw/`/`docs/` in the first place, per
   the checklist's "Storing" section.

2. **Skip if already ingested** — if `log.md` already has an `ingest`
   entry for this exact file with no changes since, skip it.

3. **Summarize and file** — read the surviving file, write or update
   `wiki/sources/<slug>.md` with a summary and key takeaways, citing
   the raw path. Update whichever `entities/`, `concepts/`, and
   `analyses/` pages are actually relevant (no fixed count — this
   project's corpus is small, so a doc might touch two or three
   related pages, not the 10-15 the source pattern describes for
   larger corpora).

4. **Update the catalog and log** — add/update the entry in
   `index.md`, append an `## [DATE] ingest | Title` line to `log.md`.

## Query workflow

`/wiki query <question>`:

1. Read `wiki/index.md` first to find candidate pages — not a scan of
   the whole wiki, and never a re-read of raw sources directly.
2. Drill into the specific `sources/`, `entities/`, `concepts/`, or
   `analyses/` pages that look relevant.
3. Answer with explicit citations to the wiki pages used (which in
   turn cite their raw sources).
4. If the answer isn't covered by any wiki page, say so plainly rather
   than guessing.
5. If the answer is a nontrivial synthesis (pulled from multiple
   pages, or required real reasoning), offer to file it into
   `wiki/analyses/` as a new page so future queries don't redo the
   work — only with confirmation, never automatically.

## Lint workflow

`/wiki lint` audits `wiki/` and reports (does not auto-fix without
confirmation):

- Direct contradictions between pages
- Claims that look outdated relative to a more recent source
- Orphan pages with no incoming links from `index.md` or other pages
- Concepts/entities referenced by name in multiple pages but missing
  their own page
- A content-level NDA check: any wiki page containing what looks like
  row-level personal data (names paired with contact info, employee
  schedules, individual transaction/loyalty records) — a backstop in
  case something restricted got in despite the ingest guard

## Error handling

- `/wiki query` or `/wiki lint` run before `/wiki init` — the agent
  creates the structure or tells the user to run `init` first, rather
  than failing confusingly
- A file the guard skips is always logged, never silently dropped
- A question with no matching wiki content gets an honest "not
  covered" answer, never a fabricated one

## Testing

No automated test suite — verification is manual, run once the skill
exists:

1. `/wiki init`, then `/wiki ingest` — confirm `wiki/sources/` gets
   pages for `raw/client-brief.md` and `docs/data-handling-checklist.md`,
   and `index.md`/`log.md` are populated correctly.
2. `/wiki query "What's restricted from AI tools?"` and `/wiki query
   "What's Meridian's annual revenue?"` — confirm correct, cited
   answers.
3. Add a dummy file matching a guard pattern (e.g. `raw/pos_extract.md`)
   and re-run `/wiki ingest` — confirm it's skipped and logged, not
   read.
4. `/wiki lint` — confirm it produces a sensible report against the
   small existing wiki (few/no findings expected at this size).

## Open questions

None blocking. The guard's keyword list and the entity/concept
taxonomy may need tuning once more real project content exists, but
both are small edits to `SKILL.md`, not design changes.
