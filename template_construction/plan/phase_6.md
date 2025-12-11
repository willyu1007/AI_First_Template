# Phase 6 – Sample Registrations and Scaffolding Dry-Runs

> Objective: use real but small **sample knowledge, abilities, and hooks** to validate scaffolding flows end-to-end and gather feedback for improvements.

---

## 1. Phase Positioning and Scope

- **Depends on:**  
  - Phase 3 knowledge structure.  
  - Phase 4 registry write scripts and coordination checks.  
  - Phase 5 scaffolding docs and orchestrator scripts.

- **Solves:**  
  - “Do the scaffolds actually work when we register real items?”  
  - “Are the registry write paths and workdocs conventions usable in practice?”

- **Non-goals:**  
  - Full coverage of all possible abilities and hooks.  
  - Deep DevOps or CI integration (those come later).

---

## 2. Overall Strategy – Three Object Types, Three-Stage Flow

### 2.1 Three Object Types

1. **Knowledge entries**  
   - Example carriers: `knowledge/architecture/*.md`, `knowledge/modules/template/*.md`, `knowledge/devops/*.md`, `knowledge/scaffolding/*.md`.  
   - Human-prepared or curated; AI helps hook them into AGENTS, routing, and indexes.

2. **Abilities**  
   - Low-level abilities: implemented by scripts or services.  
   - High-level abilities: workflows or agents built on top of low-level abilities and strategies.

3. **Hooks (registrations)**  
   - Entries in `.system/hooks/*.yaml` that define bindings for hook events.  
   - Handler implementations may remain TODO; focus is on registration and relationships.

### 2.2 Three-Stage Flow

For abilities and hooks, apply this standard flow:

1. **AI plan draft**  
   - AI proposes the registration or change, documented in scenario-local workdocs.

2. **Human confirmation and revision**  
   - Human reviews scope, side effects, and safety.  
   - AI updates the plan accordingly.

3. **Scaffolding execution**  
   - AI prepares formal parameters.  
   - AI calls the relevant scaffolding orchestrator.  
   - Registries and skeleton files are updated via Phase 4 utilities.  
   - AI records results in workdocs.

This flow must be used at least once for each major scaffolding scenario.

---

## 3. Block A – Sample Knowledge Registration

### 3.1 Tasks

1. Select a small set of knowledge docs (e.g., a subset of `knowledge/architecture` and `knowledge/devops`) that are realistic and stable.  
2. Ensure they are placed in the right subdirectories and linked from:
   - Root `AGENTS.md`.  
   - `modules/AGENTS.md`.  
   - DevOps AGENTS (`PROJECT_INIT`, `db`, `ops/packaging`, `ops/deploy`) as appropriate.  
3. Use workdocs (scenario-local) to record:
   - What knowledge was added or refined.  
   - Why it is placed where it is.  
   - Any cross-linking decisions (e.g., between module and DevOps knowledge).

### 3.2 Validation

- Confirm that an AI reading AGENTS and knowledge can navigate from high-level tasks to the correct docs.  
- Check that no knowledge doc violates its directory’s single-responsibility principle.

---

## 4. Block B – Sample Ability Registrations

### 4.1 Sample Scope

Select a small but representative set of abilities, for example:

- A few **infra-focused abilities** (e.g., running tests, checking health, verifying registries).  
- A minimal **high-level workflow** that calls these abilities in sequence.

### 4.2 Flow per Ability

For each selected ability:

1. **Planning (AI)**  
   - In scenario-local workdocs, draft a plan that includes:
     - Ability purpose and scope.  
     - Target location in `.system/registry/low-level` or `high-level`.  
     - Relationship to modules and routes.

2. **Human review**  
   - Human reviews the plan, focusing on scope and safety.  
   - AI updates the plan based on feedback.

3. **Scaffolding execution**  
   - AI prepares parameters and calls the appropriate scaffolding orchestrator (e.g., `new_ability_low.py`, `new_ability_high.py`).  
   - The orchestrator:  
     - Calls Phase 4 registry write utilities.  
     - Creates or updates any related skeleton files.  
   - AI records:  
     - Changed registry entries.  
     - Any new module skeletons or docs.  
     - Issues encountered.

### 4.3 Completion Criteria (Abilities)

- At least a few abilities (low-level and high-level) have been registered through the full three-stage flow.  
- For each such ability, registries and related docs are in sync and workdocs contain a clear audit trail.

---

## 5. Block C – Sample Hook Registrations

### 5.1 Selection

Humans should choose a small number of hooks to exercise different stages, e.g.:

- A `PromptSubmit` hook for routing hints or intent normalization.  
- A `PreAbilityCall` hook that guards critical abilities.  
- A `PostAbilityCall` hook used for logging or metrics.

### 5.2 Flow per Hook

For each selected hook:

1. **Planning (AI)**  
   - In scenario-local workdocs, create a hook plan describing:
     - Hook type and event (`PromptSubmit`, `PreAbilityCall`, etc.).  
     - Target abilities or ability groups.  
     - Expected behavior at a high level.  
     - Required registry changes in `.system/hooks/<hook_type>.yaml`.

2. **Human confirmation**  
   - Human checks whether scope is too broad or too narrow.  
   - AI adjusts as needed.

3. **Scaffolding execution**  
   - AI calls `scripts/scaffold/new_hook.py` with appropriate parameters.  
   - The orchestrator:
     - Uses Phase 4 write utilities for hooks.  
     - Runs consistency checks (e.g., referenced abilities exist).  
   - AI records results in workdocs and notes future TODOs for handler implementations.

### 5.3 Completion Criteria (Hooks)

- At least one hook has been successfully registered and associated with existing abilities.  
- Registries and workdocs information are aligned and traceable.

---

## 6. Block D – Tooling Feedback and Improvement Notes

During all sample runs, the AI should explicitly evaluate:

- Can all target registrations be completed **without manual YAML editing**?  
- Are orchestrator parameters easy to prepare and understand?  
- Are AGENTS guides for scaffolding clear?  
- Are scenario-local workdocs helpful for debugging and auditing?

Record feedback in:

- `knowledge/scaffolding/README.md` or `knowledge/scaffolding/improvement_notes.md`.  
- `template_construction/workdocs/context.md` for construction-level insights.

---

## 7. Workdocs Usage for Phase 6 (Template Construction)

In `template_construction/workdocs/`:

- `task.md` should describe the three object types and three-stage flow.  
- `todolist.md` should list each sample registration (knowledge, abilities, hooks) as separate items.  
- `context.md` should capture:
  - Observed pain points.  
  - Proposed improvements for scaffolding and registry scripts.  
  - Any contradictions between flows and architecture docs.

---

## 8. Human Review Checklist

Humans should verify Phase 6 with:

1. **Coverage**  
   - Has each major scaffolding scenario been exercised at least once with real or realistic data?  
   - Are representative abilities and hook registrations in place?

2. **Traceability**  
   - For each sample, can a human reconstruct the process from scenario-local workdocs?  
   - Are registry changes discoverable and consistent?

3. **Usability feedback**  
   - Are any scaffolds too complex or under-specified for real use?  
   - Are error messages and logs from scripts understandable?

---

## 9. Definition of Done (DoD)

Phase 6 is complete when:

1. Selected knowledge docs are correctly placed in `knowledge/` and linked from AGENTS.  
2. A representative set of low-level and high-level abilities has been registered via scaffolds, with full three-stage flow.  
3. At least one hook registration has been created and validated.  
4. Scenario-local workdocs show complete audit trails for these samples.  
5. Feedback on scaffolding and registry utilities has been written into `knowledge/scaffolding/improvement_notes.md` and/or `template_construction/workdocs/context.md`.  
6. `template_construction/workdocs/outcome.md` includes a “Phase 6 outcome” section summarizing what was registered and key improvement suggestions.
