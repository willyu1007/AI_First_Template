# Documentation & Naming Standards

This document defines concise standards for documentation and naming across the repository. It focuses on being easy for both AI and humans to read and maintain.

---

## A. Documentation Standards

### A.1 Scope and Goals

- **Goal**  
  Keep all documents AI-friendly and human-maintainable, avoiding context bloat, inconsistent naming, and broken routing.

- **Applies to**
  - All AI-facing documents (routing, policies, capability indexes, quickstarts, specs, workdocs).
  - Key human-facing documents (README, design docs, reports, evaluations).

- **AGENTS.md**  
  `AGENTS.md` is an AI-facing README, similar in role to `CLAUDE.md` and `GEMINI.md`.

---

### A.2 Audience

- Each document has a single audience: `ai` **or** `human` (never both).
- Human-facing documents such as `README.md` should state near the top:  
  `AI should ignore this file.`

---

### A.3 Format and Style

#### Encoding and File Format

- Use `UTF-8` for all text.
- Use **Markdown (`.md`)** for documentation.
- Use **YAML (`.yaml`)** for configuration and routing.
- Put metadata in **YAML front matter** at the top of Markdown files when needed.

#### Language

- Default language is **English**.
- Use other languages only when explicitly required by the project.

#### Style

- Prefer clear structure over long prose:
  - Use headings, lists, and tables where possible.
  - Write short, direct sentences.
- Emoji usage:
  - Allowed: a small, fixed set for status/signals, e.g. `✅`, `❌`, `⚠️`.
  - Not allowed: decorative emoji or emoticons without clear meaning.
- Length guidelines:
  - AI-facing documents (except knowledge-heavy docs) should normally stay under **150 lines**.
  - Human-facing documents have no hard limit but should remain easy to scan and navigate.

---

### A.4 Progressive Disclosure

#### Lightweight Entry Points

Top-level routing documents (e.g. `ROUTING.md`) should remain minimal:

- Explain the role of the directory.
- Show the routing tree and key sub-documents.
- Link to the next document to read.
- Avoid long background stories, history, or full process details.

#### Deeper Layers

- **Top level**: answer “what to do” and “where to look next”.
- **Middle level**: provide workflows, policies, and constraints.
- **Deep level**: store full details such as parameters, schemas, and examples.

---

### A.5 Minimal Front Matter Schema

Use the smallest front matter needed to keep routing and purpose clear.

```yaml
---
audience: ai | human           # Exactly one audience
purpose: <short description>   # What this document is for
updated_at: YYYY-MM-DD         # Optional: last significant update date
---
```

---

## B. Naming Conventions

### B.1 General Principles

1. **Clarity over cleverness**  
   Names should clearly reflect their role and responsibility.

2. **Explicit over implicit**  
   Avoid obscure abbreviations. Common ones such as `id`, `url`, `api`, `db` are fine.

3. **Consistency first**  
   Reuse existing styles within the project instead of inventing new ones.

4. **Reasonable length**  
   Typically 2–4 words. Very long names often indicate over-broad responsibility.

---

### B.2 Documents and Config Files

#### Markdown Documents

- Regular documents: `kebab-case.md`  
  - e.g. `data-spec.md`, `coding-guide.md`.
- Special, conventional files: `UPPER_SNAKE_CASE.md`  
  - e.g. `README.md`, `AGENTS.md`, `CAPABILITIES.md`.

#### Configuration Files

- Use lowercase names:
  - `lowercase.yaml` / `lowercase.json`
  - e.g. `defaults.yaml`, `schema.yaml`.

#### Templates

- Make the purpose explicit:
  - `kebab-case.template` or `kebab-case.md.template`
  - e.g. `workdoc-plan.md.template`.

#### Schemas / Protocols

- Use a clear suffix:
  - `something.schema.yaml`

---

### B.3 Directories

- General directories: `lowercase` or `kebab-case`  
  - e.g. `doc_agent/`, `doc_human/`.
- Module directories: prefer `lowercase`.
- Keep directory names short and descriptive.

---

### B.4 Code Naming (Python-Focused)

#### File Names

- Python files: `snake_case.py`  
  - e.g. `user_service.py`.
- TypeScript / JavaScript:
  - Logic / utilities: `camelCase.ts`
  - Components: `PascalCase.tsx`

#### Classes / Types

- Use `PascalCase`  
  - e.g. `UserService`, `DatabaseConnection`.

#### Functions / Methods

- Use `snake_case`  
  - e.g. `get_user_by_id`, `validate_input`.

#### Variables

- Use `snake_case`  
  - e.g. `user_id`, `retry_count`.

#### Constants

- Use `UPPER_SNAKE_CASE`  
  - e.g. `MAX_RETRY_COUNT`, `DEFAULT_TIMEOUT`.

#### Private Members

- Prefix with a single underscore:
  - `_private_method`, `_internal_cache`.

---

### B.5 Database Naming

- Table names: `snake_case`  
  - e.g. `user_profiles`.
- Column names: `snake_case`  
  - e.g. `created_at`.
- Index names: `idx_<table>_<columns>`  
  - e.g. `idx_users_email`.
- Constraint names:
  - Primary key: `pk_<table>`  
    - e.g. `pk_users`.
  - Foreign key: `fk_<table>_<column>`  
    - e.g. `fk_orders_user_id`.

---

### B.6 Test Naming

- Test files (Python): `test_<module>.py`  
  - e.g. `test_user_service.py`.
- Test functions (Python): `test_<description>`  
  - e.g. `test_user_service_creates_active_user`.

---

### B.7 Comments and Docstrings

#### Language and Style

- Match the project’s chosen language (default: English).
- Keep comments focused on **why** something is done, not restating the code.

#### Example (Python)

```python
def get_user_by_id(user_id: str) -> User:
    '''
    Retrieve a user by their unique identifier.

    Args:
        user_id: The unique user identifier.

    Returns:
        The User object if found.

    Raises:
        UserNotFoundError: If the user does not exist.
    '''
```
