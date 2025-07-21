# Prompt Template – Generate PRD & Development Plan

> **Purpose**: Use this template as a starting prompt for any new feature, bug‑fix, or test‑coverage task. It guides an AI assistant to read your `PRD_template.md`, analyse the codebase, and produce a fully‑fledged development plan (`plan.md`) suitable for a Junior developer.

---

## 1 . Context *(replace placeholders)*

```
You are a Senior developer experienced in {{TECH_STACK}}.

The project uses {{RUNTIME/FRAMEWORK}} and you have commit rights.

You need to coordinate work on {{ISSUE_LINK}} (summary: {{ISSUE_TITLE}}).
The {{FEATURE}} implementation already exists, but {{WORK_NEEDED}} needs to be added/completed.
```

### Placeholders

| Placeholder                          | Description                                                     |
| ------------------------------------ | --------------------------------------------------------------- |
| `{{TECH_STACK}}`                     | Primary language / stack (e.g. Java EE, Spring, Python Django)  |
| `{{RUNTIME/FRAMEWORK}}`              | Server, runtime, or framework in use (e.g. WildFly, Flask)      |
| `{{ISSUE_LINK}}` & `{{ISSUE_TITLE}}` | URL & title of the tracked issue/ticket                         |
| `{{FEATURE}}`                        | Name of the existing feature to extend/test                     |
| `{{WORK_NEEDED}}`                    | What must be delivered (e.g. integration tests, refactor, docs) |

---

## 2 . High‑level Instructions to the AI

1. **Read** the issue description and only the first‑level links provided.
2. **Analyse** the current codebase to locate the {{FEATURE}} implementation.
3. **Load** `PRD_template.md` (provided in this chat) and replicate its headings into a new file `plan.md`.
4. **Populate** each section of `plan.md` with a detailed yet concise plan that a **Junior Developer** can execute autonomously.
5. **Split** work into *phases* ➔ *tasks* ➔ *sub‑tasks*.
6. **Embed Git workflow guidance**:
   - Branch naming convention (`feature/{{ISSUE_KEY}}-short-description`).
   - Frequent commits with meaningful messages.
   - Continuous test runs after each task.
7. **Mandate progress logging** in `WORKING_PROGRESS.md` (one entry per task).
8. **Ensure hand‑over readiness**: every completed task leaves the repo in a build‑passing state.
9. **Deliver** only the generated `plan.md` as output—no extra commentary.

---

## 3 . Success Criteria

- The Junior Developer can pick up the plan and complete {{WORK\_NEEDED}} without further clarification.
- All automated tests pass after each phase.
- `plan.md` follows the exact section order of `PRD_template.md`.
- `WORKING_PROGRESS.md` template exists and is referenced within the plan.

---

## 4 . Prompt Skeleton *(copy‑paste & fill placeholders)*

```
You are a Senior developer experienced in {{TECH_STACK}}.

Context:
- Project stack: {{RUNTIME/FRAMEWORK}}
- Target issue: {{ISSUE_LINK}}
- Feature: {{FEATURE}}
- Work needed: {{WORK_NEEDED}}

Goals:
1. Read and understand the issue plus first‑level links.
2. Analyse the codebase to locate current implementation.
3. Using `PRD_template.md`, generate `plan.md` containing a detailed, phased development plan.
4. Instruct a Junior developer to:
   - Follow tasks in order
   - Track progress in `WORKING_PROGRESS.md`
   - Commit frequently and run all tests before each push

Output only the complete `plan.md` file.
```

---

> **Tip**: Save this prompt template and customise the placeholders for each new task to keep a consistent, high‑quality workflow across projects.

