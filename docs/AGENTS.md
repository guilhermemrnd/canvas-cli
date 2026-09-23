# Product Requirements — canvas-cli

canvas-cli lets a developer describe an interface to a coding agent, in any harness, and get back a
local, Figma-like canvas of it: pages of artboards holding named frames, that they can pan, zoom
and click to inspect. Everything runs on the developer's machine: no account, no cloud, no upload.

This file is the entry point for the product documentation: the theme index, the personas and
concept glossary, and the conventions for writing and maintaining the specs. When working in
`docs/`, follow these rules exactly.

**A spec is written before the code it describes.** A new capability, or a change to one, starts as
a diff to a theme file: the job it serves, every state and its exits, the degenerate cases (zero, one,
orphaned), what already exists when the action runs, and the limits it is committed to. Where a
decision is the developer's, it is raised as an Open Question and waits.

---

## Document Index

| Theme | Code | Spec | Status |
|-------|------|------|--------|
| [Canvas Project](./themes/canvas-project.md) | `CP` | `themes/canvas-project.md` | Planned |
| [Artboard Authoring](./themes/artboard-authoring.md) | `AB` | `themes/artboard-authoring.md` | Planned |
| [Canvas Viewer](./themes/canvas-viewer.md) | `CV` | `themes/canvas-viewer.md` | Planned |
| [Agent Harness](./themes/agent-harness.md) | `AH` | `themes/agent-harness.md` | Planned |

---

## Personas

| Persona | Role |
|---------|------|
| **Developer** | The person at the terminal: describes what to design, reads the canvas, inspects it. |
| **Agent** | The model inside a harness (Devin CLI, Claude Code, others) that writes the project's files and runs the CLI on the developer's behalf. |

---

## Key Concepts

- **Canvas project**: A folder of plain files, kept in the developer's working directory, that holds one canvas. The files are the only source of truth; the viewer never holds state of its own.
- **Page**: A named area of the canvas (e.g. "Supplier portal"). Holds an ordered set of boards.
- **Board (artboard)**: One screen or flow, one HTML file, drawn at a real size. Belongs to exactly one page.
- **Frame**: A named element inside a board: one screen, modal or state. Sibling states of one surface are frames in a row.
- **Tokens**: The shared stylesheet of design values (colors, type, spacing, radii, shadows) taken from the project's own guidelines. Boards read values from it and never invent their own.
- **Viewer**: The local web page that renders the canvas and lets the developer inspect an element. Read-only.
- **Harness**: The agent runtime that loads the skill and runs the CLI.

---

# Maintenance Guidelines

## Location & Structure

Every theme is **one flat file** under `docs/themes/`, named for the theme in `kebab-case`, however
long it grows. A theme that becomes hard to read is trimmed, never split into a folder.

```
docs/
  AGENTS.md                  <- This file: index, glossary, and conventions
  _template.md               <- Canonical theme format; consult before writing
  themes/
    canvas-project.md       <- All epics for this theme in one flat file
    artboard-authoring.md
```

- Never create a file per user story or per epic.
- After adding or renaming a theme, update the Document Index above.

---

## IDs & Reference Codes

### Theme codes

Each theme is assigned a short uppercase code when created. The code is declared in the theme title line and registered in the Document Index above.

```markdown
# Theme: Canvas Project · `CP`
```

| Theme | Code |
|-------|------|
| Canvas Project | `CP` |
| Artboard Authoring | `AB` |

When creating a new theme, choose a 2–4 character code that is unique and obvious from the theme name. Register it in the Document Index above.

### Epic IDs

Epics are numbered sequentially within their theme, starting at 1. The ID is part of the epic heading:

```markdown
## Epic CP-1: Create a Canvas Project
## Epic AB-3: Frames and State Rows
```

Numbers are assigned in the order epics are created and never reassigned. If an epic is deprecated, its number is retired — do not reuse it.

### User Story IDs

Every user story is assigned a sequential `S[n]` identifier within its epic. The ID appears in the story heading:

```markdown
### User Story AB-1-S1 — Single Board
### User Story AB-1-S2 — Many Boards
```

For single-story epics, the label is optional but the `S1` ID is still required:

```markdown
### User Story CP-1-S1
```

- Story IDs start at `S1` and increment within the epic.
- Story IDs are never reassigned. If a story is removed, retire its number.

### AC IDs

AC IDs always follow the format `[CODE]-[N]-S[n].[AC]`:

```
CP-1-S1.1, CP-1-S1.2, CP-1-S1.3 ...
AB-1-S1.1, AB-1-S1.2, AB-1-S1.3
AB-1-S2.1, AB-1-S2.2, AB-1-S2.3
```

ACs are numbered sequentially within their story, starting at 1 for each story. If ACs are added to an existing story, append new numbers at the end of that story's sequence. Never renumber existing ACs.

---

## Status Tracking

Every epic must declare a `**Status:**` field immediately after the epic description blockquote.

```markdown
## Epic CP-1: Create a Canvas Project

> Upload an entire product catalog from a spreadsheet file...

**Status:** Implemented
```

| Value | Meaning |
|-------|---------|
| `Planned` | Written as a spec; not yet built |
| `Partial` | Some ACs are implemented; others are not |
| `Implemented` | All ACs verified against the current implementation |
| `Deprecated` | Feature removed or replaced; epic retained for history |

**Rules:**
- New epics written ahead of implementation start as `Planned`.
- After implementing and verifying all ACs against the code, update to `Implemented`.
- After implementing a feature, re-read each AC and confirm it is true. If an AC no longer matches reality, update the AC — not just the status.
- If an epic is partially implemented, use `Partial` and note which ACs are pending in a comment inside the epic (e.g., "CP-1.4 and CP-1.5 not yet implemented").

---

## Open Questions

A theme may carry unresolved questions — gaps in the spec waiting on a product, legal, or technical
decision that isn't yet made. Collect them in a single **`## Open Questions`** section at the
**bottom of the theme file**, below the last epic. Do **not**
scatter them inside epics — a reader should find every open gap in one skimmable place.

### Format

Each open question is a `###` entry with an ID, the question, and short metadata lines:

```markdown
### OQ-[CODE]-[N] — [Short question title]

**Question:** [The unresolved decision, in one or two sentences — what must be decided, and why it's blocked.]
**Blocks:** [Epic/story/AC IDs this gates — including IDs in other themes — or "—" if none.]
**Raised:** [YYYY-MM-DD]
```

- IDs are `OQ-[CODE]-[N]`, numbered sequentially within the theme, never reused.
- Keep the question concrete. `Blocks` may reference another theme's IDs when the gap is cross-cutting.

### Resolution — remove, don't archive

When an open question is answered, **delete its entry entirely.** Never leave a "Resolved" tombstone —
the answer must flow into the durable spec instead:

- If it changes behavior → update or add the affected **AC**.
- If it settles a boundary → add an **Out of Scope** entry.
- If it settles a choice a later edit could plausibly undo → record it as a **Key Decision** (→ Format Rules → Rationale).

The Open Questions section is a working scratchpad for *unknowns only*; once known, the answer lives
in the spec and the OQ disappears. When the last question is resolved, remove the empty section too.

---

## When to Create vs. Update

### Create a new Theme when:
- A genuinely new product area exists that doesn't map to any existing theme.
- The user concern is entirely new — a new goal a persona can now achieve.

### Create a new Epic within an existing Theme when:
- A new user capability exists within the theme's domain.
- The user could **not** achieve this goal before.

### Update an existing Epic when:
- You're changing *how* an existing capability works.
- You're adding constraints, edge cases, or new variations to an existing flow.
- You're extending behavior within the same user goal.
- A bug fix changes observable behavior that the ACs describe.

> **Decision rule:** If a user can already *achieve* the goal and you're making it better or different, update. If they couldn't do it before at all, create a new epic.

### Never:
- Create a new epic that is a renamed or versioned copy of an existing one (e.g., "Submit Form v2").
- Create a new theme for a one-off feature change.

---

## Format Rules

Follow the structure in `_template.md` exactly. An epic's sections, in order, are **Status**,
**User Story**, **Contract**, **Acceptance Criteria**, and **Out of Scope**. There is no History
section: the timeline is git's, and the reasons that still hold live in Rationale (below).
**Contract** is optional — include it whenever the epic has a
concrete surface to pin down (a form's fields, a chart, a table, a derived value); omit it for a
purely behavioral epic that has nothing to enumerate.

There is **no "Implementation Notes" section** — *incidental* implementation (what the code does
today, the component/table/endpoint that happens to render it) does not belong in the spec; that
lives in the code and its git history. This is **not** a ban on concrete detail: the fields a form
accepts, their types and validation, a chart's axes, a derived metric's formula — these are *product
decisions*, and they belong in the **Contract** section (see below). The line is **decision vs.
incidental**, not concrete vs. abstract.

### Theme header

```markdown
# Theme: [Theme Name] · `[CODE]`
> [One sentence: the user problem or product area this theme addresses.]
```

### Epic header

```markdown
## Epic [CODE]-[N]: [Epic Name]

> [One sentence: the goal a user can now achieve. Start with a verb.]

**Status:** [Planned | Partial | Implemented | Deprecated]
```

### User Story

```markdown
### User Story
As a [persona], I want to [action] so that [outcome].
```

- One story per epic by default (use `S1` with no label).
- If an epic has meaningfully distinct user stories (different personas or entry points), assign each a story ID and label:
  ```markdown
  ### User Story AB-1-S1 — Single Board
  As a developer, I want to ...

  ### User Story AB-1-S2 — Many Boards
  As a developer, I want to ...
  ```
- Story IDs are `[CODE]-[N]-S[n]`; they appear in both the story heading and all AC IDs in that story's list.

### Contract

The **Contract** is where an epic's concrete surface is pinned down: exactly what a screen *is*,
so that a competent builder (human or AI) produces the intended thing rather than a plausible guess.
It sits between User Story and Acceptance Criteria — *what the thing is*, before *how it behaves*.

Acceptance Criteria (below) stay behavioral: they answer "How would I know this works?". The Contract
answers "What are the fields / columns / axes / values?" — the inventory that Given/When/Then is a
poor tool for. **Do not** dissolve a form's field list into forty ACs; put it in a Contract table.

**The precision test.** Specify down to the point of real ambiguity, then stop:

> Could two competent builders read this and ship *meaningfully different* products?

If yes, the Contract is under-specified. If you are pinning down things any reasonable builder would
default identically (exact pixel values, obvious control choices), you are over-specifying — cut it.

**Decision vs. incidental.** The Contract carries *product decisions* and never *incidental
implementation*:

- **In** — a field and its user-facing type (text / number / select / multi-select / date / phone /
  email / url), whether it is required, its accepted values and validation *stated as behavior*
  ("SKU accepts digits only"), defaults, a chart's mark / axes / aggregation / units, a derived
  metric's formula, the columns of a table.
- **Out** — component/file names, hooks, props, server-action or endpoint names, database table or
  column names, internal status codes. Use the user-facing label ("Attention Needed", not
  `attention_needed`). A domain concept from the glossary (e.g. "board") is fine; a
  code identifier is not.

**Use the densest honest representation** — match the tool to the content:

- **A form → a field table.** Columns: `Field · Type · Required · Rules / values · Notes`.
- **A table/list view → a column inventory** (each column: what it shows, how it's derived).
- **A chart → a compact spec block** — mark (bar/line/…), x, y (+units), aggregation, empty/zero
  handling, tooltip contents. Example:
  ```
  Chart: Daily scans (bar)
    x: day, trailing 30d      y: scan count (integer)
    aggregation: count of qualifying scans per day
    empty: "No scans yet" placeholder   zero-days: 0-height bars, not omitted
    tooltip: date + exact count
  ```
- **A derived value → a formula** ("coverage = frames with a name ÷ total frames").
- **A screen's states → a state table.** Columns: `State · Cause · What the user is told · Way out`.
  One row per state the screen can reach — loading, first-run empty, filtered-to-zero, partial, each
  error cause, disabled/ineligible, success, permission-denied. A state with no row is a state that
  will be discovered during build and written by whoever gets there first.
- **Sub-dialogs / repeatable rows → their own small table**, labelled.

Contract blocks are **unnumbered reference material** — they carry no `[CODE]-…` IDs. The numbered
Acceptance Criteria remain the verifiable behavioral contract, so no AC-ID cross-reference ever
breaks when the Contract is edited. When a Contract detail *also* needs behavioral verification
(e.g. "Group is required"), state it once as an AC too.

For multi-story epics, give each story its own labelled Contract block (`### Contract — [Label]`),
mirroring the labelled AC lists.

**Which sentences are Contract, and which are not.** The state table pins the *state set*, the *cause*
that distinguishes one from another, and the *way out* — those are product decisions. It carries
**verbatim wording only for strings that make a claim**: status and lifecycle labels, completeness and
verification wording, consent and data-collection text, anything of legal weight. On a compliance
product "Verified" and "Answered" are different assertions, so the spec owns which one is shown.
Every other sentence — the empty state's encouragement, a button label, a helper line — is written
against the project's own copy guidance and is not restated here; a spec that carries all its
own copy becomes a translation file and goes stale the first time the wording improves.

### Acceptance Criteria

Each AC is a `####` sub-heading with its ID and a short title, followed by Given/When/Then lines. Do **not** use tables.

```markdown
### Acceptance Criteria

#### CP-1-S1.1 — Create the project folder

Given I'm in an empty folder,
when I run init with a title,
then a canvas project is created in it.
```

- ID format is always `[CODE]-[N]-S[n].[ACn]` (e.g. `CP-1-S1.1`, `AB-2-S3.1`). ACs restart at `.1` for each story.
- The heading title is a concise (3–7 word) summary of the AC; the Given/When/Then lines are the behavioral contract.
- Each AC describes one observable behavioral contract — what the user sees or experiences.
- Keep ACs **behavioral** — reserve them for real interactions, flows, rules, and non-happy paths.
  Don't restate the Contract's inventory as ACs (no "the form has a Season field" AC); do write ACs
  for behavior the Contract can't express ("changing tier to T1 clears the parent").
- **Close the loop on every state.** For each state a story introduces, name what enters it and what
  leaves it **in every direction** — accepted, rejected, retried, abandoned — and what happens to the
  artifacts already attached when it is left. A state written with only its happy exit ships as a
  screen the user cannot leave, or as a status column written to make a card disappear. Where a
  return path is genuinely out of scope, say so in the story rather than leaving it unwritten.
- **The zero case is a required AC.** Any story whose behavior depends on a count — a list, a bulk
  action, a rollup, an export, a status derived from children — states what happens when that count
  is zero, and where it differs, at one..
- No *incidental* implementation detail (component/endpoint/table names, internal status codes).
  Concrete *product* decisions — field types, validation, formulas — live in the **Contract**.
- When an epic has multiple user stories, use labeled subsections (`### Acceptance Criteria — [Label]`) with a separate AC list per story.

### Out of Scope

```markdown
### Out of Scope
- [Something explicitly not covered by this epic.]
```

- This section is **required**. If genuinely nothing is out of scope, write "Nothing explicitly excluded at this time."
- It prevents scope creep in generated code. Include anything adjacent that might be assumed.

### Scale Envelope

One optional **theme-level** section (not per-epic), placed after the theme's `## Key Decisions` and
before the first epic. It records the volumes and limits the theme is committed to — the answers a
feature is gated on, so they are decided once and not re-litigated per feature.

```markdown
## Scale Envelope

| Quantity / limit | Committed value | At the ceiling |
|---|---|---|
| [What is counted] | [number] | [what the user sees when exceeded] |

**Timing.** [What the user waits for, and what moves to the background.]
**On failure.** [What a partial or retried run leaves behind.]
```

**Envelope vs. tuning — the same decision-vs-incidental line as the Contract.** What goes here is
*user-observable commitment*: how many we accept, what the user sees at the ceiling, how long they
wait, what a retry does to their data. What stays in code is *tuning*: batch sizes, concurrency,
index choices, chunk widths, timeouts. "A generation run accepts at most 10,000 codes and rejects
larger ones up front" is a commitment; `CONCURRENCY = 10` is tuning.

This is the one carve-out from *What NOT to Put in the Spec* → "performance benchmarks or technical
constraints". That bullet still bans benchmarks and implementation limits; a stated ceiling with a
user-facing consequence is a product decision that happens to have a number in it.

### Rationale

A spec states what is true now. *Why* it is true is recorded only where a later editor could
plausibly undo the rule without knowing the reason, and it lives next to the rule, never in a log:

- **A theme-wide choice** → a row in the theme's `## Key Decisions` table (`# · Question · Decision`),
  placed after the theme header. Edit the row in place when the decision changes; never append a
  second one that overrides it.
- **A single AC** → one `Why:` line under its Given/When/Then, one sentence, stating the constraint
  or the failure the rule prevents.

Never recorded in a spec: when something changed, what it used to do, a defect's story, or who
decided. Those belong in the commit and the PR body. A retired AC is deleted and its number is not
reused.

---

## Writing Style

- **User-focused, not incidental.** Describe what the user experiences and the product decisions
  behind it — not the code that happens to implement it. Concrete field/chart/formula precision is
  *welcome* (in the Contract); component and endpoint names are not.
- **Behavior-driven ACs.** ACs answer "How would I know this works?" not "How does this work?" The
  Contract answers "What is it?" — keep the two jobs in their own sections.
- **Concrete personas.** Use specific persona labels from the Personas section above. Do not use "the system", "the app", or "users" generically.
- **Present tense.** "I can see" not "I should be able to see."
- **Active voice.** "the viewer shows the developer" not "the developer is shown."
- **No jargon.** Avoid internal implementation names unless they are meaningful to the user (e.g., "magic link" is user-facing; "DataRequest" is not).

---

## What NOT to Put in the Spec

This is the *incidental implementation* half of the decision-vs-incidental line — it never belongs in
a user story, an AC, **or** the Contract:

- Database table names, column names, or schema details
- Function or component names, file paths, hooks, props
- API endpoints or server-action names
- Internal status codes — use the user-visible label ("attention needed", not `attention_needed`)
- Performance benchmarks or technical constraints — *except* a theme's committed ceilings and
  timing, which have their own section (→ Format Rules → Scale Envelope)
- Test cases or QA checklists

Note the distinction: *field-level product decisions* — which fields a form has, their types,
required/optional, validation, defaults — are **not** on this list. They are decisions, and they
belong in the **Contract**. What's banned is the *code that renders them*, not the fields themselves.

---

## Updating Existing ACs

When behavior changes:

1. Update the relevant AC entry in-place. Do not add a new entry that contradicts the old one.
2. If the new rule is one a later edit could plausibly undo, give it a `Why:` line or a Key Decision (→ Format Rules → Rationale).
3. If removing an AC (behavior no longer supported), delete the entry; its number is retired.

---

## Adding a New Theme

1. Assign a theme code (2–4 chars, unique, obvious from the theme name).
2. Create the flat file `themes/[theme-name].md` from `_template.md`.
3. Set the title with the code: `# Theme: [Name] · \`CODE\``.
4. Add a row to the Document Index above:
   ```markdown
   | [Theme Name](./themes/theme-name.md) | `CODE` | `themes/theme-name.md` | — | Active |
   ```
