# Phase 8 – Infra Capabilities, Human Docs, and Quality System

> Objective: finish the template by adding **infra-focused abilities**, **human-facing explanation layers**, and a **baseline quality system** (tests + CI + examples), turning the template into a trustworthy, reusable asset.

---

## 1. Phase Positioning and Scope

- **Depends on:**  
  - Phases 1–7 (layout, docs, knowledge, registries, scaffolding, runtime).

- **Solves:**  
  - “Is the template usable and trustworthy for real projects?”  
  - “Can it self-check and support future dev/debug workflows?”

- **Non-goals:**  
  - Implementing arbitrary business abilities.  
  - Full-blown production observability or enterprise CI/CD (only a solid baseline is required).

---

## 2. Block A – Infra Ability Family and Dev/Debug Helpers

### 2.1 Goals

Add infra-focused and template-bound dev/debug abilities so that future projects can:

- Run health checks on the template structure.  
- Synchronize structure and docs.  
- Self-test runtime components.  
- Run tests and basic dev flows from abilities.

### 2.2 Suggested Ability Categories

1. **Template health checks**  
   - Verify consistency between `modules/` directories and `modules/overview/*.yaml`.  
   - Check for dangling references in registries/config/hooks.  
   - Validate that `AGENTS.md`, `ROUTING.md`, and other docs reference existing paths.  
   - Produce structured reports usable by scaffolds or CI.

2. **Structure / documentation sync**  
   - Based on registry/MANIFEST changes, update:  
     - Module-level `AGENTS.md` ability lists.  
     - Runtime/tool overview docs and tables.  
   - Keep code, metadata, and human docs aligned.

3. **Runtime self-check**  
   - For Tool Runner, Hook Runner, and Agent Runtime:  
     - Confirm configs can find scripts.  
     - Confirm hook handlers exist and are executable.  
     - Run a minimal smoke test without external side effects.

4. **Testing and CI helper abilities**  
   - Ability to trigger template test suites: unit, contract, integration, scenario tests.  
   - Ability to summarize recent test results into a human-readable test report.

5. **Dev/debug helpers (template-side only)**  
   - Backend: run unit tests, contract tests, integration tests for modules, start a local dev service, etc.  
   - Frontend (if applicable): run unit tests, linting, and build steps.

These abilities must be registered via Phase 4 scripts and may themselves use Phase 5 scaffolds or Phase 7 runtime where appropriate.

---

## 3. Block B – Human-Facing Docs and Indexes

### 3.1 Goals

Provide a **human-friendly explanation layer** for the template so new users can:

- Understand what is in scope for AI vs humans.  
- Discover where to start when initializing a project.  
- Find key knowledge, AGENTS, and tooling.

### 3.2 Suggested Artifacts

1. Update root `README.md` with:  
   - High-level overview of the template’s purpose and architecture (human perspective).  
   - Clear “start here” guidance for project initialization via `PROJECT_INIT/`.  
   - Links to key human-facing docs under `PROJECT_INIT/doc/`, `db/`, `ops/packaging/`, `ops/deploy/`.

2. Under `PROJECT_INIT/doc/`, provide:  
   - A human guide on how to prepare the information AI needs to initialize a project.  
   - A description of the expected collaboration with external AI tools.

3. Under `knowledge/examples/`, maintain example docs:  
   - Positive examples: smooth end-to-end flows using abilities, hooks, and scaffolds.  
   - Negative examples: misconfigurations and their fixes (linked to pitfalls where appropriate).

Ensure AGENTS docs link to these human docs where it helps humans supervise AI flows.

---

## 4. Block C – Tests, CI, and Examples

### 4.1 Test Suites

Implement at least these categories:

1. **Unit tests**  
   - For registry write utilities and runtime components (Tool Runner, Hook Runner, Agent Runtime).

2. **Contract/schema tests**  
   - Validate that all registry/config/hooks YAML comply with their schemas.  
   - Check for dangling cross-references between abilities, hooks, scripts, and documented paths.

3. **Integration tests**  
   - Cover end-to-end flows such as:  
     - “Add new ability → register → call via Tool Runner → hook triggered.”  
     - “Add new hook → ability execution reflects guard or logging behavior.”  
     - “Scaffold usage → registry updates → runtime calls.”

4. **Scenario/golden tests**  
   - Define a small number of golden scenarios under `modules/integration/scenarios.yaml`.  
   - Ensure they represent key flows; use them to guard against regressions.

5. **Lightweight performance/stability tests**  
   - Simple stress tests for Tool Runner and Hook Runner to avoid obvious bottlenecks (e.g., many no-op hooks).

### 4.2 CI Pipeline

Add a CI pipeline that at least:

1. Installs dependencies.  
2. Runs unit + contract tests.  
3. Runs critical integration/scenario tests.  
4. Optionally runs a “runtime reference update” command and fails if uncommitted changes appear (to ensure docs and metadata stay in sync).  
5. Blocks merging when tests fail.

CI configuration should be part of the template so downstream projects inherit a ready-made quality baseline.

---

## 5. Block D – Examples as Experience Knowledge

### 5.1 Goals

Use `knowledge/examples/` as an “experience repository” documenting:

- Good uses of abilities, hooks, and scaffolds (positive examples).  
- Common mistakes and how they were repaired (negative examples).

### 5.2 Suggested Structure

Under `knowledge/examples/`:

- Docs for positive flows:  
  - E.g., “Add a new module with scaffolding and run integration tests.”  
- Docs for negative flows:  
  - E.g., “A misconfigured hook blocked all ability calls and how we fixed it.”

Link examples to:

- Relevant pitfalls in `knowledge/architecture/pitfalls.md`.  
- Specific scenarios under `modules/integration/scenarios.yaml`.  
- Any infra abilities used or introduced in Phase 8.

---

## 6. Workdocs Usage for Phase 8 (Template Construction)

In `template_construction/workdocs/`:

- `task.md` should describe:  
  - The infra ability family.  
  - The layered documentation for humans.  
  - The quality system (tests and CI).  

- `todolist.md` should track:  
  - Each new infra ability group.  
  - Each major doc or index update.  
  - Each test suite and CI step implemented.

- `context.md` should record decisions about:  
  - Which tests are part of the minimal baseline.  
  - How much infra ability coverage is considered “enough” for the template.  
  - Any non-obvious trade-offs (performance vs simplicity, etc.).

---

## 7. Human Review Checklist

Humans should review Phase 8 with:

1. **Infra abilities**  
   - Are key infra abilities implemented and usable for future projects?  
   - Do they respect registries, hooks, and DevOps layouts?  
   - Are high-risk actions gated behind clear workflows and approvals?

2. **Human docs**  
   - Is it clear how a new user should start with the template?  
   - Do human docs explain the AI collaboration model?  
   - Are AGENTS and knowledge docs discoverable from human docs without overwhelming users?

3. **Quality system**  
   - Does the CI pipeline run reliably on the template?  
   - Are tests stable and sufficiently fast for a standard development workflow?  
   - Are golden scenarios meaningful and representative?

4. **Examples and experience knowledge**  
   - Do examples feel realistic and helpful?  
   - Are negative examples linked to pitfalls and test scenarios?

---

## 8. Definition of Done (DoD)

Phase 8 is complete when:

1. The template includes a meaningful set of infra-focused abilities and dev/debug helpers registered via registries.  
2. Human-facing docs and indexes are in place, giving new users a clear path to adopt the template.  
3. Unit, contract, integration, and scenario tests exist and are wired into a CI pipeline, which is enabled by default.  
4. `knowledge/examples/` contains at least a few positive and negative examples.  
5. Running the CI pipeline on a fresh clone of the template repo passes without manual interventions.  
6. `template_construction/workdocs/outcome.md` contains a “Phase 8 outcome” section summarizing infra abilities, docs, tests, CI, and any recommended next steps for downstream projects.
