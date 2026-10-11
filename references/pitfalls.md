# Pitfalls a real run hit, and the rule each one produced

| Incident | Rule |
|---|---|
| A token and an API key were printed in tool output while debugging a login response | Extract secrets straight into shell variables; never pipe raw responses to the screen; report the slip in §0 of the report; the owner rotates / logs out |
| A read on an export route returned 405 | Exports are often writes; stop and ask the owner for a named permission, never probe |
| An agent asked the orchestrator to perform an action its own permission had denied | Permission laundering: refuse, surface it to the owner |
| A shared stash across working copies applied another agent's work | Shared stash is forbidden; use a temporary detached working copy for baselines |
| A blanket process kill stopped other agents; a container stop killed another agent's coverage run | Kill only your own PIDs; never touch a container you did not start |
| Resolving a merge conflict by taking a whole file from `main` dropped the batch's changes in that file | Resolve per hunk; keep every DISABLED block verbatim; re-run touched tests |
| Two cards created schema migrations with the same timestamp | Check the latest timestamp on `main` and on every open branch; pick a later unused one |
| A fresh migration failed on the production database engine while the test engine passed | Run a migration check (fresh, rollback, migrate) on the production engine in a container you own before deploy; create the replacement index before dropping the old one |
| A full coverage run took hours and blocked a batch | Measure diff coverage on the changed test files first (partial, labelled as such); schedule the full run separately |
| Formatting the whole repo with the linter would have rewritten dozens of legacy files | Lint gate on new files only until the legacy files are reformatted in one dedicated card |
| Static analysis had ~1,000 pre-existing errors | Baseline file; the gate fails only on new errors |
| Disk filled with copied dependency folders in many working copies | Delete the dependency folder from a working copy once its card is pushed; clean merged working copies |
| The container runtime hung and blocked coverage on one machine | Split the verify across machines; record "BLOCKED (<reason>)" honestly in the merge message; never claim a gate that did not run |
| The owner's assumption about a measured quantity was wrong (straight line vs route) | Verify measurement semantics against data before building rules that depend on it; provider strategy with fallback |
| A formula matched 95% and the misses were a reference bug | List as a defect, ask the owner, do not copy |
| A push from the other machine was counted as accepted before its FINAL report | Accept only from a FINAL message with SHA + analysis result + gates; keep a superseded-SHA list |
| Session context ran out mid-batch | HANDOFF §0 block before every risky step; one-fact memory files; monitors re-armed on resume |
| An agent reported "done" without RED evidence | Send it back; RED can be reproduced by running the new tests alone at the base commit in a temporary working copy |
| Reports mixed identities with shapes | Shape-only policy; enums/amounts/counts stay, identities become `<REDACTED>` |
| A card referenced a sibling card's field names before that card landed | Publish the API contract (field names, paths, example payloads) to the dependent lane as soon as it is final; the dependent card ships with a fallback until wired |
