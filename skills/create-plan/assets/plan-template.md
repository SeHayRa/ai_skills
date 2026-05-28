# Plan: <FEATURE_OR_PROJECT_NAME>

> Status: DRAFT — pending answers to open questions
> Created: <YYYY-MM-DD>
> Spec: `.specs/spec.md` (read this first)

---

## 1. Overview

<2–4 sentences summarizing what this plan delivers. Should mirror the spec's project goal but stated as an implementation outcome ("we will build X by doing Y") rather than a user-facing outcome. Anyone reading this section alone should know roughly what gets built and how the pieces fit together.>

**Link to spec:** `.specs/spec.md`

**Resolved questions from spec:** <Summarize how the spec's open questions were answered, since those answers drive the plan. If a question is still open, flag it here.>

---

## 2. Architecture Overview

<A short paragraph describing the new shape of the system after this work lands. What are the major components, and how do they relate? Include an ASCII or Mermaid diagram if it helps — one diagram is usually enough.>

```
<ASCII sketch or mermaid block — optional but encouraged>
```

**Key architectural choices:**

- <Choice #1 — e.g. "Synchronous request/response between API and matching engine; no queue.">
- <Choice #2>

---

## 3. Implementation Phases

<Order of operations. Each phase should be independently buildable and ideally independently testable. The agent implementing this plan should be able to stop after any phase and have working code.>

### Phase 1: <Name>
- **Goal:** <one line>
- **Scope:** <what's in, what's deferred>
- **Done when:** <observable condition>

### Phase 2: <Name>
- **Goal:**
- **Scope:**
- **Done when:**

### Phase 3: <Name>
- **Goal:**
- **Scope:**
- **Done when:**

<Add or remove phases as needed. 2–5 phases is typical.>

---

## 4. Files to Modify

<For each file, name the specific section/function/class being touched and what changes. Don't write the new code here — describe the change.>

### `<path/to/file>`
- **What changes:** <e.g. "Add a `cache_key()` helper and wire it into `get_order()`">
- **Why:** <one line>
- **Notes:** <gotchas, related callers, etc.>

### `<path/to/file>`
- **What changes:**
- **Why:**

---

## 5. Files to Create

### `<path/to/new/file>`
- **Purpose:** <one line>
- **What it contains:** <high-level — e.g. "OrderCache class with get/set/invalidate methods">
- **Why a new file (vs. adding to an existing one):** <one line>

---

## 6. Data Model / Schema Changes

<New or modified tables, structs, types, message formats. Include the actual shape — type definitions, schemas, or migration SQL are appropriate here.>

### <Entity or table name>
- **Status:** <new / modified>
- **Shape:**

```
<type definition, SQL, JSON schema, protobuf — whatever fits>
```

- **Migration notes:** <if existing data needs to be backfilled or transformed>

If no schema changes are needed, say so and move on.

---

## 7. Interface Contracts

<The "shape of the seams" between components. Function signatures, API endpoints, message formats, event shapes. Code snippets are encouraged here — they remove ambiguity.>

### <Component or endpoint name>

```
<signature, endpoint definition, message format>
```

- **Inputs:** <description>
- **Outputs:** <description>
- **Errors / edge cases:** <what can go wrong and how it's surfaced>
- **Example:** <a concrete request/response or call site>

---

## 8. Dependencies

<List anything that needs to be added, removed, or upgraded. PREFER existing dependencies — only add new ones if there's no reasonable way to do this with what's already in the codebase.>

### Already in the codebase (will use)
- `<package>` — <what it's used for in this work>

### New additions (require justification)
- `<package@version>` — <why this is needed, why an existing dependency won't work, surface area being used>

### Upgrades
- `<package>`: `<old>` → `<new>` — <why>

If no dependency changes, state that explicitly: "No new or upgraded dependencies; this work uses only what's already in the codebase."

---

## 9. Lower-Level Design Decisions

<The non-obvious calls that the implementing agent shouldn't have to re-derive. Concurrency model, error handling style, data ownership, naming conventions specific to this work, performance/memory trade-offs, etc. Frame each as a decision + rationale + alternatives considered.>

### <Decision name>
- **Decision:** <what was chosen>
- **Rationale:** <why>
- **Alternatives considered:** <what else was on the table, why rejected>

### <Decision name>
- **Decision:**
- **Rationale:**
- **Alternatives considered:**

---

## 10. Testing Strategy

<What gets tested at what level, and the specific scenarios. Don't say "write tests" — name the cases.>

### Unit tests
- `<module / function>`: <what scenarios — e.g. "empty input, single item, boundary at MAX_SIZE, overflow">

### Integration tests
- <Scenario — e.g. "Full order submission flow including DB write and cache invalidation">

### Manual / exploratory
- <Anything that's hard to automate but needs verification>

### Test data / fixtures needed
- <Any setup, mocks, seed data>

---

## 11. Risks and Mitigations

<Risks introduced by the new plan, distinct from the spec's roadblocks (which were about the existing code). Pair each risk with a concrete mitigation.>

- **Risk:** <what could go wrong>
  - **Mitigation:** <how we'll catch it or prevent it>
- **Risk:**
  - **Mitigation:**

---

## 12. Acceptance Criteria

<Checklist version of the spec's success criteria, at implementation granularity. Each item should be something you can tick off when verifying the work is done.>

- [ ] <e.g. "POST /orders returns 200 with `{id, status}` for valid input">
- [ ] <e.g. "Cache hit rate exceeds 80% in load test scenario X">
- [ ] <e.g. "All existing tests still pass">
- [ ] <e.g. "No new lint or type errors">

---

## 13. Open Questions for the User

<Exactly 5 questions about implementation details. Different in character from the spec's questions — those were about scope and behavior; these are about how to build it. Order most consequential first.>

1. **<Category — e.g. "Concurrency">:** <Specific question with options if obvious>
2. **<Category>:**
3. **<Category>:**
4. **<Category>:**
5. **<Category>:**

---

*Once these questions are answered, this plan is ready to feed into `implement-plan`.*
