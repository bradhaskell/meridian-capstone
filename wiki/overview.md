# Overview

Meridian Markets is a specialty grocery chain with fourteen stores
across Los Angeles, Orange, and Ventura counties (~$78M annual
revenue, ~620 employees), grown from six stores in five years by
taking over leases in neighborhoods other chains had left. This
project is an LMU MSBA workshop engagement kicked off by a brief from
Dana Okafor, Meridian's VP of Operations: build a dashboard analyzing
sales performance by store and category, partly to support a decision
on a likely next location (Pasadena) and partly to make better use of
an underused loyalty program (~40,000 members). The engagement runs
roughly eight weeks, with a preliminary look due for the board in
three weeks.

Data handling is tightly constrained by an NDA (see
[`wiki/concepts/nda.md`](concepts/nda.md)): customer records and
employee data — loyalty program data and labor schedules, in full or
as excerpts — may never be entered into any AI tool. The team's data
handling checklist formalizes this into a data classification table
(see [`wiki/concepts/data-classification.md`](concepts/data-classification.md))
that also flags raw POS transactions as conditional (safe only once
aggregated to store + week totals) and clears sales totals and store
attributes for AI tool use.

For external market context — not Meridian-specific data — see
[`wiki/concepts/grocery-industry-trends.md`](concepts/grocery-industry-trends.md):
specialty/fresh-format grocers are gaining share from traditional
chains, big-box players are reinvesting in stores, and grocery-anchored
real estate investment is surging, all relevant backdrop as Meridian
weighs further expansion.

For the Pasadena decision specifically, see
[`wiki/concepts/retail-site-selection.md`](concepts/retail-site-selection.md):
a structured 10-step location-evaluation framework (network mapping,
competitive effects, customer-profile matching, physical/financial
feasibility, grocery-specific weighting on parking and trade-area
saturation, and post-opening validation) that maps closely onto the
kind of analysis Dana wants before committing to the site.
