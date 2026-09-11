# Data handling checklist: Meridian Markets

Internal reference for the team. Grounded in the NDA terms in
[`raw/client-brief.md`](../raw/client-brief.md): customer records and
employee data do not go into ChatGPT, Claude, Copilot, or any other AI
tool, in full or as excerpts. That rule is not negotiable.

## Open questions for Dana

Running list — add new questions as they come up, check them off once
answered (and note the answer inline or link to where it's recorded).

- [ ] Does the raw POS transaction data carry any customer or loyalty
  identifiers, or is it anonymized at the point of sale?
- [ ] Is there a data dictionary/schema for the POS extract, especially
  given the migration to a new system last spring (did the schema
  change mid-history)?
- [ ] Besides Marcus, is there anyone else on the technical/IT side we
  can go to with data questions?
- [ ] What format do you expect for the 3-week board preview —
  informal deck, working dashboard prototype, something else?
- [ ] Are there any expectations for where we store the data during
  the engagement, or any retention/deletion requirement once it ends?
- [ ] Does the loyalty program data include direct PII (name, email,
  phone, address), or only membership IDs and purchase history?
- [ ] Who at Meridian should review outputs before anything goes to
  the board or outside the core team?

## Requesting & receiving

- [ ] NDA signed before requesting the IT extract
- [ ] Extract request scoped to only what's needed for the current
  phase of work — don't request more than necessary
- [ ] Confirm the extract arrives via a secure channel (not a plain
  email attachment)
- [ ] Log what was received, when, and by whom (dataset, date range,
  file format)
- [ ] Confirm a data dictionary/schema is received alongside the data,
  or request one if missing
- [ ] Restricted files moved to secure storage immediately on receipt
  — not left sitting in Downloads, email, or a chat app

## Storing

- [ ] Restricted data (loyalty, labor) never placed in a public or
  broadly-shared cloud folder
- [ ] Restricted data never committed to the git repo, even
  temporarily
- [ ] Access to restricted files limited to team members actively
  working with them
- [ ] Working copies/exports of restricted data cleaned up once no
  longer needed, not left scattered across laptops
- [ ] Retention/deletion plan agreed with Dana for after the
  engagement ends

## AI tools

### Data classification

| Dataset | Status | Notes |
|---|---|---|
| Loyalty membership & purchase history | 🔴 Restricted | Never in any AI tool, in full or excerpted |
| Labor scheduling & hours | 🔴 Restricted | Never in any AI tool, in full or excerpted |
| Raw / line-item POS transactions | 🟡 Conditional | Restricted until aggregated to store + week totals — line items can join back to loyalty IDs |
| Sales totals by store & week | 🟢 Safe | Explicitly cleared by Dana |
| Store attributes (sqft, opening date, lease terms) | 🟢 Safe | Explicitly cleared by Dana |

### Before you paste or upload anything into an AI tool

- [ ] Identify which dataset(s) the file or excerpt draws from
- [ ] Look it up in the classification table above
- [ ] If **Restricted** — stop. Do not use any AI tool with this data.
- [ ] If **Conditional** (raw POS) — confirm it's aggregated to store + week totals and has no customer/loyalty ID columns before proceeding
- [ ] If **Safe** — still spot-check the actual rows/columns being pasted (not just the filename) to make sure no restricted columns snuck in via a join or export
- [ ] When unsure, ask the team lead or email Dana — don't assume "probably fine"

## Presenting & sharing

- [ ] Confirm every figure in a dashboard, deck, or export is an
  aggregate (store/week level or coarser) — never row-level
  transaction, loyalty, or labor data
- [ ] Check dashboard drill-downs/filters don't let a viewer get down
  to an individual customer or employee record
- [ ] Screenshots and exported files reviewed for hidden columns or
  tooltips that could expose restricted data
- [ ] Board or external-facing materials reviewed by the team before
  sending
- [ ] Sharing links (Drive, Slack, email) checked for correct
  permissions before sending — no broader access than intended
