# Copilot Instructions — copilot-cli-workshop

This repo is a **MkDocs Material** static site: a self-guided, 8-module workshop on the standalone GitHub Copilot CLI (`copilot` binary, not the deprecated `gh copilot` extension).

## Build & Dev Commands

Dependencies are managed with `uv`:

```bash
uv sync                          # install dependencies
uv run mkdocs serve              # local dev server at http://127.0.0.1:8000
uv run mkdocs build              # build static site into site/
uv run mkdocs gh-deploy --force  # deploy to gh-pages branch (CI does this automatically)
```

## Architecture

- **`mkdocs.yml`** — single source of truth for site structure: theme, nav, plugins, and Markdown extensions. All pages must be explicitly listed under `nav:`.
- **`docs/`** — all content. Structured as `index.md` + `modules/01-08.md` + `resources/`.
- **`.github/workflows/docs-deploy.yml`** — deploys on every push to `main` using `uv sync` then `uv run mkdocs gh-deploy --force`. Requires `fetch-depth: 0` because the `git-revision-date-localized` plugin reads full git history.
- **`pyproject.toml` + `uv.lock`** — Python deps (only MkDocs Material and the date plugin). `requirements.txt` is a legacy duplicate kept for reference.

## Key Conventions

**Adding a new page:**
1. Create the `.md` file under `docs/`
2. Add an entry to the `nav:` section in `mkdocs.yml` — pages not listed there won't appear in the site navigation

**Markdown features available** (all configured in `mkdocs.yml`):
- Admonitions: `!!! note`, `!!! warning`, `!!! tip`, etc.
- Tabbed content: `=== "Tab name"`
- Mermaid diagrams: fenced code block with ` ```mermaid `
- Task lists: `- [ ]` / `- [x]`
- Keyboard keys: `++ctrl+c++`
- Code blocks with copy button and line annotations (enabled globally)
- Emoji: `:material-check:`, `:fontawesome/brands/github:`, etc.

**Content scope:** Every module follows the standalone `copilot` CLI (v1.0.44+, May 2026). The deprecated `gh copilot suggest/explain` extension (archived Oct 2025) is historical context only — do not add new content around it.

**`strict: false`** is set in `mkdocs.yml`, so build warnings won't fail CI. Don't suppress warnings by relying on this — fix broken links and references properly.

## Contributing

`main` is branch-protected — direct pushes are blocked. The required flow is:

```
git checkout -b my-feature
# make changes
git push origin my-feature
gh pr create --base main
# merge PR → deployment triggers automatically
```

Opening a PR and merging it is all that's needed — there are no required reviewers and no required status checks. The deploy workflow fires on the `push` event that the merge produces.
