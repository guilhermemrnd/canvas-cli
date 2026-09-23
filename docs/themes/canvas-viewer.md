# Theme: Canvas Viewer · `CV`

> A local page that shows the canvas as pages of boards the developer can pan, zoom and click to read, and that keeps up with the agent as it writes.

## Key Decisions

| # | Question | Decision |
|---|----------|----------|
| 1 | Can the developer change the design in the viewer? | No. The viewer is read-only; every change goes through the agent and the files. |
| 2 | Does a click reach the agent? | No. Selection is for the developer to read; they describe the element to the agent in words. |
| 3 | What does "clickable" mean? | Selecting an element to read its properties. Links in a board are not followed. |
| 4 | Where does the viewer's own state live? | In the browser, per project: current page, zoom, position. Losing it costs nothing. |
| 5 | How is a board rendered? | In its own isolated frame at its real size, with script off, then placed and scaled by the canvas. |
| 6 | Who may reach the viewer? | This machine only (→ AH-2). |

---

## Epic CV-1: Browse the Canvas

> See a page's boards laid out, and move around them.

**Status:** Planned

### User Story CV-1-S1

As a developer, I want to pan and zoom a page of boards so that I can take in a whole flow and then read one screen closely.

### Contract

**Canvas view**

| Part | What it shows / does |
|---|---|
| Page tabs | One per page, in index order; the current one is marked |
| Board | Drawn at its real size at its position, with its title above it |
| Frame name | Above each named frame, inside its board |
| Zoom | 10–400%, shown as a percentage; "fit page" and "fit selection" |
| Pan | Drag with space held, middle-drag, trackpad scroll; zoom with pinch or ctrl + wheel |

**States**

| State | Cause | What the developer is told | Way out |
|---|---|---|---|
| Loading | First render or a page switch | Boards appear as they are ready | — |
| No pages | Project has zero pages | That there are no pages yet | Ask the agent to add one |
| Empty page | Page has zero boards | That the page has no boards | Ask the agent to add one |
| Board missing | Index lists a file that is not there | The board's title and that its file is missing | Agent restores the file or removes the entry |
| Board invalid | File breaks a board rule and no earlier good render exists | The board's title and that it cannot be drawn | Agent fixes it (→ CP-4) |
| Index invalid | The index cannot be read | The problems, and the last good canvas stays shown | Agent fixes it |
| Disconnected | The local server stopped | That the viewer is disconnected | Restart `serve`; the viewer reconnects on its own |

### Acceptance Criteria

#### CV-1-S1.1 — Open on the launch page

Given the index names a launch page,
when the developer opens the viewer,
then that page is shown, otherwise the first.

#### CV-1-S1.2 — Switch pages

Given a project with several pages,
when I select another tab,
then that page's boards are shown and zoom and position are remembered per page.

#### CV-1-S1.3 — Zoom limits

Given the canvas is at 10% or at 400%,
when I zoom further in the same direction,
then it stays at the limit.

#### CV-1-S1.4 — Fit

Given a page of boards,
when I choose fit page,
then every board is visible at the largest zoom that shows all of them.

#### CV-1-S1.5 — Zero and one board

Given a page with zero boards, then it shows as empty; given one board, fit page fits that board.

#### CV-1-S1.6 — Frame names

Given a board with named frames,
when I zoom until a frame is readable,
then its name is shown above it.

#### CV-1-S1.7 — Board with a problem never blanks the page

Given one board is missing or invalid,
when I browse the page,
then the other boards render normally.

#### CV-1-S1.8 — Off-screen boards

Given a page of many large boards,
when I pan to one not yet shown,
then a placeholder of its size is drawn until it renders.

Why: rendering every board at once makes the page unusable at the top of the envelope (→ CP Scale Envelope).

### Out of Scope

- Moving, resizing, grouping or deleting boards.
- Comments, cursors of other people, sharing.
- A layer tree or outline of the elements.

---

## Epic CV-2: Focus a Board or a Frame

> Look at one board, or one state of it, at full size.

**Status:** Planned

### User Story CV-2-S1

As a developer, I want to open one board or frame by itself so that I can read it without the rest of the canvas.

### Acceptance Criteria

#### CV-2-S1.1 — Focus a board

Given the canvas view,
when I open a board,
then it is shown alone at its real size, scrolling if the window is smaller, with a way back to the same place on the canvas.

#### CV-2-S1.2 — Jump to a frame

Given a board with named frames,
when I choose a frame from the board's list,
then the view centres and fits that frame, in the canvas or the focused view.

#### CV-2-S1.3 — Step through states

Given a focused frame in a row of frames,
when I go to the next or previous,
then the neighbouring frame in flow order is shown, and stops at the ends.

#### CV-2-S1.4 — Board without frames

Given a board with zero frames,
when I focus it,
then the whole board is shown and there is no frame list.

#### CV-2-S1.5 — Focused board is removed

Given I am focused on a board that is then removed from the index,
when the change arrives,
then I am returned to its page's canvas view.

### Out of Scope

- Presenting or playing a flow by clicking through boards.
- Sharing a link to a focused board.

---

## Epic CV-3: Inspect an Element

> Click any element and read its size, position and styles.

**Status:** Planned

### User Story CV-3-S1

As a developer, I want to click any element and read its width, height, spacing and colours so that I can judge a design, and tell the agent exactly what to change.

### Contract

**Inspector — what it shows for the selected element**

| Group | Shows | Notes |
|---|---|---|
| Path | Page › Board › Frame › element | The element as its tag and class |
| Size | Width, height in px | Rendered size |
| Position | x, y in px from the board's top-left | — |
| Layout | Display, direction, gap, justify, align | Only when the element lays out children |
| Spacing | Padding and margin, four sides | Zero sides are shown as zero |
| Type | Font, size, weight, line height, colour | Only for an element that holds text |
| Fill | Background colour or image | — |
| Border | Width, style, colour, radius | — |
| Effects | Opacity, shadow | Only when set |
| Text | The text content | Cut off past a length |

**Values.** Every value is shown resolved. A value that comes from a token is shown with the token's name beside it; a value that does not is shown as it is.

### Acceptance Criteria

#### CV-3-S1.1 — Select

Given the canvas or focused view,
when I click an element in a board,
then the deepest element under the pointer is outlined and its properties are shown.

#### CV-3-S1.2 — Select without acting

Given a button or a link in a board,
when I click it,
then it is selected and nothing happens, and no link is followed.

#### CV-3-S1.3 — Parent and child

Given a selected element,
when I press the up or down arrow,
then its parent, or its first child, is selected, stopping at the board's root and at an element with no child.

Why: a container fully covered by its children could otherwise never be selected.

#### CV-3-S1.4 — Hover

Given no selection,
when I move over a board,
then the element under the pointer is outlined with its size.

#### CV-3-S1.5 — Token value

Given a fill set from the token `--ink-strong`,
when I select the element,
then the fill shows the colour and the token's name.

#### CV-3-S1.6 — Clear

Given a selection,
when I press Escape or click empty canvas,
then the selection and the inspector close.

#### CV-3-S1.7 — Selection through a change

Given an element is selected and its board is rewritten,
when the change arrives,
then the same element, at the same place in the same frame, stays selected with fresh values; if it no longer exists the inspector closes.

#### CV-3-S1.8 — Element with no size

Given an element that renders with zero width or height,
when I select it through its parent,
then its properties are shown with the zero size.

#### CV-3-S1.9 — Never leaves the machine

Given any use of the inspector,
when properties are read,
then nothing is sent or written anywhere.

### Out of Scope

- Editing any property, including text.
- Copying a selection to the agent or the clipboard (Key Decision 2).
- Measuring distances between elements.
- Selecting several elements.

---

## Epic CV-4: Follow the Agent Live

> The canvas updates as the agent writes files.

**Status:** Planned

### User Story CV-4-S1

As a developer, I want the canvas to change as the agent works so that I watch the design take shape and never reload by hand.

### Contract

**What changes what**

| Change on disk | Viewer does |
|---|---|
| A board file is saved | That board is redrawn; nothing else moves |
| The tokens file is saved | Every board is redrawn |
| The index is saved | Layout, tabs and titles are recomputed |
| An asset is saved | The boards that read it are redrawn |

### Acceptance Criteria

#### CV-4-S1.1 — Visible within a second

Given the viewer is open,
when the agent saves a board,
then the redrawn board is visible within 1 second.

#### CV-4-S1.2 — View is kept

Given I am zoomed and panned on a page,
when any file changes,
then page, zoom and position are unchanged.

#### CV-4-S1.3 — Half-written file

Given a board file is saved half-written or invalid,
when the change arrives,
then the last good render stays, marked as out of date, and the board is redrawn once a valid file arrives.

#### CV-4-S1.4 — Burst of writes

Given the agent saves many files in a row,
when the changes arrive,
then the viewer settles on the final state and never shows an earlier one after a later one.

#### CV-4-S1.5 — New and removed boards

Given the agent adds or removes a board,
when the index is saved,
then the board appears, or disappears, in place.

#### CV-4-S1.6 — Current page removed

Given I am viewing a page that the index no longer lists,
when the change arrives,
then I move to the launch page or the first; with zero pages, the no-pages state.

#### CV-4-S1.7 — Viewer opened first

Given the viewer is open before the agent has written anything,
when the first page and board arrive,
then they appear without a reload.

#### CV-4-S1.8 — Reconnect

Given the server stopped and started again,
when the viewer reconnects,
then it reloads the canvas once and keeps the current page.

### Out of Scope

- Undo, history or comparing versions of a board (git holds them).
- Two viewers keeping their positions in step.
