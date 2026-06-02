# PawOps Workbench Agent Guide

PawOps Workbench is a TypeScript workbench for development scenarios where many dog-themed agents collaborate with the user. The project should be designed module by module. Do not jump directly from an idea to implementation.

## Collaboration Protocol

For each module, use this sequence before writing implementation code:

1. Discuss user scenarios.
2. Define capability boundaries.
3. Compare 2-3 viable design options.
4. Choose the preferred tradeoff and explain why the alternatives were not chosen.
5. Record or update the module decision log as the discussion evolves.
6. Write or revise the module spec after the user confirms the direction.
7. Split the approved spec into tasks and a checklist.

Ask one focused question at a time. Prefer concrete choices when the decision space is clear. If a decision affects multiple modules, record it as an ADR.

## Current Truth vs Process Record

Keep current truth and process history separate:

- `spec.md` is the current truth. It should describe the final agreed module behavior as of now.
- `decisions.md` is the process record. It should preserve user scenarios, explored options, selected tradeoffs, rejected alternatives, and reasons for later changes.
- `tasks.md` is the current implementation plan derived from the approved spec.
- `checklist.md` is the current verification contract derived from the approved spec.

If the design changes, update `decisions.md` first to record why, then revise `spec.md`, then review whether `tasks.md` and `checklist.md` still match.

## Documentation Structure

```text
AGENTS.md
docs/
  00-vision.md
  glossary.md
  modules/
    agent-runtime-core/
      spec.md
      decisions.md
      tasks.md
      checklist.md
    agent-profile/
      spec.md
      decisions.md
      tasks.md
      checklist.md
    coding-cli-adapter/
      spec.md
      decisions.md
      tasks.md
      checklist.md
    workbench-shell/
      spec.md
      decisions.md
      tasks.md
      checklist.md
    channel-adapters/
      spec.md
      decisions.md
      tasks.md
      checklist.md
  adr/
    0001-runtime-core-first.md
```

Use module bundles for module-level work. Each module owns its own `spec.md`, `decisions.md`, `tasks.md`, and `checklist.md`. Use `docs/adr/` only for decisions that affect multiple modules or constrain future architecture.

Project-level documents have narrower roles:

- `docs/00-vision.md` defines the product intent, target users, first-version scope, and long-term direction.
- `docs/glossary.md` defines shared vocabulary such as runtime, agent profile, skill, MCP, adapter, run, step, and event.

## Document Roles

### `spec.md`

Answers what problem the module solves and what it explicitly does not solve.

Include:

- Background and current user scenario summary.
- Target users.
- Capability list, one capability per sentence.
- Non-functional requirements.
- Design skeleton.
- Out of scope.
- Accepted design and a short decision summary.
- Completion criteria.

Do not include long discussion history. Link or refer to `decisions.md` for the detailed process record.

Do not include volatile implementation details such as concrete function names, argument names, default values, line numbers, SDK type names, or exact error text unless they are part of a stable public contract.

### `decisions.md`

Answers how the module design evolved and why the current spec is shaped the way it is.

Use one entry per meaningful decision. Each entry should state:

- Date.
- Question or design pressure.
- User scenario that made it matter.
- Options considered.
- Selected option.
- Tradeoff and consequence.
- Whether it affects only this module or needs an ADR.

`decisions.md` may be append-only during active discussion. After a module stabilizes, it can be lightly reorganized for readability, but it must not contradict `spec.md`.

### `tasks.md`

Answers what to do and in what order.

Include 5-15 tasks where each task can be completed in one focused implementation session. Each task should state:

- Goal.
- Files or areas likely affected.
- Dependencies on earlier tasks.
- Reference docs or decisions.
- Verification command or manual check.

The final task list must include:

- Integration into the main flow.
- End-to-end verification.

### `checklist.md`

Answers how to verify the module is done.

Every checklist item must be observable and checkable. Avoid subjective items such as "implementation is complete" or "quality is good".

Good checklist item examples:

- Running `pnpm test` returns exit code 0.
- Creating an agent profile without a name returns a validation error.
- Starting a runtime emits `runtime.started` before any step event.
- A tool call trace includes `runId`, `stepId`, and `toolName`.

At least one checklist item must verify the module end to end.

## Initial Module Order

Start with the smallest useful runtime foundation:

1. `agent-runtime-core`: lifecycle, loop boundary, service mounting, event stream, stop conditions.
2. `agent-profile`: dog agent definition, skills, MCP config, model, permissions, budget.
3. `coding-cli-adapter`: launch, observe, interrupt, and audit external coding CLI tools.
4. `workbench-shell`: minimal UI or CLI surface for viewing runs and events.
5. `channel-adapters`: normalized inbound and outbound message channels.

## Architecture Principles

- Keep `AgentRuntime Core` small. Memory, logs, traces, skills, MCP, tools, model providers, and CLI executors should attach through explicit services or registries.
- Prefer stable interfaces over early feature breadth.
- Use the lightest capability shape that fits: skill for procedural guidance, MCP for external execution, subagent for isolated reasoning.
- Make every runtime event observable. Workbench views and channel adapters should subscribe to events rather than reaching into runtime internals.
- Keep the first version local-first and testable before adding distributed workers or many message channels.
