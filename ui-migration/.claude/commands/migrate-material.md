---
description: Start or continue the PrimeNG → Angular Material UI migration (component swap + PrimeFlex → Tailwind)
argument-hint: [continue | start | status | validate | skip <N> | help]  # default (no arg) = continue
model: sonnet
# user-invocable: true (default, omitted) — this is the user entry point (/migrate-material).
# This command has side effects (runs builds/tests, drives the migration, edits source),
# so the model must not trigger it on its own — you invoke it explicitly.
disable-model-invocation: true
allowed-tools: Agent(ui-migration-orchestrator) Agent(ui-migration-validator) Read Grep Glob Bash(npm run build) Bash(npm run lint) Bash(npm run test)
---

> Entry point for the PrimeNG → Angular Material UI swap. In Claude Code: route to the `ui-migration-orchestrator` agent (`.claude/agents/ui-migration-orchestrator.md`), which drives the migration end to end — gates → planning → approval → Phase A (component + PrimeFlex→Tailwind) → Phase B (cleanup + error recovery). **`primeng`/`primeflex`/`primeicons` are NOT removed** — they stay in `package.json` (required by the OneCX v8 libraries) and are removed only at the OneCX v9 / Angular 22 upgrade.

## Argument routing

Read the invocation argument from `$ARGUMENTS` (first token, case-insensitive). It maps to exactly one action below:

| Argument | Action |
|----------|--------|
| *(empty / bare `/migrate-material`)* | **Default → Continue execution** (see Stop Conditions; if no `UI_MIGRATION_PROGRESS.md` exists yet, this resolves to **Start**) |
| `continue` / `continue execution` | Continue execution |
| `start` | Start |
| `skip <N>` (e.g. `skip 2`) | Skip N — `<N>` is the number after the token |
| `status` | Status |
| `validate` | Validate |
| `help` | Help |
| *(anything unrecognized)* | Show the command table and ask which action to run — do NOT guess |

**Default behavior:** with no argument, resolve the progress file first — no `UI_MIGRATION_PROGRESS.md` ⇒ **Start** (gates + plan); file exists ⇒ **Continue execution** (next `[ ]` task). This is the "Start or Continue Execution" default.

## Migration Path

- **Source UI library**: PrimeNG (`p-*` tags, `@primeng/*` imports, `providePrimeNG`) + PrimeFlex (`col-*`/`g-*`/`m-*`/layout utilities)
- **Target UI library**: Angular Material (`mat-*` tags, `@angular/material/*`, `@angular/cdk`) + Tailwind (layout utilities)
- **Migration identifier**: prime-to-material
- **Progress file**: `UI_MIGRATION_PROGRESS.md` (workspace root) — separate from the Angular version migration (`MIGRATION_PROGRESS.md`)

## Read at the start of every invocation

- **Active data file**: `.claude/instructions/ui-migration-prime-to-material.instructions.md` — MCP source list, component-pair starter table, PrimeFlex→Tailwind mapping, behavioral-difference notes, repo-specific rules (module-federation sharing, `pre_loaders` scope, **PrimeNG packages kept for the OneCX v8 upgrade**).
- **Active rules file**: `.claude/instructions/ui-migration-rules.instructions.md` — hard rules (state, gates, task execution, **scoped** validation, error handling, decision protocol).
- **Project anti-patterns**: `.claude/instructions/migration-custom-user.instructions.md` — the project-specific rules + anti-patterns list (DO-NOT-REPEAT mistakes) that applies to all migrations in this repo.

## MCP Tools

- **PrimeNG MCP** (`mcp__primeng__*`) — audit the OLD PrimeNG usage (component API, template syntax, behavior)
- **Material-side MCP**:
  - `mcp__angular-cli__*` — NEW-side Angular/Material **docs** + **build/devserver validation** (`ng mcp`)
  - `mcp__ngm-dev-blocks__*` — **generate** the new Angular Material + Tailwind blocks (the dedicated Material MCP, ui.angular-material.dev/docs/mcp)

## Gate flow (enforced before planning/execution — HALT on failure)

1. **Gate 1 — Baseline**: `npm run build` / `npm run lint` / `npm run test` all green. Else HALT — never proceed.
2. **Gate 2 — Branch**: never on `main`/`master`/`dev`/`develop`. Else HALT.
3. **Gate 3 — MCP**: PrimeNG MCP + Material-side MCP configured. Else HALT, or add the config to `mcp.json` first (from `.claude/templates/mcp.example.json`).

## Routing

Check if `UI_MIGRATION_PROGRESS.md` exists in the workspace root.

- **If it does NOT exist**: gates have not been passed. The orchestrator runs Gates 1–3, then Discovery/Planning (full-repo PrimeNG inventory + PrimeFlex→Tailwind inventory + Material alternatives), creating `UI_MIGRATION_PROGRESS.md` from `.claude/templates/UI_MIGRATION_PROGRESS.template.md` (2-phase A/B — Phase A = component + PrimeFlex→Tailwind, Phase B = cleanup + error recovery; `primeng`/`primeflex`/`primeicons` kept, not removed), `[ ]`/`[x]`/`[-]` markers, 8 evidence fields per task, contiguous numbering. Then the developer reviews the plan before execution.
- **If it exists**: read it, show current status (gates, phase, completed/pending task counts), then route ONE task at a time.

Route via the Agent tool with `subagent_type: ui-migration-orchestrator`, passing the full `UI_MIGRATION_PROGRESS.md` content and both active file paths (data + rules).

## Commands

- **"Start"** — run gates + create the plan (if no progress file), or resume from the first `[ ]` task
- **"Continue execution"** — route the next `[ ]` task to the executor, then re-read the progress file
- **"Skip N"** — mark the next N `[ ]` tasks `[-]` with note "Skipped by developer on [date]"
- **"Status"** — summarize gates, current phase, task counts (`[ ]`/`[x]`/`[-]`), and open questions
- **"Validate"** — route to the `ui-migration-validator` for the latest task or a phase gate
- **"Help"** — show this table

## Stop Conditions (per executor return)

Stop and return to the user when ANY:
- Executor fixed a build, lint, or test error (user should review the fix)
- Executor changed ≥5 files in one task (review scope)
- Executor reports an ambiguous component-pair candidate, an ambiguous PrimeFlex→Tailwind mapping, a behavioral difference with no clean equivalent, or a version-mismatch (needs a decision)
- A user-visible UI task completed (visual devserver check was reported)
- A phase boundary is reached (end of Phase A, or Phase B)
- The final cleanup task completed (report final verification)

Otherwise continue to the next `[ ]` task.

## Validation discipline

Per change the executor/validator runs **scoped** validation — the specific changed file(s)' test via `--include`/`--config` **plus** `npm run build` — and does **NOT** run the whole suite (it halts the system). The full suite runs only at the baseline gate, the phase gates, and final verification.
