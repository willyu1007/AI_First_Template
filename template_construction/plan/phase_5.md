# Phase 5 – Scaffolding and Interactive Flows

> Objective: build **scaffolding knowledge and orchestrator scripts** so that AI + human interactions can drive end‑to‑end flows from requirements to concrete registry/config changes and file skeletons.

---

## 1. Phase Positioning and Scope

- **Depends on:**  
  - Phase 1–4 (layout, docs, knowledge, registry write scripts and coordination).

- **Solves:**  
  - “Given a requirement, how does an AI lead a human through a safe, repeatable flow to create modules, abilities, routes, hooks, and DevOps configs?”

- **Prepares for:**  
  - Phase 6 sample registrations (running scaffolds on real examples).  
  - Phase 7 runtime integration (Tool Runner and Hook Runner rely on registries populated by these scaffolds).

---

## 2. Scaffolding Scenario Matrix (Priority Scope)

Scaffolding is organized into scenarios, grouped into:

1. **Project-level scaffolding** (under `PROJECT_INIT/`).  
2. **Development-time scaffolding** (modules, abilities, routes, hooks).  
3. **DevOps scaffolding** (packaging and deployment skeletons).

### 2.1 Project Initialization Scaffolding

Scope: initialize a concrete project from the template.

- Runtime assets:  
  - `PROJECT_INIT/AI/AGENTS.md`  
  - `PROJECT_INIT/scripts/init_project.py`  
  - `PROJECT_INIT/workdocs/`

- Knowledge doc:  
  - `PROJECT_INIT/AI/project_init.md`

Constraint: all project initialization scripts and workdocs live under `PROJECT_INIT/`.

### 2.2 Development-Time Scaffolding

Scenarios:

1. New module instance.  
2. New low-level ability.  
3. New high-level workflow/agent.  
4. New content route.  
5. New hook registration.

### 2.3 DevOps Scaffolding

Scenarios:

1. New HTTP service packaging skeleton.  
2. New Job packaging skeleton.  
3. New HTTP service deployment skeleton.  
4. New Job deployment skeleton.

Each scenario must have:

- A **knowledge doc** under `knowledge/scaffolding/`.  
- An **orchestrator script** under `PROJECT_INIT/scripts/` or `scripts/scaffold/`.  
- Clear instructions in the relevant `AGENTS.md` files.

---

## 3. Block A – Scaffolding Knowledge Docs

### 3.1 Directory Layout

Create or populate:

```text
PROJECT_INIT/
  AI/
    init_project.md

knowledge/
  scaffolding/
    new_module.md
    new_ability_low.md
    new_ability_high.md
    new_route.md
    new_hook.md
    devops_packaging_service.md
    devops_packaging_job.md
    devops_deploy_service.md
    devops_deploy_job.md
```

`project_init.md` documents the process but runtime assets remain in `PROJECT_INIT/`.

### 3.2 Recommended Structure for Each Doc

Each scaffolding knowledge doc should follow this skeleton:

1. **Purpose & Scope**  
   - What this scaffold does and explicitly does not do.

2. **Inputs & Preconditions**  
   - Information the AI must collect from humans (e.g., module name, purpose, environment constraints).  
   - Repository state prerequisites (e.g., Phase 4 registry scripts available).

3. **Step-by-step Flow (AI + Human)**  
   - Steps for AI to analyze the requirement and propose a plan.  
   - Points where human confirmation or correction is required.  
   - When and how to call orchestrator and registry scripts.

4. **Tools & Scripts to Use**  
   - Orchestrator script path for this scenario.  
   - Underlying registry write scripts used from Phase 4.

5. **Outputs & Side Effects**  
   - Which registries and files are created or modified.  
   - Which workdocs must be updated.

6. **Safety & Rollback Notes**  
   - How to recover if the scaffold fails partway.  
   - How to revert partially applied changes.

All scaffolding docs are AI-facing and should be referenced from relevant AGENTS docs.

---

## 4. Block B – Orchestrator Scripts

### 4.1 Layout and Constraints

Scripts must be created under:

```text
PROJECT_INIT/
  scripts/
    init_project.py

scripts/
  scaffold/
    new_module.py
    new_ability_low.py
    new_ability_high.py
    new_route.py
    new_hook.py
    devops_packaging_service.py
    devops_packaging_job.py
    devops_deploy_service.py
    devops_deploy_job.py
```

Rules:

- Orchestrators **do not manage conversations directly**; they accept structured parameters and return structured results.  
- Conversational flows are defined in AGENTS and knowledge docs; orchestrators are deterministic execution engines.

### 4.2 Common Behavior Pattern

Using `new_module.py` as an example, orchestrators should:

1. Accept parameters (via CLI, JSON file, or environment variables), e.g.:  
   - `module_id`  
   - `module_type`  
   - `description`  
   - other metadata as needed.

2. Execution steps:  
   - Validate parameters against schemas (use Phase 4 `validate_entry`).  
   - Call registry write functions to update:  
     - `modules/overview/instance_registry.yaml`  
     - `instance_maturity.yaml` or others as needed.  
   - Create the module directory skeleton under `modules/<module_id>/`:
     - `AGENTS.md` (template or empty)  
     - `MANIFEST.yaml` (minimal fields per schema)  
     - `ROUTING.md`, `ABILITY.md` (empty skeletons)  
     - `workdocs/` directory with standard plan/context/tasks files.

3. Return a structured result (e.g., JSON) with:  
   - Success/failure status.  
   - List of changed files.  
   - Summary for the AI to record in workdocs.

All other orchestrators (abilities, routes, hooks, DevOps) follow this pattern, tailored to their scenario.

---

## 5. Block C – AGENTS Integration and Workdocs Usage

### 5.1 AGENTS Documentation

Update relevant AGENTS docs (root, modules, PROJECT_INIT, ops/packaging, ops/deploy) to describe for each scaffolding scenario:

- How to **initiate** the scenario from an AI conversation (what the AI should ask the human).  
- How to **prepare parameters** for the orchestrator.  
- How to **interpret results** and write to workdocs.  
- Where to store ongoing per-task workdocs (e.g., under scenario-local `workdocs/active/T-...` in project/init or modules).

### 5.2 Workdocs Conventions for Scaffolding Tasks

For each scaffolding execution:

- Use scenario-local `workdocs/` (e.g., `PROJECT_INIT/workdocs/`, `modules/<module_id>/workdocs/`, `ops/packaging/workdocs/`).  
- At a minimum, each task record should include:
  - Task metadata (ID, timestamp, scenario type, initiator).  
  - Initial requirement description (human input + AI summary).  
  - Key decisions and any human overrides.  
  - Orchestrator scripts invoked and parameter summaries (non-sensitive).  
  - Result summaries (files and registries touched).  
  - TODOs and risks if the task is not fully finished.

Keep `workdocs` writing under AI control; orchestrator scripts should not silently modify workdocs unless explicitly designed to do so.

---

## 6. Workdocs Usage for Phase 5 (Template Construction)

In `template_construction/workdocs/`:

- `task.md` should describe the scaffolding scenario matrix and how orchestrators and workdocs interact.  
- `todolist.md` should include:
  - Each knowledge doc under `knowledge/scaffolding/`.  
  - Each orchestrator script.  
  - AGENTS updates for each area.  
- `context.md` should capture feedback on scaffolding design, especially friction points you anticipate for Phase 6.

---

## 7. Human Review Checklist

Humans should verify Phase 5 with the following:

1. **Knowledge docs**  
   - Does each scaffolding scenario have a knowledge doc under `knowledge/scaffolding/`?  
   - Are flows clearly documented with AI + human roles?

2. **Orchestrator scripts**  
   - Do scripts rely on registry write utilities from Phase 4 rather than re-implementing logic?  
   - Are parameters reasonable and easy for AI to prepare?  
   - Is output structured and sufficient for auditing?

3. **AGENTS and workdocs**  
   - Do AGENTS docs explain how to initiate scaffolds and where to record workdocs?  
   - Are workdocs conventions clear enough to ensure traceability for each scaffold run?

Any missing or unclear pieces should be logged and addressed by the AI, then summarized in `outcome.md`.

---

## 8. Definition of Done (DoD)

Phase 5 is complete when:

1. `knowledge/scaffolding/` contains knowledge docs for all prioritized scaffolding scenarios.  
2. Orchestrator scripts exist for:
   - Project initialization (`PROJECT_INIT/scripts/init_project.py`).  
   - New module, low-level ability, high-level ability/workflow, content route, hook registration.  
   - DevOps packaging and deployment skeletons (service and job).  

3. AGENTS docs describe **how to use scaffolding** from AI conversations and where to store per-task workdocs.  
4. Workdocs usage for scaffolding is standardized across scenarios.  
5. At least one dry-run per scenario has been performed (even with synthetic data) to ensure scripts and docs compose correctly.  
6. `template_construction/workdocs/outcome.md` includes a “Phase 5 outcome” section summarizing the scaffolding matrix, scripts, and any known limitations.
