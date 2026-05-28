---
name: update-spec
description: Revise an existing .specs/spec.md based on new information — typically the user's answers to the spec's five open questions, but also scope changes, new constraints, or corrections surfaced during planning or implementation. Use this skill whenever the user answers spec questions, says they want to "update the spec", "revise the spec", "tweak the spec", or describes a change to a project they've already specced. Also trigger when create-plan or implement-plan surfaces that the spec needs revision. This is the maintenance counterpart to create-spec — same workflow, same template, but operates on an existing spec rather than starting fresh.
---

# Update Spec

You are revising an existing spec at `.specs/spec.md` based on new information. The spec is the top of a three-stage workflow (spec → plan → implement), so changes here ripple downstream — the plan and any in-progress implementation may need to be re-examined after this skill runs.

## What this skill is and isn't

This skill **is** focused maintenance of an existing spec. It handles:

- The user answering the spec's five open questions and wanting those baked in.
- The user revising scope ("also add Y", "actually Z is out of scope").
- The user correcting facts that turned out to be wrong.
- Downstream skills (create-plan, implement-plan) surfacing that the spec needs revision.

This skill **is not** a rewrite. The existing spec is the starting point — preserve everything that's still accurate and only change what's actually changing. A revision that touches every section is a signal you've drifted into "rewrite mode" — pause and reconsider.

This skill **does not** keep a revision log in the file. The output is a cleanly-rewritten spec that reads as if it were authored that way originally. (Git history is the revision log.)

## Inputs and outputs

- **Existing spec:** `.specs/spec.md`. Must exist — if it doesn't, this is a create-spec situation, not an update.
- **New info:** comes in two flavors, and both are valid:
  - **Chat answers**: the user provides answers conversationally ("Q1 is X, Q2 is Y, also we should drop the caching thing").
  - **Inline edits**: the user has edited `.specs/spec.md` directly — typically filling in answers under each question, or adding notes to other sections.
  - You may see both at once. Reconcile both before writing.
- **Template:** `.agents/templates/spec-template.md`. The structure of the spec hasn't changed; you're filling the same template with revised content.
- **Output:** `.specs/spec.md`, overwritten cleanly. Also marks the downstream plan stale if one exists (see "Cascading staleness" below).

## Workflow

### 1. Read the existing spec

Open `.specs/spec.md` and read it end to end. You need to know what it currently says before you can know what's changing.

Look specifically at:
- The five open questions — are they still placeholders, or did the user fill in answers inline?
- Any sections the user may have annotated, edited, or added notes to.
- Any inconsistencies that suggest the user started editing but didn't finish.

### 2. Gather the new information

Combine chat-provided info with inline edits. If the user said "Q1 is X" in chat and also wrote a note in the Background section, both inputs matter.

If the inputs conflict — the chat says one thing and the inline edit says another — flag it and ask which the user intends. Don't pick silently.

If the user answered the open questions, those answers become part of the spec proper. Don't keep the questions around as "answered Q1: X" — bake the answer into the relevant section. For example, if Q1 was "should we use Redis or in-process caching?" and the answer is Redis, that fact goes into Background, Possible Technologies, or Data Sources (wherever it naturally belongs). The Open Questions section gets replaced with a *new* set of five questions if any remain genuinely open, or removed/marked resolved if everything is settled.

### 3. Re-read the codebase if needed

If the new information changes the technical picture — different tech choice, new file in scope, different assumption about how the system works — do a quick re-exploration of the relevant code. Don't re-read the whole codebase; just verify the parts that the revision touches.

Skip this step if the changes are purely about scope, goals, or clarifying user-facing behavior — those don't require new codebase grounding.

### 4. Revise the spec

Read `.agents/templates/spec-template.md` to confirm the structure (it shouldn't have changed, but a quick check costs nothing).

Then work through the existing spec section by section, deciding for each: **keep, edit, or rewrite**. Most sections in most revisions will be *keep*. The whole point of an update is targeted change.

A few specific patterns:

- **Goal and success criteria** — usually stable across updates. Only revise if the user explicitly changed what they're trying to accomplish.
- **Out of scope** — frequently changes during revisions. New items get added when something is deferred; existing items get removed if something moves back into scope.
- **Background and assumptions** — revise to incorporate what the user clarified. Assumptions often get promoted to facts (or replaced with the actual fact) when questions get answered.
- **Data sources, technologies, roadblocks, files impacted** — update where the revision affects them. Often these change as a side effect of answered questions.
- **Open questions** — this is almost always the most-changed section. If the user answered all five, write a new set of five if any remain open, or drop the section to a single line: "All open questions resolved."

The final output should read as a fresh, coherent spec — not as a diff. Someone reading the new `.specs/spec.md` for the first time should not be able to tell which parts changed from the previous version. If you find yourself writing "previously we thought X, now we think Y," delete that and just write Y.

### 5. Write the file (clean overwrite)

Overwrite `.specs/spec.md` with the revised content. Use today's date in the header. Don't add a revision log, change history, or "v2" markers. The file always looks freshly written.

### 6. Cascade staleness to the plan

If `.plans/plan.md` exists, the plan was built against the old version of the spec — it may now be out of date. Mark it stale by editing the plan's header status line:

```
> Status: STALE — upstream spec was updated on YYYY-MM-DD. Re-run create-plan or update-plan.
```

Replace whatever the previous status line said (DRAFT, READY, etc.). Don't modify the body of the plan — that's create-plan or update-plan's job. Just flip the status flag at the top so the next stage notices.

If no plan exists yet, skip this step.

### 7. Report back

Briefly tell the user:
- Where the spec was saved.
- A one-line summary of what changed (e.g. "answered all five open questions and added a new out-of-scope item").
- If you marked a downstream plan stale, say so explicitly: "Marked `.plans/plan.md` stale — it was built against the previous spec."
- If any of the spec's questions are still open (you wrote a new set of five), remind the user those need answers before re-running create-plan.

Keep this short. The spec is the deliverable.

## Common pitfalls

- **Rewriting the whole spec.** An update is targeted change. If you find yourself revising every section, you've probably drifted into rewrite mode — pause and check whether the changes really warrant that, or whether you're just restyling.
- **Leaving answered questions in the Open Questions section.** Once a question is answered, the answer belongs in the spec proper, not as "Q1 (answered): X." The Open Questions section is only for things still open.
- **Silently picking between conflicting inputs.** If chat says one thing and the inline edit says another, ask. Don't guess.
- **Forgetting to cascade staleness.** If a plan exists, marking it stale is a required step — not a courtesy. A downstream agent reading an out-of-date plan against a fresh spec will produce incoherent output.
- **Adding a revision log.** Don't. Clean overwrite. Git tracks history; the spec doesn't need to.
- **Treating an inline edit as the final word.** The user may have started editing the file but not finished. If the inline edit looks partial (placeholder still present, sentence trails off), treat it as input to reconcile, not the final answer.
