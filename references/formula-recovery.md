# Recovering business formulas from real data

The owner's memory gives the *shape* of a rule; only real records give the exact rule
(rounding, order of operations, which inputs, which edge cases). This is how a full set of
pricing/payout/tax rules was recovered from ~13,000 production records in one night.

## 1. Pull the data (read-only, paged, PII-filtered at the edge)

1. Confirm the list endpoint and its paging, the detail endpoint, and the related entities
   needed as inputs (plans/tariffs, accounts, configuration rows).
2. Write one script that pages through everything with a 1 s pause, retries transient errors,
   and **strips PII before writing**: keep ids (hashed if only needed as keys), statuses,
   timestamps, amounts, quantities, plan references, flags. Drop names, contacts, addresses,
   coordinates, images.
3. Store under `~/Documents/<ref>-export/` (outside the repo, `chmod 600`). Record counts and
   the date range in the analysis doc.
4. Pull the reference entities the same way (plans with every rate field, accounts with their
   plan links and flags).

## 2. Build the candidate formula set

From the UI/code reports, write every formula as a hypothesis with its inputs. Typical shapes:
a fixed amount; a base quantity plus a per-unit rate beyond it; ordered brackets/tiers;
fixed overheads; conditional fees (only for one payment method, only for one order type);
taxes on a defined base; caps and floors; special cases for grouped/child records;
cancellation rules by actor and stage; dispute outcomes per party.

## 3. Test each hypothesis against the sample

For each real row compute the hypothesis and compare with the stored amounts. Report matches as
`n/N` per rule variant. Investigate the misses in clusters, not one by one — they group by:
- **rounding mode**: truncate vs half-up vs banker's; applied on the unrounded total vs per
  line; sometimes different per payment method;
- **order of operations**: tax on which base; a cap applied before or after tax;
- **quantity definition**: e.g. a distance measured as a straight line vs along a route, raw vs
  rounded, measured between which two points;
- **a missing input**: a flag on the account or plan that changes the rule;
- **a bug in the reference** (charges that should be zero, payouts that are missing): list it in
  §14 defects, ask the owner, do not copy.
Stop when the remaining misses are all explained and the owner has confirmed the bug list.

## 4. Golden vectors

Pick 40–60 real rows covering every rule variant, payment method, grouped/child case,
cancellation by each actor, dispute-ended case, zero-quantity, cap-hit. Store them as a table
in `24-formulas-golden-vectors.md` with **inputs and expected outputs only** (no identities)
and turn them into a committed test. The rebuild is "finance-correct" only when every vector
passes to the smallest unit.

## 5. Independent cross-check

Hand the same raw sample and hypothesis list to a second strong model with no access to the
first analysis. Compare: agreed rules, disagreements (resolve with more rows), what neither
could find (ask the owner). Record the second report verbatim in the summary.

## 6. Deliverables

- `23-data-analysis.md`: data counts, per-formula match rates, miss clusters, defects.
- `24-formulas-golden-vectors.md`: vectors + the exact rule text.
- `25-formula-summary-<owner-lang>.md`: for the owner, one worked example per rule, the rounding
  rules in a table, "differences with our code" (→ cards), "still open" (→ decisions).
- Cards: rule parity, rounding, caps, any external measurement provider (with fallbacks and the
  env keys needed at deploy), report columns the rules imply.

## 7. Measured quantities, the usual trap

When a rule depends on a measured quantity (distance, duration, weight), check early how the
reference measures it by comparing stored values with your own computation on a sample: a
stable ratio ≈1.0 means the same method; a stable other ratio means a different method (e.g.
road vs straight-line distance). Confirm with the owner which providers are used and when.
Design: a provider strategy with a fallback, the raw unrounded value into the formula, provider
and keys from configuration, never hardcoded.
