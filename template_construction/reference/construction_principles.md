# Repository Template Construction – Global Principles and Rules

> This document governs how external AI systems and humans collaborate to build the repository template itself. It sits **above** the 8 construction phases and applies to all of them.

---

## 1. Scope and Overall Goals

The construction plan assumes:

- The **repository template architecture** (modules, routing, registries, hooks, DevOps areas) is already defined by the architecture documents.
- **External AIs are the primary implementers**; humans act mainly as reviewers, decision-makers on ambiguous points, and approvers for sensitive operations.
- The result must be a **reusable, AI-friendly template** that supports:
  - Modular feature development
  - Ability routing and hook-based guardrails
  - Scenario-local workspaces
  - DevOps extensions (init, DB, packaging, deploy)

This document does **not** describe business features. It only defines how to construct the template so that future projects can safely build business logic on top.

---

## 2. Roles and Collaboration Model

### 2.1 Roles

- **AI developer (external AI)**  
  - Reads architecture and phase guides.  
  - Writes code, configuration, and documentation drafts.  
  - Operates script-based tools defined in the repo (registry writers, scaffolds, etc.).  
  - Maintains workdocs for traceability.

- **Human maintainer**  
  - Provides domain and architectural intent.  
  - Reviews plans and drafts based on checklists.  
  - Approves or rejects risky changes (registries, DevOps configs, runtime behavior).  
  - Runs scripts that touch external systems when necessary.

### 2.2 Collaboration Rules

1. **AI writes, human reviews, AI revises** is the default flow for any non-trivial artifact (docs, registries, runtime code, scaffolding).  
2. Humans should avoid editing YAML or code directly unless absolutely necessary; instead, they provide feedback and constraints that the AI applies.  
3. All major decisions and deviations from the guides must be recorded in the relevant workdocs (see section 4).

---

## 3. Phase Structure and Gating

Construction is organized into 8 phases:

1. **Phase 1 – Skeleton and layout**: create directories, placeholder files, and minimal AGENTS/README skeletons.  
2. **Phase 2 – Documentation conventions and AGENTS**: define DOCS_CONVENTION, basic AGENTS layout, and initial knowledge tree.  
3. **Phase 3 – Core mechanisms and standard flows**: extract architecture docs into AI-facing knowledge, standard workflows, glossary, and pitfalls.  
4. **Phase 4 – Registry write scripts and coordination**: implement schema files, registry writing utilities, and consistency checks.  
5. **Phase 5 – Scaffolding and interactive flows**: implement scaffolding knowledge and orchestrator scripts that compose Phase 4 utilities.  
6. **Phase 6 – Sample registrations and dry-runs**: register representative knowledge, abilities, and hooks through scaffolds; validate the loop.  
7. **Phase 7 – Runtime layer**: implement Tool Runner, Hook Runner, and Agent Runtime, wired to registries and hooks.  
8. **Phase 8 – Capabilities, documentation, and quality system**: enrich infra abilities, human-facing docs, tests, CI, and examples.

### 3.1 Gating Policy

- A later phase **must not** assume artifacts that earlier phases have not created.  
- If a phase precondition is not met, the AI must:
  1. Record the missing prerequisite in `template_construction/workdocs/context.md` and `todolist.md`.  
  2. Either complete the prerequisite (if clearly within scope) or escalate to a human maintainer.

---

## 4. Template-Construction Workdocs (template_construction/workdocs/)

The **template-construction scenario** has its own local workspace:

```text
template_construction/
  AGENTS.md          # Strategy for building the template (AI-only)
  workdocs/
    preparation.md   # Architecture recall and assumptions per phase
    plan.md          # High-level construction plan across phases
    task.md          # Narrative description of current phase tasks
    todolist.md      # Checklist-style tasks and DoD items
    context.md       # Rolling context, decisions, open questions
    outcome.md       # Phase-level outcomes and retrospective
```

These workdocs are for **building the template itself**, and are separate from any future scenario workdocs under `modules/`, `PROJECT_INIT/`, `db/`, `ops/packaging/`, `ops/deploy/`, etc.

### 4.1 Document Responsibilities

- `preparation.md`  
  - Used at **the start of each phase**.  
  - AI records:
    - Which architecture docs were re-read (architectural README, modular spec, routing, hooks, DevOps guides, etc.).  
    - Any updated understanding of global invariants.  
    - Phase-specific assumptions, constraints, and uncertainties.

- `plan.md`  
  - Holds the **multi-phase plan** and a short section per phase:
    - Objectives and non-goals.  
    - Expected artifacts.  
    - Dependencies on other phases.  
  - Updated when the global plan changes or a phase is re-scoped.

- `task.md`  
  - Focuses on the **current phase** only.  
  - Describes:
    - The current phase goal in 1–3 paragraphs.  
    - The main “work blocks” (A/B/C) and their rationale.  
    - How human review is expected to interact with those blocks.

- `todolist.md`  
  - A checklist for the current phase, with objectively verifiable items and explicit DoD.  
  - Must make it easy for a human to see what remains to be done.

- `context.md`  
  - Captures evolving context:
    - Progress summaries.  
    - Major decisions and rationale (with references to other docs if needed).  
    - Blockers and follow-up items.  
  - It should allow a fresh AI instance to **resume the construction task quickly**.

- `outcome.md`  
  - Written or finalized at **phase completion**.  
  - Summarizes:
    - What was actually delivered.  
    - Deviations from the phase guide and reasons.  
    - Open issues that move to the next phase or backlog.  

### 4.2 Workdocs Usage Rules

1. Workdocs are **scenario-local** to `template_construction/`. They must not be misused as a global workspace for unrelated tasks.  
2. Stable knowledge discovered during construction must be promoted into `knowledge/` or other permanent docs once it crystallizes; workdocs are not a permanent knowledge store.  
3. Workdocs are maintained primarily by the AI; humans read and comment via review comments or separate notes, which the AI then reflects into workdocs.

---

## 5. Architecture Alignment Invariants

All phases must respect the core architectural invariants defined by the architecture documents and the modular/manual specification, including:

1. **Modules as the primary place for business logic**  
   - `modules/` contains module instances, their docs, and module-local workdocs.  
   - `modules/overview/` contains project-wide registries (type graph, instance registry, interfaces, maturity).  

2. **Registries and schemas**  
   - Schema YAMLs live under `.system/registry/schemas/*.schema.yaml`.  
   - Actual registry content lives in:
     - `modules/overview/*.yaml` for module-related items.  
     - `modules/routes/*.yaml` for content routing.  
     - `.system/registry/low-level/*.yaml` and `.system/registry/high-level/*.yaml` for abilities.  
     - `.system/hooks/*.yaml` for hook registrations.  
   - All changes should go through structured scripts (Phase 4 and 5), not ad-hoc editing.

3. **Hooks semantics**  
   - Four main events: `PromptSubmit`, `PreAbilityCall`, `PostAbilityCall`, `SessionStop`.  
   - `PromptSubmit` and `PreAbilityCall` are **AI-facing, blocking**; they may emit routing hints, normalized intent, and guards.  
   - `PostAbilityCall` and `SessionStop` are **infra-only, non-blocking**; they are used for logging, tests, and reporting.

4. **DevOps extensions**  
   - `PROJECT_INIT/`, `/db/`, `/ops/packaging/`, `/ops/deploy/` each have their own `workdocs/` and `AGENTS.md`.  
   - These directories act as **scenario-local workspaces** for init, DB mirroring, packaging, and deployment, with scripts as the only bridge to external systems.

5. **Documentation roles**  
   - Human-facing `README.md` files explain areas to humans, not AI behavior policies.  
   - AI-facing behavior guidance and routes are in `AGENTS.md` and `knowledge/` docs.  
   - Workdocs record task-local planning and outcomes and must not be the only place where long-term rules are stored.

---

## 6. Behavior Rules for AI During Construction

### 6.1 At the Start of Each Phase

Before starting any phase, the AI must:

1. Re-open and skim the key architecture docs (paths are indicative, adjust to actual repo layout):
   - `ARCHITECTURE_OVERVIEW.md` or equivalent.  
   - `modular_mannual.md`.  
   - `ability_routing.md`.  
   - `content_routing.md`.  
   - `agents_strategy_guide.md`.  
   - `hooks.md`.  
   - `devops_extension_guide.md`.  
2. Write a short **“architecture recall” section** in `template_construction/workdocs/preparation.md`:
   - bullets on the global invariants relevant to this phase.  
   - Links (paths) to any particularly relevant knowledge docs.  
3. Update `template_construction/workdocs/plan.md` and `task.md` to reflect how this phase’s goals map onto the architecture.

### 6.2 While Executing a Phase

During a phase, the AI must:

- Use `todolist.md` as the primary checklist; any new sub-task gets added there.  
- Update `context.md` whenever:
  - A work block (A/B/C) reaches a natural checkpoint.  
  - A major decision is made (e.g., schema field naming, hook behavior).  
  - A blocker is found that may require human input.

For registry and DevOps changes:

- Prefer **scripts and scaffolds** over manual YAML edits.  
- If manual edits are temporarily necessary (e.g., while Phase 4 tooling is incomplete):
  - Explicitly record the change and rationale in `context.md` and `outcome.md`.  
  - Mark the manual edit as a TODO to be migrated to script/registry utilities later.

### 6.3 At Phase Completion

Upon declaring a phase done, the AI must:

1. Re-open the key architecture docs to confirm that all new artifacts are consistent with the architecture.  
2. Update `outcome.md` with:
   - A bullet list of completed checklist items.  
   - References to created/updated files.  
   - Any discrepancies between the original plan and actual results.  
3. Confirm that **all DoD items in the phase guide** are met:
   - If any DoD item is not satisfied, either:
     - Complete it before marking the phase done, or  
     - Explicitly mark it as deferred in `todolist.md` and `outcome.md` with a pointer to the next phase or backlog.

---

## 7. Safety and Change-Management Constraints

1. **Registry and hook safety**  
   - Never bypass schema validation when editing registries, except with explicit human approval recorded in `outcome.md`.  
   - For capabilities and hooks, use the three-step process wherever practical:
     1. AI proposal (plan in workdocs).  
     2. Human review and corrections.  
     3. Scaffolding or registry scripts apply the changes.  

2. **DevOps configuration**  
   - Only use the DevOps structures under `/db/`, `/ops/packaging/`, and `/ops/deploy/`.  
   - Do not introduce ad-hoc configuration trees without a clear mapping to the DevOps guides.  
   - Do not hardcode environment-specific secrets in the repo; these must be handled externally.

3. **Runtime and hooks**  
   - All runtime implementations (Tool Runner, Hook Runner, Agent Runtime) must:
     - Respect the four hook events and their blocking/non-blocking semantics.  
     - Use the ability registries as the single source of truth for ability metadata.  
     - Avoid embedding business logic; they orchestrate abilities, hooks, and workdocs updates only.

4. **Documentation hygiene**  
   - If a piece of documentation is primarily for AI execution, ensure it lives under `AGENTS.md` or `knowledge/…`, not in README.  
   - If a long-term rule is discovered while working inside workdocs, promote it to knowledge and link to it from the appropriate AGENTS file.

---

## 8. How to Use the Phase Guides (for Code Models)

When operating as an external code model (e.g., Cursor, Claude, etc.), follow this routine for each phase:

1. **Read this principles document once** at the start of the overall construction and whenever major doubts occur.  
2. For the current phase N:
   - Open `phase_N.md` (the phase guide).  
   - Summarize its objectives and DoD into `template_construction/workdocs/preparation.md` and `plan.md`.  
   - Translate its step-by-step sections into concrete checklist items in `todolist.md`.  
3. Execute the phase, keeping `context.md` up to date.  
4. At the end of the phase, use the “Human review checklist & DoD” section in the phase guide to structure your self-check, and write a short retrospective into `outcome.md`.

If any instruction in a phase guide appears to contradict this principles document or the architecture docs:

- Treat the **architecture docs + this principles document** as the higher authority.  
- Record the conflict in `context.md`.  
- Propose a resolution for human review before proceeding.

This ensures the repository template remains internally consistent and aligned with its intended architecture over all eight construction phases.
