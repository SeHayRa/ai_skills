---
name: create-spec
description: Produce a high-level, natural-language spec document for a software development task by filling in the project's spec-template.md and saving the result to .specs/spec.md. Use this skill whenever the user asks to "create a spec", "write a spec", "spec out" a feature, "scope" a piece of work, or describes a software change they want planned before any code is written. This is the first step of a three-stage spec → plan → implement workflow, so trigger it for any prompt that sounds like the start of a new feature, refactor, or non-trivial change — even when the user doesn't say the word "spec" — as long as they haven't already asked you to jump to planning or coding.
---

# Create Spec

You are turning a user's development request into a written spec — a high-level, natural-language document that captures *what* is being built and *why*, not *how*. The spec is the first stage of a three-stage workflow:

1. **create-spec** ← you are here
2. create-plan (consumes `.specs/spec.md`)
3. implement-plan (consumes the plan)

The spec must be useful as input to the next stage, which means it has to be concrete enough to plan against but free of implementation detail.

## What a spec is and isn't

A spec **is** a shared understanding of the work — goals, context, constraints, unknowns. Someone reading it should be able to describe the project back to you in their own words.

A spec **is not** a design doc or an implementation plan. No pseudocode. No code blocks except for file paths and identifiers. No "step 1, step 2, step 3" sequences describing how to build the thing. If you find yourself writing implementation steps, stop — that belongs in the plan stage.

The goal is the *minimum information needed* for someone to start planning. Resist the urge to overspecify.

## Inputs and outputs

- **Input:** the user's prompt describing what they want built, plus the surrounding codebase.
- **Template:** `.agents/templates/spec-template.md` in the project root. Read it before writing anything — it defines the exact structure to fill in. If it's missing, fall back to the copy bundled with this skill at `assets/spec-template.md`, copy it into place at `.agents/templates/spec-template.md`, and proceed.
- **Output:** `.specs/spec.md` in the project root. Create the `.specs/` directory if it doesn't exist. If `.specs/spec.md` already exists, ask the user before overwriting — they may want to version it (e.g. `spec-<feature>.md`).

## Workflow

### 1. Read the template

Open `.agents/templates/spec-template.md` first. The template defines the section order, headings, and any project-specific conventions. Match its structure exactly — don't reorder sections, rename headings, or invent new ones. If the template has placeholder markers (like `<…>`), replace them; don't leave them in the final output.

### 2. Explore the codebase

You're being asked to write a spec for a *specific* codebase, so ground your writing in what's actually there. Spend a few minutes orienting:

- List the top-level directory structure to understand the project layout.
- Read the README or any docs that describe the project's purpose and stack.
- Identify the language(s), framework(s), and key dependencies (look at `package.json`, `pyproject.toml`, `Cargo.toml`, `go.mod`, etc.).
- Find the files most relevant to the user's request and skim them. If the user mentioned a feature, module, or file by name, read it.

This exploration directly feeds three sections of the spec: **Possible Technologies** (what's already in use vs. what would be new), **Foreseeable Roadblocks** (real friction points in the current code, not generic ones), and **Files Likely Impacted** (actual paths, not guesses). A spec written without looking at the code will be vague in exactly the places where vagueness is least helpful.

Don't try to read the whole codebase. Aim for "enough to make grounded claims," then stop.

### 3. Fill in the template

Work through the template top to bottom, replacing each placeholder. Some guidance per section:

- **Project Goal** — Plain language, no jargon dump. Say what success looks like for a user or operator of the system. The "Success criteria" should be observable (you could write a test for them); the "Out of scope" list is where you preempt scope creep.

- **Background and Supporting Information** — Two jobs: (a) faithfully paraphrase what the user asked for, preserving their terminology, and (b) expand on it using context from the codebase. If the user said "add caching to the orders endpoint," your expansion should say which orders endpoint, what currently happens on a request, and why caching is interesting here. Assumptions go at the bottom — be explicit about anything you're taking for granted.

- **Data Sources** — Every input, output, and persistent store the work touches. Use the table format from the template. If genuinely no data is involved (e.g. a pure refactor), say so and move on — don't pad.

- **Possible Technologies** — Distinguish *already in the codebase* from *would be a new addition*. New dependencies are friction; flag them clearly with a one-line justification. Don't list every transitive dependency — just the ones relevant to the work.

- **Foreseeable Roadblocks** — This is where codebase exploration pays off. Name *specific* concerns: a tightly coupled module, an old version of a library, a missing test harness, an unclear ownership boundary. Generic roadblocks ("performance might be an issue") aren't useful. If nothing stands out, say so honestly — a clean codebase is a real finding.

- **Files Likely Impacted** — Real paths from the codebase, marked create / modify / delete. For large or exploratory changes, group by area rather than enumerating every file. It's fine to be approximate; the plan stage will refine this.

- **Open Questions for the User** — Exactly five. These are the highest-leverage things you genuinely don't know and that *materially change the plan*. Order them so question 1 is the most consequential. Don't ask questions you could have answered yourself from the codebase. Don't ask trivia ("what should we name the function?"). Good questions are about scope, trade-offs, constraints, and user-facing behavior. Where there are obvious options, include them inline so the user can answer quickly.

### 4. Write the file

Save the filled-in spec to `.specs/spec.md`. Create the `.specs/` directory if needed. Use the current date in the header. Don't add anything that isn't in the template structure — no preamble, no "here is your spec" wrapper.

### 5. Report back

After writing the file, tell the user:
- Where the spec was saved.
- A one-sentence summary of what it covers.
- That the five open questions at the bottom need answers before moving to `create-plan`.

Keep this report short. The spec itself is the deliverable; don't restate it in chat.

## Common pitfalls

- **Writing code or pseudocode.** Specs are natural language. If a section is tempting you to write a function signature, you're in the wrong stage.
- **Generic roadblocks.** "Testing might be complex" tells the reader nothing. "The `OrderProcessor` class has no unit tests and mocks the database in three different ways across the file" is a real roadblock.
- **Filler in Data Sources or Files Impacted.** Empty is fine. Padding makes the spec harder to plan against.
- **Six questions, or three.** The template asks for exactly five — that's a forcing function to surface the most important unknowns, not the only ones. If you have more, pick the top five and trust that the rest will surface during planning.
- **Skipping codebase exploration.** A spec written purely from the prompt will be ungrounded in the parts the user most needs grounded.
- **Overwriting an existing spec without asking.** Specs are intentional artifacts. Confirm before clobbering.
