# {{PROJECT_NAME}} – Product Requirements & Development Plan

> **Template note:** Replace every `{{PLACEHOLDER}}` with concrete information for your use‑case. Structure inspired by proven PRD examples (e.g. clustered‑singleton test plan). Remove this block when done.

---

## 0. Objective & Scope
**Objective:** {{OBJECTIVE}}

**Scope Includes**
- {{SCOPE_ITEM_1}}
- {{SCOPE_ITEM_2}}

**Out of Scope**
- {{OUT_OF_SCOPE}}

---

## 1. Key References
- **Primary Issue / Ticket:** {{ISSUE_LINK}}
- **Related PR / Design Doc:** {{PR_OR_DOC_LINK}}
- **Specification / Docs:** {{SPEC_LINK}}
- **Stakeholders / Reviewers:** {{STAKEHOLDER_LIST}}

---

## 2. Definitions & Acronyms (optional)
| Term | Definition |
|------|------------|
| SUT | {{SYSTEM_UNDER_TEST_DESCRIPTION}} |
| … | … |

---

## 3. Prerequisites & Setup
| Task ID | Description | Success Criteria |
|---------|-------------|------------------|
| 0.1 | Environment setup | Able to build & run baseline tests |
| 0.2 | Baseline build | CI passes on clean checkout |
| 0.3 | Knowledge ramp‑up | Key modules and docs reviewed |

_Add further setup tasks as needed._

---

## 4. Architecture Overview
Provide a **high‑level diagram** or bullet list of main components, data‑flow, external services, and integration points.

---

## 5. Development Phases & Tasks
Use phases to group coherent work streams. Adjust numbering/style freely.

### Phase 1 – {{PHASE1_TITLE}}
**Goal:** {{PHASE1_GOAL}}

| Task ID | Description | Acceptance Criteria | Owner | Notes |
|---------|-------------|---------------------|-------|-------|
| 1.1 | {{TASK_DESCRIPTION}} | {{CRITERIA}} | {{OWNER}} | |
| 1.2 | … | … | … | … |

### Phase 2 – {{PHASE2_TITLE}}
**Goal:** {{PHASE2_GOAL}}

| Task ID | Description | Acceptance Criteria | Owner | Notes |
|---------|-------------|---------------------|-------|-------|
| 2.1 | … | … | … | … |

_Add additional phases as required._

---

## 6. Coding Guidelines & Tooling
- **Cursor Rules**: ensure a `.cursorrules` file exists with team conventions (lint, style, commit messages).
- **Cursor Memory**: enable per‑project memory to retain context during long sessions.
- **Gemini CLI**: preferred for brainstorming, code explanation, shell automation.
- **Checkpoints**: commit after each completed task; push to branch `feature/{{TASK_ID}}`.
- **Automated Agents**:
  - Cursor Background Agent for continuous refactor/testing on specific issues.
  - Jules (optional) for PR‑level automated fixes.

---

## 7. Progress Tracking
> Update this table continuously.

| Task ID | Status (Not Started / In Progress / Done) | Notes / Blockers |
|---------|-------------------------------------------|------------------|
| 1.1 | | |

---

## 8. Review & QA Plan
- **Peer Review**: mandatory on every Pull Request.
- **CI / CD**: {{CI_PIPELINE_DETAILS}}.
- **Test Coverage Goal**: {{COVERAGE_PERCENT}}% line coverage.
- **Manual QA**: describe manual verification steps if any.

---

## 9. Deliverables
- Code merged to `main` (or `trunk`)
- Updated documentation / README
- Test suite additions
- Demo or walkthrough video (optional)

---

## 10. Timeline & Milestones
| Date | Milestone | Owner |
|------|-----------|-------|
| {{YYYY‑MM‑DD}} | Kick‑off | {{OWNER}} |
| {{YYYY‑MM‑DD}} | Phase 1 complete | {{OWNER}} |
| {{YYYY‑MM‑DD}} | Final review | {{OWNER}} |

---

## 11. Appendix
### A. Useful Commands
```bash
# Build project
<build‑command>

# Run tests
<test‑command>
```

### B. Links & Resources
- {{ADDITIONAL_LINK_1}}
- {{ADDITIONAL_LINK_2}}


---

_End of PRD template._

