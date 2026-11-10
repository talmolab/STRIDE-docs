# STRIDE-docs

Documentation site for [STRIDE](https://github.com/talmolab/STRIDE) — Stride-level Tracking and
Integrated Decision Extraction, the Talmo Lab pipeline for dual cognitive and motor phenotyping of
mice from T-maze video.

Published at https://talmolab.github.io/STRIDE-docs/ (GitHub Pages, source: GitHub Actions). Describes STRIDE 1.0.0.

| What | Where |
|---|---|
| Pages | `docs/` (Markdown) |
| Navigation and theme | `mkdocs.yml` |
| Identity (colors, footer) | `docs/stylesheets/extra.css` |
| Deploy | `.github/workflows/docs.yml` (build + Pages artifact, no bot commits) |

## Local preview

```bash
uvx --with mkdocs-material mkdocs serve          # http://127.0.0.1:8000/STRIDE-docs/
uvx --with mkdocs-material mkdocs build --strict # what CI runs; fails on broken links or nav
```

## Conventions

- Content lives in `docs/`; register every new page in `nav:` in `mkdocs.yml` (the strict build fails otherwise)
  and describe the code as it is, with numbers taken from the stage modules, `config.example.yaml`,
  the `stride init-config` templates and the commands' `--help`.
- Changes reach `main` through pull requests; deployment uses the GitHub Pages "Actions" source, so
  the repository holds no generated branch.

Three sites, do not confuse them: this is the **documentation**. The research **project page**
is `talmolab/STRIDE-page`; the **pipeline** is `talmolab/STRIDE`.
