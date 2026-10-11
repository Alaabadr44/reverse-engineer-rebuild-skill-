# Templates

## 1. Domain report (`NN-<area>.md`)

```
# <REF><NN> — <Area>
Scope: <what this report covers, sources used, what it does not cover>
Safety: <read-only, rate, redaction statement>

## 1. Actors / entities
| Entity | Fields (type, enum values) | Evidence |
## 2. Rules and lifecycle
| From | Event / actor | To | Side effects (money, notifications) | Evidence |
## 3. Formulas (as written in UI / code; tag EVIDENCE / INFERRED)
## 4. Screens (name, fields, actions, dialogs)
## 5. API (route, method, request shape, response shape — shape-only)
## 6. Settings that influence this area
## 7. Defects observed (do not copy)
## 8. Our codebase today (files, status)
## 9. Open questions (each with the call or data that would answer it)
```

## 2. Live-verification row

| Source (report §) | Item | Verified answer | Evidence (call, shape-only) |
|---|---|---|---|

## 3. Gap-analysis row

| Capability | Reference behaviour (report §) | Our status — evidence (file) | Gap to close | Card |
|---|---|---|---|---|

Status values: `EXISTS` · `PARTIAL` · `MISSING` · `ADDITIVE-EXTRA` (ours, defaults off) ·
`DEFECT-EXCLUDED` (reference bug, not copied).

## 4. Owner decision entry

```
N. **<Topic> (YYYY-MM-DD):** <question in one line>.
   Recommendation offered: <...>. Answer: <...>. Card: <id>. [interim: taken without asking, open for review]
```

## 5. Card

```
| ID | Scope (one line) | Depends on | Reference § | Size | Bundle (lands with) |
```
Agent brief for a card: repo path, rules file, base commit, working-copy path, scope lock,
inputs (spec §, decisions, golden vectors), tests to write first, test folders to run, size
gate command, commit trailer, what to report (SHA, RED evidence, GREEN results, files, open
questions in the owner's language), what not to do (merge or push `main`, touch other cards'
files, shared stash, kill others' processes).

## 6. Golden vector

| # | Case | Inputs (plan/tariff, quantities, payment method, flags) | Expected outputs (each amount) | Source row (date only) |

## 7. HANDOFF §0 block (newest first)

```
## 0x. LATEST STATE (updated YYYY-MM-DD HH:MM <tz>) — read first
- main = <sha> (<what merged>, gates: <summary>)
- Integration branch <name> @ <sha>: <cards>, verify <running/passed/failed + log path>
- Accepted from the other machine (FINAL SHAs): <list, superseded ones named>
- Running agents: machine A <n> (<cards>), machine B <n> (<cards>)
- Blocked on owner: <items>
- Resume steps: 1) ... 2) ... 3) ...
- Incidents / new rules: <...>
```

## 8. Status message to the owner (structure)

1. What merged (batch, contents in product terms).
2. What is running (agents per machine, names in product terms).
3. What was accepted / sent back from the other machine.
4. Decisions taken on recommendation (open for review) + what is needed from the owner.
5. ETA with its assumption.
