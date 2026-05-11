# SKILL: Bootstrap a MkDocs Material Site on GitHub Pages

A repeatable runbook to go from an empty directory to a fully deployed MkDocs Material site on GitHub Pages, with branch protection and automatic deployment on merge.

**Prerequisites:**
- `uv` installed (`curl -LsSf https://astral.sh/uv/install.sh | sh`)
- `gh` CLI installed and authenticated (`gh auth login`)
- `git` configured with name and email

---

## 1. Initialize the Python project

```bash
mkdir <REPO_NAME> && cd <REPO_NAME>
git init -b main
uv init --name <REPO_NAME> --no-readme
```

Add the MkDocs dependencies (the `[imaging]` extra is required for social cards):

```bash
uv add "mkdocs-material[imaging]>=9.5.0" mkdocs-git-revision-date-localized-plugin
```

Also write a `requirements.txt` as a human-readable mirror:

```
mkdocs-material[imaging]>=9.5.0
mkdocs-git-revision-date-localized-plugin>=1.2.0
```

Pin the Python version:

```bash
uv python pin 3.13
```

---

## 2. Configure MkDocs

Create `mkdocs.yml` at the repo root. Replace every `<PLACEHOLDER>` with your values:

```yaml
site_name: <SITE_NAME>
site_url: https://<GH_USER>.github.io/<REPO_NAME>/
site_description: <DESCRIPTION>
site_author: <AUTHOR>
copyright: "Copyright &copy; <YEAR> <AUTHOR>"

repo_url: https://github.com/<GH_USER>/<REPO_NAME>
repo_name: <GH_USER>/<REPO_NAME>
edit_uri: edit/main/docs/

docs_dir: docs
site_dir: site

remote_branch: gh-pages
remote_name: origin

theme:
  name: material
  language: en
  palette:
    - scheme: default
      primary: black
      accent: deep purple
      toggle:
        icon: material/brightness-7
        name: Switch to dark mode
    - scheme: slate
      primary: black
      accent: deep purple
      toggle:
        icon: material/brightness-4
        name: Switch to light mode
  features:
    - navigation.tabs
    - navigation.sections
    - navigation.top
    - navigation.footer
    - search.highlight
    - search.suggest
    - content.code.copy
    - content.code.annotate
    - content.tabs.link

nav:
  - Home: index.md

plugins:
  - search:
      lang: en
  - social:
      cards_layout_options:
        background_color: "#2563eb"  # Tailwind blue-600
  - git-revision-date-localized:
      enable_creation_date: true
      type: timeago
      fallback_to_build_date: true

markdown_extensions:
  - admonition
  - attr_list
  - def_list
  - footnotes
  - md_in_html
  - toc:
      permalink: true
      toc_depth: 3
  - pymdownx.highlight:
      anchor_linenums: true
      line_spans: __span
      pygments_lang_class: true
  - pymdownx.superfences:
      custom_fences:
        - name: mermaid
          class: mermaid
          format: !!python/name:pymdownx.superfences.fence_code_format
  - pymdownx.tabbed:
      alternate_style: true
  - pymdownx.details
  - pymdownx.inlinehilite
  - pymdownx.keys
  - pymdownx.mark
  - pymdownx.tasklist:
      custom_checkbox: true
  - pymdownx.emoji:
      emoji_index: !!python/name:material.extensions.emoji.twemoji
      emoji_generator: !!python/name:material.extensions.emoji.to_svg

extra:
  social:
    - icon: fontawesome/brands/github
      link: https://github.com/<GH_USER>

strict: false
```

---

## 3. Create the docs structure

```bash
mkdir -p docs
```

Create `docs/index.md` with at minimum a top-level heading:

```markdown
# <SITE_NAME>

Welcome.
```

Add any additional pages under `docs/` and register each one in the `nav:` section of `mkdocs.yml`. Pages not listed in `nav:` will not appear in the site navigation.

Verify the site builds and looks correct locally before continuing:

```bash
uv run mkdocs serve
# open http://127.0.0.1:8000
```

---

## 4. Create the GitHub Actions deployment workflow

Create `.github/workflows/docs-deploy.yml`:

```yaml
name: Deploy Docs

on:
  push:
    branches:
      - main

permissions:
  contents: write

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0    # required by git-revision-date-localized plugin

      - name: Configure Git Credentials
        run: |
          git config user.name github-actions[bot]
          # 41898282 is the fixed GitHub user ID for the github-actions[bot] service account
          # (this is the same value for every repository — it is not a personal account ID)
          git config user.email 41898282+github-actions[bot]@users.noreply.github.com

      - name: Install uv
        uses: astral-sh/setup-uv@v5

      - uses: actions/cache@v4
        with:
          key: mkdocs-material-${{ hashFiles('uv.lock') }}
          path: ~/.cache/uv
          restore-keys: mkdocs-material-

      - name: Install dependencies
        run: uv sync

      - name: Deploy to GitHub Pages
        run: uv run mkdocs gh-deploy --force

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true
```

> **Note:** GitHub Actions Ubuntu runners have Cairo pre-installed, so no system-level dependencies are needed for social cards.

---

## 5. Add a .gitignore

```
site/
.venv/
__pycache__/
*.pyc
.DS_Store
```

---

## 6. Create the GitHub repository

First, check whether a remote repo already exists:

```bash
gh repo view <GH_USER>/<REPO_NAME> 2>&1
# If output contains "Could not resolve to a Repository" → repo does not exist → create it
# If it returns repo metadata → repo already exists → skip creation
```

Create the repo if it doesn't exist:

```bash
gh repo create <GH_USER>/<REPO_NAME> --public --description "<DESCRIPTION>"
```

---

## 7. Initial commit and push

```bash
git add .
git commit -m "Initial commit: MkDocs site"
git remote add origin git@github.com:<GH_USER>/<REPO_NAME>.git
git push -u origin main
```

Check that the workflow started:

```bash
gh run list --repo <GH_USER>/<REPO_NAME> --limit 3
```

Wait for the workflow to complete (it creates the `gh-pages` branch):

```bash
gh run watch --repo <GH_USER>/<REPO_NAME> --exit-status
```

---

## 8. Enable GitHub Pages

> Run this **after** the workflow completes, because the `gh-pages` branch must exist first.

```bash
gh api repos/<GH_USER>/<REPO_NAME>/pages \
  --method POST \
  -H "Accept: application/vnd.github+json" \
  -f "source[branch]=gh-pages" \
  -f "source[path]=/"
```

The site will be live at `https://<GH_USER>.github.io/<REPO_NAME>/` within ~1 minute.

---

## 9. Enable branch protection on main

Blocks direct pushes to `main`. All changes must go through a pull request.

```bash
gh api repos/<GH_USER>/<REPO_NAME>/branches/main/protection \
  --method PUT \
  -H "Accept: application/vnd.github+json" \
  --input - << 'EOF'
{
  "required_status_checks": null,
  "enforce_admins": false,
  "required_pull_request_reviews": {
    "dismiss_stale_reviews": false,
    "require_code_owner_reviews": false,
    "required_approving_review_count": 0
  },
  "restrictions": null,
  "allow_force_pushes": false,
  "allow_deletions": false,
  "block_creations": false
}
EOF
```

`required_approving_review_count: 0` means PRs are required but no reviewer approval is needed (suitable for solo repos). Set to `1` for team repos.

---

## 10. Day-to-day contributing flow

Once branch protection is active, all changes follow this flow:

```bash
git checkout -b <branch-name>
# make changes
git push origin <branch-name>
gh pr create --base main --title "<title>" --body "<description>"
gh pr merge --merge           # merges the PR
# → GitHub Actions deploys automatically
```

---

## Key facts to remember

| Thing | Detail |
|---|---|
| Dev server | `uv run mkdocs serve` → `http://127.0.0.1:8000` |
| Add a page | Create `.md` in `docs/` **and** add it to `nav:` in `mkdocs.yml` |
| Deploy trigger | Any push (or merge) to `main` |
| Social cards | Auto-generated at `site/assets/images/social/` — no extra config needed beyond the `social` plugin |
| `fetch-depth: 0` | Required in CI — the `git-revision-date-localized` plugin reads full git history |
| `gh-pages` branch | Managed entirely by `github-actions[bot]` — do not protect it |
