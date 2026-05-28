# ai_skills

A collection of [AI Skills] I use across projects.

## Skills

### `create-spec` → `create-plan` → `implement-plan`

A three-stage workflow for software development. Each stage produces an artifact that feeds the next, so context can be refreshed (or handed to a fresh agent) between stages without losing the thread.

1. **create-spec** — Turns a feature request into a high-level natural-language spec at `.specs/spec.md`. No code, just goals, context, data sources, foreseeable roadblocks, and five questions for the user to answer.
2. **create-plan** — Turns the answered spec into a concrete implementation plan at `.plans/plan.md`. Architecture, phases, interface contracts, file-by-file changes, design decisions, and five more questions on implementation details.
3. **implement-plan** — Executes the answered plan. Phase by phase, pausing for confirmation between phases. Tests run at the end.

Each stage stops if the previous stage's open questions are still unanswered — guesses propagate.

### `frontend-design`

Anthropic's frontend-design skill. Used as-is.

## Layout

Skills live under `.agents/skills/` and shared templates under `.agents/templates/`:

```
.agents/
├── skills/
│   ├── create-spec/
│   ├── create-plan/
│   ├── implement-plan/
│   └── frontend-design/
└── templates/
    ├── spec-template.md
    └── plan-template.md
```

The spec/plan skills bundle their templates and self-populate `.agents/templates/` on first run, so dropping the folder into a new project is enough to bootstrap.

## Adding more

More skills to come. The spec → plan → implement loop is the spine; everything else slots around it.
