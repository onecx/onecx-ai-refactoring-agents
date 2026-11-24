---
description: Start or continue the PrimeNG → (Angular Material | Optimus UI) UI migration (Material = component swap; Optimus v1 = component package/identifier rename plus version-specific OneCX theme/CSS compatibility; both include PrimeFlex → Tailwind). The target library is chosen by the developer (never defaulted).
argument-hint: [continue | start | status | validate | skip <N> | help] # default (no arg) = continue
model: sonnet
# user-invocable: true (default, omitted) — this is the user entry point (/migrate-primeng).
# This command has side effects (runs builds/tests, drives the migration, edits source),
# so the model must not trigger it on its own — you invoke it explicitly.
disable-model-invocation: true
allowed-tools: Agent(ui-migration-orchestrator) Agent(ui-migration-validator) Read Grep Glob Bash(npm run build) Bash(npm run lint) Bash(npm run test)
---

> Entry point for the PrimeNG → **target** UI swap — target is chosen by the developer and never defaulted. Route to `ui-migration-orchestrator`. For Optimus, the v1 component migration is a documented rename, but OneCX v8 requires a separately verified theme/CSS compatibility plan. The final progress file and response must include an evidence-backed checklist of temporary v8 items to remove/replace/retain after Angular 22 / OneCX v9.

## Argument routing

Read the invocation argument from `$ARGUMENTS` (first token, case-insensitive). It maps to exactly one action below:

| Argument                            | Action                                                                                                                          |
| ----------------------------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| _(empty / bare `/migrate-primeng`)_ | **Default → Continue execution** (see Stop Conditions; if no `UI_MIGRATION_PROGRESS.md` exists yet, this resolves to **Start**) |
| `continue` / `continue execution`   | Continue execution                                                                                                              |
| `start`                             | Start                                                                                                                           |
| `skip <N>` (e.g. `skip 2`)          | Skip N — `<N>` is the number after the token                                                                                    |
| `status`                            | Status                                                                                                                          |
| `validate`                          | Validate                                                                                                                        |
| `help`                              | Help                                                                                                                            |
| _(anything unrecognized)_           | Show the command table and ask which action to run — do NOT guess                                                               |

**Default behavior:** with no argument, resolve the progress file first — no `UI_MIGRATION_PROGRESS.md` ⇒ **Start** (gates + plan); file exists ⇒ **Continue execution** (next `[ ]` task). This is the "Start or Continue Execution" default.

## Target Selection (NEVER default)

The target UI library is the developer's choice. On a **fresh start** (no `UI_MIGRATION_PROGRESS.md`) **ASK which target** if not stated — never default to either. Record the choice in the progress file's `Target:` field. On **resume** (file exists) use the recorded `Target:` — do not re-ask or switch.

## Migration Path

- **Source UI library**: PrimeNG (`p-*` tags, `@primeng/*` imports, `providePrimeNG`) + PrimeFlex (`col-*`/`g-*`/`m-*`/layout utilities)
- **Target UI library** (chosen, never defaulted): **Angular Material** (component swap) OR **Optimus UI** (v1 package/identifier rename; `p-*`/`pi-*` unchanged) — plus Tailwind. For Optimus, read `.claude/instructions/ui-migration-onecx-version-compat.instructions.md`; the rename alone does not integrate OneCX v8 theme services or PrimeFlex styles.
- **Migration identifier**: `prime-to-material` or `prime-to-optimus` (matches the chosen target)
- **Progress file**: `UI_MIGRATION_PROGRESS.md` (workspace root) — separate from the Angular version migration (`MIGRATION_PROGRESS.md`)

## Read at the start of every invocation

- **Active data file** (per the `Target:` field): Material → `.claude/instructions/ui-migration-prime-to-material.instructions.md`; Optimus → `.claude/instructions/ui-migration-prime-to-optimus.instructions.md`. Both point to the shared PrimeFlex→Tailwind mapping and evidence rules. For Optimus, also read the mandatory OneCX version-compatibility file; dependency decisions are based on installed peers, styles and runtime references.
- **Optimus compatibility file**: `.claude/instructions/ui-migration-onecx-version-compat.instructions.md` — mandatory profile detection, then the fixed OneCX v8 steps (theme adapter, `tw-` prefix, single icon font, feature-module imports) and the Angular 22 / OneCX v9 cleanup handoff.
- **Active rules file**: `.claude/instructions/ui-migration-rules.instructions.md` — hard rules (state, gates, task execution, **scoped** validation, **size-difference check**, error handling, decision protocol).
- **Project anti-patterns**: `.claude/instructions/migration-custom-user.instructions.md` — the project-specific rules + anti-patterns list (DO-NOT-REPEAT mistakes) that applies to all migrations in this repo.

## MCP Tools (target-aware)

- **PrimeNG MCP** (`mcp__primeng__*`) — audit the OLD PrimeNG usage (component API, template syntax, behavior). Both targets.
- **Angular CLI MCP** (`mcp__angular-cli__*`) — **build/devserver validation** for either target (`ng mcp`) + general Angular docs. Both targets.
- **Tailwind MCP** (`mcp__tailwind__*`, remote `https://mcp.tailwind.ai`) — PrimeFlex→Tailwind utility + size-difference lookups. Both targets (PrimeFlex→Tailwind is target-independent; full tooling + doc-source fallback in `ui-migration-primeflex-to-tailwind.instructions.md`). Companion skills (optional, separate): `npx @tailwind-app/skills`.
- **Material target only** — `mcp__ngm-dev-blocks__*`: **generate** the new Angular Material + Tailwind blocks (the dedicated Material MCP, ui.angular-material.dev/docs/mcp).
- **Optimus UI target only — NO MCP.** Use version-matched docs (v1/Angular 21: installation, PrimeNG migration, Tailwind, theming, configuration; v2/Angular 22: current docs). Inspect installed OneCX packages for theme/stylesheet integration. Do not equate API rename evidence with OneCX theme/CSS parity.

## Gate flow (enforced before planning/execution — HALT on failure)

1. **Gate 1 — Baseline**: `npm run build` / `npm run lint` / `npm run test` all green. Else HALT — never proceed.
2. **Gate 2 — Branch**: never on `main`/`master`/`dev`/`develop`. Else HALT.
3. **Gate 3 — Docs / MCP (TARGET-AWARE)**: always PrimeNG MCP + Angular CLI MCP. **Material**: + `ngm-dev-blocks` + Material MCP configured (bootstrap from `mcp.json` if missing; if it cannot be configured, VERIFY the Material API from `node_modules/@angular/material/` + `@angular/cdk/` before HALT). **Optimus UI**: no MCP — verify the two Optimus URLs are reachable via WebFetch (fall back to `curl -sSL`); if both are unreachable, fall back to VERIFYING the rename from `node_modules/@openng/optimus-ui/` + `@openng/icons/`. **Last-resort fallback (both targets): when a doc source (MCP or URL) is unreachable/policy-blocked, trace + VERIFY the installed `node_modules/` (`node_modules/primeng/`, `@openng/optimus-ui/`, `@angular/material/`, `primeflex/`, `tailwindcss/` — the `.d.ts`, `package.json`, and CSS/asset files); the scoped build confirms it.** HALT only if `node_modules` also yields no evidence — or add a missing MCP config to `mcp.json` first (from `.claude/templates/mcp.example.json`).

## Routing

Check if `UI_MIGRATION_PROGRESS.md` exists in the workspace root.

- **If it does NOT exist**: gates have not been passed. The orchestrator confirms target, runs gates, and plans. For Optimus, record OneCX version and compatibility profile; dependency retention/removal is evidence-based. The template includes the mandatory v9 cleanup handoff. The developer reviews the plan before execution.
- **If it exists**: read it (including the `Target:` field — use it, don't re-ask), show current status (gates, phase, completed/pending task counts), then route ONE task at a time.

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
- Executor reports an ambiguous **target mapping** (Material: a candidate **pair**; Optimus: app-owned vs `pre_loaders`/OneCX-v8 scope), a behavioral difference with no clean equivalent, or a version-mismatch (needs a decision)
- A user-visible UI task completed (visual devserver check was reported)
- A phase boundary is reached (end of Phase A, or Phase B)
- The final cleanup task completed (report final verification)

Otherwise continue to the next `[ ]` task.

## Validation discipline

Per change the executor/validator runs **scoped** validation — the specific changed file(s)' test via `--include`/`--config` **plus** `npm run build` — and does **NOT** run the whole suite (it halts the system). The full suite runs only at the baseline gate, the phase gates, and final verification.
