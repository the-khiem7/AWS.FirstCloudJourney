# Repository Guidelines

## Project Structure & Module Organization
This Hugo workspace stores markdown source in `content/`, with event modules such as `content/4-Events-Participated/4.1-AWS Vietnam Cloud Day 2025/`. Visual assets live under `static/`, while `public/` holds generated output -- do not edit it manually. Layout overrides belong in `layouts/`, shared theme components live in `themes/`, `archetypes/` defines default front matter, and `.obsidian/` is only for the team knowledge base.

## Build, Test, and Development Commands
Install dependencies once with `npm install`. Use `npm run dev` to launch `hugo server -D`, enabling drafts and hot reload at http://localhost:1313. Run `npm run build` (wraps `hugo -D --gc --minify`) before submitting to ensure the site renders cleanly and stale artifacts are collected. If submodules look stale, refresh them with `npm run prebuild`.

## Coding Style & Naming Conventions
All session notes live in paired `_index.vi.md` and `_index.md` files. Mirror front matter keys exactly, translating only human-readable strings. Keep files UTF-8 without BOM, prefer two-space indentation inside lists, and stick to ASCII bullets unless the source requires otherwise. Image filenames follow the existing slide-name patternâ€”reuse casing to avoid broken links. Reference `AGENT.md` for translation-specific steps.

## Testing & Verification
We do not ship automated tests, so treat the Hugo build as the primary gate. Run `npm run build` and watch the console for warnings, then spot-check pages via the local dev server. When adding assets, confirm they resolve through the expected `/images/...` path. For large imports, run `hugo --panicOnWarning` to surface front matter or shortcode issues early.

## Commit & Pull Request Guidelines
Commits use concise, sentence-case subjects (see `git log`). Group related edits into a single commit rather than multiple micro changes. Pull requests should describe scope, affected directories, and manual verification steps, and link to any tracking issue or Notion card. Include screenshots or rendered snippets when altering visuals, and highlight new or removed files so reviewers can evaluate publishing side effects.

## Agent-Specific Notes
Automation agents must follow `AGENT.md`, limit scope to requested subdirectories, and avoid touching generated `public/` assets. Always report modified paths with line numbers and call out untracked files in summaries.
