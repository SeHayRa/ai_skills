---
name: implement-plan
description: Execute an existing implementation plan by reading .plans/plan.md and making the actual code changes it specifies — modifying files, creating new ones, and adhering to the design decisions already made. Use this skill whenever the user asks to "implement the plan", "build it", "execute the plan", "do the implementation", or references an existing .plans/plan.md and wants the code written. This is the third and final stage of a three-stage spec → plan → implement workflow, so trigger it when the user has a plan ready (with the open questions answered) and wants the changes applied to the codebase. Also trigger if the user describes a development task and mentions they already have a plan written.
---

# Implement Plan

You are executing a plan that already exists. The thinking has been done — your job is to make the code match. This is the third stage of a three-stage workflow:

1. create-spec (produced `.specs/spec.md`)
2. create-plan (produced `.plans/plan.md`)
3. **implement-plan** ← you are here (consumes `.plans/plan.md`, produces real code changes)

There is no document to produce. The output is working code that matches the plan.

## What this skill is and isn't

This skill **is** disciplined execution. Read the plan, follow it phase by phase, change the files it names, create the files it specifies, write code that honors the interface contracts and design decisions already documented. Match the codebase's existing patterns.

This skill **is not** the place to re-litigate design decisions. The plan already chose between alternatives — it says so, in its "Lower-level design decisions" section. If you disagree with a choice, the right move is to surface it to the user, not to silently implement a different design. Quietly going off-plan defeats the entire point of having a plan.

## Inputs and outputs

- **Input:** `.plans/plan.md` in the project root, with all five open questions answered. If those answers are still placeholders or empty, stop and tell the user — same logic as the previous stages.
- **Output:** code changes in the repo. No `.md` file. No deliverable to save.

## Workflow

### 1. Read the plan, fully

First, check the status line in the header. If it says STALE — upstream spec was updated, the spec changed after the plan was written and the plan may be out of date. Stop. Tell the user: "The plan is marked stale because the spec was updated after it was written. Run update-plan first, or confirm you want me to proceed against the current plan anyway." Wait for the user's explicit instruction before continuing.
If the status line is DRAFT or READY, proceed.

Open `.plans/plan.md` and read it end to end before touching any code. Specifically internalize:

- **The answers to the plan's open questions** — these are constraints on the implementation.
- **The architecture overview and design decisions** — these tell you the shape to build toward.
- **The phases** — the order to work in.
- **Files to modify and create** — your full scope. Anything outside this list is out of scope.
- **Interface contracts** — the seams you must match. Function signatures, schemas, payloads.
- **Dependencies** — the only packages you may add are the ones listed under "New additions." Don't add anything else.
- **Testing strategy** — the cases you'll verify at the end.
- **Acceptance criteria** — your checklist for "done."

If the plan's open questions are still unanswered, stop. Don't guess. Tell the user which questions need answers before implementation can start.

### 2. Read the relevant code

Before changing a file, read it. You need to know what's actually there — current function bodies, imports, tests, callers — not what you assume is there from the plan's description. The plan is precise but it isn't a substitute for the actual code.

For each file the plan says you'll modify, also do a quick check of who calls the symbols you're changing. The plan should have flagged this, but verify — breaking unrelated callers is the most common way implementations go sideways.

### 3. Execute phase by phase, pausing between phases

The plan has implementation phases for a reason: each one is a coherent unit of work that leaves the code in a usable state. Work through them in order.

**For each phase:**

1. State what you're about to do — name the phase and list the files you'll touch in this phase.
2. Make the changes. Modify and create files according to the plan's specifications for this phase.
3. Confirm the phase is complete against the plan's "done when" condition.
4. **Pause.** Report what was done in the phase — files changed, files created, anything notable — and explicitly ask the user before starting the next phase. Wait for their go-ahead.

This pause gives the user a chance to catch problems early. If they say "looks good, continue," move on. If they have feedback, address it before proceeding.

Don't run ahead to the next phase even if it seems trivial. The point of the pause is for the user to check the trajectory, not for you to optimize throughput.

### 4. Stay inside the plan

If during implementation you discover that the plan is incomplete — a file needs to change that wasn't listed, an interface needs an extra parameter, a dependency is actually required that the plan didn't anticipate — **stop and surface it**. Don't silently expand scope. The right response looks like:

> "While implementing Phase 2, I found that `auth/middleware.py` also needs to change because it calls the function we're modifying. The plan didn't list this file. Should I (a) include it in this implementation, (b) leave it as-is and accept the regression, or (c) revisit the plan?"

This is also true for dependencies. The plan's Dependencies section is exhaustive — those are the *only* packages you may add. If you need another one, ask first.

Same goes for design decisions. If you find yourself wanting to do something different from what the plan specifies, that's a signal to pause and discuss, not to deviate.

### 5. Match the codebase's conventions

The plan likely called out specific patterns to follow, but reinforce this in practice. Match the surrounding code's:

- Naming conventions (snake_case vs camelCase, prefixes, etc.)
- Error handling style (exceptions vs result types, error wrapping conventions)
- Logging style and verbosity
- Import organization
- Comment style (or absence of comments)
- Test structure and naming

When the plan and the surrounding code conflict, the plan wins for *what* to build but the code wins for *how to style it*. If the plan said "add a method `getUserOrders`" and the codebase uses snake_case, write `get_user_orders` and flag the discrepancy.

### 6. After the final phase: run the tests

Once all phases are complete and the user has approved the final phase, run the test suite. The acceptance criteria in the plan tell you what to verify.

- Run unit and integration tests. Report results.
- Run linters and type checkers if the project uses them.
- Walk through the acceptance criteria checklist explicitly — for each item, state whether it's met and how you verified it.

If tests fail or acceptance criteria aren't met, report what failed and what you think is needed to fix it. Don't silently patch failing tests by lowering the bar — if a test fails, either the implementation is wrong or the test expectation needs to change, and that's a decision to surface, not to make alone.

### 7. Final report

After tests pass and acceptance criteria are checked off, give a short summary:

- Phases completed.
- Files modified and created (full list).
- Dependencies added (should match the plan's list exactly).
- Test and lint results.
- Any deviations from the plan that came up, and how they were resolved.
- Anything the user should verify manually (the plan's "Manual / exploratory" testing items, if any).

## Common pitfalls

- **Implementing before reading the whole plan.** Skipping ahead to "files to modify" without reading design decisions leads to code that contradicts the plan's intent.
- **Silently expanding scope.** Touching files not listed in the plan, or adding dependencies not in the Dependencies section. If you need to do this, ask.
- **Re-litigating decisions.** The plan already chose. If you disagree, raise it; don't quietly do something else.
- **Skipping the inter-phase pause.** The pause is a feature. Use it.
- **Ignoring codebase conventions.** A plan-correct implementation that doesn't match local style still creates friction for everyone who reads it after.
- **Burying test failures.** If tests fail, the right move is to report and discuss — not to weaken the test until it passes.
- **Writing a plan-update doc.** This skill doesn't produce a `.md` file. The output is code.
