---
name: design-docs
description: >
  Use this skill at the START of any new project or feature before writing code.
  Triggers when the user says things like "I want to build X", "starting a new project",
  "new flow", "new tool", or "let's design X". Also triggers when the user asks to
  document an existing project retroactively.

  This skill grills the user one question at a time to capture design decisions,
  then produces the full docs/ folder structure: overview.md, data_model.md, a
  Mermaid flow diagram, and ADR files — matching the python-boilerplate repo structure.

  Always use this skill before starting to code. The output is the pre-coding
  documentation that keeps the user in ownership of their own project.
---

# design-docs

Produces pre-coding documentation for a python project by grilling the user
one question at a time, then writing the docs/ folder structure.

## When to use

- User is starting a new project or feature
- User wants to document an existing project retroactively (read the code first,
  then grill for what the code can't answer)
- User says `/design-docs` explicitly

## Output

```
docs/
├── decisions/
│   ├── README.md
│   ├── TEMPLATE.md
│   └── NNN_<decision>.md      (one per decision surfaced during grilling)
└── design/
    ├── overview.md
    ├── data_model.md
    └── flows/
        ├── README.md
        └── main_flow.md       (Mermaid diagram)
```

See `assets/` for the file templates.

---

## Step 1 — Read the codebase (if it exists)

If the user has shared a repo or files, read them before asking anything.
Extract what you can already answer:
- What entities exist (from models, dataclasses, DB columns)
- What flows and tasks exist and their schedules
- What environment/config variables are defined
- Any TODOs or stubs that indicate incomplete work

**Do not ask the user for things the code already answers.**

---

## Step 2 — Grill

Ask one question at a time. Wait for the answer before asking the next.
Offer a suggested answer where you can — the user confirms or corrects.

Work through these areas in order, skipping what the code already answered:

### Problem
- What problem does this solve? (one sentence)
- Who experiences this problem?

### Users and stories
- Who are the actors? (who submits, who receives, who monitors)
- For each actor: what do they want, and why? (user story format)

### Entities
- What are the "things" in this system?
- For each entity: what are its key fields? what is its lifecycle?

### Data flow
- Where does data come from?
- Where does it go?
- What external systems are involved? (APIs, databases, email, queues)

### Flow design
- What are the distinct phases? (e.g. pre-notify, process, post-notify, report)
- Should these be one flow or multiple? Why?
- What schedules do they run on?

### Boundaries
- What is explicitly out of scope?
- What depends on something not yet built?

### Open questions
- What is not yet decided?
- What needs to be confirmed with stakeholders before go-live?

### Decisions
During grilling, whenever the user makes a non-obvious choice — anything where
they picked one approach over another — flag it:
> "That sounds like a decision worth recording. I'll add it as an ADR."

A decision worth recording is:
- Hard to reverse later
- Non-obvious without context
- The result of a real tradeoff

Do NOT record every implementation detail. Most sessions produce 3–6 ADRs.

---

## Step 3 — Write the docs

Once grilling is complete, write the files directly into `docs/` in the
user's project using the Write tool. Do not present them as markdown code
blocks for the user to copy — write them straight to disk.

### If docs/ already exists

Don't regenerate everything from scratch. Diff the existing docs against what
the codebase and grilling actually show:
- Update `overview.md` and `data_model.md` in place where they've drifted
  from the code (stale fields, missing flows, wrong types).
- Update `flows/main_flow.md` if the process has changed.
- For new decisions surfaced during grilling, continue ADR numbering from the
  highest existing number in `decisions/` — don't restart at 001.
- Leave existing ADRs alone unless one has been explicitly superseded (then
  add a new ADR and mark the old one's Status as `Superseded by [link]`).

### overview.md
Use the template in `assets/overview_template.md`.
Fill every section. Mark genuinely unresolved items as open questions.
Do not leave template placeholder text.

### data_model.md
Use the template in `assets/data_model_template.md`.
Include all entities, fields, and their types.
Note which fields are runtime-enriched vs persisted.
Note source and destination systems.

### flows/main_flow.md
A Mermaid `flowchart TD` diagram showing the end-to-end data and process flow.

Shape conventions (also documented in flows/README.md):
- `([Label])` — person or external actor
- `[Label]` — process or step
- `{Label}` — decision point
- `[(Label)]` — database or data store
- `(Label)` — start or end
- `[/Label/]` — input or output

Render edge labels as `-->|description|` to show what moves between steps.

### decisions/NNN_<title>.md
One file per decision surfaced during grilling.
Use the template in `assets/decisions/TEMPLATE.md`.
Number sequentially from 001 (or from the next free number — see
"If docs/ already exists" above).

### Static files (copy as-is, do not edit)
- `assets/decisions/README.md` → `docs/decisions/README.md`
- `assets/decisions/TEMPLATE.md` → `docs/decisions/TEMPLATE.md`
- `assets/design/flows/README.md` → `docs/design/flows/README.md`

---

## Step 4 — Confirm

After presenting all files, ask:
> "Anything missing or wrong? Any decisions we didn't capture?"

Revise on feedback. When the user is satisfied, remind them:
> "Keep `docs/decisions/` open while you code. Any time you think 'I could do X or Y' — that's a decision. Write it down before you continue."
