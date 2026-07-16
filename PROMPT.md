$tech-debt

Run a deletion-first technical-debt audit and implementation over the current repository. If I append paths or a subsystem after this prompt, those paths are the complete scope. Exclude generated output, vendored code, dependency caches, and build artifacts unless the scope explicitly names them.

Use the project custom agents named below. Do not substitute built-in agents. Begin every delegated task message with `$tech-debt`; every subagent must load and follow it before inspecting code. The root agent coordinates, deduplicates, and decides; it does not perform a competing audit.

## Invariants

- Preserve unrelated user changes.
- Ground every finding in `path:line`, the offending snippet, and the deletion that removes it.
- Do not accept prose-only findings.
- Prefer deletion over wrapping, shimming, compatibility branches, redundant validation, broad catches, or new abstraction.
- Do not add empty catches, catch-log-rethrow blocks, defensive checks already enforced upstream, or comments that explain obsolete structure.
- Treat a wrapper or adapter as valid only at a proven third-party, protocol, persistence, generated-code, or public API boundary.
- Order encompassing fixes before their contained findings.
- Use the skill priority formula exactly: `(Impact + Risk) × (6 - Effort)`.
- Keep audit agents read-only. Only one implementation agent may edit at a time.
- Do not broaden the scope to opportunistic cleanup.
- Do not commit, push, publish, or open a pull request unless my task explicitly requests it.

## Orchestration

1. Inspect repository guidance and the current worktree. Record the exact scope, repository-native validation commands, and unrelated changes that must be preserved.
2. Spawn `debt_router`. Give it my request and the exact scope. Wait for `EXPLICIT_DELETE` or `DISCOVER_DEBT`, retain its result, then close its thread.
3. For `EXPLICIT_DELETE`, spawn `debt_contracts` for the named targets. Wait, retain its result, and close its thread. Use the router and contracts results as the complete arbiter input. Do not run a repository-wide discovery wave.
4. For `DISCOVER_DEBT`, spawn these four agents in parallel over non-overlapping lenses, then wait for all four:
   - `debt_accretion`
   - `debt_lifecycle`
   - `debt_aot`
   - `debt_contracts`
5. After the selected branch completes, ensure every discovery or contracts thread is closed. Deduplicate its findings by root cause without dropping evidence. Spawn `debt_arbiter` with the complete candidate set. It must classify every candidate as `DELETE`, `KEEP_BOUNDARY`, or `REJECT` and return one canonical ordered chat TODO list. Retain the canonical list and close its thread.
6. When no candidate survives, make no source change, skip steps 7 and 8, and continue to verification with an intentional no-op.
7. When `DELETE` items exist, partition an oversized canonical list into scoped, cohesive sessions of about 20 items while preserving encompassing-before-contained order. Process the sessions sequentially with one `debt_deleter` task at a time. Retain each result and close its thread before starting the next session.
8. When `KEEP_BOUNDARY` items exist, partition them by boundary into sessions of about 20 items and process the sessions sequentially with one `debt_boundary_fix` task at a time after all deletions finish. It may edit only when the arbiter supplied concrete boundary evidence. If the evidence is insufficient, it returns `REJECTED_NO_BOUNDARY` and makes no edit. Retain each result and close its thread before starting the next session.
9. Spawn `debt_verifier` after implementation. Give it the original scope, canonical TODO list, worktree baseline, and validation commands. It must inspect the final diff, reject unproven accretion, and return `PASS`, `FAIL`, or `PAUSED` with evidence.
10. On `FAIL`, run one repair pass with the responsible implementation agent, then run `debt_verifier` once more. On a second failure or any unexpected contract dependency, stop and return the blocker. Do not improvise a compatibility layer.
11. Close the final verifier thread after retaining its result. No completed subagent thread remains open.

## Final response

Return one chat TODO list, not a report file. Show completed items as checked and unresolved or paused items as unchecked. Each finding retains its `path:line`, snippet, deletion remedy, and priority score. Follow it with:

- `Looks bad but is fine`: intentional structures and the evidence for keeping them.
- `Validation`: exact commands and outcomes.
- `Net deletion`: files and lines removed versus added.
- `Blocked`: only concrete unresolved dependencies or failing checks.
