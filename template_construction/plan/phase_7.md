# Phase 7 – Agent Runtime, Tool Runner, and Hook Runner

> Objective: implement the **runtime layer** that ties abilities, registries, and hooks together: a Tool Runner library, a Hook Runner engine, and a single Agent Runtime that cooperates with external AIs.

---

## 1. Phase Positioning and Scope

- **Depends on:**  
  - Phases 1–6 (layout, docs, knowledge, registries, scaffolding, sample registrations).

- **Solves:**  
  - “How do abilities actually execute, with hooks and registries, under a single runtime contract?”

- **Prepares for:**  
  - Phase 8 quality system (tests, CI, infra abilities).  
  - Real-world usage of the template with external AIs.

---

## 2. Runtime Roles and Responsibilities

Introduce three core runtime roles:

1. **Tool Runner**  
   - Library/utility that executes abilities.  
   - Uses registries to locate scripts/handlers.  
   - Emits hook events around ability calls.  
   - Centralizes usage/telemetry logging.

2. **Hook Runner**  
  - Library that loads `.system/hooks/*.yaml` and executes hook handlers.  
  - Supports the five standard events (`PromptSubmit`, `PreAbilityCreate`, `PreAbilityCall`, `PostAbilityCall`, `SessionStop`) with blocking/non-blocking semantics.  
  - Shared by Tool Runner and Agent Runtime.

3. **Agent Runtime**  
   - The only long-lived process.  
   - Manages sessions.  
   - Orchestrates PromptSubmit and SessionStop hooks.  
   - Interprets “action primitives” from external AIs (e.g., create/update/run task) and delegates to Tool Runner.  
   - Maintains per-session state and workdocs integration.

---

## 3. Block A – Tool Runner

### 3.1 API Surface

Tool Runner should expose functions like:

- `task.create(...)`  
- `task.run(task_id or spec, ...)`  
- `task.get(task_id)`  
- `task.update(task_id, ...)`  
- `task.delete(task_id)`  
- `direct_run(ability_id, input_payload)`

### 3.2 Behavior

Tool Runner is the single execution entry point for the **ability system**. For ability-based flows (via `task.*`), it must:

1. Look up ability metadata in `.system/registry/low-level` or `high-level` and resolve configuration (including ability-level hook bindings such as `hooks.pre_create` / `hooks.pre_call` / `hooks.post_call`).  
2. On `task.create`:
   - Validate the input against registry schemas (shape-level validation).  
   - Trigger `PreAbilityCreate` hooks via Hook Runner with the appropriate context:
     - Global `PreAbilityCreate` hooks discovered from `.system/hooks/*.yaml`.  
     - Ability-specific pre-create hooks referenced from ability registries/config (`hooks.pre_create`).  
   - If preflight denies creation, return an “unavailable” result with a clear reason and do **not** allocate a task id or create `.system/implement/<task-id>/`.  
   - If allowed, allocate a task id, create the task folder and `tool_call.md`, and cache input + resolved configuration.  
3. On `task.run`:
   - Load cached input and configuration.  
   - Trigger `PreAbilityCall` hooks with the appropriate context:
     - Global `PreAbilityCall` hooks.  
     - Ability-specific pre-call hooks from registries/config (`hooks.pre_call`).  
   - If guardrails deny execution, return a structured decision (for example via `ability_guard` signals) and **do not** execute the underlying implementation.  
   - If allowed, execute the underlying script or handler (based on registry data).  
   - Record execution results and telemetry (e.g., success/failure, duration, environment) in `tool_call.md` or equivalent store.  
   - Trigger `PostAbilityCall` hooks (global + ability-specific `hooks.post_call`) for telemetry and reporting; these remain infra-only and must not inject AI-facing signals back into the current turn.  

For **direct_run**, Tool Runner may offer an escape hatch that:

- Invokes scripts or other implementations directly, without going through the ability task lifecycle (`task.create` / `task.run`).  
- May skip some or all ability-level hooks and state management when used for low-risk utilities or debugging.  
- Leaves it up to each project whether to reuse parts of the Hook Runner pipeline for direct calls; Phase 7 does **not** require direct_run to participate in the full ability routing lifecycle.

Tool Runner should not embed business logic; it orchestrates abilities and hooks based on registries.

---

## 4. Block B – Hook Runner

### 4.1 API Surface

Hook Runner should expose at least:

- `run_hooks(event_type, context, blocking_only)`

### 4.2 Responsibilities

1. Read `.system/hooks/*.yaml` to discover hook registrations.  
2. For a given `(event_type, context)`:
   - Select matching hooks based on their `match` activation rules (e.g., ability scope, ids, changed paths).  
   - Execute handlers (e.g., scripts under `scripts/hooks/` or other mechanisms).  
3. For blocking events (`PromptSubmit`, `PreAbilityCreate`, `PreAbilityCall`):
   - Collect hook signals (`routing_hint` / `normalized_intent` for `PromptSubmit`; `ability_preflight` for `PreAbilityCreate`; `ability_guard` for `PreAbilityCall`).  
   - Merge results in a structured way for the caller (Tool Runner or Agent Runtime).  
4. For non-blocking events (`PostAbilityCall`, `SessionStop`):
   - Execute hooks without affecting current decision flow.  
   - Use them for logging, builds/tests, reporting, etc. Any AI-facing fields (for example `hook_signals`) from these events must be ignored by callers or treated as configuration errors.

---

## 5. Block C – Agent Runtime

### 5.1 Role

Agent Runtime is the single long-lived runtime process responsible for:

- Managing conversational sessions.  
- Triggering PromptSubmit and SessionStop hooks.  
- Translating external AI “action primitives” into Tool Runner calls.  
- Tracking session-level workdocs updates if needed.

### 5.2 External Contract

Define how external AI clients interact with Agent Runtime, for example:

1. External client sends a **PromptSubmit** request with user task and context.  
2. Agent Runtime:
   - Triggers `PromptSubmit` hooks via Hook Runner.  
   - Provides hook signals to the AI as part of the context.  
3. External AI returns a list of **actions** (e.g., `CREATE_TASK`, `UPDATE_TASK`, `RUN_TASK`, optionally some `DIRECT_RUN`-style escapes for humans/infra).  
4. Agent Runtime:
   - For each action, delegates to Tool Runner. For example:
     - `CREATE_TASK` → `task.create` (Tool Runner runs `PreAbilityCreate` and returns a task id only if preflight passes).  
     - `RUN_TASK` → `task.run` (Tool Runner runs `PreAbilityCall`, then executes or denies according to guardrail signals).  
     - Non-task-based direct runs, if supported, may call underlying scripts via Tool Runner’s `direct_run` without participating in the ability task lifecycle.  
   - Updates session state and optionally session-local workdocs.  
5. When the session ends or a major phase completes, Agent Runtime:
   - Triggers a `SessionStop` event via Hook Runner.  
   - Uses non-blocking hooks for builds/tests/summaries, without injecting new signals into the current AI turn.

The exact wire protocol and process model can be minimal, as long as it satisfies the above contract.

---

## 6. Block D – Registry and Config Adjustments

During this phase, adjust registry/config behavior to match runtime needs:

- Replace any older monolithic `guardrail` fields for abilities with explicit hook bindings at the ability/operation level:
  - `hooks.pre_create` for preflight (`PreAbilityCreate`).  
  - `hooks.pre_call` for execution guardrails (`PreAbilityCall`).  
  - `hooks.post_call` for telemetry (`PostAbilityCall`).  
- Ensure `.system/hooks/*.yaml` is the source of truth for **hook definitions** (id, event_type, handler, match rules), while ability registries/configs are the source of truth for **which hooks are bound** to which abilities/operations.  
- Align ability registry entries with Tool Runner expectations (e.g., handler types, script paths, configuration references) and populate `routing_hints` metadata so that PromptSubmit routing hooks can suggest abilities based on intent, scope, and environment.

Record any new configuration conventions in appropriate knowledge docs and AGENTS files.

---

## 7. Workdocs Usage for Phase 7 (Template Construction)

In `template_construction/workdocs/`:

- `task.md` should describe the three runtime roles (Tool Runner, Hook Runner, Agent Runtime) and their interactions.  
- `todolist.md` should have separate checklists for:
  - Tool Runner implementation and tests.  
  - Hook Runner implementation and tests.  
  - Agent Runtime implementation and tests.  
  - Registry/config alignment tasks.  
- `context.md` should record:
  - Design decisions about runtime API.  
  - Any constraints for the external AI contract.  
  - Open questions to be resolved in Phase 8 or later projects.

---

## 8. Human Review Checklist

Humans should validate Phase 7 using these scenarios:

1. **Preflight + pre-call guard scenario**  
   - An ability has preflight and pre-call hooks configured (via `hooks.pre_create` / `hooks.pre_call`).  
   - External AI requests to create a task via Agent Runtime (`CREATE_TASK` action).  
   - Tool Runner runs `task.create`, triggers `PreAbilityCreate`, and either:
     - Returns a task id (available), possibly with warnings, or  
     - Returns an “unavailable” result with a clear reason (no task id allocated).  
   - For a valid task, external AI later requests to run it (`RUN_TASK` action).  
   - Tool Runner runs `task.run`, triggers `PreAbilityCall`; Hook Runner enforces guardrails or demands confirmation.  
   - Agent Runtime surfaces both preflight results and guardrail decisions correctly to the AI (and/or humans).

2. **Ability execution + post-call hook**  
   - An ability has `hooks.post_call` configured.  
   - External AI runs a task; Tool Runner executes ability and Hook Runner triggers PostAbilityCall behavior (e.g., logging).

3. **Session-level hooks**  
   - PromptSubmit hooks provide routing hints and normalized intent.  
   - SessionStop hooks run build/tests or produce summaries.  
   - Agent Runtime correctly triggers these events.

Humans should also review:

- Whether the runtime interfaces are minimal yet sufficient for the architecture.  
- Whether error handling and logging are adequate for debugging.

---

## 9. Definition of Done (DoD)

Phase 7 is complete when:

1. Tool Runner exposes task management and direct execution functions, and for ability-based flows:
   - Runs `PreAbilityCreate` on `task.create` and returns structured preflight results.  
   - Runs `PreAbilityCall` / `PostAbilityCall` around `task.run` according to hook bindings.  
2. Hook Runner supports all five events with correct blocking/non-blocking semantics:
   - `PromptSubmit`, `PreAbilityCreate`, and `PreAbilityCall` are blocking and may emit AI-facing `hook_signals`.  
   - `PostAbilityCall` and `SessionStop` are infra-only; any AI-facing fields they emit are ignored.  
3. Agent Runtime is implemented as the single long-lived process, managing sessions and coordinating PromptSubmit/SessionStop events and Tool Runner calls. 
4. Agent Runtime can provide ability routing and knowledge routing hints to external AI via PromptSubmit routing hooks that read `ROUTING.md` / `ABILITY.md` and ability registries/configs (including `routing_hints`).  
5. Ability and hook registries/configs use explicit `hooks.pre_create` / `hooks.pre_call` / `hooks.post_call` bindings instead of vague guardrail fields, and hooks are defined in `.system/hooks/*.yaml` with clear `event_type` and `match` rules.
6. At least a small set of end-to-end tests validate that Agent Runtime, Tool Runner, and Hook Runner work together as designed.  
7. `template_construction/workdocs/outcome.md` contains a “Phase 7 outcome” section summarizing runtime APIs, known limitations, and planned refinements.
