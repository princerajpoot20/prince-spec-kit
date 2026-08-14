# Prince Spec Kit

A lightweight, context-efficient spec kit for AI-assisted development in brownfield projects.

Built to solve three scaling problems common spec kits run into: **monolithic context files** that consume too many tokens, **overly strict guardrails** that block legitimate work, and **knowledge structures that assume greenfield conditions** where everything is clean and well-documented.

Instead of one giant constitution file that every AI call must load, this kit uses a navigable knowledge structure — like a book with a table of contents, chapters, and detail pages. The AI reads the index first, loads only the chapters relevant to the current task, and goes deeper only when needed.

---

## Table of contents

- [Installation](#installation)
- [First-time setup — /discover](#first-time-setup----discover)
- [The development workflow](#the-development-workflow)
  - [1. Start work — /start](#1-start-work----start)
  - [2. Challenge the plan — /challenge](#2-challenge-the-plan----challenge)
  - [3. Implement](#3-implement)
  - [4. Complete and archive — /complete](#4-complete-and-archive----complete)
- [Code review — /code-review](#code-review----code-review)
- [The knowledge structure](#the-knowledge-structure)
- [Command reference](#command-reference)
- [Design principles of this kit](#design-principles-of-this-kit)

---

## Installation

Copy the `.cursor/` and `.spec/` folders from this repo into your project root:

```bash
cp -r /path/to/prince-spec-kit/.cursor /your/project/
cp -r /path/to/prince-spec-kit/.spec /your/project/
```

Add to your project's `.gitignore`:
```
# Active work items — local planning files
.spec/work/active/
```

Add to `.cursorignore` (keeps large archive files out of Cursor's AI indexing):
```
.spec/work/archive/completed-tasks.md
```

Then open your project in Cursor and run `/discover` to initialize.

> Full installation details: [INSTALL.md](./INSTALL.md)

---

## First-time setup — /discover

`/discover` is the one-time setup command. Run it once when you first add this spec kit to a project. After that, you only run it again when you want to refresh or expand the knowledge.

```
/discover
```

**What happens:**

**Phase 1 — Initialization**
The AI reads your `package.json` (or equivalent manifest), detects the project name, gets the current git SHA, and prints:
```
Initializing Prince Spec Kit for: my-project
Current SHA: abc1234
Knowledge base will be created at: .spec/knowledge/

If the project name looks wrong, say the correct name now.
Otherwise press Enter or say 'proceed'.
```
Confirm or correct the name. The AI stamps the project name into `.cursor/rules/principles.mdc` and proceeds.

**Phase 2 — Knowledge build**
The AI explores the codebase and builds:

```
.spec/knowledge/
├── index.md          ← Master navigation — what's where and when to open each chapter
├── overview.md       ← Project purpose, architecture, tech stack, conventions
├── deploy.md         ← Deployment, CI/CD, environments, rollback
└── {domain}/
    └── README.md     ← Chapter for each major domain area (editor, state, parser, etc.)
```

When done:
```
✓ Prince Spec Kit initialized for my-project.

Knowledge base built — 5 chapters created under .spec/knowledge/.

Chapters:
- editor       → Monaco integration, keybindings, file handling
- parser       → AsyncAPI parsing, Spectral rules, $ref resolution
- state        → Zustand stores, persistence, what lives where
- visualizer   → React Flow graph, node types, layout
- services     → External calls, service wiring, DI

Next steps:
- Run /start {description} to begin a task or feature
- Run /discover --deep {domain} to expand a chapter with detail pages
- Open .cursor/rules/principles.mdc to add project design principles
```

**Refresh options** (for after the initial setup):

| Command | What it does |
|---------|-------------|
| `/discover` | Print summary of current knowledge |
| `/discover --refresh` | Regenerate all knowledge from scratch |
| `/discover --refresh overview` | Regenerate project overview only |
| `/discover --refresh deploy` | Regenerate deployment knowledge only |
| `/discover --refresh {domain}` | Regenerate one chapter and its detail pages |
| `/discover --deep {domain}` | Add detail pages to a chapter (goes deeper without changing other chapters) |
| `/discover "where is auth handled?"` | Answer a question from the knowledge base |

---

## The development workflow

Once the knowledge base is built, your workflow for any piece of work follows this path:

```
/start → (optional) /challenge → implement → /complete
```

---

### 1. Start work — /start

Run `/start` to begin any task or feature. You don't need a polished description — give it your raw idea and the AI will refine it.

```
/start add a valid/invalid status chip to the editor header
```
```
/start fix the parser not clearing diagnostics when the document becomes valid
```
```
/start I want users to be able to toggle the visualizer panel with a keyboard shortcut
```

**What the AI does:**

1. Reads `knowledge/index.md` first — always
2. Identifies which chapters are relevant and loads only those
3. Loads specific detail pages if needed
4. Asks clarifying questions if anything is unclear (max 4 at a time, waits for answers)
5. Creates a work item at `.spec/work/active/{id}.md` with:
   - Refined description and why
   - Which knowledge files were referenced
   - Questions asked and answers received
   - Design approach and reasoning
   - Step-by-step implementation plan
   - Risks and edge cases
   - Success criteria
   - Files to change

**Complexity is self-assessed:**

| Level | Meaning |
|-------|---------|
| `minimal` | 1 file, < 30 min, no new patterns |
| `small` | 2–3 files, uses existing patterns, < 2 hours |
| `medium` | 3–8 files, crosses 2+ domains, 2–8 hours |
| `large` | Many files, architectural decisions, > 1 day |

After creating the work item, the AI stops and waits. It does not write code. Review the plan before proceeding.

---

### 2. Challenge the plan — /challenge

Run `/challenge` to stress-test the plan before writing code. Strongly recommended for medium and large work items.

```
/challenge
```

The AI evaluates the plan across six dimensions and produces a challenge report:

| Dimension | What it checks |
|-----------|---------------|
| **Assumption risks** | What breaks if the plan's assumptions are wrong? |
| **Missing edge cases** | Empty states, concurrent actions, invalid input, unexpected user behavior |
| **Knowledge conflicts** | Does the plan contradict documented conventions or patterns? |
| **Test gaps** | What changes have no way to verify they're correct? |
| **Scope and complexity** | Is the complexity rating accurate? Any hidden follow-ons? |
| **Simpler alternatives** | Could the same outcome be achieved with less? |

**Output example:**
```
# Challenge Report — add-valid-status-chip

## Findings

### 1. Assumption risks
Medium — Plan assumes `documents.state.valid` is always a boolean, but it may be
`undefined` before the first parse completes. Steps 1 and 3 need null guards.

### 2. Missing edge cases
- What does the chip show while the document is being parsed? (loading state)
- What if the active file has no diagnostics key yet in the store?

### 3. Knowledge conflicts
None identified — plan is consistent with documented patterns.

...

## Before implementing — address these
1. Add null guard for `valid` (could be undefined pre-parse)
2. Decide on loading/pending state for the chip
```

The AI produces the report and stops. It does not revise the work item — you decide which concerns to address. Update `.spec/work/active/{id}.md` if you change the plan.

---

### 3. Implement

With the plan reviewed (and optionally challenged), implement the changes.

The work item at `.spec/work/active/{id}.md` is your reference throughout implementation. It tells you exactly which files to change and what each change is.

There are no restrictions on how you implement — use AI code generation, write manually, or both. The spec kit doesn't get involved during implementation, only before and after.

---

### 4. Complete and archive — /complete

When implementation is done, run `/complete` to close the work item and update the knowledge base.

```
/complete
```

**What the AI asks:**
> "Before archiving, two quick questions:
> 1. Did the implementation differ from the plan? Any surprises?
> 2. Did you discover anything about the codebase worth adding to the knowledge base?"

Answer both. Then the AI:

1. **Updates the knowledge base** — if you discovered something (a new pattern, undocumented behavior, architectural fact), it adds it to the relevant chapter with a traceability note pointing back to this work item
2. **Archives the work item:**
   - Minimal/small items → appended to `work/archive/completed-tasks.md`
   - Medium/large items → their own `work/archive/{id}.md` file
3. **Updates the archive index** — adds a row to `work/archive/README.md` with a link to the archived item
4. **Removes the active work item** from `work/active/`

**Archive index (work/archive/README.md) example:**

| Date | Type | Title | Summary | Detail |
|------|------|-------|---------|--------|
| 2026-08-10 | feature | [add-valid-status-chip](./work/archive/add-valid-status-chip.md) | Added valid/invalid chip to editor status bar | [→](./work/archive/add-valid-status-chip.md) |
| 2026-08-10 | fix | [fix-diagnostics-clear](./work/archive/completed-tasks.md#fix-diagnostics-clear) | Fixed diagnostics not clearing on valid document | [→](./work/archive/completed-tasks.md#fix-diagnostics-clear) |

Everything is traceable: the archive links to the knowledge, and the knowledge links back to the archive.

---

## Code review — /code-review

Use `/code-review` to review a pull request and produce structured, copy-paste-ready GitHub comments.

**Input options:**

```
/code-review https://github.com/org/repo/pull/42
```
```
/code-review feature/add-valid-chip
```

No input — the AI asks you to provide a PR URL or branch name.

**What the AI does:**
- Fetches the diff (via GitHub MCP for PR URLs, or `gh` CLI for branch names)
- Reads the relevant knowledge chapters for the changed files
- Produces a full review with an overall verdict and per-comment blocks

**Comment format (copy-paste ready for GitHub):**
```
**`src/components/Editor/EditorSidebar.tsx:87`** — [Architecture]
> `const [valid, setValid] = useState(false)`

**Concern:** This creates local component state for document validity, but
`documents.state.ts` already tracks `valid` as part of the parsed document
state. This creates two sources of truth.

**Suggestion:** Use `useDocumentsState(s => s.documents[activeUri]?.valid)`
instead of local state.

**Explanation:** Per the state chapter knowledge, all document-related state goes
through Zustand stores — not component-local state. This keeps validity reactive
to parser updates and consistent across components that read the same document.
```

Each comment has:
- **File and line** — precise location
- **Category** — Bug | Architecture | Style | Test | Security | Performance | Docs
- **The exact code** — what you're commenting on
- **Concern** — what is wrong and why
- **Suggestion** — what to do instead
- **Explanation** — context the author needs if the concern isn't self-evident

The review ends with "Must fix before merging" and "Nice to have (non-blocking)" summaries.

---

## The knowledge structure

The knowledge base in `.spec/knowledge/` is the core of this spec kit. It is what makes AI work context-efficient.

**The book metaphor:**
- `index.md` is the **table of contents** — always read first
- `{domain}/README.md` files are **chapters** — read only the relevant ones
- `{domain}/{topic}.md` files are **detail pages** — loaded only when specifically needed

**For a small bug fix**, the AI loads:
- `index.md` (~50–100 lines)
- 1 chapter README (~50–100 lines)
- Total: ~200 lines of context

**For a large feature**, the AI loads:
- `index.md`
- 2–3 chapter READMEs
- 2–3 detail pages
- Total: ~500–800 lines of context

Compare this to a single monolithic constitution file that every request must load in full (commonly 5,000–15,000+ lines for large brownfield projects). The reduction in token cost is 5–30x for most tasks.

**Building depth incrementally:**

The initial `/discover` creates the index, overview, deploy, and chapter-level READMEs. It does not go deeper. Depth is added on demand:

```
/discover --deep parser
```

This adds `knowledge/parser/spectral-rules.md`, `knowledge/parser/schema-parsers.md`, etc. — detailed pages that future work in that area can load without re-exploring.

**Adding design principles:**

Open `.cursor/rules/principles.mdc` and add your project's non-negotiable design decisions:

```markdown
**State lives in Zustand stores**
All persistent UI and domain state goes through dedicated Zustand stores.
Do not use local component state for anything shared or persistent.

*Why:* Keeps state predictable, testable, and shareable across components.
*Where:* `src/state/*.state.ts`
*Exception:* Truly transient UI state that doesn't survive remounts (e.g. hover).
```

These principles are referenced during `/start` (planning) and `/challenge` (stress-testing). When a plan conflicts with a principle, the AI flags it.

---

## Command reference

| Command | When to use | Key options |
|---------|------------|-------------|
| `/discover` | First-time setup, refresh knowledge, answer a knowledge question | `--refresh`, `--refresh {domain}`, `--deep {domain}` |
| `/start [description]` | Begin any task or feature | Just describe what you want — raw ideas are fine |
| `/challenge` | Stress-test the active plan before implementing | No options — reads the active work item automatically |
| `/complete` | Archive finished work and update knowledge | No options — finds the active work item automatically |
| `/code-review [PR URL or branch]` | Review a pull request | PR URL (GitHub MCP), branch name (`gh` CLI), or no arg (AI asks) |

**Agents** (invoked automatically or on request):

| Agent | Role |
|-------|------|
| `architect` | Reviews plan for architectural soundness — does it respect layer boundaries, existing patterns, dependency health? |
| `challenger` | Powers `/challenge` — asks hard questions across six dimensions |

**Rules** (always active, no invocation needed):

| Rule | What it enforces |
|------|-----------------|
| `workflow.mdc` | One step per turn, no skipping, no code during planning |
| `knowledge.mdc` | Always start from index, load only what's needed, mark gaps as TBD |
| `principles.mdc` | Project-specific design principles flagged during planning and review |

---

## Design principles of this kit

**Context-efficient by default** — The AI never loads what isn't needed for the current task. Knowledge is navigated, not dumped.

**Brownfield-honest** — Knowledge files can say TBD. Observed gaps are marked explicitly rather than papered over with guesses.

**One flow, variable depth** — `/start` handles everything from a typo fix to a month-long feature. The depth and formality scale to the work.

**Guardrails explain, they don't block** — `principles.mdc` describes the why behind each principle. Departures are documented and justified, not prohibited.

**Incremental knowledge** — The knowledge base starts shallow (index + chapters) and goes deeper on demand. It doesn't need to be complete to be useful.

**Everything is traceable** — Archive items link to knowledge pages they updated; knowledge pages link back to the work that changed them. You can always answer "why is this here?" and "what changed this?"

**Copy-paste ready outputs** — Code review comments are formatted for direct paste into GitHub. Work items are immediately actionable. Nothing needs post-processing.
