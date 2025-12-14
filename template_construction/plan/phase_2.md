# Phase 2 – Documentation Conventions and AGENTS Structure

> Objective: define clear **documentation roles and conventions**, create the core `AGENTS.md` files, and establish the `knowledge/` skeleton so that AI systems can navigate and extend the repository safely.

---

## 1. Phase Positioning and Scope

- **Depends on:** Phase 1 skeleton and placeholder files.  
- **Solves:** “When the AI arrives in a directory, how does it know what it may do, read, and modify?”  
- **Prepares for:** Phase 3, where core mechanism knowledge and workflows are added on top of these conventions.

This phase focuses on **rules and templates** rather than detailed content.

---

## 2. Preconditions

Before starting Phase 2, the AI must verify:

1. Phase 1 layout exists (see Phase 1 DoD).  
2. The following files exist (possibly as empty placeholders):
   - Root `AGENTS.md`.  
   - `DOCS_CONVENTION.md`.  
   - `modules/AGENTS.md`.  
   - `knowledge/README.md` and `knowledge/architecture/overview.md`.  
   - `knowledge/modules/template/*`.  
   - `knowledge/devops/packaging_template.md` and `deploy_template.md`.  
   - `PROJECT_INIT/AI/AGENTS.md`.  
   - `ops/packaging/AGENTS.md`, `ops/deploy/AGENTS.md`.  

In `template_construction/workdocs/preparation.md`, add a **Phase 2 recap** of architecture constraints specifically related to docs and workspaces (roles of README vs AGENTS vs knowledge vs workdocs).

---

## 3. Work Blocks Overview

Phase 2 is structured into three work blocks:

- **Block A – Documentation conventions (`DOCS_CONVENTION.md`)**  
- **Block B – AGENTS structure and content skeletons**  
- **Block C – Initial knowledge templates**

Each block has specific deliverables and review points.

---

## 4. Block A – DOCS_CONVENTION.md

This document will be used as part of our AI reading strategy going forward，please keep it AI-friendly：write in a clear structure manner， organize the content into chapters and provide a table of contents.

### 4.1 Purpose

`DOCS_CONVENTION.md` is the **single source of truth** for documentation roles and naming conventions. External AIs will rely on it whenever they need to create or modify docs.

### 4.2 Required Content

Please refer to the documentation `reference/documentation_standards.md` during the build process. Ensure `DOCS_CONVENTION.md` covers at least:

1. **Document types and roles**  
   - `README.md` – human-facing, area overview, no AI behavior rules.  
   - `AGENTS.md` – AI-facing, defines allowed operations, routing hints, and workdocs usage.  
   - `knowledge/*.md` – stable, reusable knowledge and architecture explanations.  
   - `workdocs/` – scenario-local, task-level plans, context, tasks, and outcomes.

2. **Path and naming conventions**  
   - How module docs are organized under `modules/<module_id>/`.  
   - Where project-wide registries live (`modules/overview/`, `.system/registry/`, `.system/hooks/`).  
   - Where DevOps docs live (`PROJECT_INIT/`, `db/`, `ops/packaging/`, `ops/deploy/`).  

3. **AI ↔ human collaboration flow**  
   - “AI drafts → human reviews against a checklist → AI revises” as standard.  
   - Where humans should leave comments (e.g., in review tools, not inline YAML editing).  
   - How to record decisions and lessons in workdocs and promotion rules to knowledge docs.

### 4.3 Workdocs Integration

- In `template_construction/workdocs/task.md`, document key design decisions made for DOCS_CONVENTION (e.g., what belongs to AGENTS vs knowledge).  
- In `todolist.md`, add one checklist item for each major DOCS_CONVENTION section and mark them done once written and reviewed by AI.

---

## 5. Block B – AGENTS Structure and Content Skeletons

### 5.1 Target Files

By the end of Phase 2, the following `AGENTS.md` files must exist and follow a **consistent section structure**:

- Root `AGENTS.md`.  
- `modules/AGENTS.md`.  
- `knowledge/modules/template/AGENTS.md`.  
- `PROJECT_INIT/AI/AGENTS.md`.  
- `ops/packaging/AGENTS.md`.  
- `ops/deploy/AGENTS.md`.  
- (Optional but recommended) `db/AGENTS.md`.  
- Later phases will add more (module-local AGENTS, `.system/AGENTS.md`, etc.).

### 5.2 Recommended Section Structure

All AGENTS docs should use a shared skeleton, for example:

1. **Scope & Audience**  
   - What this area covers and who should read this doc (external AI vs human).  

2. **Allowed Operations**  
   - What an AI is allowed to do in this area (read/update which files, call which scripts).  
   - What is explicitly disallowed without human approval.

3. **Knowledge Routing**  
   - Which `knowledge/` docs are relevant for this area.  
   - How to choose the right document to read before acting.

4. **Workdocs Usage**  
   - Whether this area has its own workdocs directory and how it should be used.  
   - How to name task IDs and structure per-task workdocs.

5. **Safety & Escalation**  
   - When to escalate to humans (e.g., production-affecting changes, registry schema changes).  
   - Any environment-specific constraints (dev vs prod).

### 5.3 Concrete Instructions

For each of the target files:

1. Start from the skeleton above.  
2. Fill in a first-pass content based on architecture docs and DOCS_CONVENTION.  
3. Ensure paths are correct and consistent with Phase 1 layout.  
4. Add references to relevant knowledge docs (from Block C below).

Add a checklist entry in `todolist.md` for each AGENTS file and mark them done after the AI draft is complete and self-checked.

---

## 6. Block C – Initial Knowledge Templates

### 6.1 Target Files

Populate the following files with initial, **AI-readable** content (not just TODO):

- `knowledge/AGENTS.md` – explains the purpose and structure of `knowledge/`.  
- `knowledge/architecture/overview.md` – high-level architecture overview (modular structure, registries, routing, hooks, DevOps).  
- `knowledge/modules/template/layout.md` – standard module layout description.  
- `knowledge/modules/template/manifest_fields.md` – description of MANIFEST.yaml fields and constraints.  
- `knowledge/modules/template/registration_flow.md` – the high-level process for creating/registering modules.  
- `knowledge/modules/template/examples/homework_system.md` – placeholder example scenario (may be mostly TODO at this stage).  
- `knowledge/devops/packaging_template.md` – abstract packaging flow and constraints.  
- `knowledge/devops/deploy_template.md` – abstract deployment flow and constraints.

### 6.2 Style Requirements

All knowledge docs must:

- Be written for **AI consumption** – concise, structured, and with clear callouts of “do / don’t”.  
- Avoid duplicating AGENTS content; instead, AGENTS should reference these docs.  
- Avoid concrete platform-specific commands; focus on abstract flows and constraints.

Record any key structural decisions (e.g., how MANIFEST fields map to registries) in `template_construction/workdocs/context.md`.

---

## 7. Human Review Checklist

A human maintainer should validate Phase 2 using the following questions:

1. **DOCS_CONVENTION.md**  
   - Does it clearly distinguish roles of `README`, `AGENTS`, `knowledge`, and `workdocs`?  
   - Does it describe path conventions accurately?  
   - Does it reflect the AI-first collaboration model?

2. **AGENTS files**  
   - Do all required AGENTS files exist?  
   - Do they share a consistent section structure?  
   - Do they **avoid** duplicating human-only explanations from README?  
   - Do they point to the correct `knowledge/` docs?

3. **Knowledge templates**  
   - Are the knowledge docs in the right locations?  
   - Do they respect the one-line responsibility for each directory?  
   - Are there obvious contradictions with the architecture docs?

Any issues should be recorded in `template_construction/workdocs/context.md` and result in revisions by the AI.

---

## 8. Definition of Done (DoD)

Phase 2 is complete when:

1. `DOCS_CONVENTION.md` exists and:
   - Clarifies roles of document types.  
   - Documents key path conventions.  
   - Describes AI↔human collaboration flows.

2. All target `AGENTS.md` files listed in Section 5.1:
   - Use the shared section structure.  
   - Describe allowed operations, knowledge routing, and workdocs rules.  
   - Point to correct knowledge docs.

3. The main `knowledge/` areas exist and are populated with initial content that matches their responsibilities.

4. All of the above have been reviewed by a human based on this checklist, and `template_construction/workdocs/outcome.md` contains a “Phase 2 outcome” section summarizing:
   - Main conventions decided.  
   - Any remaining TODOs for later phases.  
   - Confirmed alignment with the architecture docs.
