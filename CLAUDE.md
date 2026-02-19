# CLAUDE.md

Project-level instructions for Claude Code when working on this repository.

## Project overview

"A First Course in Quantitative Economics with Python" — an undergraduate lecture series by Thomas J. Sargent and John Stachurski. Built with Jupyter Book, lectures are MyST Markdown files with executable Python code cells. Live at https://intro.quantecon.org/.

## Key files

- `lectures/_config.yml` — Jupyter Book config (theme, Sphinx extensions, intersphinx, MathJax macros, execution settings)
- `lectures/_toc.yml` — table of contents defining 13 parts and the order of 50 lectures
- `lectures/_static/quant-econ.bib` — BibTeX bibliography for all citations
- `environment.yml` — conda environment spec (Python 3.13, Jupyter Book, Sphinx extensions)
- `lectures/*.md` — the lecture source files (MyST Markdown with Jupytext)

## Build commands

All commands require the `quantecon` conda environment. Use the full path if not activated:

```bash
# Build HTML (output: lectures/_build/html/)
conda run -n quantecon jb build lectures

# Build PDF (requires xelatex)
conda run -n quantecon jb build lectures --builder=pdflatex

# Clean cache (required before full rebuild)
conda run -n quantecon jb clean lectures --all

# Deploy to GitHub Pages
conda run -n quantecon ghp-import -n -p -f lectures/_build/html
```

## Lecture file conventions

Every lecture `.md` file must start with this frontmatter:

```yaml
---
jupytext:
  text_representation:
    extension: .md
    format_name: myst
    format_version: 0.13
    jupytext_version: 1.17.1
kernelspec:
  name: python3
  display_name: Python 3 (ipykernel)
  language: python
---
```

After frontmatter, the structure is:

1. Reference label: `(lecture-name)=`
2. Title: `# Lecture Title` (capitalize all major words)
3. Brief introduction
4. Import code cell
5. Sections with `##` headings (capitalize only first word and proper nouns)

Code cells use this syntax:

````
```{code-cell} ipython3
import numpy as np
```
````

Tags for display control: `:tags: [hide-output]`, `:tags: [hide-input]`

## Style rules

### Python code

- Follow PEP8; capitals acceptable for matrices (e.g., `A`, `B`) to match math notation
- Use Unicode Greek in code: `α`, `β`, `γ`, `σ`, `θ` — never spelled-out names
- Spaces around operators (`a * b`) but not powers (`a**b`)
- Use `lw=2` for all line charts
- No `ax.set_title()` — figures must have no embedded titles
- Figure captions and axis labels: lowercase
- Keep the default matplotlib box; do not hide spines
- Prefer code-generated figures over static images
- Performance timing: `qe.Timer()` context manager or `qe.timeit()`

### Math notation

- Narrative text: use UTF-8 characters (α, β, γ)
- Math environments (`$`, `$$`): use LaTeX (`\alpha`, `\beta`)
- Transpose: `\top` (e.g., `$A^\top$`)
- Matrices: `\begin{bmatrix}...\end{bmatrix}` — never bold vectors or matrices
- Sequences: `\{ x_t \}_{t=0}^{\infty}`
- Multi-line: `\begin{aligned}...\end{aligned}` inside `$$` blocks

### References and links

- Citation: `` {cite}`bibtex-label` `` or `` {cite:t}`bibtex-label` `` for in-text
- BibTeX entries go in `lectures/_static/quant-econ.bib`
- Internal links: `[text](filename)` or `[](filename)` for auto-title
- Cross-series: `` {doc}`series:page_name` `` (intersphinx keys: `intermediate`, `pyprog`, `dle`, `dps`, `eqm`, `stats`, `tools`, `dynam`)

### Exercises

- Use gated syntax (`exercise-start`/`exercise-end`) for exercises with executable code
- Solutions use the `dropdown` class
- Proof directives: `prf:theorem`, `prf:definition`, `prf:proof`, etc.

## Adding a new lecture

1. Create `lectures/new_lecture.md` with the frontmatter template above
2. Add `(new-lecture)=` reference label before the title
3. Add `- file: new_lecture` to the appropriate section in `lectures/_toc.yml`
4. Add any BibTeX entries to `lectures/_static/quant-econ.bib`
5. Place static assets in `lectures/_static/lecture_specific/new_lecture/` or `lectures/datasets/`
6. Build and verify: `jb build lectures`

## Known pitfalls

- **anaconda metapackage**: `anaconda=2025.12` in `environment.yml` is only available with the full Anaconda distribution. On Miniforge/Miniconda, create the env manually with `conda create -n quantecon python=3.13 pip -y` and install dependencies separately.
- **numba**: Must be installed via `conda install -c conda-forge numba` — pip attempts to build llvmlite from source and fails on macOS.
- **Git push size**: The HTML build is ~54MB. Set `git config --local http.postBuffer 157286400` before pushing to `gh-pages`.
- **Pre-existing build warnings**: ~29 warnings about undefined label references in `french_rev.md`, `inequality.md`, `inflation_history.md`, `networks.md`, and `unpleasant.md` are known and pre-existing in the upstream repo.
- **Cache staleness**: After modifying code in a lecture, clean the cache before rebuilding: `jb clean lectures --all`.
- **Execution timeout**: Notebooks have a 10-minute execution timeout (set in `_config.yml`).

## Do not

- Edit anything in `lectures/_build/` — this is generated output
- Commit `lectures/_build/` or `.ipynb_checkpoints/` (already in `.gitignore`)
- Modify `lectures/_config.yml` without understanding the Sphinx/Jupyter Book implications
- Use `ax.set_title()` in matplotlib figures
- Use spelled-out Greek letter names (`alpha`, `beta`) in Python code — use Unicode (`α`, `β`)
- Bold matrices or vectors in LaTeX
- Add bare image files when a code-generated figure is feasible
