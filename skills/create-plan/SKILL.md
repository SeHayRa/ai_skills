---
name: create-plan
description: Produce a detailed implementation plan from an existing spec by filling in the project's plan-template.md and saving the result to .plans/plan.md. Use this skill whenever the user asks to "create a plan", "plan out", "design", or "break down" a feature that already has a spec, or whenever they reference an existing .specs/spec.md and want the next step. This is the second stage of a three-stage spec → plan → implement workflow, so trigger it when the user has a spec ready (with the open questions answered) and wants concrete implementation detail before writing code. Also trigger if the user describes a development task and says they already wrote or filled out a spec.
---

# Create Plan

You are turning an existing spec into a concrete implementation plan. The plan is the second stage of a three-stage workflow:

1. create-spec (produces `.specs/spec.md`)
2. **create-plan** ← you are here (consumes `.specs/spec.md`, produces `.plans/plan.md`)
3. implement-plan (consumes `.plans/plan.md`)

The plan exists so that a fresh agent can pick it up, read it once, and start writing code without having to re-derive architectural decisions. That's the bar — if the plan leaves an architectural question unanswered, the implementing agent will guess, and the guess will probably be wrong.

## What a plan is and isn't

A plan **is** the bridge from intent to code. It specifies the shape of the work: which files change, what new files exist, what the interfaces between components look like, what the data model is, what decisions have already been made. Code snippets are encouraged where they reduce ambiguity — function signatures, type definitions, schemas, example payloads, migration SQL. These make the plan precise.

A plan **is not** the implementation itself. No full function bodies. No "here is the working code." If you find yourself writing a complete implementation, stop — that's the implement-plan stage. The plan describes the seams and the shape; the implementation fills them in.

The plan is also not a re-pitch of the spec. The spec already covers *what* and *why*. The plan covers *how*. Don't restate goals at length — link to the spec and move on.

## Inputs and outputs

- **Input:** `.specs/spec.md` in the project root, with the spec's open questions already answered. If those questions are still unanswered placeholders, stop and tell the user — the plan will be guesswork without them.
- **Template:** `.agents/templates/plan-template.md` in the project root. Read it before writing anything. If it's missing, fall back to the copy bundled with this skill at `assets/plan-template.md`, copy it into place at `.agents/templates/plan-template.md`, and proceed.
- **Output:** `.plans/plan.md` in the project root. Create the `.plans/` directory if needed. If `.plans/plan.md` already exists, ask the user before overwriting.

## Workflow

### 1. Read the spec

Open `.specs/spec.md` and read it end to end. Pay specific attention to:
- The **answers** to the five open questions — these are decisions that constrain the plan.
- The **files likely impacted** list — your starting point for what to modify and create.
- The **roadblocks** — concrete obstacles the plan must address.
- The **out of scope** items — don't plan for things the spec explicitly excludes.

If any of the spec's open questions are still unanswered (placeholders, "TBD", or empty), stop. Tell the user which questions need answers before the plan can be drafted. Don't guess on the user's behalf — the whole point of those questions was that the answers materially change the plan.

### 2. Read the template

Open `.agents/templates/plan-template.md`. The template defines the section order and headings — match it exactly. Don't reorder, rename, or invent sections. Replace all `<…>` placeholders.

### 3. Explore the impacted code

The spec listed files that will probably be touched. Read them now — actually open them, don't infer from filenames. For each file you plan to modify, you need to know:
- What's already there (functions, classes, exports).
- Who calls it (a quick grep for the symbols you're changing).
- What tests cover it (so you know what might break).
- What patterns/conventions the surrounding code uses (naming, error handling, logging).

This grounding is what separates a real plan from a generic one. The plan should reference *specific* functions, classes, and line ranges where useful — not "the orders module" but "the `submit_order` function in `api/orders.py:42`".

While exploring, also note:
- **Existing dependencies you can reuse** — this matters for the dependency rule below.
- **Existing patterns to follow** — if the codebase uses a particular error type, naming convention, or testing approach, the plan should call that out so the implementation matches.

### 4. Fill in the template

Work through the template top to bottom. Section-by-section guidance:

- **Overview & spec link** — Short. Mirror the spec's goal but phrased as an implementation outcome. Note how the spec's open questions were answered.

- **Architecture overview** — One paragraph describing the new system shape after the work lands. An ASCII or Mermaid sketch is helpful when there are multiple components. List the 2–4 most important architectural choices as bullets.

- **Implementation phases** — Order of operations. Each phase should be independently buildable; the agent should be able to stop after any phase and have working (if incomplete) code. 2–5 phases is typical. State each phase's goal, scope, and a "done when" condition.

- **Files to modify** — For each file, name the specific function/class/section being touched and describe the change. Don't write the new code; describe it. Include any callers or related code that might be affected.

- **Files to create** — Path, purpose, contents (high-level), and a one-line justification for why this needs a new file rather than additions to an existing one.

- **Data model / schema changes** — Where code snippets really earn their keep. Include actual type definitions, SQL, JSON schemas, protobuf, or whatever fits the project. Note any migration concerns (backfills, data transformations, backward compatibility).

- **Interface contracts** — The seams between components. Function signatures, API endpoints, message formats, event shapes — written out concretely. For each, include inputs, outputs, error/edge cases, and a concrete example call. This is where ambiguity dies.

- **Dependencies** — Read the rule below carefully. Prefer existing dependencies; flag new ones loudly.

- **Lower-level design decisions** — The non-obvious calls the implementing agent shouldn't have to re-derive. Concurrency, error handling style, data ownership, performance/memory trade-offs. Frame each as decision + rationale + alternatives considered. If you skip alternatives, the implementing agent doesn't know whether you considered them or not.

- **Testing strategy** — Name the actual cases, not "write tests." "Empty input, single item, boundary at MAX, overflow" is useful; "unit tests for the cache" is not.

- **Risks and mitigations** — Distinct from the spec's roadblocks. Roadblocks are about the existing code; risks are about *this plan* ("the matching engine could deadlock if X"). Pair every risk with a concrete mitigation.

- **Acceptance criteria** — Implementation-level checklist. Each item should be tickable when verifying the work is done.

- **Open questions** — Exactly five. These are *implementation* questions, not scope questions (scope was the spec's job). Things like: "Should the cache be in-process or shared via Redis?" or "Async or sync for the notification path?" — questions that materially change how the code gets written. Most-consequential first.

### 5. Write the file

Save to `.plans/plan.md`. Create the `.plans/` directory if needed. Use the current date in the header. Don't add a preamble — the plan is the deliverable.

### 6. Report back

Briefly tell the user:
- Where the plan was saved.
- A one-sentence summary of what it covers.
- **If you added any new dependencies, call them out explicitly here** — this is the user's chance to push back before implementation starts.
- That the five open questions need answers before moving to `implement-plan`.

## The dependency rule

**Default to no new dependencies.** Before adding any package, library, or service, ask: *can this be done with what's already in the codebase?* Usually the answer is yes — the project already has an HTTP client, a JSON parser, a testing framework, a logger. Reach for those.

A new dependency is only justified when:
- The functionality is genuinely complex (cryptography, parsing a specific format, etc.) and reimplementing it would be error-prone or wasteful.
- An existing dependency in the project doesn't cover it and no reasonable workaround exists.
- The value is high relative to the cost of an additional dep (install time, attack surface, version maintenance, learning curve).

When you do add one, the plan must:
- List it in the **Dependencies → New additions** section with the specific version.
- State explicitly why an existing dep won't work.
- Note the surface area being used (one function vs. the whole library).
- Flag it in the report-back message so the user sees it before implementation.

If you're tempted to add a dependency for convenience rather than necessity — don't. The plan can note "could be simpler with library X" in the design decisions section without making the addition.

## Common pitfalls

- **Drafting a plan before the spec's questions are answered.** The whole point of those questions was that they constrain the plan. Stop and ask the user instead.
- **Restating the spec.** The plan links to the spec, doesn't duplicate it. If you find yourself rewriting the project goal at length, cut it.
- **Vague file changes.** "Update the orders module" is not a plan. "Add a `cache_key()` helper to `api/orders.py` and wire it into `get_order()` at line 47" is.
- **Skipping interface contracts.** This is the section that most reduces ambiguity for the implementer. If you leave a contract underspecified, the implementer will improvise.
- **Adding dependencies without serious justification.** See the dependency rule.
- **Writing full implementations in the plan.** Signatures and shapes are great; complete function bodies belong in the implement stage.
- **Generic risks.** "Performance could be a concern" tells the reader nothing. Name the specific risk and a concrete mitigation.
- **Filler in any section.** Empty is fine when nothing applies; padding makes the plan harder to execute against.
