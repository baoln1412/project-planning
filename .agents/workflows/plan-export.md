---
description: Export business proposal to docx format for stakeholder presentation
---

# /plan-export — Export Business Proposal to DOCX

## Prerequisites

- Pandoc must be installed (`pandoc --version` to verify)
- A completed `business-proposal.md` must exist in the project folder

## Steps

### 1. Identify the project
// turbo
List all folders in `projects/` (excluding `_template`).
If only one project exists, auto-select it. Otherwise ask which project to export.

### 2. Verify the business proposal exists
// turbo
Check that `projects/{{project-slug}}/business-proposal.md` exists and is populated (not just template placeholders).

If it doesn't exist or is still a template, suggest running Phase 8 of the planning flow first.

### 3. Export to docx
// turbo
Run the pandoc conversion:

```bash
pandoc "projects/{{project-slug}}/business-proposal.md" -o "projects/{{project-slug}}/business-proposal.docx" --from markdown --to docx --standalone --toc --toc-depth=2 --metadata title="Business Proposal: {{PROJECT_NAME}}"
```

### 4. Confirm the export
// turbo
Verify the `.docx` file was created and report the file path and size.

Tell the user: "Your business proposal has been exported to `projects/{{project-slug}}/business-proposal.docx`. You can open and edit it in Microsoft Word."

## Notes

- The docx will include a Table of Contents (--toc) for easy navigation.
- Mermaid diagrams in the markdown will NOT render in docx — they appear as code blocks. To include diagrams, export them as images first and embed them in the markdown before running pandoc.
- If pandoc is not installed, run: `winget install --id JohnMacFarlane.Pandoc`
