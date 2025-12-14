# Hooks for AI-Centric Development

> **Status**: Draft, evolving with real usage.  
> **Audience**: People (and AIs) building repos where an AI developer is the primary actor, and hooks are the infra around it.

This document defines a small, event‑driven **hook system** that lives next to abilities, tools, and knowledge routing in an AI‑centric repo.

The core assumptions are:

- The **AI is the primary developer**: it reads code, plans work, edits files, runs tools.
- A long‑running **Agent Runtime** owns the lifecycle: it receives user input, emits events, runs hooks, and calls abilities.
- **Hooks are repository‑local infrastructure**: they wrap the AI’s work with logging, guardrails, and automation, without being hard‑wired into the AI’s prompt.

The goals of this design:

1. **Help the AI**, don’t fight it  
   - Provide routing hints, guardrails, and summaries in a structured, predictable way.
2. **Stay lightweight and optional**  
   - The repo must work with all hooks disabled; hooks are an “extra layer,” not core logic.
3. **Keep responsibilities sharp**  
   - Only some events may directly influence the AI’s current turn or a tool call. Others are infra‑only.
4. **Be easy to scaffold and maintain**  
   - Hooks are small YAML files + handler scripts; a dedicated scaffold flow helps create them.

---

## 0. Overview: Roles and Architecture

### Roles & Terms

This document follows the shared glossary defined for AI-facing docs in this repository (for example, the project-level `AGENTS.md` and any shared glossary it references). It uses the standard terms `AI developer`, `Agent Runtime`, `Tool Runner`, `Hook Runner`, and `humans` as defined there and does not introduce additional role names.

We assume four main roles in this repo:

- **AI developer**  
  - The code-capable model loop that plans, edits, and proposes actions (for example, “create task”, “run task”).
- **Agent Runtime**  
  - The **only long-running** process or service that:
    - Receives user input.
    - Emits session-level events (`PromptSubmit`, `SessionStop`).
    - Owns session state and turn pipelines.
    - Interprets the AI’s actions (e.g. `LIST_ABILITIES`, `CREATE_TASK`, `RUN_TASK`) and delegates to the Tool Runner.
- **Tool Runner**  
  - A non-resident execution library/CLI that:
    - Provides task interfaces for abilities (`task.create`, `task.run`, etc.).
    - Emits ability-level events (`PreAbilityCreate`, `PreAbilityCall`, `PostAbilityCall`).
    - Calls concrete implementations (scripts, MCP tools, APIs) according to ability registries and configuration.
- **Hook layer / Hook Runner**  
  - Repository-local configuration + handler scripts that run on specific events.
  - Exposed as a library function such as `run_hooks(event_type, context, blocking_only)`.
  - Reused by both the Agent Runtime (for `PromptSubmit` / `SessionStop`) and the Tool Runner (for `PreAbilityCreate` / `PreAbilityCall` / `PostAbilityCall`).

High‑level flow for a typical interactive session:

1. User or system submits a new task/message.
2. The Agent Runtime emits `PromptSubmit` and runs blocking hooks (`PromptSubmit` is AI-facing).
3. The Agent Runtime merges hook signals into a **turn context**, then hands control to the AI developer.
4. The AI plans and produces a list of actions (e.g. `CREATE_TASK`, `RUN_TASK`).
5. For each `CREATE_TASK`:
   - The Agent Runtime delegates to the Tool Runner (`task.create`).
   - The Tool Runner emits `PreAbilityCreate` and runs blocking hooks (availability + preflight).
   - If allowed, the Tool Runner allocates a task key and persists task metadata.
6. For each `RUN_TASK`:
   - The Agent Runtime delegates to the Tool Runner (`task.run`).
   - The Tool Runner emits `PreAbilityCall` and runs blocking hooks (guardrails).
   - If allowed, the Tool Runner executes the underlying implementation.
   - After execution, the Tool Runner emits `PostAbilityCall` and runs infra-only hooks (telemetry).
7. When the session ends:
   - The Agent Runtime emits `SessionStop` and runs infra-only hooks (build/test, summaries, cache refresh, etc.).

Hooks are entirely driven by the Agent Runtime and Tool Runner; the AI never calls hooks directly. Instead, the AI sees **structured “hook signals” and control messages** injected into the turn context and/or returned as part of tool/task responses.

---

## 1. Event Model and Lifecycle

Hooks attach to **events** emitted by the Agent Runtime and Tool Runner. Each event type has a clear purpose and a specific interaction contract with the AI developer loop.

### 1.1 Event types

This document uses the following events:

- `PromptSubmit`  
  Emitted when a new user task/message is submitted. Used to:
  - Normalize intent.
  - Suggest abilities and documents.
  - Attach routing hints for the first turn.

- `PreAbilityCreate`  
  Emitted right before a task is created for an ability (`task.create`), **without executing the ability**. Used to:
  - Validate basic availability (registry exists, config resolvable, env permitted).
  - Validate input shape enough to create a task.
  - Emit structured “preflight” results (available/unavailable + reason).
  - Avoid mixing availability checks with execution guardrails.

- `PreAbilityCall`  
  Emitted right before an ability/tool is executed (`task.run` or direct run). Used to:
  - Enforce guardrails and safety checks.
  - Validate arguments and environment at execution time.
  - Decide whether the ability call is allowed to run now.

- `PostAbilityCall`  
  Emitted right after an ability/tool finishes. Used to:
  - Log usage, latency, and outcomes.
  - Update caches and quality metrics.
  - Attach metadata, but **not** to influence the current turn’s logic.

- `SessionStop`  
  Emitted when a session or logical phase ends. Used to:
  - Run build/test/lint across changed components.
  - Aggregate and persist summaries.
  - Prepare signals for future sessions (but not retroactively change this one).

### 1.2 Per‑event AI interaction contract

Not all events are allowed to interact with the AI in the same way. To keep the AI’s execution loop predictable, we distinguish:

- **AI‑facing, blocking events** – may directly shape the current turn and must run synchronously:
  - `PromptSubmit`
  - `PreAbilityCreate`
  - `PreAbilityCall`
- **Infra‑facing, post‑processing events** – may log, cache, or summarize, but **do not affect the current turn**:
  - `PostAbilityCall`
  - `SessionStop`

Conceptually:

| Event            | Timing                     | Blocking? | May directly shape current turn/call? | Typical use cases                                           |
|------------------|----------------------------|-----------|---------------------------------------|-------------------------------------------------------------|
| PromptSubmit     | Before planning            | Yes       | **Yes**                               | Routing hints, normalized intent, doc suggestions            |
| PreAbilityCreate | Before task creation        | Yes       | **Yes**                               | Availability checks, input preflight, pre-create policy      |
| PreAbilityCall   | Before ability execution    | Yes       | **Yes**                               | Guardrails, permission checks, approval requirements         |
| PostAbilityCall  | After ability execution     | No (for AI) | **No**                              | Telemetry, quality metrics, logging, cache updates           |
| SessionStop      | After session phase ends    | No (for AI) | **No**                              | Build/test/lint aggregation, summaries for future sessions   |

---

## 2. Hook Declaration (YAML schema)

Hooks are declared as small YAML files under `.system/hooks/` (one file per hook). The runtime loads these files, filters them by event type and match rules, and executes the handlers.

A typical hook file looks like:

```yaml
id: ability_usage_tracker
event_type: PostAbilityCall
enabled: true
blocking: false

summary: >
  Track each ability call and update ability_usage.json. For slow calls,
  append to slow_abilities.log.

match:
  ability_scope: "*"
  min_duration_ms: 0

handler:
  kind: script
  command: ./scripts/hooks/ability_usage_tracker.py

effects:
  - update_usage_cache: .system/cache/ability_usage.json
  - append_to: .system/logs/slow_abilities.log
```

### 2.1 Required fields

- `id`  
  - Unique identifier within the repo. Use `kebab-case`.
- `event_type`  
  - One of: `PromptSubmit`, `PreAbilityCreate`, `PreAbilityCall`, `PostAbilityCall`, `SessionStop`.
- `enabled`  
  - `true` / `false`. Disabled hooks are ignored by the runtime.
- `blocking`  
  - Controls whether hook failure or its decisions may block the main flow:
    - For `PromptSubmit`, `PreAbilityCreate`, and `PreAbilityCall`, blocking hooks are run **synchronously** and may affect whether planning, task creation, or execution proceeds.
    - For `PostAbilityCall` and `SessionStop`, `blocking` is usually `false` (AI-facing behavior is disallowed; blocking only affects infra flows).

### 2.2 Matching rules

Hooks may limit when they run with a `match` block:

```yaml
match:
  ability_scope: "*"                # glob over ability ids (ability-related events)
  min_duration_ms: 100              # only for slow calls (PostAbilityCall)
  only_if_changed_paths:
    - "services/**"
    - "apps/**"
```

Common fields:

- `ability_scope`  
  - Glob or list of globs over ability ids (for ability‑related events).
- `min_duration_ms`  
  - For `PostAbilityCall`, only run on calls slower than this threshold.
- `only_if_changed_paths`  
  - For `SessionStop`, only run if any changed files match these patterns.

#### Important: `match` is hook activation, not routing

The `match` block decides **whether this hook runs**.

It is not the repository’s routing rule language. Ability suggestions and knowledge suggestions should be produced by a small number of routing hooks (usually on `PromptSubmit`) that read routing documents and registry metadata, rather than spreading routing logic across many hook YAML files.

### 2.3 Handler spec

The `handler` block defines how to run the hook:

```yaml
handler:
  kind: script
  command: ./scripts/hooks/session_stop_build_check.sh
```

- `kind`  
  - Currently `script`. Future kinds might include `builtin` or `http`.
- `command`  
  - Script or executable path. The runtime:
    - Sends the event context as JSON on stdin.
    - Expects a JSON `HookResult` on stdout.

### 2.4 Effects (human‑readable)

The `effects` list is **descriptive only**: it documents what the handler intends to do. It is not an execution spec.

---

## 3. Contexts, Hook Results, and Hook Signals

Handlers receive structured JSON context and return structured results. The runtime and AI developer both rely on this structure to reason about hook behavior.

### 3.1 Event contexts (overview)

Each event type has its own context shape; here we outline the key fields.

#### PromptSubmitContext (AI‑facing)

```ts
interface PromptSubmitContext {
  event_type: "PromptSubmit";
  session_id: string;
  user_raw_input: string;
  current_files?: string[];
}
```

Hooks for this event may emit **routing hints** and **normalized intent**.

#### PreAbilityCreateContext (AI‑facing)

Preflight checks run at task creation time and should not assume the ability will run.

```ts
type InvocationSource =
  | "ai_session"
  | "background_job"
  | "ci_pipeline"
  | "manual_cli";

interface PreAbilityCreateContext {
  event_type: "PreAbilityCreate";
  session_id?: string;

  // The target ability (high-level id) or low-level operation key.
  ability_ref: {
    kind: "ability_id" | "operation_key";
    value: string;
  };

  // A summary of input, not necessarily the full payload.
  input_summary?: string;

  // Optional runtime environment (dev/staging/prod), if known.
  environment?: string;

  caller: {
    source: InvocationSource;
    session_id?: string;
    job_id?: string;
    triggered_by?: string;
  };
}
```

Hooks for this event may emit **preflight** signals (available/unavailable + reason) and may block task creation.

#### PreAbilityCallContext (AI‑facing)

```ts
interface PreAbilityCallContext {
  event_type: "PreAbilityCall";
  session_id?: string;
  ability_id: string;

  // Task key is present for task-based runs.
  task_key?: string;

  // Optional summarized args / targets.
  args_summary?: string;

  caller: {
    source: InvocationSource;
    session_id?: string;
    job_id?: string;
    triggered_by?: string;
  };
}
```

Hooks for this event may **allow, deny, or require human approval** for a specific ability call.

#### PostAbilityCallContext (infra‑facing)

```ts
interface PostAbilityCallContext {
  event_type: "PostAbilityCall";
  session_id?: string;
  ability_id: string;
  task_key?: string;
  status: "success" | "failure" | "partial";
  duration_ms?: number;
  input_summary?: string;
  output_summary?: string;
  error_message?: string;
}
```

PostAbilityCall hooks are infra‑only: they log, track usage, and update caches. They do **not** send AI‑facing signals for the current turn.

#### SessionStopContext (infra‑facing)

```ts
interface SessionStopContext {
  event_type: "SessionStop";
  session_id: string;
  changed_files?: string[];
  abilities_used?: string[];
  tasks_summary?: Array<{
    task_key: string;
    ability_id: string;
    status: "success" | "failure" | "partial";
  }>;
  started_at?: string;
  ended_at?: string;
}
```

### 3.2 HookResult: raw handler output

Each handler writes a JSON object to stdout. The runtime merges all results for an event.

```ts
interface HookResult {
  // Infra fields: logs, telemetry, cache hints
  logs?: string[];
  usage_recorded?: boolean;

  // AI-facing events (PromptSubmit / PreAbilityCreate / PreAbilityCall) only:
  hook_signals?: HookSignal[];

  error?: {
    code: string;
    message: string;
  };
}
```

### 3.3 HookSignal: AI‑visible signals

To avoid every hook inventing its own schema, we standardize hook signals.

```ts
type HookSignalKind =
  | "routing_hint"       // PromptSubmit: ability/doc suggestions
  | "normalized_intent"  // PromptSubmit: structured task description
  | "ability_preflight"  // PreAbilityCreate: availability + preflight result
  | "ability_guard";     // PreAbilityCall: allow/deny/require-human

interface HookSignal {
  source_event: "PromptSubmit" | "PreAbilityCreate" | "PreAbilityCall";
  hook_id: string;
  kind: HookSignalKind;
  code: string;
  severity?: "info" | "warning" | "error";
  payload: any;
}
```

#### PromptSubmit hook signals

PromptSubmit hooks may emit:

- `routing_hint` (abilities + documents)
- `normalized_intent` (scope/topic/stage aligned with knowledge routing)

Example `routing_hint`:

```jsonc
{
  "source_event": "PromptSubmit",
  "hook_id": "prompt_router",
  "kind": "routing_hint",
  "code": "ROUTE_SUGGESTION",
  "payload": {
    "suggested_abilities": [
      { "id": "signup_e2e_test", "reason": "User asked for signup regression.", "confidence": 0.91 }
    ],
    "suggested_documents": [
      { "path": "integration/ROUTING.md", "reason": "Cross-module E2E request." }
    ]
  }
}
```

#### PreAbilityCreate hook signals

PreAbilityCreate hooks may emit `ability_preflight` signals:

```jsonc
{
  "source_event": "PreAbilityCreate",
  "hook_id": "nonprod_availability_check",
  "kind": "ability_preflight",
  "code": "ABILITY_UNAVAILABLE",
  "severity": "error",
  "payload": {
    "ability_ref": { "kind": "operation_key", "value": "db.write.user_row" },
    "reason": "Operation is not enabled in prod environments.",
    "suggestion": "Switch to staging/dev, or request human override."
  }
}
```

Interpretation:

- `ABILITY_AVAILABLE` – task creation may proceed (optionally with warnings).
- `ABILITY_UNAVAILABLE` – no task should be created; return reason.
- `ABILITY_REQUIRES_HUMAN` – task creation should stop and ask for approval/input.

#### PreAbilityCall hook signals

PreAbilityCall hooks may emit `ability_guard` signals:

```jsonc
{
  "source_event": "PreAbilityCall",
  "hook_id": "prod_config_guard",
  "kind": "ability_guard",
  "code": "ABILITY_DENIED",
  "severity": "error",
  "payload": {
    "ability_id": "edit_file",
    "reason": "Editing prod config files is not allowed from ai_session.",
    "require_human": true
  }
}
```

### 3.4 Event‑level constraints

The Agent Runtime and Tool Runner enforce per‑event constraints on `HookResult`:

- For `PromptSubmit`, `PreAbilityCreate`, and `PreAbilityCall`:
  - `hook_signals` are allowed and may be surfaced to the AI (either as part of turn context or as part of tool/task responses).
- For `PostAbilityCall` and `SessionStop`:
  - Any AI‑facing fields (e.g. `hook_signals`) are ignored (and optionally logged as misconfiguration).

---

## 4. Agent Runtime Behavior

The Agent Runtime is the **single source of truth** for when session-level events are emitted and how hooks are orchestrated around AI turns.

### 4.1 Turn pipeline (PromptSubmit)

High-level pseudocode for the start of a turn:

```ts
async function handleUserInput(rawInput: string, session: SessionState) {
  const promptCtx = buildPromptSubmitContext(rawInput, session);
  const promptResults = await runHooks("PromptSubmit", promptCtx, { blockingOnly: true });
  const promptSignals = collectHookSignals(promptResults);

  const turnContext = {
    user_raw_input: rawInput,
    hook_signals: promptSignals,
  };

  const startControl = {
    type: "control",
    turn_ready: true,
    source_event: "PromptSubmit",
    hook_signals: promptSignals,
  };

  const plan = await callAiPlanner(startControl, turnContext);

  // plan may include CREATE_TASK and RUN_TASK steps, handled below...
}
```

### 4.2 Task creation (PreAbilityCreate)

When the AI requests `CREATE_TASK`, the Agent Runtime delegates to the Tool Runner (`task.create`). The Tool Runner:

1. Builds a `PreAbilityCreateContext`
2. Runs blocking hooks for `PreAbilityCreate`
3. If denied/unavailable, returns the preflight result
4. If allowed, allocates a task key and persists the task

The Agent Runtime should surface the preflight result to the AI as part of the `CREATE_TASK` response (and may also log it in session state).

### 4.3 Task execution (PreAbilityCall / PostAbilityCall)

When the AI requests `RUN_TASK`, the Tool Runner:

1. Builds a `PreAbilityCallContext`
2. Runs blocking hooks for `PreAbilityCall`
3. Interprets `ability_guard` signals
4. Executes the ability if allowed
5. Emits `PostAbilityCall` and runs infra-only hooks

### 4.4 Background and non‑AI callers

Abilities may also be invoked by cron/background jobs, CI pipelines, or manual CLI scripts.

To keep hooks effective in these scenarios:

- All invocations must go through the Tool Runner API/CLI.
- The Tool Runner sets `caller.source` appropriately.
- Guardrails and preflight hooks apply consistently across AI and non-AI callers.

---

## 5. Patterns and Recipes

### 5.1 Telemetry and usage tracking (PostAbilityCall)

- **Event**: `PostAbilityCall`  
- **blocking**: `false`  
- **Goal**: centralize telemetry about ability use.

Typical YAML:

```yaml
id: ability_usage_tracker
event_type: PostAbilityCall
enabled: true
blocking: false

match:
  ability_scope: "*"

handler:
  kind: script
  command: ./scripts/hooks/ability_usage_tracker.py

effects:
  - update_usage_cache: .system/cache/ability_usage.json
```

### 5.2 Ability and document suggestions (PromptSubmit)

- **Event**: `PromptSubmit`  
- **blocking**: `true`  
- **Goal**: suggest relevant abilities/documents and normalize intent.

Typical YAML:

```yaml
id: prompt_router
event_type: PromptSubmit
enabled: true
blocking: true

summary: >
  Suggest a small set of relevant abilities and documents as routing hints.

handler:
  kind: script
  command: ./scripts/hooks/prompt_router.py

effects:
  - suggest_abilities
  - suggest_documents
  - emit_normalized_intent
```

### 5.3 Availability / preflight checks (PreAbilityCreate)

- **Event**: `PreAbilityCreate`  
- **blocking**: `true`  
- **Goal**: block task creation when an ability is not callable in the current env/config.

Typical YAML:

```yaml
id: nonprod_availability_check
event_type: PreAbilityCreate
enabled: true
blocking: true

match:
  ability_scope: "*"

handler:
  kind: script
  command: ./scripts/hooks/nonprod_availability_check.py

effects:
  - enforce_environment_availability
  - validate_minimum_config
```

### 5.4 Safety and permission checks (PreAbilityCall)

- **Event**: `PreAbilityCall`  
- **blocking**: `true`  
- **Goal**: prevent dangerous or misconfigured ability executions.

Typical YAML:

```yaml
id: prod_config_guard
event_type: PreAbilityCall
enabled: true
blocking: true

match:
  ability_scope: "edit_file"

handler:
  kind: script
  command: ./scripts/hooks/prod_config_guard.py

effects:
  - enforce_prod_safety
```

### 5.5 Build/test aggregation and summaries at SessionStop

- **Event**: `SessionStop`  
- **blocking**: usually `false` (for AI)  
- **Goal**: run lightweight build/test commands and persist summaries.

Typical YAML:

```yaml
id: session_stop_build_check
event_type: SessionStop
enabled: true
blocking: false

match:
  only_if_changed_paths:
    - "services/**"
    - "apps/**"

handler:
  kind: script
  command: ./scripts/hooks/session_stop_build_check.sh

effects:
  - run_local_build_checks
  - summarize_errors_for_humans
```

---

## 6. Observability, Debugging, and Governance

Hooks can improve observability but can also become a source of confusion if not governed carefully.

### 6.1 Listing hooks

Provide a simple command or ability to list all registered hooks, e.g.:

```bash
./scripts/hooks/list
```

Example output:

```text
id                      event_type      enabled blocking match_summary
----------------------  -------------   ------- -------- --------------------------
skill_activation_prompt PromptSubmit    true    true     all sessions
prod_config_guard       PreAbilityCall  true    true     ability=edit_file
ability_usage_tracker   PostAbilityCall true    false    ability=* 
session_stop_build_check SessionStop    true    false    paths=services/**,apps/**
```

This helps both humans and the AI quickly understand what infra is in play.

### 6.2 Logging

Hooks should log enough to debug issues without overwhelming logs:

- At least log:
  - `hook_id`
  - `event_type`
  - `status` (success/failure)
  - `duration_ms`
- For blocking hooks, log when they **deny** or **require human** decisions.

### 6.3 Handling failures and noise

- **Blocking hooks**:
  - On failure, the runtime may:
    - Treat it as a soft failure and skip the hook (logging an error), or
    - Fail the event entirely, depending on configuration.
  - For guardrails, prefer explicit `ability_guard` or `ability_preflight` signals over throwing exceptions.
- **Non‑blocking hooks**:
  - Failures should not block AI progress.
  - Log errors and continue.

If a hook becomes too noisy (e.g., constantly denying calls the team considers safe), treat it as a design issue and revisit its logic or matching rules.

### 6.4 Avoiding hook sprawl

To keep `.system/hooks/` manageable:

- Use clear, descriptive ids (`prod_config_guard`, `ability_usage_tracker`).
- Group hooks logically by event_type or purpose when browsing.
- Periodically review and prune unused or obsolete hooks.
- Prefer shared patterns (logging, simple guardrails) over deeply nested hook logic.

---

## 7. Design Principles (Recap)

- Hooks are **event-driven infrastructure**: they wrap the AI’s work with routing hints, guardrails, telemetry, and summaries.
- Only `PromptSubmit`, `PreAbilityCreate`, and `PreAbilityCall` may shape the current turn or a tool call.
- `PostAbilityCall` and `SessionStop` are infra-only: they should not modify the current planning loop.
- Keep `match` blocks small and stable; encode routing suggestions in routing docs and registry metadata, surfaced by a small number of routing hooks.
