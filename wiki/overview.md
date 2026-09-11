# Overview

Meridian Markets is a specialty grocery chain with fourteen stores
across Los Angeles, Orange, and Ventura counties (~$78M annual
revenue, ~620 employees), grown from six stores in five years by
taking over leases in neighborhoods other chains had left. This
project is an MSBA capstone engagement kicked off by a brief from
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
