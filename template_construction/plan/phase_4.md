# Phase 4 – Registry Write Scripts and Coordination Mechanisms

> Objective: provide **robust, script-based write paths** for all registry types (modules, content routes, abilities, hooks) and define coordination and consistency rules between them.

---

## 1. Phase Positioning and Scope

- **Depends on:**  
  - Phase 1 layout (`modules/overview/`, `modules/routes/`, `.system/registry/`, `.system/hooks/`).  
  - Phase 2 conventions and AGENTS.  
  - Phase 3 knowledge, especially standard workflows and terminology.

- **Solves:**  
  - “How do we safely write to registries?”  
  - “How do registries depend on each other and stay consistent?”

- **Prepares for:**  
  - Phase 5 scaffolding (which will orchestrate these scripts).  
  - Phase 6 sample registrations.

---

## 2. Preconditions

The AI must confirm that:

1. The layout and placeholder files from Phase 1 exist.  
2. Phase 3 has defined clear concepts for module types, instances, interfaces, abilities, routes, and hooks.  
3. `DOCS_CONVENTION.md` and relevant `AGENTS.md` already mention registries conceptually (even if write flows are not yet implemented).  
4. `template_construction/workdocs/preparation.md` contains a Phase 4 section listing registry types and their intended locations.

---

## 3. Block A – Registry Types and Schema Files

### 3.1 Directory and File Layout

All **schema YAML** files must live under:

```text
.system/
  registry/
    schemas/
      module_type.schema.yaml
      module_instance.schema.yaml
      module_maturity.schema.yaml
      module_interface.schema.yaml
      route.schema.yaml
      ability_low.schema.yaml
      ability_high.schema.yaml
      hook.schema.yaml
      # extension schemas as needed
```

Registry **content** is split as follows:

- Modules (content only) under `modules/overview/`:
  - `type_registry.yaml`  
  - `instance_registry.yaml`  
  - `instance_maturity.yaml`  
  - `interfaces.yaml`  

- Content routing under `modules/routes/*.yaml`.  
- Abilities under `.system/registry/low-level/*.yaml` and `.system/registry/high-level/*.yaml`.  
- Hooks under `.system/hooks/*.yaml` (one file per hook id, e.g. `ability_usage_tracker.yaml`; each file declares an `event_type` such as `PromptSubmit`, `PreAbilityCreate`, `PreAbilityCall`, `PostAbilityCall`, or `SessionStop`).

### 3.2 Tasks

1. In `.system/registry/AGENTS.md`, list all registry types and for each:  
   - Schema file path.  
   - Content file path(s).  
   - One-line purpose.

2. For each registry type, create a schema file with at least:  
   - Primary key fields (`id`, `module_id`, `ability_id`, etc.).  
   - Relationship fields (`module_id`, `ability_ids`, etc.).  
   - Descriptive fields (`description`, `status`, `tags`).  
   - Enumerations and required/optional constraints.

3. Ensure schema docs are aligned with Phase 3 knowledge (terminology and fields). Any conflicts must be resolved and recorded in `template_construction/workdocs/context.md`.

---

## 4. Block B – Minimal Registry Write Library

### 4.1 Code Layout

Create a general-purpose registry write library under:

```text
scripts/
  devops/
    registry/
      __init__.py
      common.py
      modules.py
      routes.py
      abilities.py
      hooks.py
```

### 4.2 Responsibilities

- `common.py`:
  - `load_yaml(path)` / `dump_yaml(path, data)` utilities.  
  - `insert_or_update_entry(data, key_field, new_entry)` with stable ordering and deduplication.  
  - `validate_entry(entry, schema_path)` that uses the schema YAML for structure validation.

- `modules.py`:
  - Functions to add/update entries in `type_registry.yaml`, `instance_registry.yaml`, `instance_maturity.yaml`, `interfaces.yaml` based on corresponding schemas.

- `routes.py`:
  - Functions to add/update content routes in `modules/routes/*.yaml` using `route.schema.yaml`.

- `abilities.py`:
  - Functions to add/update abilities in `.system/registry/low-level/*.yaml` and `.system/registry/high-level/*.yaml` using `ability_low`/`ability_high` schemas.

- `hooks.py`:
  - Functions to add/update hook registrations in `.system/hooks/*.yaml` using `hook.schema.yaml`.

### 4.3 CLI Entry Points (Optional but Recommended)

Provide a small script (e.g., `scripts/devops/registry_add_entry.py`) that can:

1. Parse a `--type` argument to select the appropriate module (`modules`, `routes`, `abilities`, `hooks`).  
2. Load an entry from `--entry-file path/to/entry.yaml` or `--entry-json '{...}'`.  
3. Validate the entry against the schema.  
4. Call the appropriate insert/update function to write into the registry content file.

This script is the **canonical imperative entry point** for registry modifications in later phases and for human/manual debugging.

---

## 5. Block C – Coordination Rules and Consistency Checks

### 5.1 Documentation

Create `.system/registry/registry_coordination.md` describing coordination relationships, at minimum:

1. **Ability ↔ Module instance**  
   - When adding a new ability, which module instances and routes should be checked or updated?  
   - Which checks are **blocking errors** vs warnings?

2. **Routing ↔ Module instance**  
   - When adding a new content route, how to check for conflicts with existing routes (overlapping scopes/topics) and module ownership?

3. **Hooks ↔ Abilities / Routing**  
   - When adding a hook registration, how to validate:
     - The referenced abilities exist.  
     - The referenced hook event types and routing stages are valid.

For each relationship, explicitly state:

- “When A changes, check B/C.”  
- Whether issues should cause:
  - Write failure.  
  - Warning but allowing write.  
  - Recording-only behavior.

### 5.2 Code-Level Consistency Functions

In `scripts/devops/registry/` implement consistency functions, e.g.:

- `modules.py`:
  - `check_module_instance_consistency(new_instance_entry) -> list[Problem]`

- `routes.py`:
  - `check_route_addition_consistency(new_route_entry) -> list[Problem]`

- `abilities.py`:
  - `check_ability_addition_consistency(new_ability_entry) -> list[Problem]`

- `hooks.py`:
  - `check_hook_addition_consistency(new_hook_entry) -> list[Problem]`

Each function should:

- Load current registry content.  
- Apply the coordination rules from `registry_coordination.md`.  
- Return a structured problem list (at least error vs warning message types).

Write scripts should, by default:

- Invoke these checks before write.  
- Abort on severe errors, with clear messages.  
- Provide a `--no-check` escape hatch reserved for special cases (not used in scaffolding by default).

---

## 6. Workdocs Usage for Phase 4

- In `task.md`, describe Blocks A/B/C, highlight that they are **dependencies** for scaffolding.  
- In `todolist.md`, track:
  - Schema files created and reviewed.  
  - Library modules implemented.  
  - CLI entry point built.  
  - Coordination doc and check functions done.  
- In `context.md`, record:
  - Any tricky consistency rules.  
  - Any deliberate deviations from architecture docs.

---

## 7. Human Review Checklist

Humans should verify Phase 4 with the following:

1. **Schema clarity**  
   - Are schema files complete enough to catch obvious errors?  
   - Do field names and relationships match the architecture docs?

2. **Write library behavior**  
   - Can a manually crafted sample entry be written to each registry type via the scripts without breaking existing content?  
   - Are entries sorted and deduplicated as expected?

3. **Coordination rules**  
   - Are key cross-registry relationships documented?  
   - Are blocking vs warning behaviors sensible?  
   - Do check functions implement the documented rules?

4. **Usage guidance**  
   - Is it clear from AGENTS and knowledge docs that **registry changes must go through scripts**, not manual editing?

Issues identified in review should be addressed by the AI and logged in `outcome.md`.

---

## 8. Definition of Done (DoD)

Phase 4 is complete when:

1. `.system/registry/schemas/*.schema.yaml` exist and have passed at least one human review.  
2. `.system/registry/AGENTS.md` documents all registry types and their schema/content mappings.  
3. Script-based write functions exist for all major registry types (modules, routes, abilities, hooks).  
4. At least one sample entry per registry type can be written via the scripts without corrupting the YAML structure.  
5. `registry_coordination.md` and corresponding `check_*_consistency` functions are in place and invoked by default in write flows.  
6. `template_construction/workdocs/outcome.md` contains a “Phase 4 outcome” section summarizing scripts, schemas, and coordination rules, plus any planned extensions.
