# Import report — {{PROJECT_SLUG}}

<!--
  TEMPLATE — fill in all {{PLACEHOLDER}} values.
  This doc is the stakeholder-facing deliverable. Readable by designers,
  PMs, and engineers. For internal assistant context, see CONVERSION_NOTES.md.

  Tone:
    - "What was imported" and "What was fixed" sections: plain English,
      minimize jargon. Parenthetical explanations for technical terms.
    - "TypeScript health" and "Appendix": fully technical is OK.
    - "Known issues": always state WHO is impacted.
-->

**Source:** Figma Make file `{{FIGMA_FILE_ID}}` / "{{HUMAN_PROJECT_NAME}}"
**Ported to:** `Code Conversion Output/Projects/{{PROJECT_SLUG}}/`
**Date:** {{YYYY-MM-DD}}
**Status:** {{Runnable with known issues | Clean | Blocked on ...}}

---

## What was imported

<!--
  1–3 sentences: what the project does, what the main UI flows are.
  Written for someone who hasn't seen the Figma file.
-->

{{PROJECT_OVERVIEW}}

### Source structure

<!--
  Table of key folders/files and what they contain. Include counts
  where useful (e.g. "41 shadcn/ui wrappers", "11 Figma-node folders").
-->

| Folder | Contents |
|---|---|
| `src/app/App.tsx` | {{ROLE}} |
| `src/app/components/ui/` | {{COUNT}} shadcn/ui component wrappers |
| `src/imports/` | {{COUNT}} Figma-node-generated component folders + {{COUNT}} image assets |
| `src/styles/` | {{DESCRIPTION}} |
| {{OTHER_FOLDERS}} | {{DESCRIPTION}} |

### Stack

- React {{VERSION}} + Vite {{VERSION}} + TypeScript {{VERSION}}
- Tailwind CSS v{{VERSION}} (`@tailwindcss/vite`)
- shadcn/ui (Radix primitives)
- {{OTHER_NOTEWORTHY_DEPS}}

---

## What was fixed during the port

<!--
  Numbered list. One item per fix. Pull from CONVERSION_NOTES.md review
  items marked "fixed on port" but rewrite for a non-technical audience.
  Lead each item with a bold one-liner summarizing the user impact.
-->

1. **{{PLAIN_ENGLISH_SUMMARY}}.** {{DETAIL}}

2. **{{PLAIN_ENGLISH_SUMMARY}}.** {{DETAIL}}

…

---

## Known issues (not fixed)

<!--
  Numbered list. Pull from CONVERSION_NOTES.md review items marked
  "flagged, not fixed". For each:
    - Bold one-liner
    - Body (1–2 sentences)
    - *Impact:* who notices (designer, developer, end-user, a11y auditor)
    - *Fix:* one-sentence resolution path
-->

1. **{{SUMMARY}}.**
   {{BODY}}
   *Impact:* {{WHO}}
   *Fix:* {{HOW}}

2. **{{SUMMARY}}.**
   {{BODY}}
   *Impact:* {{WHO}}
   *Fix:* {{HOW}}

…

---

## TypeScript health

- `pnpm dev`: {{boots clean / boots with warnings / fails}}
- `pnpm typecheck`: **{{N}} errors** / clean

<!--
  If errors exist, list each in a table. Keep it brief — one-line
  summaries, not full error messages.
-->

| Error | File | Summary |
|---|---|---|
| {{TS_CODE}} | `{{FILE}}:{{LINE}}` | {{ONE_LINE}} |
| … | … | … |

---

## Next steps

<!--
  Ordered list of recommended actions, most impactful first.
  Pull from the open threads (T1..Tn) in CONVERSION_NOTES.md but
  rewrite for a mixed audience (designer + engineer).
-->

1. {{ACTION}}
2. {{ACTION}}
…

---

## Appendix: file name mapping

<!--
  ALWAYS include this. Designers look for their Figma frame names and
  won't find them without this table. Infer domain meaning from App.tsx
  imports, EditMode.tsx imports, and the chat transcript.
-->

| Generated name | Domain meaning | Used by |
|---|---|---|
| `{{FOLDER}}/` | {{DESCRIPTION}} | `{{CONSUMER_FILE}}` |
| … | … | … |
