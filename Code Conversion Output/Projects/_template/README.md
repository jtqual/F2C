# Code Conversion Output/Projects/_template

This directory defines the standard structure for every ported Figma Make
project. When running the **figma-make-import** skill, copy these
templates into the new project folder and fill them in.

## Files

| File | Purpose | Audience |
|---|---|---|
| `CONVERSION_NOTES.template.md` | Internal context for the AI assistant: resume-from-here + code review | Assistant only |
| `IMPORT_REPORT.template.md` | Stakeholder-facing summary of the port | Designers, PMs, engineers |
| `README.template.md` | Project README with dev setup instructions | Engineers |

## Usage

```bash
cp -R "Code Conversion Output/Projects/_template/" "Code Conversion Output/Projects/<slug>/"
# Rename *.template.md → *.md
mv "Code Conversion Output/Projects/<slug>/CONVERSION_NOTES.template.md" "Code Conversion Output/Projects/<slug>/CONVERSION_NOTES.md"
mv "Code Conversion Output/Projects/<slug>/IMPORT_REPORT.template.md"    "Code Conversion Output/Projects/<slug>/IMPORT_REPORT.md"
mv "Code Conversion Output/Projects/<slug>/README.template.md"            "Code Conversion Output/Projects/<slug>/README.md"
```

The matching Figma Make export should already be at
`Figma Make/Projects/<slug>/` — same `<slug>` in both trees.

Then follow the **figma-make-import** skill checklist to fill each file.
