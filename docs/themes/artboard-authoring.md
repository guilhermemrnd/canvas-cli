# Theme: Artboard Authoring · `AB`

> A board is one plain HTML file the agent writes at a real size, with named frames for its states and design values taken from the project's own guidelines.

## Key Decisions

| # | Question | Decision |
|---|----------|----------|
| 1 | What is a board's format? | Plain HTML and CSS. No template language, no components, no script: what the agent writes is what renders. |
| 2 | What is a frame? | Any element the agent marks with a name. Unmarked elements stay selectable; marked ones become the unit the developer and agent both name. |
| 3 | Can boards reach the network? | No. Fonts and images are local files or inline data. A board that needs a remote file fails validation. |
| 4 | Can boards run script? | No. A board is a picture with real markup, so it cannot fetch, track or change itself. |
| 5 | Where do design values come from? | The project's own guidelines, written once into the shared tokens file. A value the guidelines do not cover is raised to the developer, never made up. |
| 6 | Can frames nest? | No. A frame is a screen or a state; a frame inside a frame is a component, which is just markup. |

---

## Epic AB-1: Write a Board File

> Write a board the viewer can render exactly as drawn.

**Status:** Planned

### User Story AB-1-S1

As an agent, I want a small, strict board format so that what I write renders the same way for the developer without my checking it.

### Contract

**Board rules**

| Rule | Severity |
|---|---|
| A complete HTML document with a single root element in the body | Error |
| The root has the board's exact width and height | Error |
| The document reads the shared tokens | Warning |
| No script, no event-handler attributes, no `iframe`, `object` or `embed` | Error |
| No `javascript:` links | Error |
| Real form controls and links are used as such, never a styled `div` in their place | Warning |
| Icon-only buttons carry a label | Warning |

### Acceptance Criteria

#### AB-1-S1.1 — Renders at its size

Given a valid board of 1440 × 900,
when the viewer shows it,
then it fills exactly 1440 × 900 canvas pixels, with nothing scaled, clipped or scrolling inside it.

#### AB-1-S1.2 — Script is refused

Given a board with a script or an inline event handler,
when validation runs,
then it is an error naming the file and line, and the viewer does not run it.

#### AB-1-S1.3 — Root size mismatch

Given the root element's size differs from the index,
when validation runs,
then the error gives both sizes.

#### AB-1-S1.4 — Content taller than the board

Given content that extends past the board's height or width,
when the viewer shows it,
then the board keeps its size and the overflow is drawn as it overflows, not cropped or resized.

Why: an artboard sized to fit its content hides the layout decision it exists to show.

#### AB-1-S1.5 — Empty board

Given a board whose root has no content,
when the viewer shows it,
then it renders as an empty board at its size, and validation reports no error.

### Out of Scope

- Components, loops, conditions or data binding.
- Interactivity of any kind: hover, click-through, animation.
- Dark or light variants of one board.

---

## Epic AB-2: Name Frames and Lay Out States

> Draw the states of one surface as named frames side by side.

**Status:** Planned

### User Story AB-2-S1

As a developer, I want every state of a screen drawn and named next to the others so that I can compare loading, empty, error and success at a glance.

### Contract

**Frame**

| Field | Type | Required | Rules / values | Notes |
|---|---|---|---|---|
| Name | text | Yes | Non-empty; unique within the board | Shown above the frame in the viewer |
| Element | any | Yes | Marked with the name; not inside another frame | The frame's box is the element's box |

**Convention (carried by the skill, → AH-3):** the states of one surface are frames in one row, left to right in flow order, evenly spaced. A different surface starts a new board.

### Acceptance Criteria

#### AB-2-S1.1 — Frames are listed

Given a board with three named frames,
when the developer opens it,
then each frame's name is shown above it and the viewer can jump to any of them (→ CV-2).

#### AB-2-S1.2 — Zero frames

Given a board with no named frames,
when the developer opens it,
then the whole board is the only unit: shown, selectable and focusable, with no frame list.

#### AB-2-S1.3 — One frame

Given a board with one named frame,
when the developer opens it,
then it is shown as any other, with its name.

#### AB-2-S1.4 — Duplicate name

Given two frames in a board share a name,
when validation runs,
then it is an error naming both.

#### AB-2-S1.5 — Nested frames

Given a frame inside another frame,
when validation runs,
then it is an error naming the inner one.

#### AB-2-S1.6 — Frame past the board

Given a frame that extends outside the board,
when the developer opens the board,
then the frame is drawn where it is and marked as not fitting.

Why: the board size is the promise that every frame is shown whole.

### Out of Scope

- A frame list or layer tree editable in the viewer.
- Frame-level export.

---

## Epic AB-3: Read Design Values from the Project's Guidelines

> Make every colour, size and font on a board traceable to something the project decided.

**Status:** Planned

### User Story AB-3-S1

As a developer, I want the agent's designs to use my project's tokens and geometry so that the canvas shows something I can build, not a picture of another product.

### Contract

**Tokens file.** One stylesheet of custom properties: colours, type sizes and families, spacing, radii, shadows, control heights. Each is named as the project's own guidelines name it.

### Acceptance Criteria

#### AB-3-S1.1 — Boards read tokens

Given a token named `--ink-strong`,
when a board colours text with it,
then the viewer shows the resolved colour and validation reports nothing.

#### AB-3-S1.2 — Raw value

Given a board uses a raw colour, size or font that no token holds,
when validation runs,
then it is a warning naming the value and file.

#### AB-3-S1.3 — Raw value equal to a token

Given a raw value equal to a token's value,
when validation runs,
then the warning names the token to use.

#### AB-3-S1.4 — Gap in the guidelines

Given the agent needs a value the guidelines do not cover,
when it draws,
then it asks the developer for the value and does not make one up (→ AH-3).

#### AB-3-S1.5 — Empty tokens

Given a project whose tokens file is empty,
when validation runs,
then it warns that boards have nothing to read, and errors on nothing.

#### AB-3-S1.6 — Token changed

Given a token's value is edited,
when the file is saved,
then every board reading it shows the new value (→ CV-4).

### Out of Scope

- Extracting tokens automatically from a stylesheet or a design tool.
- Installing a third-party design system.
- Multiple themes, or dark mode, from one tokens file.

---

## Epic AB-4: Keep Assets Local

> Boards render the same with no network and leak nothing.

**Status:** Planned

### User Story AB-4-S1

As a developer, I want boards that never call out so that the canvas works offline and nothing about my product leaves my machine.

### Acceptance Criteria

#### AB-4-S1.1 — Local file

Given a board refers to a font or image under `assets/` by a relative path,
when the viewer shows it,
then the file renders.

#### AB-4-S1.2 — Remote reference

Given a board or the tokens file refers to a remote address for a font, image or stylesheet,
when validation runs,
then it is an error naming the address and where it is.

#### AB-4-S1.3 — Missing asset

Given a relative reference to a file that does not exist,
when validation runs,
then it is an error naming the reference.

#### AB-4-S1.4 — Inline data

Given a small image inlined as data,
when validation runs,
then it is accepted, within the board file's size limit (→ CP Scale Envelope).

#### AB-4-S1.5 — Reference outside the project

Given a path that leaves `.canvas/`,
when validation runs,
then it is an error, and the viewer does not serve it.

### Out of Scope

- Fetching stock images or fonts for the agent.
- Asset management in the viewer.

---

## Open Questions

### OQ-AB-1 — How is "fits the board" checked?

**Question:** AB-1-S1.3 is static, but AB-2-S1.6 and overflow need a real layout. Options: only the viewer measures and marks it; or `validate` also measures using a locally installed browser when one is found; or the tool bundles one. The middle option gives the agent feedback with no download; the third makes the install heavy.
**Blocks:** AB-2-S1.6, CP-4-S1
**Raised:** 2026-09-23
