# The 12 phases, step by step

Numbers in brackets are calibration points from one real run; scale them to the system at hand.

## Phase 1 — Source inventory and safety setup

1. List every source with its trust level and freshness (SKILL.md §2 precedence). Typical:
   - live hosts: each web UI, each API host (test and production), file storage/CDN;
   - API collections (Postman/Insomnia/OpenAPI) — stale until a live call confirms a route;
   - app sources or binaries (decompile only what the owner owns), built JS bundles of SPAs;
   - exports the owner can produce (core records, master data, report files);
   - the owner's memory (highest precedence for *intent*, lowest for exact numbers).
2. Write the access rules into the decisions doc: read-only, rate, page size, which writes are
   allowed by name, where exports are stored (outside the repo), the PII policy.
3. Credentials: owner provides a token once → read into a variable → delete the file → owner
   logs out / rotates at the end. Never print it. If a read returns 405 the route is a write:
   stop and ask for a named permission; do not probe for workarounds.
4. Create `plans/<ref>/` and the numbering scheme (two-digit prefix, letters for splits: `07a`,
   `07b`). Keep an index in `00-sources.md`.

## Phase 2 — Domain reports (parallel agents, one per area)

Give each agent: the sources it may read, the safety rules, the report template
(templates.md §1), a scope lock, and the instruction to end with `§9 Open questions`.
Typical split [14 reports]:
- authentication, roles/permissions, organisational hierarchy, global settings;
- the core entity's lifecycle (status machine), its money flows, disputes/exceptions;
- customers/tenants and their configuration, price plans, payout plans, transactions;
- workforce/resources, scheduling, their API;
- geography, notifications, reports, exports;
- payment modules and third-party integrations;
- each mobile app split in two (operational flow; money/account) from source or binary;
- each web surface (admin screens, customer portal, each role-scoped view);
- built bundles (route tables, API calls, realtime channels found in compiled JS).

Each report: actors; entities and fields with an evidence column; lifecycles as transition
tables; formulas as written in the UI/code; screens with fields and actions; API routes with
request/response shapes (shape-only); defects observed; open questions.

## Phase 3 — Master spec synthesis (strong model)

Merge the reports into `00-master-spec.md`: changelog header; the precedence rule used for
conflicts; one section per area in the same order as the reports; **§14 Defects (not copied)**;
**§15 Remaining unknowns** (CLOSED / PARTIAL / OPEN, each with the call that would resolve it);
a final section per app. Keep `UNVERIFIED` tags; never silently promote an inference to a fact.

## Phase 4 — Live API verification (read-only)

Goal: resolve §15 with read calls only. Rules: ≈1 request/s, page size ≤10, shape-only output,
login only through the documented login route. Output per surface
(`13a-live-verification-<surface>.md`): a table *source § → item → verified answer → evidence
call*, then the full field shapes of the key resources. Record anything surprising: non-JSON
responses, routes that 404, role gates, endpoints that ignore paging. Self-report any safety
slip in §0 of the report.

## Phase 5 — UI crawl, screen by screen

A read-only browser script: log in, visit every navigation route, full-page screenshot, open
each Add/Edit/View dialog, dump labels/inputs/selects, close. Screenshots and the label dump
live on the owner's machine (`~/Documents/<ref>-screens/`), not in the repo. Then compare field
by field with our code into `gaps/<area>.md` (✅ same, ⚠️ partial, ❌ missing) and a register
`22-live-crawl-gaps.md` with totals, caveats (cropped dialogs, dialogs not opened, which branch
was compared) and proposed card IDs [71 screens, 244 shots, 165 dialogs].

## Phase 6 — Real-data formula recovery

See `formula-recovery.md`. Outputs: analysis doc, golden-vectors doc, a summary in the owner's
language with one worked example per rule, and a "differences with our code" section that
becomes cards.

## Phase 7 — Reports and exports parity

Pull every report endpoint (shape) and every export the owner can produce (a test environment
is fine). For each: filters, columns, totals, export formats. Build `reports-parity.md`:
reference report → ours → done / partial (what is missing) / missing. Mark owner rules that
intentionally hide data from a role.

## Phase 8 — Owner decisions

`17-owner-decisions.md`: numbered, dated entries with the question, our recommendation, the
final answer and the card that implements it. Binding defaults unless the owner says otherwise:
exact clone + additions only; additions default to the reference behaviour; reference defects
are not copied; where the reference was never observed, offer the safest rule and mark it
interim. Ask with AskUserQuestion, recommendation first.

## Phase 9 — Gap analysis vs our codebase

`15-gap-analysis.md`, one table per area: capability → reference behaviour (report §) → our
status with file evidence (`EXISTS` / `PARTIAL` / `MISSING` / `ADDITIVE-EXTRA` /
`DEFECT-EXCLUDED`) → gap to close → card ID. Verify "ours" by reading the code (models, schema,
routes, services), not from memory. Re-check against unmerged branches before creating
duplicate cards.

## Phase 10 — Clone plan → cards

`16-clone-plan.md`: §1 scope edits to existing roadmap tasks (old → new → why → evidence);
§2 new cards (ID, one-line scope, depends on, reference §, size S/M/L); §3 critical path as a
dependency diagram; §4 phase mapping. Size: S < half a day of one agent, M ≈ one day, L ≥ 2
days. Cards that must land together (API + client screen) are marked as a bundle.

## Phase 11 — Gated rebuild in batches

Per card: isolated working copy, implementing agent works test-first, keeps RED evidence,
reports. Reviewer checks. Cards go into an integration branch (~8), synced with `main`,
cross-domain test folders first, then the full verify in the background with a unique log and
a timeout. Split the verify across machines when available. Merge `--no-ff`, docs, notify,
next card queued per lane. When a second machine owns a lane, it never merges `main`; it sends
FINAL reports (SHA, RED, analysis, gates) and the first machine accepts or sends back. Keep a
running list of accepted SHAs per batch and name superseded ones explicitly.

## Phase 12 — Parity audits and ETA

Every few batches: `PARITY-REMAINING.md`, built by checking every card ID against the tree
(route/class/file presence + commit messages), with counts done / partial / missing per area,
what is blocked on the owner, and the remaining days computed from cards left × size, agents in
parallel, verify time per batch. Give the owner the number and the assumption behind it.
