# Conventions

- One page per source in `sources/`, named by a kebab-case slug of
  its title (e.g. `client-brief.md`).
- Every wiki page cites the raw file(s) it was built from.
- `entities/` = people and organizations. `concepts/` = abstract
  ideas, terms, policies. `analyses/` = synthesized answers to
  complex questions, filed back only with the user's confirmation.
- Never ingest a file the NDA guard flags (see the `ingest` section of
  `.claude/skills/wiki/SKILL.md`) — data classification is documented
  in `docs/data-handling-checklist.md`.
- Files under `docs/superpowers/**` are excluded from ingest entirely
  — they're this wiki-building project's own process artifacts, not
  project knowledge (see `.claude/skills/wiki/SKILL.md`).
- Keep `index.md` and `log.md` up to date with every change.
