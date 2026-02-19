# A First Course in Quantitative Economics with Python

An undergraduate lecture series on the foundations of computational economics, designed and written by [Thomas J. Sargent](http://www.tomsargent.com/) and [John Stachurski](https://johnstachurski.net/).

**Live site:** <https://intro.quantecon.org/>

The lectures cover topics from economic data analysis and supply-demand models through linear and nonlinear dynamics, probability, optimization, and estimation — all using Python. Each lecture is a MyST Markdown file that contains executable Python code, mathematical notation, and narrative text, built into a website using [Jupyter Book](https://jupyterbook.org/).

## Table of contents

- [A First Course in Quantitative Economics with Python](#a-first-course-in-quantitative-economics-with-python)
  - [Table of contents](#table-of-contents)
  - [Project structure](#project-structure)
  - [Prerequisites](#prerequisites)
  - [Environment setup](#environment-setup)
    - [Option A: Anaconda distribution (recommended)](#option-a-anaconda-distribution-recommended)
    - [Option B: Miniforge or Miniconda](#option-b-miniforge-or-miniconda)
    - [Verify the setup](#verify-the-setup)
  - [Building locally](#building-locally)
  - [Lecture format](#lecture-format)
  - [Adding a new lecture](#adding-a-new-lecture)
  - [Modifying existing lectures](#modifying-existing-lectures)
  - [Style guide summary](#style-guide-summary)
    - [Writing conventions](#writing-conventions)
    - [Code style](#code-style)
    - [Math formatting](#math-formatting)
    - [Figures](#figures)
    - [Exercises and proofs](#exercises-and-proofs)
    - [References and linking](#references-and-linking)
  - [CI/CD pipeline](#cicd-pipeline)
  - [Pull request workflow](#pull-request-workflow)
  - [Related QuantEcon projects](#related-quantecon-projects)

## Project structure

```text
lecture-python-intro/
├── environment.yml              # Conda environment specification
├── README.md                    # This file
├── .github/workflows/           # CI/CD pipelines (build, publish, link check)
├── lectures/
│   ├── _config.yml              # Jupyter Book configuration
│   ├── _toc.yml                 # Table of contents (defines lecture order)
│   ├── _static/
│   │   ├── quant-econ.bib       # BibTeX bibliography
│   │   ├── lecture_specific/    # Images and data used by specific lectures
│   │   └── *.png, *.ico         # Logos and favicons
│   ├── datasets/                # Data files used across lectures
│   ├── intro.md                 # Landing page
│   └── *.md                     # 50 lecture files (MyST Markdown)
└── _notebook_repo/              # Supporting notebook repository config
```

Key files:

- **`lectures/_config.yml`** — Jupyter Book settings: theme, execution mode (`cache`), Sphinx extensions, intersphinx mappings, MathJax macros, and deployment metadata.
- **`lectures/_toc.yml`** — Defines the 13 parts of the lecture series and the order of lectures within each part.
- **`environment.yml`** — Conda environment specification with Python version and all pip-installed build tools.
- **`lectures/_static/quant-econ.bib`** — Bibliography entries referenced by lectures via `{cite}` directives.

## Prerequisites

| Requirement | Notes |
| --- | --- |
| **Conda** | [Anaconda](https://www.anaconda.com/download) or [Miniforge](https://github.com/conda-forge/miniforge) |
| **Python 3.13** | Installed automatically by the conda environment |
| **Git** | For cloning the repository and version control |
| **xelatex** (optional) | Only required for PDF builds; install via `texlive-xetex` |

## Environment setup

### Option A: Anaconda distribution (recommended)

If you use the full Anaconda distribution, the environment file will resolve directly:

```bash
conda env create -f environment.yml
conda activate quantecon
```

This installs Python 3.13, the Anaconda metapackage (which bundles NumPy, SciPy, Matplotlib, Pandas, and other scientific packages), and all pip-based build tools (Jupyter Book, Sphinx extensions, etc.).

### Option B: Miniforge or Miniconda

The `anaconda=2025.12` metapackage in `environment.yml` is not available on conda-forge. If you use Miniforge or Miniconda, create the environment manually:

```bash
conda create -n quantecon python=3.13 pip -y
conda activate quantecon
```

Install the build tools:

```bash
pip install jupyter-book==1.0.4post1 quantecon-book-theme==0.15.1 \
    sphinx-tojupyter==0.6.0 sphinxext-rediraffe==0.3.0 \
    sphinx-exercise==1.2.1 sphinx-proof==0.3.0 \
    sphinxcontrib-youtube==1.4.1 sphinx-togglebutton==0.3.2 \
    sphinx-reredirects==1.1.0
```

Install the scientific computing packages needed to execute lecture notebooks. Install `numba` via conda first (pip builds from source can fail on some platforms), then the rest via pip:

```bash
conda install -n quantecon -c conda-forge numba -y
pip install numpy matplotlib scipy pandas quantecon networkx sympy \
    statsmodels openpyxl xlrd
```

### Verify the setup

```bash
python --version          # Should show Python 3.13.x
jb --version              # Should show jupyter-book 1.0.4.post1
python -c "import numpy"  # Should succeed without errors
```

## Building locally

All build commands should be run from the repository root with the `quantecon` environment activated.

**Build HTML:**

```bash
jb build lectures
```

The output is written to `lectures/_build/html/`. Open `lectures/_build/html/index.html` in a browser to preview.

**Build PDF** (requires xelatex):

```bash
jb build lectures --builder=pdflatex
```

**Clean the build cache** (useful when notebooks need to be re-executed from scratch):

```bash
jb clean lectures --all
```

The first build takes longer because every notebook is executed and cached. Subsequent builds reuse the cache and only re-execute changed notebooks.

## Lecture format

Lectures are written in [MyST Markdown](https://myst-parser.readthedocs.io/) — an extended markdown syntax that supports executable code cells, LaTeX math, cross-references, and Sphinx directives. Each `.md` file begins with a YAML frontmatter block:

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

After the frontmatter, a typical lecture includes:

- **Reference label** — `(lecture-name)=` enables cross-referencing from other lectures
- **Title** — `# Lecture Title`
- **Narrative text** — standard markdown with LaTeX math (`$...$` and `$$...$$`)
- **Code cells** — fenced with `` ```{code-cell} ipython3 `` and `` ``` ``
- **Directives** — `{exercise}`, `{solution}`, `{prf:theorem}`, `{note}`, etc.
- **Citations** — `` {cite}`bibtex-key` `` referencing `quant-econ.bib`

Example code cell:

````text
```{code-cell} ipython3
import matplotlib.pyplot as plt
import numpy as np
```
````

Code cells can include tags for display control:

````text
```{code-cell} ipython3
:tags: [hide-output]

# This cell's output is hidden by default
```
````

## Adding a new lecture

1. **Create the lecture file.** Add a new `.md` file in the `lectures/` directory with the standard frontmatter (copy from an existing lecture like `solow.md`).

2. **Add a reference label.** Place `(your-lecture-name)=` on the line before the title so other lectures can link to it.

3. **Register it in the table of contents.** Edit `lectures/_toc.yml` and add `- file: your-lecture-name` under the appropriate part:

   ```yaml
   - caption: Section Name
     numbered: true
     chapters:
     - file: existing_lecture
     - file: your_new_lecture    # Add here
   ```

4. **Add bibliography entries.** If your lecture cites references, add BibTeX entries to `lectures/_static/quant-econ.bib`.

5. **Add static assets.** If your lecture uses images or data files, place them in `lectures/_static/lecture_specific/your_lecture_name/` or `lectures/datasets/`.

6. **Build and verify.** Run `jb build lectures` and check that your lecture renders correctly, all code cells execute, and cross-references resolve.

## Modifying existing lectures

- **Edit content** directly in the `.md` file — change text, code, or math as needed.
- **Re-execute a single lecture** by clearing its cache and rebuilding:

  ```bash
  jb clean lectures --all
  jb build lectures
  ```

  Alternatively, delete the specific cached notebook from `lectures/_build/.jupyter_cache/` and rebuild.

- **Test code changes** by running the lecture as a Jupyter notebook. You can convert it with Jupytext:

  ```bash
  jupytext --to notebook lectures/your_lecture.md
  jupyter notebook lectures/your_lecture.ipynb
  ```

## Style guide summary

Please read the [QuantEcon Operations Manual](https://manual.quantecon.org/intro.html) for the full style reference. Below is a summary of the most important conventions.

### Writing conventions

- Keep content clear and concise; when choosing between options, prefer the simplest approach
- Use **bold** for definitions and technical terms, *italics* for emphasis
- Lecture titles: capitalize all major words (e.g., "How it Works: Data, Variables and Names")
- Section headings: capitalize only the first word and proper nouns

### Code style

- Follow PEP8; capital letters are acceptable for matrices to match mathematical notation
- Use Unicode Greek letters in code (`α`, `β`, `γ`, `σ`, `θ`) instead of spelled-out names
- Surround operators with spaces (`a * b`), but not powers (`a**b`)
- Use `qe.Timer()` context manager for performance timing and `qe.timeit()` for benchmarking
- Install non-Anaconda packages at the top of the lecture

### Math formatting

- Use UTF-8 unicode characters (α, β, γ) in narrative text; use LaTeX commands (`\alpha`, `\beta`) inside math environments (`$`, `$$`)
- Transpose notation: `\top` (e.g., `$A^\top$`)
- Matrices: always use `\begin{bmatrix}...\end{bmatrix}` with square brackets; never bold matrices or vectors
- Sequences: `\{ x_t \}_{t=0}^{\infty}`
- Multi-line equations: use `\begin{aligned}...\end{aligned}` within `$$` blocks

### Figures

- No embedded titles in figures (avoid `ax.set_title`)
- Captions: lowercase (~5-6 words); axis labels: lowercase
- Use `lw=2` for all line charts
- Keep the default matplotlib box around figures; do not hide spines
- Prefer code-generated figures over static image files

### Exercises and proofs

- Use gated syntax (`exercise-start`/`exercise-end`) for exercises containing executable code
- Solutions should use the dropdown class by default
- Proof directives require the `prf:` prefix (e.g., `prf:theorem`, `prf:definition`)

### References and linking

- Standard citation: `` {cite}`bibtex-label` ``; in-text citation: `` {cite:t}`bibtex-label` ``
- Add new BibTeX entries to `lectures/_static/quant-econ.bib`
- Internal links: `[link text](filename)` or `[](filename)` for auto-title
- Cross-series links: `` {doc}`series:page_name` `` (requires intersphinx config in `_config.yml`)

## CI/CD pipeline

The repository uses five GitHub Actions workflows in `.github/workflows/`:

| Workflow | Trigger | Purpose |
| --- | --- | --- |
| **ci.yml** | Pull requests | Builds HTML, PDF, and notebooks; deploys a Netlify preview |
| **publish.yml** | `publish*` tags | Builds and deploys to GitHub Pages at intro.quantecon.org |
| **collab.yml** | Pull requests | Tests execution in a Google Colab environment |
| **cache.yml** | Weekly (Monday 3am UTC) | Pre-builds and caches notebooks for faster CI runs |
| **linkcheck.yml** | Weekly (Sunday 11pm UTC) | Checks for broken links and opens issues if found |

All CI builds run on Ubuntu with Python 3.13, the full texlive suite, and a 10-minute per-notebook execution timeout.

## Pull request workflow

1. Fork the repository and create a feature branch
2. Make your changes following the [style guide](#style-guide-summary)
3. Build locally to verify (`jb build lectures`)
4. Open a pull request with a detailed description
   - Use checkboxes for multi-task PRs
   - Link related issues; use `fixes #number` to auto-close on merge
5. Mark the PR as **Draft** until it is ready for review
6. Once reviewed and approved, apply the `ready` tag for merging

CI will automatically build a preview of your changes on Netlify.

## Related QuantEcon projects

This is one of several lecture series in the QuantEcon ecosystem:

| Series | URL |
| --- | --- |
| Quantitative Economics with Python (Intermediate) | <https://python.quantecon.org/> |
| Python Programming for Economics and Finance | <https://python-programming.quantecon.org/> |
| Dynamic Linear Economies | <https://dle.quantecon.org/> |
| Dynamic Programming Squared | <https://dps.quantecon.org/> |
| Equilibrium Models | <https://eqm.quantecon.org/> |
| Quantitative Economics with Data | <https://stats.quantecon.org/> |
| Tools and Techniques | <https://tools-techniques.quantecon.org/> |
| Economic Dynamics | <https://dynamics.quantecon.org/> |

Cross-references between series are supported via intersphinx. See the `intersphinx_mapping` section in `lectures/_config.yml` for the configuration.
