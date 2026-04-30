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

`<output-slug>` below is the kebab-ascii destination computed from the
input folder name (e.g. `AI Scoring` → `ai-scoring`); see
`figma-make-import` preflight #3 for the rule.

```bash
cp -R "Code Conversion Output/Projects/_template/" "Code Conversion Output/Projects/<output-slug>/"
# Rename *.template.md → *.md
mv "Code Conversion Output/Projects/<output-slug>/CONVERSION_NOTES.template.md" "Code Conversion Output/Projects/<output-slug>/CONVERSION_NOTES.md"
mv "Code Conversion Output/Projects/<output-slug>/IMPORT_REPORT.template.md"    "Code Conversion Output/Projects/<output-slug>/IMPORT_REPORT.md"
mv "Code Conversion Output/Projects/<output-slug>/README.template.md"            "Code Conversion Output/Projects/<output-slug>/README.md"
```

The matching Figma Make export lives at
`Figma Make/Projects/<input-name>/` — same project, intentionally
different folder names. The input name preserves whatever the user
dropped; the output slug serves the toolchain.

Then follow the **figma-make-import** skill checklist to fill each file.
