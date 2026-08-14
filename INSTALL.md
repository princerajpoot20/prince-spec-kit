# Installing Prince Spec Kit

## What gets added to your project

```
your-project/
├── .cursor/
│   ├── commands/         ← Slash commands (discover, start, complete, challenge, code-review)
│   ├── agents/           ← Specialist agents (architect, challenger)
│   ├── rules/            ← Always-on rules (workflow, knowledge, principles)
│   └── templates/        ← Reusable templates for work items and knowledge chapters
└── .spec/
    └── knowledge/        ← Your project's knowledge base (built by /discover)
        └── work/         ← Active and archived work items
```

---

## Steps

### 1. Copy the kit files

Copy `.cursor/` and `.spec/` from this repo into your project root.

```bash
cp -r /path/to/prince-spec-kit/.cursor /your/project/
cp -r /path/to/prince-spec-kit/.spec /your/project/
```

### 2. Configure .gitignore

Add to your project's `.gitignore`:

```
# Spec kit — active work items (optional, remove if you want to commit plans)
.spec/work/active/
```

Knowledge files (`.spec/knowledge/`) and archives (`.spec/work/archive/`) should be committed — they are the project's living documentation.

### 3. Configure .cursorignore (recommended)

Add to `.cursorignore` to keep large archive files out of Cursor's indexing:

```
.spec/work/archive/completed-tasks.md
```

### 4. Run /discover

Open Cursor in your project and run:
```
/discover
```

This is the one-time setup. It runs in two phases:

**Phase 1 — Initialization:**
- Detects the project name from `package.json` (or equivalent manifest)
- Gets the current git SHA
- Prints: "Initializing Prince Spec Kit for: **{your-project-name}**" — confirm or correct
- Updates `principles.mdc` with the project name

**Phase 2 — Knowledge build:**
- Explores the codebase and builds the knowledge files from scratch:
  - `knowledge/index.md` — master navigation
  - `knowledge/overview.md` — project overview
  - `knowledge/deploy.md` — deployment knowledge
  - `knowledge/{domain}/README.md` — a chapter file for each major domain

After this, you're ready to use all commands.

### 5. Populate principles (optional)

Open `.cursor/rules/principles.mdc` and add project-specific design principles.
These are guidelines the AI will reference when planning and reviewing work.
The `/discover` command will suggest candidates based on ADRs and code conventions it finds.

---

## Upgrading

The kit is designed to be copy-paste portable. To upgrade to a newer version, copy updated `.cursor/` files over your existing ones. Your `.spec/` knowledge and work archives are not touched by upgrades.

---

## Notes for brownfield projects

- Run `/discover` even if the project is partially documented — it will capture what exists and mark gaps explicitly as TBD
- Principles don't need to be complete on day one; add them as you discover them through real work
- The knowledge base is a living document — it gets better the more you use the kit
