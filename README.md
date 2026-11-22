# Agent Harness Codes

Reference code for harness-based agentic applications, written to accompany the paper
"Engineering the Harness: Control, Budget, and Audit for Agentic Applications"
(Dr. Anish Roychowdhury).

Everything here runs on synthetic data with a scripted model. No network access, API key
or third-party package is needed. Requires Python 3.9 or later.

## Quick start

```bash
unzip harness_reference_impl.zip -d agent-harness
cd agent-harness
python3 demo.py
```

Expected last line: `ALL SCENARIOS PASSED`.

## Contents

Keep this table current. Add one row per file or folder when new code is added.

| Path | Kind | Purpose | Added |
|------|------|---------|-------|
| `harness_triage.py` | Core library | Harness: budget, tool contract, registry, audit log, policy gate, async-generator agent loop, subagents | 2026-11-22 |
| `tools_aml.py` | Domain tools | Six synthetic AML tools and the registry builder | 2026-11-22 |
| `demo.py` | Scenarios and tests | Eight scenarios with assertions on harness behaviour | 2026-11-22 |
| `README.md` | Docs | This file | 2026-11-22 |

## Architecture in brief

The model proposes; the harness decides. The model has no direct path to a tool. Every call
passes: registry lookup, argument validation, policy decision (allow, ask, deny), analyst
approval when required, execution with result truncation, and an audit entry.

| Responsibility | Where it lives |
|----------------|----------------|
| State ownership | `TriageAgent` (messages, denials, usage) |
| Tool contract | `ToolSpec`, `ToolRegistry`, `validate_args` |
| Permission | `TriageAgent._decide`, `_ask_analyst`, `ToolSpec.check_permissions` |
| Budget | `IterationBudget` (shared across the agent tree), cost ceiling in `investigate` |
| Observability | `investigate` is an async generator yielding `Event` objects |
| Audit | `AuditLog` (append-only, hash-chained) |

## Scenarios covered by `demo.py`

1. Normal investigation with one subagent
2. Injected instruction leads the model to request a freeze; analyst refuses
3. Runaway loop with root and subagent sharing one budget
4. Cost ceiling (soft: checked before each model call)
5. Wrong argument type, out-of-policy look-back, unknown tool
6. Audit-log tamper detection
7. Abort during a run
8. Approval timeout fails closed

## Conventions

- One domain per tools file (`tools_<domain>.py`), exposing `build_registry()`.
- The harness core (`harness_triage.py`) stays domain-neutral. Domain rules go in tool
  `check_permissions` hooks, not in the core.
- Every new control gets a scenario in a demo or test file that asserts on side effects,
  not on model text.
- Prose and comments: no em dashes.

## Adding new material

When a new zip arrives, follow these steps so this file stays accurate.

1. Unzip into the project root, or into a subfolder if names would clash.
2. Add a row to the Contents table: path, kind, one-line purpose, date.
3. If the new code changes the harness core, update the Architecture table and the
   Changelog.
4. If it adds scenarios, extend the scenario list or add a new section below.
5. Re-run `python3 demo.py` and confirm it still ends with `ALL SCENARIOS PASSED`.

### Template for a new component section

```markdown
## <Component name>

- Files: `<paths>`
- Purpose: <one or two sentences>
- Depends on: <modules>
- Run: `<command>`
- Status: <working | experimental | stub>
- Notes: <limitations, assumptions>
```

## Known gaps

These are limitations of the current reference, also discussed in the paper.

- No live model; the injection scenario is simulated by scripting the model.
- Audit log is in memory; production needs durable storage and external anchoring of the
  head hash (asynchronous, with synchronous anchoring before irreversible actions).
- A timed-out approval is recorded as a denial and the run ends. There is no checkpoint,
  park, or resume.
- No liveness metrics (required-step coverage, completion rate) with a real model.
- Argument validation checks presence, type and unexpected fields only.
- Concurrency-safety declarations on tools are trusted, not verified.

## Changelog

| Date | Change |
|------|--------|
| 2026-11-22 | Initial release: harness core, AML tools, eight scenarios |

## Safety and scope

Synthetic data only. This is an engineering reference, not compliance guidance. Rules on
suspicious activity reporting and record retention differ by jurisdiction.

## References

The paper's bibliography is the authoritative list. The starting point for the pattern is
K. Huang and G. Huang, *Harness Patterns for Agentic AI with Claude Code and Hermes Agent*.
