---
name: reverse-engineer-rebuild
description: Reverse-engineer any running reference or legacy system (web dashboards, portals, mobile apps, APIs, databases, real data exports) into a verified spec, recovered business formulas, a gap analysis and a card-based rebuild plan, then execute the rebuild through a gated test-first pipeline with parallel agents. Use when the user wants to analyze a system in order to re-program, clone or migrate it (تحليل سيستم بالهندسة العكسية، إعادة برمجة سيستم، نسخة من سيستم قديم، تحليل سيستم موجود عشان نبنيه من جديد), extract its pricing/finance/business rules from real records, or plan and run the rebuild in batches on one or more machines.
---

# Reverse-engineer → spec → formulas → gaps → cards → gated rebuild

A stack-agnostic playbook for turning a running system you do not have (full) source for into a
rebuilt system that behaves the same, with evidence for every claim. It was distilled from a real
run (14 domain reports, ~70 screens crawled, finance formulas recovered from ~13,000 real
transactions with golden vectors, 40+ owner decisions, ~10 merged batches run by up to 19
parallel agents on two machines) and generalized: nothing here depends on a language, framework
or domain.

Reference files, read when you reach the matching phase:
- `references/phases.md` — the 12 phases, step by step, with deliverables and numbering.
- `references/formula-recovery.md` — recovering business formulas from real data.
- `references/templates.md` — report / spec / gap / decision / card / golden-vector / handoff templates.
- `references/pitfalls.md` — incidents a real run hit, and the rule each one produced.

Vocabulary: **reference system** = the system being analyzed; **owner** = the person who owns
it and decides; **our codebase** = where the rebuild happens; **card** = one unit of work with
its own branch; **gate** = an automated check that must pass before a merge.

## 0. First conversation with the owner (ask before analyzing)

Ask with AskUserQuestion, one topic per question, your recommendation first and marked as the
recommended option, in the owner's language:
1. **What exists to analyze?** Live URLs (admin / portal / API, test and production), API
   collections (Postman/Insomnia/OpenAPI), app sources or binaries, DB dumps or exports
   (CSV/XLSX/JSON), screenshots or videos, written docs, the owner's own memory of the rules.
2. **What access is allowed?** Read-only reads are the default. Any write (including "export"
   endpoints that are POST) needs a named exception. Production data: read-only, PII never stored.
3. **Target shape:** exact clone + additions only (default), or a redesign? Which surfaces are in
   scope (each web app, each portal/role view, each mobile app, integrations)? What from the
   existing codebase must be kept?
4. **Working rules:** owner's language, code/docs language, never-delete-code policy,
   test-first, which model implements and which reviews, notification channel, who pushes the
   main branch, how many machines/agents may run in parallel.

Record every answer in `plans/<ref>/17-owner-decisions.md` (numbered, dated, with the
recommendation offered and the final answer). `<ref>` is a short slug for the reference system.

## 1. Non-negotiables (security and scope)

- Read-only against the reference system. Rate ≈1 request/s, small page sizes (≤10) on lists,
  no writes unless a named exception exists.
- Tokens, API keys, passwords: read into shell variables at runtime, never echoed, never written
  to the repo or the scratchpad. Credential files handed over by the owner are deleted after use.
  If a secret appears in chat, tell the owner to rotate it or log out, and write that in the handoff.
- PII (names, phones, emails, addresses, coordinates, national IDs, plates, device IDs, images)
  never enters the repo or any analysis output. Reports are **shape-only**: enums, statuses, counts
  and amounts stay; identities become `<REDACTED>` / `<uuid>`.
- Owner-provided exports live outside the repo (e.g. `~/Documents/<ref>-export/`, `chmod 600`).
- A denied tool permission is a stop, not a workaround. Never route it through another agent or
  session. Give the owner the exact permission line to add; never edit your own permission settings.
- Scope lock per card; report anything outside it, don't touch it.

## 2. Evidence rules (apply to every sentence written about the reference system)

- **Precedence on conflict:** owner-stated rule > real transaction data > live API responses >
  live UI crawl > app source / built bundles > API collections (often stale) > third-party docs >
  inference.
- **Tag every claim:** `EVIDENCE` (seen live or in data), `INFERRED` (deduced from code/UI),
  `PROPOSED` (our design), `UNVERIFIED` / `UNVERIFIED-LIVE` (not yet seen; carry the tag through
  every later document until resolved). Cite the source as `<report> §<section>`.
- Every report ends with `§9 Open questions`; the master spec collects them in `§15 Remaining
  unknowns` (CLOSED / PARTIAL / OPEN) and reference-system defects in `§14 Defects (do not copy)`.
- A formula is "recovered" only when it reproduces ≥95% of a real sample exactly (to the smallest
  unit) and every miss is explained (rounding mode, a reference bug, a missing input). Anything
  less is a hypothesis.

## 3. The method at a glance

| # | Phase | Output (under `plans/<ref>/`) |
|---|---|---|
| 1 | Source inventory + safety setup | `00-sources.md`; access rules in the decisions doc |
| 2 | Domain reports, one per area, parallel agents | `01..NN-<area>.md` (auth/roles/settings, core entity lifecycle, money rules, each user type, each app, each portal, integrations, built bundles) |
| 3 | Master spec synthesis | `00-master-spec.md` (changelog, precedence, §14 defects, §15 unknowns) |
| 4 | Live API verification (read-only) | `13-live-verification-<surface>.md` per surface; the UNVERIFIED table resolved |
| 5 | UI crawl, screen by screen | `22-live-crawl-gaps.md` + `gaps/<area>.md` with ✅/⚠️/❌ per field; screenshots outside the repo |
| 6 | Real-data formula recovery | `23-data-analysis.md`, `24-formulas-golden-vectors.md`, `25-formula-summary-<owner-lang>.md` |
| 7 | Reports and exports parity | `26-reports-compare.md`, `reports-parity.md` |
| 8 | Owner decisions | `17-owner-decisions.md` (numbered, dated, recommendation + answer) |
| 9 | Gap analysis vs our codebase | `15-gap-analysis.md` (EXISTS / PARTIAL / MISSING / ADDITIVE-EXTRA / DEFECT-EXCLUDED) |
| 10 | Clone plan → cards | `16-clone-plan.md` (scope edits, new cards, critical path, sizes S/M/L) |
| 11 | Gated rebuild in batches | branches + `STATUS.md`, `HANDOFF.md`, owner report, `TIMELINE.md`, merge log |
| 12 | Parity audits + ETA | `PARITY-REMAINING.md` with counts (done / partial / missing) and days left |

Phases 2, 4, 5 and 6 run as parallel agents (a fast model for extraction, a strong model for
synthesis and review; use a second strong model to cross-check recovered formulas independently).

## 4. The rebuild pipeline (phase 11), compressed

1. One card = one branch = one isolated working copy (copy dependency folders, never symlink;
   copy env files; unique migration/schema timestamps across open branches).
2. Implementing agent: tests first and keep the **RED evidence** (the failing summary before any
   production code), a case-matrix docblock per new/changed function, size limits agreed with the
   owner (e.g. class ≤250 lines, method ≤30, view ≤150), never delete code (comment out with a
   `DISABLED(<card>, <date>): reason` marker or move to a `disabled_code/` mirror), all user-facing
   labels in every supported language.
3. Reviewing agent checks the report; a card without RED evidence goes back.
4. Cards merge `--no-ff` into an integration branch (≈8 cards per batch), synced with `main`,
   then one combined verify: unit/feature tests, linters, type-check, secret scan, diff coverage
   on changed lines (Lines ≥95%, Branches ≥90%), size gate, duplication gate, never-delete guard.
   Split the verify across machines when there are several (e.g. backend + web on one, mobile on
   another).
5. All gates pass → merge into local `main` with `--no-ff` → the owner pushes.
6. After every merge: update status / handoff / owner report / timeline / merge log, notify the
   owner, and queue the next card per lane before the current one merges.
7. Keep every machine at capacity (≈7–10 agents each). Watch the other machine's pushes with a
   monitor re-armed every 30 minutes; accept a card only from its FINAL report (SHA + analysis
   result + gates), never from a push alone; keep an explicit list of superseded SHAs.

## 5. Owner communication

- Every "where are we?" answer gives: what merged, what is running (agents per machine, in
  product terms), what is blocked on the owner, and a completion estimate with its assumption
  (e.g. "full verify ≈70 min, one batch ≈3 h").
- Every decision goes through AskUserQuestion with the recommendation marked. When told not to
  ask (night shift), take the recommendation, mark it "interim, open for review" in the decisions
  doc, and list it in the next status message.
- Explain a finding in plain language with one concrete, de-identified example before the formula.
- Before a usage limit or the end of a session: a HANDOFF §0 block and a notification with resume steps.

## 6. Done criteria for the analysis half

- The master spec has no `UNVERIFIED` item that a read-only call could have resolved.
- Every money/business formula has golden vectors from real data committed as tests.
- Every reference-system defect is listed and marked "not copied".
- Every open product question is in the decisions doc with a recommendation.
- Gap-analysis rows map 1:1 to cards with sizes and a critical path; the owner has an ETA.
