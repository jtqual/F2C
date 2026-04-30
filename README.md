# F2C — Figma Make to Cursor

> **Cloning?** Use the `clone-me` branch (the stable, release-tagged branch). `main` is the development trunk and may be ahead of what's been tested end-to-end:
>
> ```bash
> git clone -b clone-me https://github.com/jtqual/F2C.git
> ```

A small workspace for turning **Figma Make** prototypes into projects that
a developer (or an AI coding assistant in Cursor or Claude) can actually
work with.

## What this is for

Figma Make is great at getting from an idea to a running prototype fast.
You describe what you want in a chat, and Make writes the code. But when
you download that project and try to hand it off, there's usually a gap:
fonts don't load, the TypeScript config is missing, file names are Figma
node IDs, and the AI's "I've fixed it" messages don't always match what's
actually in the code.

This repo is the bridge across that gap. It holds the tools, notes, and
repeatable steps that make the handoff smoother — so you can spend your
time on the design, not on wrestling the export into shape.

## Who this is for

- **UX designers** using Figma Make who want their prototypes to survive
  the trip out of Figma.
- **Designers handing off to developers** (or to Cursor/Claude) who want
  the receiver to land on something that runs and is legible.
- **Anyone curious** what a "real" Figma Make export looks like once you
  open the hood.

## How it's organized

```
F2C/
├── Figma Make/Projects/             ← drop your Figma Make exports here (gitignored, your workspace)
│   └── _template/                   ← tracked: structural placeholder + drop-zone README
├── Code Conversion Output/Projects/ ← the ported, runnable versions (gitignored, your workspace)
│   └── _template/                   ← tracked: doc-standard templates (CONVERSION_NOTES, IMPORT_REPORT, README)
├── resources/                       ← fonts and shared assets the ports need
└── .skills/                         ← the repeatable steps, written as "skills" an AI can run
```

Each project gets the **same slug** in both trees: a Figma Make export
at `Figma Make/Projects/<slug>/` lines up with its ported version at
`Code Conversion Output/Projects/<slug>/`. Both trees are gitignored
(only the `_template/` folder under each is tracked), so they're pure
scratch space for running, tweaking, and demoing the design.

## The skills

Most of the value here is in the **skills** — short, portable playbooks
that automate the mechanical parts of a Figma Make handoff. When you open
this repo in Cursor or Claude, the assistant can run these for you:

- **`figma-make-import`** — takes a fresh Make export and gets it to a
  running state (fixes the package setup, adds TypeScript config,
  removes Make's pnpm artifacts since F2C runs on npm, etc.).
- **`figma-make-sap72-font`** — wires SAP's "72" typeface (the family
  most Make exports reference but don't ship) using the .ttf files
  bundled under `resources/typefaces/`.
- **`figma-make-chat-replay`** — reads Make's `chat.txt` and turns it into
  a clean "here's what the designer asked for" document that survives the
  handoff.
- **`figma-make-code-review`** — looks for the recurring rough spots in
  Make's output (duplicate components with subtle drift, absolute
  positioning, accessibility gaps) and flags them.
- **`figma-make-import-report`** — writes a final report of what was
  fixed during the port and what's still open.

You don't need to learn these — just ask your assistant to "import this
Figma Make project" or "review this Make export" and it will pick the
right one.

## A few things worth knowing

A lot of what's in this repo comes from lessons learned on the first real
port. The full story is in `Code Conversion Output/Projects/POST_MORTEM.md`
(which lives in your local `Code Conversion Output/Projects/` folder and
is gitignored). The short version:

- **Make's "I've done it" is intent, not proof.** The chat is a great
  record of what you asked for, but cross-check the code before calling
  something done.
- **Download and run the project locally before demoing.** Make's preview
  may hide issues (missing fonts, viewport responsiveness) that appear
  once the code leaves Figma.
- **Your chat is the spec.** When handing off, share the chat alongside
  the code — it's the clearest record of design intent.
- **Expect a small porting step.** The skills here handle most of it;
  plan for a short refactor pass after to get the prototype into shape.

## Getting started

1. Drop your Figma Make export into `Figma Make/Projects/<your-slug>/`
   (pick a short kebab-case slug — the ported version will use the same
   one).
2. Open this repo in Cursor or Claude Code.
3. Ask the assistant to "import this as a new F2C project" — it will pick
   up the `figma-make-import` skill and walk through the port.
4. The ported, runnable version lands in
   `Code Conversion Output/Projects/<your-slug>/`.

That's it. The goal is to make the boring parts boring, so the interesting
parts — the design, the interactions, the feedback — get the attention.

## Branches

| Branch | Role |
|--------|------|
| **`clone-me`** | The branch end users clone. Fast-forwarded from `main` only after a release is tested end-to-end and tagged. **Default branch** on GitHub. |
| **`main`** | Development trunk. May contain unreleased work. Maintainers commit here. |
| **feature branches** | Short-lived. Merged into `main` and deleted. |

**Release flow (maintainers):** merge feature → `main`, smoke-test, tag `vX.Y.Z` on `main`, fast-forward `clone-me` to that tag, push. Never commit directly to `clone-me`.

```bash
# on main, after merging features
git tag -a vX.Y.Z -m "..."
git push origin main vX.Y.Z

# fast-forward clone-me to that tag and push
git checkout clone-me && git merge --ff-only main && git push
```
