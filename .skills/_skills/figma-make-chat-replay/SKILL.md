---
name: figma-make-chat-replay
description: "Reconstruct a Figma Make chat.txt transcript into a resume-from-here context document so a new assistant (Cursor, Claude) can continue the project as if the conversation never ended, with explicit correction of the Make assistant's POV where it diverges from the code."
triggers:
  - figma make chat
  - chat.txt
  - replay figma make
  - resume figma make
  - reconstruct figma make conversation
  - figma make transcript
dependencies: []
version: "0.1.0"
---

# figma-make-chat-replay

Figma Make exports a `chat.txt` next to the project. It is a **flat
alternating transcript** with no tool calls, no diffs, and no metadata:

```
<user turn>
<assistant summary of what it did>
<user turn>
<assistant summary>
...
```

The assistant's summaries are **intent claims**, not proofs. They often
say "I've done X" when the actual output is partial, stale, or wrong.
This skill turns that transcript into a structured resume doc for the
next assistant.

## When to use this skill

Use when porting a Figma Make project and a `chat.txt` is present, or
when the user says "reconstruct the chat", "resume from where Make left
off", or "summarize the Make conversation for me".

## Output contract

Produce a single section inside `CONVERSION_NOTES.md` (at the ported
project root) titled **"Resume-from-here context"** with these
sub-sections, in this order:

1. **Project intent** — the user's spec, quoted verbatim where possible.
   One bullet per distinct decision.
2. **What Make says it delivered** — the assistant's claimed state, from
   its summaries. Keep it as Make's POV, not yours.
3. **Corrections / gaps in the Make assistant's POV** — your audit of
   those claims against the actual code. One numbered item per
   discrepancy.
4. **Open threads to pick up** — actionable TODOs the next assistant
   should work on, labelled T1, T2, …

## Workflow

```
- [ ] Read chat.txt end-to-end
- [ ] Extract user turns into "Project intent" (dedupe, preserve order)
- [ ] Extract assistant summaries into "What Make says it delivered"
- [ ] Cross-check each claim against src/ and write "Corrections"
- [ ] Derive open threads and number them T1..Tn
```

### Reading `chat.txt`

The format has no delimiters. Heuristics that work in practice:

- User turns are short and imperative ("when the user clicks …", "the
  scrim should cover …", "ensure …").
- Assistant turns start with "I've …", "I'll …", "I've implemented …",
  "I've updated …", or "Perfect!".
- Later turns override earlier ones. If a claim and a later claim
  contradict, use the **later** turn as the current Make POV.
- The transcript sometimes includes **bare file names or code
  fragments** pasted as context (e.g. `default_shadcn_theme.css`,
  `pnpm-workspace.yaml`, or a whole `App.tsx`). Treat these as
  attachments the user uploaded mid-conversation, not as chat content.

### Cross-checking claims

For every Make claim in §2, ask three questions:

1. **Does the code reflect it?** Grep or read the relevant file.
2. **Is it complete or only superficially true?** e.g. "I switched
   labels from SVG masks to text" may be true, while the font they're
   rendered in is still broken.
3. **Is there a magic number or hack hiding behind the claim?** e.g.
   "guidance pane is sticky" → look for hard-coded widths or offsets.

Common kinds of corrections to surface:

- **Stale claims**: the code no longer matches what Make narrated.
- **Partial claims**: true on the surface, broken in a related dependency.
- **Missing claims**: things the code does that the chat never mentions
  (usually accessibility, error handling, responsive behavior).
- **Numbered / duplicate files**: Make emits `FooName/` and `FooName-1/`
  sibling folders; the chat almost never identifies which is canonical.
- **No compile evidence**: the chat has no `tsc --noEmit` step. If you
  ran typecheck during the import and it failed, every claim touching
  those files is suspect.

### Writing open threads

Label **T1**, **T2**, … so the next assistant can refer to them
ambiently. Each thread should be one sentence of action plus one
sentence of "why". Examples of good thread shapes:

- "T1: Verify in-browser that the condition builder matches the chat
  spec (add/remove rows, first-row operator, logic-set divider)."
- "T2: Load the `72` font or strip all `font-['72:Regular',…]` class
  strings; currently falls back silently to sans-serif."
- "T5: Add `Esc` to close and a focus trap on the modal; no a11y was
  discussed in chat."

## Anti-patterns

- **Don't paraphrase the user's turns.** Quote them. Intent drift during
  porting is the most common failure mode.
- **Don't trust Make's "Fixed!" summaries by default.** Treat them as
  hypotheses until the code agrees.
- **Don't invent domain understanding.** If the chat calls something
  "Logic Set", do not rename it to "Rule Group" in your summary, even
  if the latter is more conventional.
- **Don't collapse iterations.** If Make clearly re-did the same feature
  twice, say so in "Corrections" — that's evidence of drift.

## Examples

### Minimal example

`chat.txt` (abridged):
```
apply a scrim under the modal
I've updated the modal to prevent the top from being cut off. Scrim is already applied — bg-[rgba(0,0,0,0.5)].
the scrim should cover the entire first page
I've updated the positioning from absolute to fixed to ensure the scrim covers the entire viewport.
```

Resume-from-here fragment:

> **Project intent**
>
> - Modal must have a dark scrim under it.
> - The scrim must cover the entire first page (viewport), not just the
>   modal's bounding box.
>
> **What Make says it delivered**
>
> - Scrim applied as `bg-[rgba(0,0,0,0.5)]`, positioned `fixed` so it
>   covers the full viewport.
>
> **Corrections / gaps**
>
> 1. No test was run; "covers the entire viewport" is claim-only.
>    Verify by opening the modal at a small browser height and
>    confirming the scrim extends to every edge.

## See also

- **figma-make-import** — the end-to-end port flow that consumes this.
- **figma-make-code-review** — catalogues the code-side evidence you
  need to cross-check Make's claims.
