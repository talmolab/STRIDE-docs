# STRIDE-docs — the STRIDE documentation site

MkDocs Material site for the STRIDE pipeline (`talmolab/STRIDE`). Private repo, public site once
Pages is enabled. Keep pages short and task-focused; describe the code as it is, never as it
should be. Numbers (thresholds, defaults) come from `config.yaml` and the stage modules.

## Authorship rule (non-negotiable)

Every commit is authored and committed by **Yipeng Li**
`<134640652+LeoMeow123@users.noreply.github.com>` (GitHub: LeoMeow123). Never add
`Co-Authored-By:`, `Claude-Session:` or any other trailer, and never commit with another
identity. The deploy workflow publishes through the Pages Actions source precisely so that no
bot ever commits to this repository (no `gh-pages` branch). Check
`git log --format='%an <%ae>'` before pushing.

## Three sites, do not confuse them

| URL | Repo | What |
|---|---|---|
| talmolab.github.io/STRIDE-docs | **this repo** | documentation (install, guide, how it works) |
| talmolab.github.io/STRIDE-page | `talmolab/STRIDE-page` | research project page |
| github.com/talmolab/STRIDE | `talmolab/STRIDE` | the pipeline |

## Stack and layout

- Content in `docs/`, one folder per nav tab. Navigation in `mkdocs.yml`.
- Identity in `docs/stylesheets/extra.css` (indigo = cognition, emerald = gait; same tokens as
  the project page). Fonts Inter / JetBrains Mono via the theme.
- Diagrams are Mermaid code fences.

```bash
uvx --with mkdocs-material mkdocs serve
uvx --with mkdocs-material mkdocs build --strict   # run before every commit
```

## Adding a page — do all three

1. Create the `.md` under the right `docs/` folder.
2. Register it in `nav:` in `mkdocs.yml` (strict build fails otherwise).
3. Link it from the Home page quick links if a user would look for it there.

## Source of truth for content

`talmolab/STRIDE`: `README.md` (public API), `config.yaml` (every default), `src/stride/cli.py`
(commands), `src/stride/stages/*.py` (thresholds, outputs), `docs/example_run_manifest.yaml`.
The internal `RUNBOOK.md` is a lab SOP, not public API; borrow its gate logic, not its paths.
