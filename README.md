# Tech-debt subagent workflow

This bundle turns the diagram into a Codex custom-agent workflow while enforcing the deletion-first contract in `$tech-debt`.

## Diagram mapping

| Diagram step | Agent workflow |
| --- | --- |
| User request | Root task receives `PROMPT.md` |
| Does the task include deletion? | `debt_router` |
| “Should I delete code?” | `debt_arbiter` validates the explicit target |
| Is code obsolete or faulty? | `debt_accretion`, `debt_lifecycle`, `debt_aot`, and `debt_contracts` scan in parallel; `debt_arbiter` validates their evidence |
| Remove code or files | `debt_deleter` |
| Wrapper or shim proposal | `debt_boundary_fix` permits one only at a proven external boundary; otherwise it rejects the proposal without editing |
| Changed, shorter code | `debt_verifier` checks the diff and repository-native validation |

The diagram's automatic wrapper fallback is intentionally narrowed. A wrapper is itself technical debt unless it isolates a real third-party, protocol, persistence, or public API boundary.

## Install

Copy `.codex` into the repository root. If `.codex/config.toml` already exists, merge the required settings instead of replacing the file.

The older bundled skill was removed in favor of the stronger [canonical tech-debt skill](https://github.com/ANcpLua/skill-library/tree/main/skills/tech-debt). From a checkout of that private library, run `python3 scripts/install.py tech-debt --dest ~/.codex/skills`. Existing installations are not overwritten. For the expanded DELETE / TRANSFORM / KEEP_BOUNDARY workflow and its current role definitions, install `debt-run` from the same library; it also installs `tech-debt`. The Codex agent configuration in this repository remains a separate host adapter.

Start a new Codex task from the repository root and paste `PROMPT.md`. Append an exact path or subsystem after the prompt when the audit must be narrower than the repository.

Project-scoped custom agents are loaded only for trusted repositories. The agent name inside each TOML file is authoritative.

Select a parent permission mode that permits repository edits. Live parent permissions can override the `sandbox_mode` declared by an implementation agent.
