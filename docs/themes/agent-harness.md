# Theme: Agent Harness · `AH`

> Any coding-agent harness can drive the canvas from a skill and a local command, with the project and its guidelines in the agent's context.

## Key Decisions

| # | Question | Decision |
|---|----------|----------|
| 1 | What does a harness need to support? | Loading a skill, running a shell command and writing files. Nothing else: no plugin, no server protocol. |
| 2 | Where does the design know-how live? | In the skill, as plain instructions. The command holds mechanics only (create, check, serve) and never judges a design. |
| 3 | First harness? | Devin CLI; Claude Code and others use the same skill unchanged. |
| 4 | Does anything leave the machine? | No. No account, no telemetry, no upload, no remote fetch. |
| 5 | Who says a design looks right? | Nobody, until the developer has looked. The agent reports what it wrote and what validation said, not how it renders. |

## Scale Envelope

| Quantity / limit | Committed value | At the ceiling |
|---|---|---|
| Viewers per project | 1 | A second `serve` reports the running address and starts nothing |
| Viewers per machine | Unbounded; each picks a free port | — |

**Timing.** `init`, `add-page`, `add-board`, `list` and `validate` return immediately. `serve` stays running until stopped.
**On failure.** A refused command changes nothing and says why in one line.

---

## Epic AH-1: Drive the Canvas from a Command

> Create, check and show a canvas with a handful of commands.

**Status:** Planned

### User Story AH-1-S1

As an agent, I want a few predictable commands so that I create and check a canvas without hand-writing the index.

### Contract

| Command | Does | Output |
|---|---|---|
| `init <title>` | Creates the project (→ CP-1) | The project's location |
| `add-page <name>` | Adds a page (→ CP-2) | The page id |
| `add-board` | Adds a board to a page at a size (→ CP-3) | The file name |
| `list` | Lists pages, boards, sizes and frames | Plain text; `--json` for a list |
| `validate` | Checks every file (→ CP-4) | Findings; `--json` for a list; exit non-zero on error |
| `serve` | Starts the viewer (→ AH-2) | The address |

Every command prints which project it used and exits non-zero when it changed nothing because of a problem.

### Acceptance Criteria

#### AH-1-S1.1 — Unknown command or missing input

Given a command with a missing or unknown argument,
when it runs,
then nothing changes and the message names the argument and the correct form.

#### AH-1-S1.2 — No project

Given no project in this folder or any parent,
when any command but `init` runs,
then it stops and says to run `init`.

#### AH-1-S1.3 — Listing

Given a project with pages and boards,
when the agent runs `list`,
then it sees each page's boards with their sizes and frame names, ready to edit in place.

#### AH-1-S1.4 — Listing zero

Given a project with zero pages or zero boards,
when the agent runs `list`,
then the output says so and exits zero.

### Out of Scope

- Commands that rename, move or delete boards and pages (the agent edits files).
- Commands that change a board's content.
- Export to image or PDF.

---

## Epic AH-2: Serve the Viewer Locally

> Show the canvas in a browser without anything leaving this machine.

**Status:** Planned

### User Story AH-2-S1

As a developer, I want the agent to give me a local address for the canvas so that I can look at it as it works.

### Acceptance Criteria

#### AH-2-S1.1 — Start

Given a project,
when the agent runs `serve`,
then the viewer is reachable at a printed local address and the command stays running.

#### AH-2-S1.2 — Local only

Given the viewer is running,
when a request arrives from another machine or with a non-local host name,
then it is refused.

Why: the canvas holds unreleased product designs, and a local page reachable by any site can be read through name tricks.

#### AH-2-S1.3 — Port in use

Given the default port is taken,
when the agent runs `serve`,
then the next free port is used and printed.

#### AH-2-S1.4 — Already running

Given a viewer is already running for this project,
when the agent runs `serve` again,
then it prints the running address and starts nothing.

#### AH-2-S1.5 — Stale record

Given a previous viewer stopped without cleaning up,
when the agent runs `serve`,
then it starts normally and does not report the dead one as running.

#### AH-2-S1.6 — Stop

Given a running viewer,
when it is interrupted,
then it stops, releases the port and leaves every project file untouched.

#### AH-2-S1.7 — Only the project is served

Given any request path,
when it leaves `.canvas/`,
then it is refused.

#### AH-2-S1.8 — Nothing goes out

Given the viewer running,
when the developer uses it for any time,
then it makes no request to any other machine.

### Out of Scope

- Public sharing, tunnels or hosting.
- Opening the browser for the developer (the agent gives the address).
- Running as a background service after the terminal closes.

---

## Epic AH-3: Give the Agent the Design Protocol

> A skill that makes any agent design from the project's own guidelines.

**Status:** Planned

### User Story AH-3-S1

As a developer, I want to say what I need designed and have the agent draw it from my project's guidelines so that the canvas shows something I can build.

### Contract

**What the skill tells the agent to do, in order**

| Step | Rule |
|---|---|
| 1 Find the project | Use the canvas project here, or run `init` with a title when there is none |
| 2 Read the guidelines | Read the project's own instructions, design guidance and existing UI code before drawing |
| 3 Fill the tokens | Write the values the guidelines name into the tokens file; a value they do not cover is raised, not invented |
| 4 Revise, never fork | Change the existing boards for a screen that already has one; add a board only for a new surface |
| 5 Draw in context | A page inside the app's real chrome, at the app's real width; only a modal, menu or toast bare, over a scrim |
| 6 Draw every state | Loading, empty, partial, error, permission-denied and success, and every non-happy path the request names, as frames in one row |
| 7 Write copy as product copy | Short, real, no filler text; a missing fact is a visible placeholder, not invented |
| 8 Check | Run `validate` and fix every error |
| 9 Report | Give the address, what was drawn, what was assumed and what was not covered; say it has not been looked at |

### Acceptance Criteria

#### AH-3-S1.1 — Guidelines found

Given a project with instructions, design guidance or tokens,
when the agent starts a design,
then the tokens file is filled from them before any board is drawn.

#### AH-3-S1.2 — No guidelines

Given a project with no design guidance at all,
when the agent is asked to design,
then it asks the developer what to follow before drawing, and does not invent a system.

#### AH-3-S1.3 — Gap raised

Given the agent needs a colour or size the guidelines lack,
when it draws,
then it asks the developer, or leaves a marked placeholder and lists it in the report.

#### AH-3-S1.4 — Revise in place

Given a board for the screen already exists,
when the developer asks for a change,
then that board's file is edited, its frame names stay, and no second copy is added.

#### AH-3-S1.5 — States

Given a flow with several states,
when the agent draws it,
then each state that looks different is a named frame in one row in flow order.

#### AH-3-S1.6 — Honest report

Given the agent finished,
when it reports,
then it states the validation result and does not say the design looks right.

Why: the agent wrote markup it has not seen rendered.

#### AH-3-S1.7 — Verify only when asked

Given the developer has not asked for a screenshot check,
when the agent finishes,
then it does not open a browser or render an image, and offers to.

### Out of Scope

- Variant exploration protocols (the developer asks for options in the prompt).
- Teaching the agent a visual style; the guidelines carry that.
- Anything in the project's own instructions about how to design that the skill would repeat.

---

## Epic AH-4: Install in Any Harness

> Drop the skill into a harness and design from its next session.

**Status:** Planned

### User Story AH-4-S1

As a developer, I want to add the skill to Devin CLI, or any other harness, by copying a folder so that I do not wire anything per tool.

### Acceptance Criteria

#### AH-4-S1.1 — One folder

Given the skill folder placed where the harness reads skills,
when a session starts,
then the harness lists the skill and it can start a design from a plain request.

#### AH-4-S1.2 — Same skill, every harness

Given Devin CLI, Claude Code, or a harness that only reads a plain instructions file,
when each is given the skill's text,
then each can follow all nine steps with shell and file access alone.

#### AH-4-S1.3 — Command missing

Given the skill is installed but the command is not,
when the agent tries a command,
then it stops and tells the developer how to install it, and writes no project files by hand.

Why: hand-writing the index is how the files drift from what validation expects.

#### AH-4-S1.4 — Version drift

Given the skill and the command are of different versions,
when the agent runs a command,
then it reports the mismatch and stops.

### Out of Scope

- A per-harness plugin or MCP server.
- Auto-updating the skill.

---

## Open Questions

### OQ-AH-1 — How is the command installed?

**Question:** As an npm package run with `npx` (needs Node; easy to update and to hand to any harness that has it), or as a single binary (no runtime, heavier release work)? The choice also fixes what "installed" means in AH-4-S1.3.
**Blocks:** AH-4-S1
**Raised:** 2026-09-23

### OQ-AH-2 — Does the skill cover a harness that cannot run a long process?

**Question:** `serve` must stay running. Some harnesses run each command to completion; either the skill tells the agent to start it in the background, or `serve` detaches itself and gets a `stop` command. The second adds a command and a stop rule.
**Blocks:** AH-2-S1.1, AH-2-S1.6
**Raised:** 2026-09-23
