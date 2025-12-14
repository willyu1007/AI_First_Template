# Phase 3 – Core Mechanism Knowledge and Standard Workflows

> Objective: turn architecture documents into **AI-facing knowledge** and define **standard workflows** for modules, abilities, content routing, and hooks, plus a shared glossary and pitfalls.

---

## 1. Phase Positioning and Scope

- **Depends on:**  
  - Phase 1 skeleton and registries/paths.  
  - Phase 2 DOCS_CONVENTION, AGENTS skeletons, and initial knowledge templates.

- **Solves:**  
  - “When the AI makes key design decisions, where is the authoritative knowledge?”

- **Prepares for:**  
  - Phase 4 registry write scripts (schemas must be aligned with these concepts).  
  - Phase 5 scaffolding (flows will reference these standard workflows).

---

## 2. Preconditions

Before starting Phase 3, the AI must:

1. Confirm that Phase 1 and 2 DoD are satisfied.  
2. Re-read the architecture docs: modular spec, routing docs, agents guide, hooks, DevOps extension.  
3. In `template_construction/workdocs/preparation.md`, write a **Phase 3 section** capturing:
   - Key architectural concepts for modules, abilities, content routing, and hooks.  
   - Any mismatches or ambiguities discovered across architecture docs.
4. `reference/workdocs_mannual.md` is a user manual for working documents， refer to it when designing related content.

---

## 3. Block A – Architecture Knowledge Mapping

### 3.1 Goal

Refactor the architecture documents into **small, position-aware knowledge files** under:

- `knowledge/architecture/` – global architecture concepts and relationships.  
- `knowledge/modules/template/` – module template mechanisms and best practices.  
- `knowledge/devops/` – DevOps-related flows.

### 3.2 Tasks

1. Review the source architecture docs (human-oriented).  
2. For each important concept (e.g., module types, type graph, interface lifecycle, ability types, routing concepts, hook events, DevOps scenarios), create or refine a dedicated section in `knowledge/architecture/*.md`.  
3. Ensure that:
   - Each knowledge file has a **narrow responsibility**.  
   - The root `AGENTS.md` and `modules/AGENTS.md` point to these files explicitly as references.  
4. Update `knowledge/modules/template/*` where necessary to:
   - Explain how module manifest fields link to registries.  
   - Describe module-level workdocs and outcomes in relation to the global architecture.

Record mapping decisions in `template_construction/workdocs/context.md` (e.g., “interfaces.yaml describes X, Y, Z; the authoritative explanation is now in knowledge/architecture/interfaces.md”).

---

## 4. Block B – Standard Workflows

### 4.1 Goal

Define **standard workflows** for critical areas and document them in AI-readable form. At minimum, create or refine the following docs (paths indicative):

- `knowledge/architecture/module_workflow.md`  
- `knowledge/architecture/ability_workflow.md`  
- `knowledge/architecture/content_routing_workflow.md`  
- `knowledge/architecture/hook_lifecycle.md`

### 4.2 Required Content for Each Workflow Doc

Documents need to be designed so that AI can efficiently understand the content and avoid unnecessary token burden.

For each workflow:

1. **Purpose & Scope**  
   - Which problems this workflow solves.  
   - What is explicitly out of scope.

2. **Inputs & Preconditions**  
   - Required registry and knowledge state.  
   - Required DevOps or environment preconditions if applicable.

3. **Step-by-step Flow**  
   - The canonical sequence of steps the AI should follow (including where to ask humans).  
   - Explicit references to which registries/docs must be read or updated.

4. **Outputs & Side Effects**  
   - Which files/registries are changed.  
   - How workdocs are used to record the process.

5. **Safety Notes**  
   - Common failure modes and how to mitigate them.  
   - Which steps require human approval.

Update AGENTS docs to reference these standard workflows as the “default path” for new work in each area.

---

## 5. Block C – Glossary and Pitfalls

### 5.1 Glossary – `knowledge/architecture/glossary.md`

Define key terms with concise explanations and examples, including but not limited to:

- `module`, `module_type`, `module_instance`.  
- `ability`, `low-level ability`, `high-level workflow`, `agent`.  
- `hook` (concept only).  
- `workdocs`, `knowledge`, `template`, `overview`, `registry`.  
- `routing`, `content routing`, `ability routing`.  
- `scope`, `topic`, `route`, `doc set`.

Each entry should follow this pattern:

- **Name**  
  - One-sentence definition.  
  - One-sentence example referencing actual or future paths in this template.

### 5.2 Pitfalls – `knowledge/architecture/pitfalls.md`

Document “don’t do this” cases with a **wrong vs right** contrast, such as:

- Keeping long-term knowledge only in `workdocs/` instead of promoting it to `knowledge/`.  
- Defining AI behavior rules in README instead of AGENTS/knowledge.  
- Creating ad-hoc module directories without updating `modules/overview/instance_registry.yaml`.  
- Editing another module’s `AGENTS.md` without coordination.

Each pitfall entry should include:

- **Wrong:** short description of the problematic behavior.  
- **Right:** recommended alternative aligned with architecture.  
- Optionally: a reference to standard workflows or registry scripts that should be used instead.

---

## 6. Workdocs Usage for Phase 3

- In `task.md`, describe the three work blocks (A/B/C) and their dependencies.  
- In `todolist.md`, track at least:
  - Knowledge mapping completed for architecture docs.  
  - Standard workflows docs created and linked from AGENTS.  
  - Glossary and pitfalls created and self-reviewed.  
- In `context.md`, log:
  - Any conceptual disagreements found among the original architecture docs and how you resolved them.  
  - Any terminology decisions that might impact future phases (e.g., names of schema fields).

---

## 7. Human Review Checklist

Humans should review Phase 3 with these questions:

1. **Knowledge mapping**  
   - Do `knowledge/architecture/*` and `knowledge/modules/template/*` cover the key mechanisms at the correct level of abstraction?  
   - Are responsibilities clearly separated (no giant catch-all docs)?  

2. **Standard workflows**  
   - For modules, abilities, routing, and hooks, is there a clear, step-by-step standard flow?  
   - Do these flows match the architectural intent and constraints?  
   - Are references to registries and AGENTS correct?

3. **Glossary and pitfalls**  
   - Are key terms defined accurately and concisely?  
   - Do pitfalls capture the most dangerous or costly mistakes?  
   - Are “right” alternatives realistic under the planned architecture?

Any required corrections should be described in human comments and then applied by the AI, with results recorded in `context.md` and `outcome.md`.

---

## 8. Definition of Done (DoD)

Phase 3 is complete when:

1. The architecture docs have been mapped into a set of **stable, AI-facing knowledge files** under `knowledge/architecture/` and `knowledge/modules/template/`.  
2. There is at least one standard workflow doc for **modules**, **abilities**, **content routing**, and **hooks**, and relevant AGENTS docs reference them.  
3. `knowledge/architecture/glossary.md` and `knowledge/architecture/pitfalls.md` exist and are filled with meaningful content.  
4. A human review confirms that the knowledge structure:
   - Reflects the intended architecture.  
   - Avoids obvious gaps or contradictions.  
5. `template_construction/workdocs/outcome.md` contains a “Phase 3 outcome” section summarizing the created docs, key decisions, and any planned refinements for later phases.
