---
name: update-plan
description: Revise an existing .plans/plan.md based on new information — typically the user's answers to the plan's five open questions, deviations surfaced by implement-plan, an upstream spec change that left the plan stale, or design adjustments mid-flight. Use this skill whenever the user answers plan questions, says they want to "update the plan", "revise the plan", "amend the plan", or describes a change to a plan they've already drafted. Also trigger when the plan is marked STALE due to an upstream spec change, or when implement-plan surfaces that the plan needs revision. This is the maintenance counterpart to create-plan — same workflow, same template, but operates on an existing plan rather than starting fresh.
---

# Update Plan

You are revising an existing plan at `.plans/plan.md` based on new information. The plan is the middle of a three-stage workflow (spec → plan → implement), so changes here can ripple in both directions — they may answer questions raised by the spec, or invalidate work-in-progress against the previous version of the plan.

## What this skill is and isn't

This skill **is** focused maintenance of an existing plan. It handles:

- The user answering the plan's five open questions.
- Deviations surfaced by implement-plan ("the plan didn't list this file", "we need an extra dependency", "the interface contract is wrong for case X").
- Upstream spec changes that left the plan stale.
- Mid-implementation design adjustments ("let's switch from sync to async after all").

This skill **is not** a rewrite. The existing plan is the starting point — preserve what's still accurate, change what's actually changing. If you're revising every section, you've drifted into rewrite mode.

This skill **does not** keep a revision log in the file. The output is a cleanly-rewritten plan that reads as if it were authored that way originally.

## Inputs and outputs

- **Existing plan:** `.plans/plan.md`. Must exist — if it doesn't, this is a create-plan situation.
- **Spec:** `.specs/spec.md`. Read this for context, especially if the plan's status line says STALE (which means the spec was updated after the plan was written).
- **New info:** comes in two flavors, both valid:
  - **Chat answers**: the user provides answers or change requests conversationally.
  - **Inline edits**: the user has edited `.plans/plan.md` directly.
  - You may see both. Reconcile both before writing.
- **Template:** `.agents/templates/plan-template.md`. The structure hasn't changed; you're filling the same template with revised content.
- **Output:** `.plans/plan.md`, overwritten cleanly. Also marks the downstream implementation stale (see "Cascading staleness" below) if there's evidence implementation has begun.

## Workflow

### 1. Check for upstream staleness first

Open the plan and look at the status line in the header. If it says `STALE — upstream spec was updated`, the spec changed after this plan was written, and the plan may need substantial revision — not just the targeted edits the user is asking for.

In that case, before doing anything else:
1. Read `.specs/spec.md` end to end.
2. Identify what changed from the perspective of the plan — new files in scope, removed scope, changed tech choices, different success criteria.
3. Tell the user: "The spec was updated since this plan was written. Here's what I see as the impact on the plan: [list]. Want me to incorporate those changes along with your other revisions, or address them separately?"

Don't silently reconcile spec changes — they may be larger than the user is mentally tracking. Get explicit acknowledgment.

If the status line isn't STALE, proceed normally.

### 2. Read the existing plan

Read it end to end. You need to know what it currently says before you can know what's changing. Pay specific attention to:

- The five open questions — are they still placeholders, or did the user answer them inline?
- Any sections the user may have annotated or edited.
- The Dependencies section — this is the most common place for deviation, and the rule against unjustified new dependencies still applies.

### 3. Read the spec for grounding

Open `.specs/spec.md` and refresh on the spec's constraints — especially the answered open questions, success criteria, and out-of-scope items. The plan's revisions still have to honor what the spec says.

If you handled step 1 because the spec was updated, you've already done this — skip it.

### 4. Gather the new information

Combine chat-provided info with inline edits. If they conflict, flag it and ask which the user intends.

If the user answered the plan's open questions, bake the answers into the relevant sections (interface contracts, design decisions, dependencies, etc.) rather than leaving them as "Q1 (answered): X" in the Open Questions section.

If the new information came from implement-plan surfacing a deviation, treat the deviation report as authoritative: the implementer found something real, and the plan was incomplete. Update the plan to reflect what's actually needed.

### 5. Re-explore the relevant code if needed

If the revision changes which files are touched, adds a new file, or changes an interface contract, do a quick read of the relevant code to ground the revised plan in reality — same approach as create-plan, just narrower in scope.

Skip this if the revision is purely about design decisions or open-question answers that don't touch the file layout or interfaces.

### 6. Revise the plan

Confirm the template structure at `.agents/templates/plan-template.md` (it shouldn't have changed). Then work through the existing plan section by section, deciding **keep, edit, or rewrite** for each. Most sections in most revisions are *keep*.

Specific patterns:

- **Overview, architecture overview** — usually stable. Revise only if the answered questions or deviations changed the high-level shape.
- **Resolved questions from spec (in Overview)** — update if upstream spec changed.
- **Implementation phases** — often stable, but a deviation might split or merge a phase, or change the order. Revise if so.
- **Files to modify / create** — frequently changes. Add files surfaced by deviation reports; remove files no longer needed; update the per-file change notes if scope shifted.
- **Data model / interface contracts** — revise if the user's answers or deviations affected the seams. This is where most plan revisions land.
- **Dependencies** — if a new dependency really is needed (justified by deviation, not convenience), add it under "New additions" with the same justification rigor as create-plan. The "no new dependencies unless necessary" rule still applies during updates.
- **Lower-level design decisions** — update if the user's answers or deviations reversed or refined a previous decision. When a decision changes, you can keep it in the alternatives-considered list of the new decision so the trail is visible without a revision log.
- **Testing strategy and acceptance criteria** — revise if scope changed.
- **Open questions** — almost always changes. If the user answered all five, write a new set if any remain open, or drop the section to "All open questions resolved."

The final output should read as a fresh, coherent plan. Don't write "previously the plan said X, now Y" — just write Y.

### 7. Write the file (clean overwrite)

Overwrite `.plans/plan.md` with the revised content. Use today's date. No revision log. Status line goes back to `DRAFT` (if questions remain open) or `READY` (if all questions are answered).

### 8. Cascade staleness to the implementation

If there's evidence implementation has begun — files mentioned in "Files to Modify" or "Files to Create" have actually been changed in the repo since the plan was last written — flag this to the user explicitly:

> "Implementation appears to be in progress (I see changes to `path/to/file.py` and `path/to/other.py`). The revised plan may invalidate some of that work. Want me to summarize the diff so you can decide what to re-do?"

Don't try to "un-implement" or modify code. Just surface the situation. The user decides what to do.

To check for implementation progress, you can:
- Look at `git log` or `git status` for changes to files named in the plan.
- Look at the files themselves to see if they match the plan's expected state.

If implementation hasn't started yet (no relevant changes), skip this step.

### 9. Report back

Briefly tell the user:
- Where the plan was saved.
- A one-line summary of what changed.
- If you marked anything stale or flagged in-progress implementation, say so.
- **If you added any new dependencies, call them out explicitly** — same as create-plan, this is the user's chance to push back.
- If any plan questions remain open, remind the user those need answers before re-running implement-plan.

## The dependency rule still applies

Same rule as create-plan: **default to no new dependencies.** A revision is not an excuse to slip new packages in. If a deviation surfaced the need for a new dep, it still has to clear the same bar: justified, scoped, and explicitly flagged. If you're adding one because the revision happens to make it convenient, don't.

## Common pitfalls

- **Ignoring an upstream STALE flag.** If the spec changed, the plan may need bigger revisions than the user is asking for. Surface this before quietly reconciling.
- **Rewriting the whole plan.** Targeted change. If you're touching every section, pause.
- **Leaving answered questions in the Open Questions section.** Bake answers into the relevant sections; the Open Questions section is only for things still open.
- **Silently picking between conflicting inputs.** Ask.
- **Slipping in new dependencies during a revision.** The dependency rule applies the same way it does during create-plan.
- **Modifying or "un-implementing" code in response to plan changes.** That's implement-plan's job (and the user's call). This skill stops at the plan file and the staleness flag.
- **Adding a revision log.** Don't. Clean overwrite.
