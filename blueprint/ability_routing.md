# Ability Routing

## 0. Purpose and Design Goals

### Roles & Terms

This document follows the shared glossary defined for AI-facing docs in this repository (for example, the project-level `AGENTS.md` and any shared glossary it references). It uses the standard terms `AI developer`, `Agent Runtime`, `Tool Runner`, `Hook Runner`, and `humans` as defined there and does not introduce additional role names.

Ability routing is the infrastructure that makes it easy for an AI-first project to reuse existing tools instead of “reinventing the wheel”, while keeping the code base transparent, consistent, and controllable.

In this template:

- An **ability** is a packaged piece of functionality that can be invoked by an AI or a human.
- The project maintains a **single ability pool** and a **standard Tool Runner** so that abilities can be discovered, validated, and called in a uniform way.
- Abilities are split into **low-level** and **high-level** layers to keep orchestration manageable and to align with how a code model plans and executes work.

The core design goals are:

1. **Maximize AI development efficiency**: the AI should be able to quickly discover and apply the right abilities for a given task, without reading large amounts of implementation code.
2. **Keep orchestration manageable**: orchestration logic should remain at an abstract level while low-level details are encapsulated behind well-defined abilities.
3. **Support continuous evolution**: as the project grows and the AI generates new scripts and flows, the ability routing system should absorb these outputs and evolve with real usage.
4. **Maintain control and safety**: all ability executions go through a common Tool Runner and Hook Runner with configuration and hooks, so that behavior is auditable and adjustable.

---

## 1. Concepts and Rules

### 1.1 Routing Documents (`ABILITY.md`)

A routing document is the human- and AI-readable entry point into the ability system. Every routing document is named `ABILITY.md`.

Routing documents serve two purposes:

1. **Discovery** – help the AI quickly understand which abilities exist in the current working domain.
2. **Navigation** – provide stable paths to the underlying ability registries, without exposing low-level implementation details.

The template follows a modular development model. Code is organized into **module instances**. Each module instance defines its own working boundary. For each module instance, there is a dedicated routing document: `<module-root>/ABILITY.md`.

In addition, the project maintains a **top-level integration routing document** for end-to-end or cross-module scenarios, typically under an integration or system directory, for example: `integration/ABILITY.md`.

In normal development, the AI’s working scope is well-defined, so one orchestration run should only need to read **one** `ABILITY.md`:

- Module-level `ABILITY.md` for feature development and local testing.
- Integration-level `ABILITY.md` for end-to-end or cross-module testing.

Routing documents evolve together with the project. When orchestration hits gaps (for example, no ability exists for a common operation), the Agent Runtime, Tool Runner, and CI tools can capture that usage and feed it back to improve `ABILITY.md` over time.

#### High-level vs. Low-level Abilities in `ABILITY.md`

Both high-level and low-level abilities live in the **same ability pool** but are presented differently in routing documents:

- **High-level abilities** are preferred and shown first. They represent workflows or agents that cover meaningful chunks of work (e.g. “run end-to-end signup test”).
- **Low-level abilities** represent atomic operations (e.g. “insert row into user table”) and are listed as fallbacks when no suitable high-level ability exists.

The AI should:

1. Scan high-level abilities first, matching **intent and scenario**.
2. Only fall back to low-level abilities when high-level ones cannot cover all orchestration steps.

#### Example: High-level Ability Routing Structure

A typical `ABILITY.md` may describe high-level abilities in a table:

```markdown
## High-level abilities

| Id                    | Type     | Scope                | When to use                                             | Registry path                                  |
| --------------------- | -------- | -------------------- | ------------------------------------------------------- | ---------------------------------------------- |
| signup_e2e_test       | workflow | integration/test     | Run full user signup E2E test against non-prod env     | .system/registry/high-level/signup_e2e.yaml    |
| billing_regression    | workflow | integration/billing  | Run billing regression suite with seeded test data     | .system/registry/high-level/billing_reg.yaml   |
| code_review_agent     | agent    | dev/code-review      | Review PRs for style, safety, and interface changes    | .system/registry/high-level/code_review.yaml   |
```

Key fields:

- `Id`: unique ability identifier used by the Tool Runner and Agent Runtime.
- `Type`: `workflow` or `agent`.
- `Scope`: domain or module where this ability is valid.
- `When to use`: natural-language intent description to help semantic matching.
- `Registry path`: path to the YAML registry that defines the ability.

#### Example: Low-level Ability Routing Structure

Low-level abilities are usually grouped by operation type and technology:

```markdown
## Low-level abilities (atomic operations)

| Operation key                 | Kind    | Scope               | Summary                                           | Registry path                                       |
| ---------------------------- | ------- | ------------------- | ------------------------------------------------ | --------------------------------------------------- |
| db.write.user_row            | script  | infra/db            | Write a single user row into the users table     | .system/registry/low-level/db.write.user_row.yaml   |
| http.call.billing_service    | api     | infra/http          | Call billing service HTTP endpoints              | .system/registry/low-level/http.call.billing.yaml   |
| mcp.query.logs               | mcp     | infra/observability | Query logs via MCP log server                    | .system/registry/low-level/mcp.query.logs.yaml      |
```

The AI primarily matches:

- **High-level abilities** by *intent* and *scope*.
- **Low-level abilities** by *operation key* and *summary*.

Routing documents should stay compact and focused on what is actually useful in that working domain. They do not need to list every ability in the global pool, only those that are relevant to that module or integration scenario.

---

### 1.2 Ability Pool

The project maintains a centralized **ability pool** – a set of all registered abilities that are allowed to be called. The ability pool is the single source of truth for:

- What the ability is supposed to do.
- How to call it (via the Tool Runner task interface).
- Where the implementations live.
- Which configuration and hook bindings apply.

We distinguish between **low-level abilities** and **high-level abilities**.

#### 1.2.1 Low-level abilities

Low-level abilities are **atomic operations**, usually tied to concrete implementations:

- Repository scripts
- External services (through MCP)
- External APIs or cloud providers
- Database operations, queue operations, etc.

Low-level abilities are classified by *implementation kind*:

- `script` – project-internal scripts or binaries.
- `mcp` – MCP (Model Context Protocol) tools exposed by external or internal MCP servers.
- `api` – HTTP/RPC API calls to external services or cloud providers.

For each kind, the project maintains **operation-level** and **implementation-level** metadata.

##### Example: `script` ability (conceptual)

- Operation key: `db.write.user_row`
- Implementation: `script`
- Implementation data:
  - Script path: `scripts/db/write_user_row.sh`
  - Invocation style: `./scripts/db/write_user_row.sh <json-payload>`
  - Required environment variables: `DB_HOST`, `DB_USER`, `DB_NAME`
  - Side effects: writes to the `users` table in the non-production database

##### Example: `mcp` ability (conceptual)

- Operation key: `mcp.query.logs`
- Implementation: `mcp`
- Implementation data:
  - MCP server id: `observability-logs`
  - MCP tool: `search_logs`
  - Typical filters: service name, time window, severity
  - Constraints: available only in certain environments

##### Example: `api` ability (conceptual)

- Operation key: `http.call.billing_service`
- Implementation: `api`
- Implementation data:
  - Base URL: `https://billing.dev.internal/api`
  - Authentication: `BILLING_API_TOKEN` or service account
  - Endpoints: `/charge`, `/refund`, `/invoice`
  - Rate limits: per minute and per day

In all cases, the AI should not call implementations directly. Instead, it uses the **Agent Runtime**, which calls the **Tool Runner** to load configuration, run hooks (via the Hook Runner), and execute the correct implementation for the selected operation key.

#### 1.2.2 High-level abilities

High-level abilities are semantic capabilities composed from low-level abilities. They represent the tasks that the AI developer or AI planner should prefer.

High-level abilities come in two types:

- `workflow` – relatively deterministic pipelines, usually sequential or with limited branching (e.g. “run build + unit tests + integration tests”).
- `agent` – AI-driven flows that can dynamically choose which low-level abilities to call, often requiring reasoning loops and flexible decision making (e.g. “debug failing test given logs and codebase”).

##### Example: `workflow` ability (conceptual)

- Id: `signup_e2e_test`
- Type: `workflow`
- Description: run the full user-signup flow against staging, including DB seeding and cleanup.
- Steps:
  1. Call `db.write.user_row` to seed a test user.
  2. Call `http.call.signup_service` with test data.
  3. Call `mcp.query.logs` to verify no errors were emitted.
  4. Call `db.read.user_row` to verify persisted state.

##### Example: `agent` ability (conceptual)

- Id: `code_review_agent`
- Type: `agent`
- Description: review a pull request, focusing on safety, style, and public interfaces.
- Behavior:
  - Reads PR diff and context.
  - Calls diagnostic abilities as needed.
  - Generates structured review comments.

In both cases, high-level abilities are executed via the same **Tool Runner**, not by the AI calling low-level tools directly.

---

### 1.3 Ability Registries

Routing documents (`ABILITY.md`) **do not** contain full technical specifications. Instead, they point to **ability registries**: YAML documents that define:

- Input and output contracts.
- Scope and usage conditions.
- Constraints and hook bindings.
- Mapping to implementations and configuration.

Registries are the **single source of truth** (SSOT) for ability semantics. The AI uses routing documents to **find** abilities and registry entries to **understand and safely call** them.

There are separate registry formats for low-level and high-level abilities.

#### 1.3.1 Low-level Ability Registry

A low-level ability corresponds to an **atomic operation type**, not a specific implementation. For example, `db.write.user_row` is an operation; it may have multiple concrete implementations.

The primary question for orchestration time is:

> “Can this operation type be inserted into my execution chain?”

The Tool Runner will then choose a concrete implementation based on configuration and environment.

##### Primary registry contents (operation-level)

Each low-level registry entry describes an **operation key**. Typical fields:

- **Registration info**
  - `operation_key` (primary key, e.g. `db.write.user_row`)
  - `version` (optional semantic version of the contract)
- **Operation description**
  - Natural-language summary
  - Intended use cases
  - Idempotency / side-effect notes
- **Scope**
  - Module(s) or domain(s) where this operation is valid
  - Allowed environments (e.g. `dev`, `staging`, but not `prod`)
- **Input**
  - Structured description or schema (JSON-schema-like; at minimum: required fields, types, and constraints)
- **Output**
  - Structured description or schema (success result, error shape, partial results)
- **Routing metadata (ability-side, human-maintained)**
  - `routing_hints`: small intent-oriented hints used to suggest this operation during `PromptSubmit`
- **Hook bindings**
  - `hooks.pre_create`: Preflight hooks for task creation (`PreAbilityCreate`)
  - `hooks.pre_call`: Guardrail hooks for execution (`PreAbilityCall`)
  - `hooks.post_call`: Telemetry hooks after execution (`PostAbilityCall`)

Example (conceptual, YAML):

```yaml
operation_key: db.write.user_row
version: 1
summary: Write a single user row into the users table (non-prod only).
scope:
  modules: ["users.api", "integration/tests"]
  environments: ["dev", "staging"]

input_schema:
  type: object
  required: ["user_id", "email"]
  properties:
    user_id: { type: string }
    email: { type: string, format: email }

output_schema:
  type: object
  required: ["status"]
  properties:
    status: { type: string, enum: ["success", "error"] }
    error_code: { type: string }

routing_hints:
  keywords: ["insert user", "seed user", "create test user"]
  negative_keywords: ["prod", "production"]
  notes: "Prefer high-level signup workflows if available."

hooks:
  pre_create: ["nonprod_availability_check"]
  pre_call: ["prod_write_guard"]
  post_call: ["ability_usage_tracker"]
```

##### Implementation documents

Implementation documents describe **how** an operation is implemented. This is mostly for humans and automation, not required reading for the AI.

Typical fields:

- Registration info
  - Implementation id
  - Implementation kind: `script` | `mcp` | `api`
- Operation linkage
  - Operation key this implementation belongs to
- Invocation & behavior
  - Invocation method (script path, MCP server/tool id, API endpoint, etc.)
  - Constraints and limitations
  - Default parameters and override rules
- Tags
  - E.g. `fast`, `experimental`, `deprecated`, `high-cost`

##### Configuration documents

Configuration documents store environment-specific settings and behavior. They are referenced by both operations and implementations.

For each operation key, typical fields:

- Implementation selection
  - Current default implementation id
  - List of allowed implementation ids
- Runtime parameters (per environment or context)
  - `mcp_server`, `mcp_tool`
  - `endpoints` / base URLs
  - Credentials references (never inline secrets)
  - Prompt templates or hints for AI
- Hook bindings (optional overrides)
  - Ability-level hook bindings may be declared at operation-level, implementation-level, or config-level.
  - **Resolution rule** should be deterministic and documented by the Tool Runner (for example: operation → implementation → env config).

The AI interacts with configuration only indirectly through the Agent Runtime and Tool Runner. The Tool Runner is responsible for interpreting configuration, resolving hook bindings, and selecting the correct implementation.

#### 1.3.2 High-level Ability Registry

A high-level ability corresponds to an **abstract functional capability** rather than a single operation. It combines multiple low-level operations to achieve a larger goal.

The primary key is the **ability id**, not an operation key. One registry entry maps to one orchestrated capability.

Typical fields:

- **Registration info**
  - `id`: unique ability id (e.g. `signup_e2e_test`)
  - `type`: `workflow` | `agent`
  - `version`: optional semantic version
- **Intent and scenarios**
  - Natural-language description of what the ability does
  - Example user stories or scenarios
- **Scope**
  - Modules, domains, or integration boundaries where this ability applies
- **Input / Output**
  - Schemas and example payloads
- **Execution behavior**
  - Orchestration description
  - Constraints (runtime, cost, environment)
  - Callable low-level operation keys (whitelist)

High-level abilities may also include:

- `routing_hints` (used for suggestions during `PromptSubmit`)
- `hooks.pre_create` / `hooks.pre_call` / `hooks.post_call` for preflight, guardrails, and telemetry.

Example (conceptual, YAML):

```yaml
id: signup_e2e_test
type: workflow
version: 2
summary: Run the full signup E2E regression against staging (seed + run + verify + cleanup).
scope:
  environments: ["staging"]

input_schema:
  type: object
  required: ["build_id"]
  properties:
    build_id: { type: string }

output_schema:
  type: object
  required: ["status"]
  properties:
    status: { type: string, enum: ["success", "failure"] }
    report_path: { type: string }

routing_hints:
  keywords: ["signup e2e", "registration regression", "create account end-to-end"]
  notes: "Preferred over assembling low-level operations for signup verification."

hooks:
  pre_create: ["staging_readiness_check"]
  pre_call: ["approval_required_if_high_cost"]
  post_call: ["ability_usage_tracker"]
```

---

### 1.4 Execution Lifecycle and Hook Integration

Ability routing interacts with hooks at **three distinct phases**. This separation is intentional and avoids mixing “availability checks” with “execution guardrails”.

1. **PromptSubmit (routing suggestions)**
   - The Agent Runtime emits `PromptSubmit`.
   - PromptSubmit hooks may emit `routing_hint` and `normalized_intent` signals.
   - These signals are suggestions: which abilities and documents to consider.
   - **Important:** PromptSubmit is about *routing hints*, not about *permission to execute*.

2. **Task creation (preflight, non-executing)**
   - The Tool Runner receives `task.create` (or an equivalent “create task” action).
   - The Tool Runner emits `PreAbilityCreate` and runs `hooks.pre_create` + any matching global hooks.
   - The goal is to answer: **“Is this ability callable in the current environment/config, and is the input shape valid enough to create a task?”**
   - If the preflight denies creation, no task key is allocated.

3. **Task execution (guardrails, executing)**
   - The Tool Runner receives `task.run`.
   - The Tool Runner emits `PreAbilityCall` and runs `hooks.pre_call` + any matching global hooks.
   - The goal is to answer: **“Is it safe and allowed to execute right now?”**
   - If allowed, the Tool Runner executes the implementation.
   - After execution, it emits `PostAbilityCall` (infra-only) and runs `hooks.post_call`.

This lifecycle also applies to one-off “direct run” calls: direct execution should still run `PreAbilityCreate` (preflight) and `PreAbilityCall` (guardrails) in order, even if no durable task key is created.

---

## 2. Usage and Typical Flows

High-level and low-level abilities form a single, unified tool pool. Ability routing ensures that once the AI has a task plan, it can map each step to one or more concrete abilities and execute them through the common Tool Runner.

### 2.1 Task Orchestration Perspective

From the AI developer’s internal orchestration perspective (working through the Agent Runtime in interactive sessions), the typical process is:

1. **Identify the correct `ABILITY.md`**
   - Based on the current working scope (module or integration).
   - Read `ABILITY.md` to inventory available high-level and low-level abilities.

2. **Select candidate abilities**
   - Prefer high-level abilities that match intent and scope.
   - Use low-level abilities to fill gaps.

3. **Create tasks (preflight and staging)**
   - For each selected ability, ask the Agent Runtime to create a task via the Tool Runner.
   - Task creation runs `PreAbilityCreate` hooks. This may return:
     - “available + warnings” (task key issued),
     - or “unavailable” (no task key issued, with reason).

4. **Finalize and execute**
   - Construct an execution graph from task keys.
   - Execute tasks (`task.run`) in the planned order.
   - Execution runs `PreAbilityCall` guardrails and may be denied even if task creation succeeded.

5. **React and iterate**
   - Handle success/failure outputs.
   - If a call is denied or unavailable, adjust the plan (different ability, different env, request human approval, etc.).

#### Example scenario (conceptual)

- Task: “Run signup E2E regression for the latest build.”
- Plan:
  - Prefer `signup_e2e_test` high-level workflow.
  - If not available, assemble low-level operations:
    - `db.write.user_row` to seed data.
    - `http.call.signup_service` for API calls.
    - `mcp.query.logs` to validate no errors.

---

### 2.2 Ability Routing Perspective

From the ability routing perspective, once the AI knows what it wants to do, the flow looks like:

1. The AI reads the relevant `ABILITY.md` and selects a set of candidate registry paths.
2. The AI reads the corresponding registry entries to decide which abilities to use.
3. For each selected ability, the AI asks the Agent Runtime to **create a task** (Tool Runner `task.create`) and receives a task key **only if preflight passes**.
4. The AI builds its orchestration plan around those task keys.
5. When it is time to execute, the AI asks the Agent Runtime to run the task (`task.run`).
6. The Tool Runner executes the underlying implementation(s) and returns outputs in the registered format.

---

### 2.3 Directional Invocation (summary)

Directional invocation means explicitly selecting and calling an ability, either manually or by giving the AI a precise instruction to do so. There are three main patterns:

1. **Task-based (recommended, via Agent Runtime / Tool Runner)**
   - Follow the standard flow: `task.create` (with `PreAbilityCreate` preflight) → `task.run` (with `PreAbilityCall` guardrails).
   - Use this path for most abilities that touch shared state; it records intent, configuration, and results under `.system/implement/<task-key>/tool_call.md`.

2. **Direct Tool Runner execution (non-session / low-risk contexts)**
   - A human or automation reads the registry and calls the Tool Runner directly with:
     - Ability id or operation key.
     - Input payload according to the registry.
   - This still runs `PreAbilityCreate` + `PreAbilityCall`, but may skip durable task storage; use for utilities and debugging where traceability is less critical.

3. **Raw tool invocation (not recommended)**
   - Call underlying scripts, MCP tools, or APIs directly, without going through the ability system.
   - This bypasses ability-level logging, hooks, and configuration; reserve it for exceptional scenarios only (for example emergency debugging with human experts).

In all cases, the preferred direction for AI-driven work is: **start from `ABILITY.md` → pick abilities → call them via the Tool Runner**, falling back to raw tools only when strictly necessary.

---

### 2.4 Tool Runner (standard ability executor)

The template provides a **single, global Tool Runner** (library and/or CLI) as the execution entry point for abilities.

In interactive sessions, external AI systems normally call the Agent Runtime, which in turn uses the Tool Runner instead of calling individual scripts, MCP tools, or APIs directly.

In background contexts (CI, cron, admin CLIs), it is acceptable to call the Tool Runner directly, subject to project policy.

#### Interfaces

The Tool Runner exposes at least the following conceptual interfaces:

1. **Create task** *(does not execute)*
   - Input:
     - Ability id or operation key.
     - Structured input payload according to the registry.
   - Behavior:
     - Validates input against registry schemas (shape-level validation).
     - Resolves configuration (environment, implementation selection).
     - Triggers `PreAbilityCreate` hooks via the Hook Runner:
       - Global `PreAbilityCreate` hooks (from `.system/hooks/*.yaml`)
       - Ability-level `hooks.pre_create` bindings from registries/config
     - If preflight denies creation:
       - Return “unavailable” with a clear reason.
       - Do **not** allocate a task key or create task folders.
     - If allowed:
       - Allocate a unique task key.
       - Create a folder: `/.system/implement/<key>/`.
       - Create a record file: `/.system/implement/<key>/tool_call.md` (or equivalent).
       - Cache input and resolved configuration.
   - Output:
     - Task key (e.g. `task-123`) when available.
     - Availability status + warnings.

   Example response (conceptual):

   ```json
   {
     "status": "available",
     "task_key": "task-123",
     "warnings": [
       "This ability is slow; prefer running it off-peak in staging."
     ]
   }
   ```

   Example “unavailable” response:

   ```json
   {
     "status": "unavailable",
     "reason": "Missing required configuration: BILLING_API_TOKEN (staging).",
     "suggestion": "Ask a human to provide credentials or run in dev with mock billing."
   }
   ```

2. **Run task** *(executes using cached data)*
   - Input:
     - Task key.
   - Behavior:
     - Loads cached input and configuration.
     - Triggers `PreAbilityCall` hooks via the Hook Runner:
       - Global `PreAbilityCall` hooks
       - Ability-level `hooks.pre_call` bindings
     - If allowed:
       - Executes the underlying implementation(s).
     - Records execution details and results in `tool_call.md`.
     - Triggers `PostAbilityCall` hooks (infra-only):
       - Global `PostAbilityCall` hooks
       - Ability-level `hooks.post_call` bindings
   - Output:
     - Standardized result structure from the registry (success/error/payload).

3. **Direct run (no durable task)** *(execute once without persisting a task)*
   - Input:
     - Ability id or operation key.
     - Input payload.
   - Behavior:
     - Runs `PreAbilityCreate` (preflight) followed by `PreAbilityCall` (guardrails).
     - Executes directly if allowed.
     - Runs `PostAbilityCall` hooks (infra-only).
     - May optionally write an ephemeral record for observability if configured.
   - Output:
     - Execution result.

4. **Query/modify task payload** *(optional but recommended)*
   - Query by task key to inspect current input.
   - Modify one or more input fields by key.
   - Useful when the AI or a human wants to adjust parameters before execution.

5. **Delete task**
   - Deleting a task cleans up cached data and any related local artifacts.

#### Why split PreAbilityCreate vs PreAbilityCall?

This split keeps semantics clean:

- `PreAbilityCreate` answers: **can we form a runnable task?** (registry/config/input availability)
- `PreAbilityCall` answers: **can we execute now?** (safety/permissions/side effects)

This avoids accidental behavior like “task creation triggers production guardrails”, which makes orchestration brittle.

---

## 3. Registration and Maintenance

### 3.1 Ability Registration

Apart from template-provided implementations, abilities enter the system through:

1. **Continuous integration of development outputs** (AI or humans add new scripts or flows).
2. **Manual registration** using template-provided scaffolding tools.

Every ability, regardless of level or type, must be registered through a common registration process. Registration ensures:

- The registry entries follow the schemas.
- References to implementations and configurations are valid.
- `ABILITY.md` documents are updated consistently.

### 3.2 Routing Maintenance

Day-to-day development uses:

- Module-level `ABILITY.md` for feature work.
- Integration-level `ABILITY.md` for end-to-end and integration testing.

These two entry points are the primary maintenance targets.

#### 3.2.1 Sustainable Evolution

In this template, ability routing is expected to evolve continuously with real usage, especially in AI-heavy development.

- During normal module work, new helper scripts are typically stored under `modules/<module_id>/workdocs/outcomes/scripts/` with metadata defined in `modular_mannual.md`.
- When a script proves reusable, it can be promoted into a low-level ability in the registry and referenced from configuration.
- Routing entries in `ABILITY.md` should then be updated to prefer this new ability over ad-hoc scripts.

Over time, the system should:

- **Harvest actual outputs**:
  - Capture the intent of new scripts and flows (what problem they solve).
  - Record which abilities were used vs. when new code was created.
  - Keep track of where new scripts live and how they are documented.
- **Feed usage back into the pool**:
  - Compare new scripts/flows with existing abilities and decide when to promote them.
  - Add new low-level operations where gaps are obvious.
  - Identify stable combinations of low-level abilities that deserve promotion into new high-level workflows or agents.

The long-term goal is that the ability pool and routing documents (`ABILITY.md`) **adapt to real work**: frequently used patterns become first-class abilities, while obsolete or rarely used entries can be pruned or deprecated.

### 3.3 Other Considerations

#### 3.3.1 Hooks: Routing Hints, Preflight, and Guardrails

Hooks and ability routing are intentionally aligned, but they must not be confused.

- **`routing_hints` (ability-side metadata)**:
  - Lives next to the ability definition (registry/config).
  - Is maintained by humans close to the ability itself.
  - Is used by PromptSubmit routing hooks to suggest abilities.
  - It is *not* a hook selector.

- **`match:` (hook-side activation filter in `.system/hooks/*.yaml`)**:
  - Decides whether a hook runs for a given event.
  - It is infra-level and should remain small and stable.
  - It must not become the primary place where ability suggestions are encoded.

Recommended pattern:

- Keep one or a small number of **PromptSubmit routing hooks** that always run (or run for most sessions).
- Those routing hooks read:
  - Knowledge routing sources (`ROUTING.md` / route files; see `content_routing.md`)
  - Ability routing sources (`ABILITY.md` + registries/config with `routing_hints`)
- The hooks then emit `routing_hint` and `normalized_intent` signals.

**Preflight vs guardrails**:

- Preflight checks that block task creation live in `hooks.pre_create` (`PreAbilityCreate`).
- Execution guardrails that block actual tool calls live in `hooks.pre_call` (`PreAbilityCall`).
- Post-processing telemetry lives in `hooks.post_call` (`PostAbilityCall`, infra-only).

Example `routing_hints` (conceptual):

```yaml
routing_hints:
  keywords: ["homework grading", "grade submissions", "teacher grading"]
  scopes: ["modules/grading.*", "integration/homework"]
  notes: "Suggest this ability for teacher workflows; students should not see it."
```

#### 3.3.2 Configuration Maintenance

Configuration is critical: incorrect configuration can cause abilities to behave unexpectedly, even if their registry definitions are correct. To keep configuration reliable:

- **Separate configuration from semantics**
  - Registries define what an ability does and its input/output contracts.
  - Configuration defines how and where it runs in a specific environment.
- **Validate configuration**
  - Provide CI checks or `ability lint` commands to ensure:
    - Referenced implementations exist.
    - Required env vars are present (as references, not secrets).
    - Endpoints and timeouts conform to policy.

By keeping configuration healthy and validated, we ensure that:

- The AI can trust availability signals from `PreAbilityCreate`.
- Execution results are predictable and consistent.
- Safety policies are consistently applied by `PreAbilityCall`.

---

This document defines the conceptual model and operational workflow for ability routing within the template repository. It is designed from the perspective of a code-oriented AI model and its internal orchestration loop, with the single metric of success being **higher AI development efficiency under controlled and observable execution**.
