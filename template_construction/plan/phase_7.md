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
   - Supports four event types and blocking/non-blocking semantics.  
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

For each ability execution, Tool Runner must:

1. Look up ability metadata in `.system/registry/low-level` or `high-level`.  
2. Trigger `PreAbilityCall` hooks via Hook Runner with relevant context.  
3. Execute the underlying script or handler (based on registry data).  
4. Trigger `PostAbilityCall` hooks.  
5. Record usage/telemetry (e.g., success/failure, duration) in a structured format (e.g., logs, reports).

Tool Runner should not embed business logic; it orchestrates abilities and hooks based on registries.

---

## 4. Block B – Hook Runner

### 4.1 API Surface

Hook Runner should expose at least:

- `run_hooks(event_type, context, blocking_only)`

### 4.2 Responsibilities

1. Read `.system/hooks/*.yaml` to discover hook registrations.  
2. For a given `(event_type, context)`:
   - Select matching hooks based on registry fields (e.g., ability scope, ids, routing stage).  
   - Execute handlers (e.g., scripts under `scripts/hooks/` or other mechanisms).  
3. For blocking events (`PromptSubmit`, `PreAbilityCall`):
   - Collect hook signals (routing hints, normalized intent, ability guards).  
   - Merge results in a structured way for the caller (Tool Runner or Agent Runtime).  
4. For non-blocking events (`PostAbilityCall`, `SessionStop`):
   - Execute hooks without affecting current decision flow.  
   - Use them for logging, builds/tests, reporting, etc.

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
3. External AI returns a list of **actions** (e.g., `CREATE_TASK`, `UPDATE_TASK`, `RUN_TASK`).  
4. Agent Runtime:
   - For each action, delegates to Tool Runner.  
   - Updates session state and optionally session-local workdocs.  
5. When the session ends or a major phase completes, Agent Runtime:
   - Triggers a `SessionStop` event via Hook Runner.  
   - Uses non-blocking hooks for builds/tests/summaries, without injecting new signals into the current AI turn.

The exact wire protocol and process model can be minimal, as long as it satisfies the above contract.

---

## 6. Block D – Registry and Config Adjustments

During this phase, adjust registry/config behavior to match runtime needs:

- Replace any older `guardrail` fields for abilities with explicit `hooks.pre_call` and `hooks.post_call` bindings.  
- Ensure `.system/hooks/*.yaml` is the **only** source of truth for hook registrations.  
- Align ability registry entries with Tool Runner expectations (e.g., handler types, script paths).

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

1. **Pre-call guard scenario**  
   - An ability has a pre-call hook configured.  
   - External AI requests to create/run a task via Agent Runtime.  
   - Tool Runner triggers PreAbilityCall; Hook Runner enforces guard or demands confirmation.  
   - Agent Runtime surfaces results correctly to the AI.

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

1. Tool Runner exposes task management and direct ability execution functions and triggers Pre/PostAbilityCall hooks.  
2. Hook Runner supports all four events with correct blocking/non-blocking semantics.  
3. Agent Runtime is implemented as the single long-lived process, managing sessions and coordinating PromptSubmit/SessionStop events and Tool Runner calls.  
4. Ability and hook registries/configs use explicit `hooks.pre_call` / `hooks.post_call` bindings instead of vague guardrail fields.  
5. At least a small set of end-to-end tests validate that Agent Runtime, Tool Runner, and Hook Runner work together as designed.  
6. `template_construction/workdocs/outcome.md` contains a “Phase 7 outcome” section summarizing runtime APIs, known limitations, and planned refinements.
