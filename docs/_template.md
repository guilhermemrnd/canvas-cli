# Theme: [Theme Name] · `CODE`

> [One sentence: the user problem or product area this theme addresses.]

---

## Epic CODE-1: [Epic Name]

> [One sentence: the goal a user can now achieve. Start with a verb — "Send", "Upload", "Review".]

**Status:** Planned

### User Story CODE-1-S1

As a [persona], I want to [action] so that [outcome].

<!--
Contract — optional. Include when the epic has a concrete surface to pin down (form fields, a chart,
a table, a derived value). Omit for a purely behavioral epic. Carries product decisions, never
incidental implementation (no component/table/endpoint names). See AGENTS.md → Format Rules → Contract.
Match the tool to the content — a form is a field table, a chart is a spec block, a metric is a formula.
-->

### Contract

**[Screen / form name] — fields**

| Field | Type | Required | Rules / values | Notes |
|---|---|---|---|---|
| [Field] | text | Yes | [validation stated as behavior] | [default / note] |
| [Field] | select | No | [allowed values] | — |

### Acceptance Criteria

#### CODE-1-S1.1 — [Short title]

Given [precondition],
when [action],
then [observable result].

#### CODE-1-S1.2 — [Short title]

Given [precondition],
when [action],
then [observable result].

### Out of Scope

- [Something explicitly not covered by this epic.]

---

## Epic CODE-2: [Another Epic Name]

> [One sentence goal.]

**Status:** Planned

<!-- Multi-story epic: give each story an ID + label and a separate AC list. -->

### User Story CODE-2-S1 — [Label]

As a [persona], I want to [action] so that [outcome].

<!-- Multi-story epic: give each story its own labelled Contract block too. -->

### Contract — [Label]

```
Chart: [name] ([mark])
  x: [dimension]        y: [measure] ([units])
  aggregation: [rule]
  empty: [placeholder]  tooltip: [contents]
```

### Acceptance Criteria — [Label]

#### CODE-2-S1.1 — [Short title]

Given [precondition],
when [action],
then [observable result].

### User Story CODE-2-S2 — [Label]

As a [persona], I want to [action] so that [outcome].

### Acceptance Criteria — [Label]

#### CODE-2-S2.1 — [Short title]

Given [precondition],
when [action],
then [observable result].

### Out of Scope

- [Something explicitly not covered by this epic.]
