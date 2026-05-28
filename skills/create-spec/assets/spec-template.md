# Spec: <FEATURE_OR_PROJECT_NAME>

> Status: DRAFT — pending answers to open questions
> Created: <YYYY-MM-DD>
> Source prompt: <one-line summary of what the user asked for>

---

## 1. Project Goal

<2–5 sentences describing, in plain language, what we are trying to accomplish and why it matters. State the outcome a successful implementation produces, not how to build it.>

### Success criteria

- <Observable, testable condition #1 — e.g. "Users can submit an order via the new endpoint and receive a confirmation ID.">
- <Observable condition #2>
- <Observable condition #3>

### Out of scope

- <Thing this spec is explicitly NOT trying to do. Out-of-scope items prevent scope creep during planning.>
- <Another non-goal>

---

## 2. Background and Supporting Information

### What the user asked for

<Paraphrase the user's original prompt faithfully. Preserve any specific terms, names, or constraints they used.>

### Expanded context

<Fill in the picture around the prompt: domain background, why this is being built now, who the users are, how this fits into the existing system. Use what you learned from exploring the codebase. If the user mentioned existing files, modules, or features, describe their current role here.>

### Assumptions

- <Assumption being made — e.g. "Authentication is already handled upstream by the API gateway.">
- <Assumption #2>

---

## 3. Data Sources

<List every input, output, and persistent store this work will touch. For each, note where it lives, what shape it has, and whether it already exists.>

| Source | Type | Status | Notes |
|---|---|---|---|
| <e.g. `users` table> | <Postgres / file / API / stream> | <Exists / new> | <Schema notes, ownership, access pattern> |

If no data sources are involved, say so explicitly and explain why.

---

## 4. Possible Technologies

<List candidate languages, frameworks, libraries, databases, services. Note which already exist in the codebase vs. which would be new additions. Flag anything that would be a meaningful new dependency.>

- **Language(s):** <e.g. Python 3.11 — matches existing service>
- **Framework(s):** <e.g. FastAPI — already in use>
- **Database / storage:** <e.g. Postgres via SQLAlchemy>
- **External services:** <e.g. Stripe API for payments>
- **New dependencies under consideration:** <list, with a one-line justification each>

---

## 5. Foreseeable Roadblocks

<Based on the current state of the codebase, list things that are likely to cause friction during implementation. Each item should name a real concern, not a generic one.>

- **<Roadblock name>:** <What it is, where in the code it surfaces, why it's a problem for this work.>
- **<Roadblock name>:** <…>

If the codebase is in good shape and nothing stands out, say so.

---

## 6. Files Likely Impacted

<List specific paths the implementation will probably touch. Mark each as create / modify / delete. If a file's role isn't obvious, add a short note.>

- `<path/to/file>` — modify — <why>
- `<path/to/new/file>` — create — <purpose>
- `<path/to/file>` — read-only reference — <why it matters>

If the change is large or exploratory, group files by area (e.g. "API layer," "data layer," "tests") instead of listing every one.

---

## 7. Open Questions for the User

<Exactly 5 questions whose answers materially change the plan. Don't ask questions you can answer yourself from the codebase or the prompt. Order them so the most consequential question is first.>

1. **<Question category — e.g. "Auth model">:** <Specific question. Include the options you're choosing between if there are obvious ones.>
2. **<Category>:** <Question>
3. **<Category>:** <Question>
4. **<Category>:** <Question>
5. **<Category>:** <Question>

---

*Once these questions are answered, this spec is ready to feed into `create-plan`.*
