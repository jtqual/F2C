---
name: figma-make-a11y-modal
description: "Add the missing accessibility primitives to Figma Make-generated modals: role=dialog, aria-modal, labelled heading, Escape-to-close, focus trap, and initial focus."
triggers:
  - figma make a11y
  - figma make modal accessibility
  - fix modal a11y
  - add focus trap
  - escape to close modal
  - aria-modal figma make
dependencies: []
version: "0.1.0"
---

# figma-make-a11y-modal

Figma Make modals ship without a11y primitives. The component renders,
looks right, and is completely inaccessible by keyboard or screen
reader. The fix is identical across projects; this skill applies it.

## When to use this skill

Use when the code review's "UI hygiene" section flags a modal without
`role="dialog"`, or the user asks to "make the modal accessible", "add
focus trap", or "close on Escape".

## What "done" looks like

A Make-generated modal is accessible when all of these hold:

- The outer container has `role="dialog"` and `aria-modal="true"`.
- The modal is labelled: either `aria-labelledby="<heading-id>"`
  pointing at the title element, or `aria-label="<text>"`.
- A visible close affordance exists and has an accessible name
  (`aria-label="Close"` on an icon-only button).
- **Escape** closes the modal.
- **Tab** and **Shift+Tab** cycle only through focusable elements
  inside the modal (focus trap).
- On open, focus moves to the first focusable element (or the heading
  if that's more appropriate).
- On close, focus returns to the element that opened the modal.
- The page behind the modal is inert: either `inert` attribute on the
  body content, or `aria-hidden="true"` + pointer-events disabled.

## Preferred approach — use a library

If the project has `@radix-ui/react-dialog` already (common via
shadcn/ui — check `src/app/components/ui/dialog.tsx`), wrap the Make
modal's body in a `<Dialog>` and let Radix handle all eight bullets
above. This is the correct answer 90% of the time.

```tsx
import * as Dialog from "@radix-ui/react-dialog";

<Dialog.Root open={open} onOpenChange={setOpen}>
  <Dialog.Portal>
    <Dialog.Overlay className="fixed inset-0 bg-black/50" />
    <Dialog.Content className="…existing Make classes…">
      <Dialog.Title>{title}</Dialog.Title>
      {/* existing Make modal body */}
      <Dialog.Close aria-label="Close">×</Dialog.Close>
    </Dialog.Content>
  </Dialog.Portal>
</Dialog.Root>
```

Check first:

```bash
rg "@radix-ui/react-dialog" package.json src
cat src/app/components/ui/dialog.tsx 2>/dev/null | head
```

If present, use it. Do not re-author focus trap logic by hand when Radix
is a line away.

## Fallback — manual a11y

Use only if no dialog primitive is available and the user declines
adding one.

1. Add attributes to the outer modal div:

   ```tsx
   <div
     role="dialog"
     aria-modal="true"
     aria-labelledby="make-modal-title"
     ref={containerRef}
   >
   ```

2. Give the heading an id matching `aria-labelledby`.

3. Escape handler:

   ```tsx
   useEffect(() => {
     if (!open) return;
     const onKey = (e: KeyboardEvent) => {
       if (e.key === "Escape") onClose();
     };
     document.addEventListener("keydown", onKey);
     return () => document.removeEventListener("keydown", onKey);
   }, [open, onClose]);
   ```

4. Focus trap — the minimal version:

   ```tsx
   useEffect(() => {
     if (!open || !containerRef.current) return;
     const focusables = containerRef.current.querySelectorAll<HTMLElement>(
       'a[href], button:not([disabled]), input:not([disabled]), select:not([disabled]), textarea:not([disabled]), [tabindex]:not([tabindex="-1"])',
     );
     const first = focusables[0];
     const last = focusables[focusables.length - 1];
     first?.focus();
     const onKey = (e: KeyboardEvent) => {
       if (e.key !== "Tab") return;
       if (e.shiftKey && document.activeElement === first) {
         e.preventDefault();
         last?.focus();
       } else if (!e.shiftKey && document.activeElement === last) {
         e.preventDefault();
         first?.focus();
       }
     };
     document.addEventListener("keydown", onKey);
     return () => document.removeEventListener("keydown", onKey);
   }, [open]);
   ```

5. Return focus on close — store `document.activeElement` at open time
   and call `.focus()` on it in the cleanup.

## Verification

After the change:

- Open the modal. Focus should be inside it.
- Press **Tab** until you'd expect to escape the modal. Focus should
  wrap back to the first focusable element, not leave.
- Press **Escape**. Modal closes. Focus returns to the trigger.
- Run axe or the browser's Accessibility panel. Zero violations on the
  dialog itself.

## Do not

- Don't add `tabIndex={-1}` to the container as a shortcut — the screen
  reader will stop announcing the content.
- Don't trap focus with a global keydown listener that stays mounted
  after unmount. Always clean up in the effect's return.
- Don't set `aria-hidden="true"` on the modal itself. It's the
  *background* that's hidden.

## Examples

- User: "The modal can't be closed with Escape." → this skill.
- Code review flags "modal missing role=dialog and focus trap" → this skill.

## See also

- **figma-make-code-review** — where the a11y gap gets flagged.
- **figma-make-refactor** — this skill is pass 2 of that sequence.
