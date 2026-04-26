# Figma Make export — drop zone

Each Figma Make export goes in its own folder under
`Figma Make/Projects/<slug>/`. Pick a short, kebab-case `<slug>` (e.g.
`real-time-trigger-proto`) — the matching ported output lives at
`Code Conversion Output/Projects/<slug>/` and **must share the same
slug** so input and output line up.

A typical export contains:

```
Figma Make/Projects/<slug>/
├── chat.txt                        ← the Figma Make conversation
├── <Human project name>/           ← the actual exported project
│   ├── package.json
│   ├── src/
│   └── ...
└── ...
```

This folder is **gitignored**. Only this `_template/` (which you're
reading) is tracked, as a structural placeholder.
