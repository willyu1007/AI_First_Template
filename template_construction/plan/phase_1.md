# Phase 1 – Repository Skeleton and Directory Layout

> Objective: establish the **complete directory tree and placeholder files** for the repository template so that later phases can directly focus on content, not paths.

This guide assumes you are an **external AI** acting as the main implementer, with humans reviewing and approving.

---

## 1. Phase Positioning and Scope

- **Previous phases:** none (this is the starting point).  
- **This phase solves:** “Where does everything live?”  
- **Later phases depend on:** stable paths and placeholder files created here.

This phase is intentionally **content-light**: create structures and minimal placeholders, avoid writing heavy documentation or code.

---

## 2. Preconditions and Inputs

Before you start, you must:

1. Read or re-read the core architecture docs (paths are indicative):  
   - Root `README.md` (if it exists yet in this construction repo).  
   - `modular_mannual.md`.  
   - `ability_routing.md`.  
   - `content_routing.md`.  
   - `agents_strategy_guide.md`.  
   - `hooks.md`.  
   - `devops_extension_guide.md`.  
2. Open `template_construction/workdocs/` and ensure the following files exist (create them if necessary):  
   - `preparation.md`, `plan.md`, `task.md`, `todolist.md`, `context.md`, `outcome.md`.  
3. In `preparation.md`, add a **Phase 1 section** summarizing the architecture invariants that affect layout:
   - Modules as primary place for business logic.  
   - Central registries under `modules/overview/` and `.system/registry/`.  
   - Scenario-local workdocs in `PROJECT_INIT/`, `/db/`, `/ops/packaging/`, `/ops/deploy/`.  
   - Hooks under `.system/hooks/`.  

You may assume the repo is otherwise empty or near-empty.

---

## 3. Target Layout (High-Level)

You must create a directory tree that supports the architecture and DevOps extensions. At minimum, the root should contain:

```text
/README.md
/AGENTS.md
/ARCHITECTURE_OVERVIEW.md
/DOCS_CONVENTION.md
/modular_mannual.md            # architecture spec (provided or copied in)
/ability_routing.md
/content_routing.md
/agents_strategy_guide.md
/hooks.md
/devops_extension_guide.md

/modules/
  AGENTS.md
  ROUTING.md
  routes/
    README.md
  overview/
    README.md
    type_registry.yaml
    instance_registry.yaml
    instance_maturity.yaml
    interfaces.yaml

/knowledge/
  README.md
  architecture/
    overview.md
  modules/
    template/
      AGENTS.md
      layout.md
      manifest_fields.md
      registration_flow.md
      examples/
        homework_system.md
  devops/
    packaging_template.md
    deploy_template.md

/.system/
  hooks/
    README.md
  registry/
    README.md
    schemas/           # schema files will be created in Phase 4
    low-level/
    high-level/

/PROJECT_INIT/
  doc/
    README.md
  AI/
    AGENTS.md
  workdocs/
    README.md

/db/
  schema/
    README.md
  migrations/
    README.md
  config/
    README.md
  samples/
    README.md
  workdocs/
    README.md

/ops/
  packaging/
    AGENTS.md
    scripts/
      README.md
    workdocs/
      README.md
  deploy/
    AGENTS.md
    scripts/
      README.md
    workdocs/
      README.md

/scripts/
  hooks/
    README.md
  devops/
    README.md

/template_construction/
  AGENTS.md
  workdocs/
    preparation.md
    plan.md
    task.md
    todolist.md
    context.md
    outcome.md
```

You may refine this structure slightly in later phases, but **paths used by architecture docs must remain stable**.

---

## 4. Step-by-Step Instructions (for AI)

### 4.1 Update Workdocs for Phase 1

1. In `template_construction/workdocs/plan.md`:
   - Add a “Phase 1 – Skeleton” subsection with:
     - Purpose, non-goals, and key artifacts (directory tree, placeholder files).  
2. In `template_construction/workdocs/task.md`:
   - Describe Phase 1 in 2–4 paragraphs:
     - What you will create.  
     - Why the layout matters for routing, registries, hooks, and DevOps.  
3. In `todolist.md`:
   - Add checklist items for each major area:
     - Root files.  
     - `modules/` structure.  
     - `knowledge/` structure.  
     - `.system/` layout.  
     - `PROJECT_INIT/`, `/db/`, `/ops/packaging/`, `/ops/deploy/`.  
     - `scripts/`.  
     - `template_construction/` and its workdocs.  

### 4.2 Create Root-Level Files and Directories

- Create the root-level Markdown files (`README.md`, `AGENTS.md`, `ARCHITECTURE_OVERVIEW.md`, `DOCS_CONVENTION.md`, and architecture specs).  
- For now, most may contain `"TODO: to be filled in later phases"`. Do **not** write detailed content yet.  
- Create directories `modules/`, `knowledge/`, `.system/`, `PROJECT_INIT/`, `db/`, `ops/`, `scripts/`, `template_construction/`.

Mark the corresponding checklist items in `todolist.md` as done when finished.

### 4.3 Create Module and Knowledge Skeletons

1. Under `modules/`:
   - Create `AGENTS.md` and `ROUTING.md` with minimal placeholder headings.  
   - Create `routes/README.md`.  
   - Create `overview/` with the YAML files listed above, each containing a minimal, valid YAML header (e.g. `# TODO: fill in registry` plus an empty structure).

2. Under `knowledge/`:
   - Create `README.md`, `architecture/overview.md`.  
   - Create `modules/template/` with:
     - `AGENTS.md`, `layout.md`, `manifest_fields.md`, `registration_flow.md`.  
     - `examples/homework_system.md` with placeholder text.  
   - Create `devops/packaging_template.md` and `devops/deploy_template.md` with placeholder text.

Update `todolist.md` as you complete each sub-area.

### 4.4 Create .system and DevOps Layouts

1. Under `.system/`:
   - Create `hooks/README.md`.  
   - Create `registry/README.md`.  
   - Create empty `registry/schemas/`, `registry/low-level/`, and `registry/high-level/` directories.

2. Under `PROJECT_INIT/`, `db/`, `ops/packaging/`, and `ops/deploy/`:
   - Create the subdirectories and placeholder READMEs and AGENTS files as in the layout section above.

3. Under `scripts/`:
   - Create `hooks/README.md` and `devops/README.md` only; actual scripts come in later phases.

### 4.5 Initialize template_construction/ Workspace

1. Under `template_construction/`, create `AGENTS.md` with:
   - A title.  
   - A short note that this area is the workspace for building the template itself.  
   - A pointer to this phase guide and `template_construction_principles.md`.

2. Under `template_construction/workdocs/`, ensure that:
   - `preparation.md`, `plan.md`, `task.md`, `todolist.md`, `context.md`, `outcome.md` all exist.  
   - Each has at least a top-level title and a “Phase 1” subsection.

Update `context.md` with a summary of what you have created so far and any deviations from the planned tree.

---

## 5. Human Review Checklist

A human maintainer should validate Phase 1 using the following questions:

1. **Layout completeness**  
   - Does the directory tree match the intended architecture (modules, knowledge, `.system`, DevOps areas)?  
   - Are all registry-related locations present (`modules/overview/`, `.system/registry/`, `.system/hooks/`)?  

2. **Workspaces and workdocs**  
   - Does each DevOps scenario (`PROJECT_INIT/`, `db/`, `ops/packaging/`, `ops/deploy/`) have a `workdocs/` directory?  
   - Does `template_construction/workdocs/` exist with all six documents?  

3. **Placeholders vs. content**  
   - Are placeholders clearly marked as TODO and not pretending to be final content?  
   - Are any unintended or out-of-scope files/directories present? If yes, record them in `context.md` and decide whether to keep or remove them.

4. **Git status**  
   - Does `git status` (if used) show a clean, understandable set of new files that reflect the structure above?

Humans should not manually adjust the tree unless necessary. When they do, they should instruct the AI to update the structure in a follow-up iteration and record it in workdocs.

---

## 6. Definition of Done (DoD)

Phase 1 is considered complete when:

1. The directory tree and placeholder files listed in **Section 3** exist in the repository.  
2. All required `AGENTS.md` and key `README.md` files exist, even if they only contain TODO content.  
3. `template_construction/workdocs/` is in place, with Phase 1 sections written in `preparation.md`, `plan.md`, `task.md`, `todolist.md`, and `context.md`.  
4. A human maintainer has reviewed the structure using the checklist above and confirmed that later phases can rely on these paths without adjustment.  
5. `template_construction/workdocs/outcome.md` contains a short “Phase 1 outcome” section summarizing:
   - What was created.  
   - Any deviations from this guide.  
   - Open questions or follow-ups for later phases.
