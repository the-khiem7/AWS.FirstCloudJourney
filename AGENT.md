# Agent Translation Playbook

This repository keeps paired Markdown files for every Cloud Day session. Each directory under `content/7-AWS Vietnam Cloud Day 2025/` contains:

- `_index.vi.md` - Vietnamese source material
- `_index.md` - English translation target

Follow the steps below whenever a translation update is requested.

## 1. Scope & Prep
- Work only inside the specific subfolders that the user names; ignore the parent `_index.*` files at `content/7-AWS Vietnam Cloud Day 2025` unless explicitly asked.
- Confirm both `_index.vi.md` and `_index.md` exist in each target subfolder before making changes.
- Use UTF-8 without BOM when writing files and keep content ASCII-only unless the source already relies on non-ASCII characters (for example, image filenames containing an en dash).

## 2. Translation Workflow
- Mirror the front matter from `_index.vi.md` into `_index.md`. Translate human-readable fields such as `title`, but keep structural keys and values (dates, weights, tags, shortcodes) identical unless the user instructs otherwise.
- Translate all body content from Vietnamese to English while preserving:
  - Section headings, lists, paragraphs, and callouts
  - Code fences, inline code, and pseudo diagrams (for example, ```cli blocks)
  - Image references and filenames
- If the source uses emoji or other non-ASCII bullets, replace them with simple ASCII equivalents (e.g., `### - Session Name`).
- Maintain the same ordering of sections, separators (`---`), and spacing so the Notion export remains aligned.

## 3. Validation Checklist
- Re-read each translated section to ensure terminology stays consistent with AWS vocabulary and the original intent.
- Run `git status --short` to verify only the expected `_index.md` files changed.
- Spot-check at least one heading per file with `Select-String` (or similar) to confirm the content matches the intended English translation.

## 4. Deliverables
- Report the updated file paths with line references when summarizing work.
- Highlight any files that are newly created or still untracked so the user can add them before committing.

Adhering to this playbook keeps the bilingual content synchronized and avoids regression in the Cloud Day documentation.
