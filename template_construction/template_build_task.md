# Template Repository Build – Task Description (Prompt for External AI)

> **Goal**: turn this repo into a usable AI‑first template that matches the architecture specs, using an 8‑phase roadmap and a shared `workdocs/` workspace.

---

## 1. Your Role and the Target Scenario

You are an external AI **developer**. In the final, real‑world usage:

- You (or other code models like you) will be the **primary implementer** of application code.
- Humans act mainly as **reviewers, constraint providers, and approvers**.
- A thin **Agent Runtime** is the **only long‑lived runtime**. It:
  - Receives user input.
  - Triggers hooks via a **Hook Runner**.
  - Calls abilities and tools via a **Tool Runner**.
- **Tool Runner** and **Hook Runner** are *supporting scripts/libraries* that are *only* invoked by the Agent Runtime.
- External AIs never talk directly to Tool Runner or Hook Runner; they talk to the **Agent Runtime only**, through documented entrypoints.

Your task in this repository is to **build that world**: implement the template layout, docs, registries, runtime scripts and scaffolding so that future AIs can work efficiently and safely.

---

## 2. Files You Must Treat as Authoritative

The repository already contains several **architecture specification documents**. Treat these as the target design:

- `blueprint/modular_mannual.md` – modular development specification and module skeleton.
- `blueprint/ability_routing.md` – ability pool, `ABILITY.md` usage, execution script contract.
- `blueprint/content_routing.md` – knowledge routing (`ROUTING.md` + `/routes` + knowledge docs).
- `blueprint/agents_strategy_guide.md` – how `AGENTS.md` is structured and used at different levels.
- `blueprint/hooks.md` – hook events, Agent Runtime responsibilities, Hook Runner behavior.
- `blueprint/devops_extension_guide.md` – template support for init, DB, packaging, deployment.
- `blueprint/README.md` – architecture design and key concept overview.

There are also 8 planning documents:

In addition, `template_construction/construction_principles.md` defines the **global principles and rules** for building the template. It governs collaboration between external AIs and humans, phase gating, and `workdocs/` usage, and it **sits above the 8 construction phases and applies to all of them**.

- `template_construction/plan/phase_1.md` – repo skeleton (directories + empty files) and global layout.
- `template_construction/plan/phase_2.md` – documentation conventions and `AGENTS.md` structure.
- `template_construction/plan/phase_3.md` – core mechanism docs and standard flows.
- `template_construction/plan/phase_4.md` – registration scripts and registry collaboration.
- `template_construction/plan/phase_5.md` – scaffolding and interactive flows (AI + human).
- `template_construction/plan/phase_6.md` – initial registration of knowledge/abilities/hooks and validation.
- `template_construction/plan/phase_7.md` – Agent Runtime, Tool Runner, Hook Runner.
- `template_construction/plan/phase_8.md` – ability/knowledge completion, documentation refinement, quality system.

**Interpretation rules:**

1. Treat the architecture specs as **desired end state**.  
   If current files differ from the specs, your job is to move the repo **toward** the specs.
2. Treat `raw_plan_*.md` as **phase‑by‑phase intent**.  
   They may contain outdated or overly detailed suggestions; you are allowed to adjust details, but you must preserve **their goals and boundaries**.
3. When there is a conflict:
   - Confirm with user.
   - Make conflicts explicit in `template_construction/workdocs/context.md` and propose fixes.

---

## 3. Non‑Negotiable Rules For Actual Development Environment

When building this template, please remember that the following constraints will apply in actual use.

### 3.1 README vs AGENTS

- `AGENTS.md` is **for AI**.
  - It is the *only* place where AI‑facing behavioral rules are defined.
  - Every directory that needs AI guidance should use its own `AGENTS.md`.
- `README.md` is **for humans only**.
  - You must **not** read `README.md` as a source of rules or instructions.
  - If you see `README.md` mention AI behavior, treat that as **outdated**; propose aligning it with the AGENTS strategy instead.
- When you need norms or behavior rules, always prefer:
  1. Local `AGENTS.md` in the working directory.
  2. `agents_strategy_guide.md` for the general strategy network.

### 3.2 Runtime and tools

- There is **exactly one** long‑lived runtime: **Agent Runtime**.
- Agent Runtime:
  - Receives user inputs and session events.
  - Emits events (`PromptSubmit`, `PreAbilityCall`, `PostAbilityCall`, `SessionStop`) as described in `hooks.md`.
  - Delegates to **Hook Runner** for hook execution.
  - Delegates to **Tool Runner** for ability/operation execution according to `ability_routing.md`.
- **Tool Runner**:
  - Implements the *standard execution script* described in `ability_routing.md`.
  - Exposes operations like `task.create`, `task.run`, `task.get`, `task.update`, `task.delete`, `direct_run`.
  - Is never called directly by external AIs; only by Agent Runtime.
- **Hook Runner**:
  - Executes hook YAML + handler scripts for each event, exactly as described in `hooks.md`.
  - Is never called directly by external AIs; only by Agent Runtime.

Any code or docs you add must respect this single‑runtime model.

### 3.3 Humans vs AI responsibilities

- **AI (you)**: write docs, code, scripts, and registry entries **inside the repo**.
- **Humans**: review changes, run commands that require credentials or local access, decide on production deployments.
- When you propose actions that require humans, record them explicitly (see `workdocs/todolist.md`).

---

## 4. Shared `workdocs/` for Building the Template

The **template build itself** is a long‑running scenario. It uses a dedicated root‑level workspace:

```text
workdocs/
  plan.md
  task.md
  context.md
  todolist.md
  preparation/
  outcome/
```

This `workdocs/` is **only** for the “build the repo template” meta‑task. All other scenarios still use their own local workspaces (e.g. `PROJECT_INIT/workdocs/`, `db/workdocs/`, `ops/packaging/workdocs/`, `modules/<module_id>/workdocs/`), as described in the specs.

### 4.1 `workdocs/plan.md`

- Holds the **overall 8‑phase roadmap** and its current interpretation.
- For each phase, record:
  - Phase ID (`1`–`8`).
  - One‑sentence goal (from `plan/phase_*.md`).
  - Scope and main outputs.
  - Dependencies on previous phases.
- When you refine how a phase should be executed (without changing its intent), update `plan.md` and briefly explain why.

### 4.2 `workdocs/task.md`

Use `task.md` as the **active task ledger** during template construction.

Suggested structure:

- A table or list with fields like:
  - `phase_id`
  - `task_id` (short slug)
  - `description`
  - `status` (`PLANNED | IN_PROGRESS | BLOCKED | DONE | CANCELLED`)
  - `related_files`
- Update this file whenever you:
  - Start or finish a task.
  - Discover a new task required to make a phase viable.

### 4.3 `workdocs/context.md`

`context.md` is the **fast resume entry** for the entire template build.

Each time you complete a work session or a phase, update:

- Current phase and sub‑phase.
- What has been completed.
- What is blocked and why.
- Which parts of the architecture specs are still “draft” vs “trustworthy”.
- Pointers to important decisions (for example, to `knowledge/` docs or module‑level workdocs once they exist).

Think of `context.md` as “if I lost all in‑memory context, what is the minimum I must read to continue the build safely?”.

### 4.4 `workdocs/todolist.md`

Use `todolist.md` for **backlog items** that are **not yet attached** to a specific phase (or that span multiple phases).

Examples:

- “Reconcile README with AGENTS.md once docs are stable.”
- “Add CI check for ability registry schema.”
- “Introduce example modules for smoke‑testing the template.”

When a todo becomes concrete work inside a phase, link or move it into `task.md`.

### 4.5 `workdocs/preparation/`

Use `preparation/` for **scratch material**:

- Temporary notes on design options.
- Early drafts of prompts or docs before they stabilize.
- Experiments that may or may not be kept.

Content here is **not** treated as canonical knowledge. Before you finish a phase, either:

- Promote useful material into the appropriate permanent location (e.g. `knowledge/`, `modules/…/docs/`, `.system/` registries), or
- Delete it if it is no longer useful.

### 4.6 `workdocs/outcome/`

Use `outcome/` to record **stable results** per phase, for example:

- `phase_1_skeleton_summary.md`
- `phase_2_docs_and_agents_summary.md`
- …
- `phase_8_quality_and_completion_report.md`

Each outcome file should:

- Summarize what was actually done vs the original `raw_plan_*.md`.
- Note any intentional deviations and their justification.
- Link to key files or directories created in that phase.
- Indicate whether the phase passed the phase review (see the “Phase Review Workflow” prompt) and under what conditions.

---

## 5. Eight‑Phase Working Loop

Use the following loop to gradually build the template. The high‑level intent of each phase is defined by `template_construction/plan/phase_*.md`; the steps below define **how you work**.

### Phase 0 – Orientation

Before starting Phase 1:

1. Read all architecture specs:
   - `blueprint/modular_mannual.md`
   - `blueprint/ability_routing.md`
   - `blueprint/content_routing.md`
   - `blueprint/agents_strategy_guide.md`
   - `blueprint/hooks.md`
   - `blueprint/devops_extension_guide.md`
   - `blueprint/README.md`
   - `template_construction/construction_principles.md` - global principles and rules for template construction.
2. Skim `phase_1.md`–`phase_8.md` to understand the overall flow.
3. Initialize `workdocs/plan.md`, `task.md`, `context.md`, and `todolist.md`.

### General pattern for each phase N (1–8)

For each phase `N`:

1. **Refresh context**
   - Re‑read `workdocs/context.md`.
   - Re‑read the section for phase `N` in `workdocs/plan.md`.
   - Carefully read `phase_N.md` and extract:
     - Phase goal in one sentence.
     - In‑scope and out‑of‑scope items.
     - Expected outputs.
2. **Align with architecture docs**
   - Identify which architecture specs are most relevant to this phase.
   - Note any constraints they impose (for example, module skeleton, registry schema, hook event model).
3. **Plan concrete tasks**
   - Break the phase into small tasks and write them into `workdocs/task.md`.
   - Mark which tasks require human actions (e.g. running scripts, validating decisions).
4. **Execute tasks**
   - Create or update directories, empty files, docs, registries, or scripts as required.
   - Keep content consistent with the specs and the cross‑doc alignment rules.
   - **Do not** invent new conceptual layers or roles unless they are clearly aligned with existing terms (module, ability, knowledge routing, runtime, hook, DevOps scenario).
5. **Run the Phase Review Workflow**
   - Use the “Phase Review Workflow (pseudo‑agent)” prompt (see the companion document `phase_review_workflow.md`) to review the phase’s results.
   - Fix blocking issues identified by the review before marking the phase as complete.
6. **Update workspace**
   - Update `workdocs/context.md` with what was achieved, what remains, and how the repo changed.
   - Write a phase summary into `workdocs/outcome/phase_<N>_*.md`.
   - Update `workdocs/task.md` and `workdocs/todolist.md` to reflect the new state.

Move to the next phase only after the current phase passes review and its key artifacts are stable enough for other phases to depend on.

---

## 6. Completion Criteria

You may consider the **template build task complete** when:

1. **Structure**
   - The repo’s directory layout matches the modular skeleton and DevOps structure described in:
     - `modular_mannual.md`
     - `devops_extension_guide.md`
   - Scenario‑specific workspaces exist and are documented (`PROJECT_INIT/workdocs/`, `db/workdocs/`, `ops/packaging/workdocs/`, `ops/deploy/workdocs/`, `modules/<module_id>/workdocs/`, plus this root `workdocs/`).

2. **Docs and routing**
   - `AGENTS.md` files exist at the required levels and follow `agents_strategy_guide.md`.
   - Knowledge routing (`ROUTING.md` + `/routes`) is configured for the modular system and any necessary global scopes, following `content_routing.md`.
   - Core mechanism docs (modular, ability routing, knowledge routing, hooks, DevOps) are internally consistent.

3. **Abilities, hooks, runtime**
   - A basic ability pool and registries exist, with at least a small set of low‑level and high‑level abilities.
   - Hooks are declared under `.system/hooks/` and wired into the event model described in `hooks.md`.
   - Tool Runner implements the standard execution script contract.
   - Agent Runtime exists as the **only long‑lived runtime**, delegating to Tool Runner and Hook Runner correctly.

4. **Scaffolding and examples**
   - Scaffolding flows (for modules, integration scenarios, knowledge docs, abilities, hooks) are in place and documented.
   - At least one or two example modules and integration scenarios exist so that future AIs can “play through” the entire flow.

5. **Workdocs**
   - `workdocs/plan.md`, `task.md`, `context.md`, `todolist.md`, and `outcome/` accurately describe **what was really done**.
   - Future AIs can read these files and resume work (for extension or refactoring) without needing the original human author.

---

## 7. How to Behave While Reading This Prompt

When you are started with this prompt:

1. **Do not** modify this document itself. Treat it as a specification.
2. Begin by checking whether `workdocs/` already exists:
   - If not, create it and initialize `plan.md`, `task.md`, `context.md`, `todolist.md`, `preparation/`, and `outcome/`.
3. Follow the **Phase 0 + phase‑by‑phase loop** above.
4. Prefer small, incremental changes with clear notes in `workdocs/context.md` over large, monolithic edits.
5. When you are unsure how to interpret a conflict:
   - Re‑consult the architecture specs.
   - Document the uncertainty and your chosen resolution in `workdocs/context.md`.
   - Prefer the safer / more conservative interpretation.

If you strictly follow this prompt, the result should be a **coherent, AI‑first repository template** whose structure, docs, and tooling are all discoverable and usable by future AIs and humans.
