# Virtual Environments and Package Management in Python

Modern Python development relies on isolated environments and reproducible package management. Without proper isolation and dependency pinning, projects break due to conflicting library versions, OS-managed Python corruption, and inconsistent deployments.

---

## Table of Contents

1. [Why Virtual Environments?](#1-why-virtual-environments)
2. [`venv` Module (Creating, Activating, Deactivating)](#2-venv-module-creating-activating-deactivating)
3. [`virtualenv` (Third-Party)](#3-virtualenv-third-party)
4. [`pip` — Python Package Installer](#4-pip--python-package-installer)
5. [`pip` Commands (`install`, `uninstall`, `list`, `freeze`, `show`, `search`)](#5-pip-commands-install-uninstall-list-freeze-show-search)
6. [`requirements.txt` (Creating, Using)](#6-requirementstxt-creating-using)
7. [`pip install` from Different Sources (PyPI, Git, Local)](#7-pip-install-from-different-sources-pypi-git-local)
8. [`pipenv` Overview](#8-pipenv-overview)
9. [`poetry` Overview](#9-poetry-overview)
10. [`conda` Overview](#10-conda-overview)
11. [`pyproject.toml`](#11-pyprojecttoml)
12. [Publishing Packages to PyPI](#12-publishing-packages-to-pypi)
13. [Version Specifiers and Dependency Resolution](#13-version-specifiers-and-dependency-resolution)
14. [Best Practices for Dependency Management](#14-best-practices-for-dependency-management)

---

## 1. Why Virtual Environments?

By default, Python installs third-party packages into a single global `site-packages` directory. Installing everything globally creates several serious problems:

- **Version conflicts:** Project A requires `django==3.2`, while Project B requires `django==5.0`. A single global environment cannot hold two versions of the same package simultaneously.
- **System Python protection:** Linux and macOS often depend on the system Python for OS utilities. Installing or upgrading global packages can break system tools (modern distros enforce PEP 668 `externally-managed-environment` errors to prevent this).
- **Reproducibility:** When everything lives in one global pool, `pip freeze` outputs dozens of unrelated packages from other projects, making it hard to know what a specific project actually needs.
- **Clean upgrades & testing:** You can test a new Python or library version in a disposable environment without risking existing projects.

### How Virtual Environments Work Under the Hood

A virtual environment is simply a self-contained directory containing:
- A symlink or copy of the Python interpreter binary.
- Its own isolated `site-packages/` directory.
- Scripts (`activate`, `pip`, etc.) that temporarily prepend the environment's `bin/` (or `Scripts\`) folder to your shell's `PATH`.
- A `pyvenv.cfg` file pointing back to the base Python installation.

---

## 2. `venv` Module (Creating, Activating, Deactivating)

`venv` is built into the Python standard library (since Python 3.3) and is the recommended default tool for creating lightweight virtual environments.

### Creating a Virtual Environment

```bash
# Create an environment named .venv in the current directory
python3 -m venv .venv

# Create an environment with a specific Python version
python3.12 -m venv .venv

# Create an environment that can still see globally installed packages
python3 -m venv --system-site-packages .venv

# Create an environment without installing pip (minimal)
python3 -m venv --without-pip .venv
```

> **Note:** On Debian/Ubuntu systems, you may need to install the `python3-venv` system package first (`sudo apt install python3-venv`).

### Activating the Environment

Activation updates your shell's `PATH` and `VIRTUAL_ENV` environment variables so that `python` and `pip` resolve to the environment's binaries.

| Platform / Shell | Command |
| :--- | :--- |
| **Linux / macOS (bash, zsh)** | `source .venv/bin/activate` |
| **Linux / macOS (fish)** | `source .venv/bin/activate.fish` |
| **Linux / macOS (csh/tcsh)** | `source .venv/bin/activate.csh` |
| **Windows (Command Prompt)** | `.venv\Scripts\activate.bat` |
| **Windows (PowerShell)** | `.venv\Scripts\Activate.ps1` |
| **Windows (Git Bash)** | `source .venv/Scripts/activate` |

> **PowerShell tip:** If execution policy blocks activation, run:
> `Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser`

### Verifying Activation

Once activated, your shell prompt usually prefixes `(.venv)`. You can verify with:

```bash
which python      # Linux/macOS -> /path/to/project/.venv/bin/python
where python      # Windows
python -c "import sys; print(sys.prefix)"
```

### Deactivating and Deleting

```bash
# Return to the system/global Python environment
deactivate

# Remove a virtual environment completely (just delete the folder)
rm -rf .venv      # Linux/macOS
rmdir /s /q .venv # Windows CMD
```

> **Tip:** You can also invoke an environment without activating it by calling its Python executable directly:
> `.venv/bin/python script.py` or `.venv/bin/pip install requests`. This is especially useful in cron jobs, CI pipelines, and Dockerfiles.

---

## 3. `virtualenv` (Third-Party)

`virtualenv` is the older, third-party predecessor to `venv` that remains actively maintained and widely used.

### Installing `virtualenv`

```bash
python3 -m pip install --user virtualenv
# Or install globally in an isolated CLI tool via pipx:
pipx install virtualenv
```

### Usage

```bash
# Create an environment
virtualenv .venv

# Target a specific Python interpreter
virtualenv -p /usr/bin/python3.11 .venv
virtualenv --python=3.11 .venv

# Activate and deactivate identically to venv
source .venv/bin/activate
deactivate
```

### `venv` vs `virtualenv`

| Feature | `venv` (stdlib) | `virtualenv` (third-party) |
| :--- | :--- | :--- |
| **Installation** | Built into Python 3.3+ | Requires separate installation |
| **Creation speed** | Standard | Faster (uses caching & `app-data` seeding) |
| **Python versions** | Creates env for the interpreter running it | Can discover & target other installed Python versions (`-p`) |
| **Extensibility** | Minimal | Rich plugin & seeding options (`--seeder`, `--no-periodic-update`) |
| **Default choice** | Yes, for most modern Python projects | Useful in CI, multi-version testing (`tox`), or older setups |

---

## 4. `pip` — Python Package Installer

`pip` (*Pip Installs Packages*) is the standard package installer for Python. It fetches packages from the **Python Package Index (PyPI)** or other configured indexes and installs them into the active Python environment.

### Checking and Upgrading `pip`

Always invoke `pip` via `python -m pip` when possible—this guarantees you are using the `pip` tied to that exact Python interpreter.

```bash
# Check pip version and which Python it belongs to
python -m pip --version

# Upgrade pip inside your active virtual environment
python -m pip install --upgrade pip
```

### Understanding Package Formats

- **Source Distribution (`sdist`, `.tar.gz`):** Contains raw source code and build metadata (`pyproject.toml` / `setup.py`). May require a C/Rust compiler on the user's machine if the package has native extensions.
- **Built Distribution (`wheel`, `.whl`):** Pre-built binary/archive format (e.g., `numpy-2.1.0-cp312-cp312-manylinux_2_17_x86_64.whl`). Installs rapidly without compiling code.

### `pip` Configuration & Caching

```bash
# View active pip configuration
python -m pip config list

# Inspect or clear the download cache
python -m pip cache info
python -m pip cache purge
```

---

## 5. `pip` Commands (`install`, `uninstall`, `list`, `freeze`, `show`, `search`)

### `pip install`

Installs one or more packages and their dependencies.

```bash
# Install the latest version
pip install requests

# Install a specific version or range
pip install "django==5.0.2"
pip install "fastapi>=0.110,<0.116"

# Upgrade an already-installed package
pip install --upgrade requests

# Install optional "extras" defined by a package
pip install "fastapi[standard]"
pip install "sqlalchemy[asyncio]"

# Install without caching or without dependencies (advanced)
pip install --no-cache-dir requests
pip install --no-deps mypackage
```

### `pip uninstall`

Removes installed packages (note: `pip uninstall` does **not** automatically remove orphaned transitive dependencies).

```bash
# Uninstall with confirmation prompt
pip uninstall requests

# Uninstall without prompting
pip uninstall -y requests

# Uninstall multiple packages or everything listed in a file
pip uninstall -y requests urllib3
pip uninstall -y -r requirements.txt
```

### `pip list`

Displays installed packages in a human-readable table.

```bash
# List all installed packages
pip list

# List packages that have newer versions available on PyPI
pip list --outdated

# List only top-level packages not depended on by other installed packages
pip list --not-required

# Machine-readable JSON output
pip list --format=json
```

### `pip freeze`

Outputs installed packages in exact `package==version` format suitable for `requirements.txt`.

```bash
# Print pinned dependencies to stdout
pip freeze

# Save pinned dependencies to a file
pip freeze > requirements.txt

# Exclude editable installs or list only local env packages
pip freeze --local > requirements.txt
```

### `pip show`

Displays detailed metadata for one or more installed packages (version, summary, license, location, dependencies, and reverse dependencies).

```bash
pip show requests
```

Example output:

```text
Name: requests
Version: 2.32.3
Summary: Python HTTP for Humans.
Home-page: https://requests.readthedocs.io
Author: Kenneth Reitz
License: Apache-2.0
Location: /home/user/project/.venv/lib/python3.12/site-packages
Requires: certifi, charset-normalizer, idna, urllib3
Required-by:
```

### `pip search` (Deprecated / Disabled)

Historically, `pip search <query>` searched PyPI from the terminal. However, **PyPI permanently disabled the XML-RPC search endpoint** in 2020 due to excessive abusive traffic. Running `pip search` now returns an error (`XMLRPC request failed`).

**Alternatives to `pip search`:**
- Browse or search directly on [https://pypi.org](https://pypi.org).
- Install a third-party CLI tool such as `pip-search` (`pip install pip-search`).
- Query PyPI's JSON API for a known package name: `curl -s https://pypi.org/pypi/requests/json | python -m json.tool | head -n 30`.

---

## 6. `requirements.txt` (Creating, Using)

A `requirements.txt` file lists packages to install via `pip`. It is the simplest, most universally supported way to share dependencies across environments, servers, and Docker containers.

### Creating `requirements.txt`

There are two common conventions:

1. **Logical / direct dependencies (hand-maintained or `requirements.in`):**
   ```text
   # requirements.txt
   fastapi>=0.115,<0.116
   uvicorn[standard]>=0.30
   pydantic~=2.9
   ```
2. **Exact pinned lockfile (`pip freeze`):**
   ```bash
   pip freeze > requirements.txt
   ```
   ```text
   annotated-types==0.7.0
   anyio==4.6.0
   fastapi==0.115.0
   pydantic==2.9.2
   starlette==0.38.6
   ```

### Using `requirements.txt`

```bash
# Install all packages listed in requirements.txt
pip install -r requirements.txt
```

### Splitting Environments (Base, Dev, Prod)

You can include one requirements file inside another using `-r`:

```text
# requirements.txt (production dependencies)
fastapi==0.115.0
uvicorn==0.30.6
```

```text
# requirements-dev.txt (local development & testing)
-r requirements.txt
pytest==8.3.3
ruff==0.6.9
mypy==1.11.2
```

Install development dependencies with:

```bash
pip install -r requirements-dev.txt
```

### Using `pip-tools` (`pip-compile` / `pip-sync`)

For clean separation between direct dependencies and fully pinned lockfiles, `pip-tools` is a popular workflow:

```bash
pip install pip-tools

# Write direct dependencies in requirements.in, then compile exact transitive pins:
pip-compile requirements.in          # Generates requirements.txt
pip-compile --generate-hashes requirements.in  # With SHA256 hashes for security

# Synchronize the active environment to match requirements.txt exactly (adds & removes packages)
pip-sync requirements.txt
```

### Constraints Files (`-c`)

A constraints file restricts versions without forcing those packages to be installed unless requested:

```bash
pip install -r requirements.txt -c constraints.txt
```

---

## 7. `pip install` from Different Sources (PyPI, Git, Local)

`pip` can install packages from many sources beyond the default PyPI index.

### 1. PyPI and Alternate Package Indexes

```bash
# Default PyPI
pip install httpx

# TestPyPI or private artifact registry (Artifactory, Nexus, AWS CodeArtifact)
pip install --index-url https://test.pypi.org/simple/ my-package

# Use a private index as an extra fallback alongside PyPI
pip install --extra-index-url https://pypi.internal.example.com/simple/ internal-lib
```

> **Security Warning:** Be cautious with `--extra-index-url` on private package names—attackers can exploit *dependency confusion* by registering the same package name with a higher version on public PyPI. Prefer configuring a single unified proxy or `--index-url`.

### 2. Git Repositories

Install directly from GitHub, GitLab, or private Git servers (requires `git` on `PATH`):

```bash
# Default branch
pip install git+https://github.com/psf/requests.git

# Specific tag, branch, or commit SHA (recommended for reproducibility)
pip install git+https://github.com/psf/requests.git@v2.32.3
pip install git+https://github.com/psf/requests.git@main
pip install git+https://github.com/psf/requests.git@98b4e38c

# Package located in a subdirectory of a monorepo
pip install "git+https://github.com/org/monorepo.git@v1.0#subdirectory=packages/my-lib"

# Private repo over SSH
pip install git+ssh://git@github.com/org/private-repo.git@v1.2.0

# Named PEP 508 syntax (usable inside requirements.txt or pyproject.toml)
pip install "requests @ git+https://github.com/psf/requests.git@v2.32.3"
```

### 3. Local Directories and Archives

```bash
# Install from a local directory containing pyproject.toml or setup.py
pip install ./my-local-package
pip install /abs/path/to/my-local-package

# Install in editable (development) mode — edits to source files take effect immediately
pip install -e ./my-local-package
pip install --editable .

# Install from a local wheel or source archive
pip install ./dist/my_package-0.1.0-py3-none-any.whl
pip install ./dist/my_package-0.1.0.tar.gz

# Offline / air-gapped installation from a pre-downloaded folder of wheels
pip download -r requirements.txt -d ./wheelhouse
pip install --no-index --find-links=./wheelhouse -r requirements.txt
```

---

## 8. `pipenv` Overview

**Pipenv** combines `pip` and `virtualenv` into a single CLI workflow, introducing deterministic builds via `Pipfile` and `Pipfile.lock`.

### Key Files

- **`Pipfile`**: Human-readable TOML manifest listing direct `[packages]` and `[dev-packages]`.
- **`Pipfile.lock`**: Auto-generated JSON lockfile containing exact resolved versions and cryptographic SHA-256 hashes of every transitive dependency.

### Common Workflow

```bash
# Install pipenv
pip install --user pipenv

# Initialize an environment and install a package
pipenv install requests

# Install a development-only package
pipenv install --dev pytest ruff

# Run a command inside the pipenv virtual environment without manual activation
pipenv run python app.py
pipenv run pytest

# Spawn a shell with the virtual environment activated
pipenv shell

# Install everything strictly from Pipfile.lock (for CI/production)
pipenv sync
pipenv install --deploy --system   # Fail if Pipfile.lock is out of date

# Inspect the dependency tree
pipenv graph
```

### Example `Pipfile`

```toml
[[source]]
url = "https://pypi.org/simple"
verify_ssl = true
name = "pypi"

[packages]
requests = ">=2.31"
flask = "~=3.0"

[dev-packages]
pytest = "*"

[requires]
python_version = "3.12"
```

---

## 9. `poetry` Overview

**Poetry** is a modern, all-in-one dependency management, virtual environment, and packaging tool built around `pyproject.toml` and `poetry.lock`.

### Strengths

- Unified workflow: manages virtual environments, resolves dependencies cleanly, builds wheels/sdists, and publishes to PyPI.
- Uses standard `pyproject.toml` plus a deterministic, cross-platform `poetry.lock`.
- Supports dependency groups (`main`, `dev`, `docs`, etc.).

### Common Workflow

```bash
# Install Poetry (recommended via pipx or official installer)
pipx install poetry

# Create a new project with src layout
poetry new my-project
# Or initialize Poetry in an existing directory
poetry init

# Add runtime and group-specific dependencies
poetry add fastapi pydantic
poetry add --group dev pytest ruff

# Install dependencies from poetry.lock (creates virtualenv automatically)
poetry install

# Install excluding dev dependencies (for production)
poetry install --only main

# Run commands inside Poetry's managed environment
poetry run pytest
poetry run python -m my_project

# Update lockfile to newer allowed versions
poetry update

# Build and publish a package
poetry build
poetry publish
```

### Example Poetry `pyproject.toml` (v1/v2)

```toml
[project]
name = "my-app"
version = "0.1.0"
description = "Sample Poetry project"
requires-python = ">=3.11"
dependencies = [
    "fastapi (>=0.115,<0.116)",
    "pydantic (>=2.9,<3.0)",
]

[tool.poetry.group.dev.dependencies]
pytest = "^8.3"

[build-system]
requires = ["poetry-core>=2.0.0,<3.0.0"]
build-backend = "poetry.core.masonry.api"
```

---

## 10. `conda` Overview

**Conda** is an open-source, language-agnostic package and environment manager widely used in **data science, machine learning, and scientific computing**.

### Why Conda Differs from `venv` + `pip`

- **Manages Python itself:** Conda installs different Python versions directly—no separate system Python installation needed.
- **Manages non-Python binary dependencies:** Conda handles C/C++ libraries, CUDA toolkits, GDAL, FFmpeg, MKL, and R packages cleanly without requiring local compilation.
- **Channels:** Packages come from channels such as `defaults` (Anaconda) or `conda-forge` (community-maintained, most up-to-date).

> **Note:** **Mamba** (`mamba` / `micromamba`) and Conda's built-in `libmamba` solver are fast C++ drop-in replacements for Conda's historically slow dependency solver.

### Common Conda Commands

```bash
# Create an environment with a specific Python version and packages
conda create -n ml-env python=3.12 numpy pandas scikit-learn

# Create from conda-forge channel
conda create -n geo-env -c conda-forge python=3.12 geopandas

# Activate and deactivate
conda activate ml-env
conda deactivate

# List environments and installed packages
conda env list
conda list

# Export environment specification for sharing
conda env export --from-history > environment.yml

# Recreate environment from environment.yml
conda env create -f environment.yml

# Update an existing environment from environment.yml
conda env update -f environment.yml --prune
```

### Example `environment.yml`

```yaml
name: ml-env
channels:
  - conda-forge
  - defaults
dependencies:
  - python=3.12
  - numpy>=2.0
  - pandas>=2.2
  - scikit-learn
  - pip
  - pip:
      - fastapi==0.115.0
```

### Using `pip` Inside Conda Safely

1. Install as many packages as possible with `conda` first.
2. Only use `pip` for packages unavailable on Conda channels.
3. Avoid running `conda install` repeatedly *after* heavy `pip` installs, as Conda does not always track `pip`-only metadata cleanly.

---

## 11. `pyproject.toml`

Defined across **PEP 518**, **PEP 517**, and **PEP 621**, `pyproject.toml` is the unified configuration file for Python projects. It replaces legacy `setup.py`, `setup.cfg`, and `MANIFEST.in` files for most projects and acts as a single home for build settings, metadata, dependencies, and tool configurations (`ruff`, `pytest`, `mypy`, `black`).

### Anatomy of `pyproject.toml`

```toml
# 1. Build System (PEP 517 / PEP 518)
# Tells pip/build which backend builds the package
[build-system]
requires = ["hatchling>=1.25"]
build-backend = "hatchling.build"

# 2. Project Metadata & Dependencies (PEP 621)
[project]
name = "awesome-toolkit"
version = "0.2.0"
description = "A clean utility toolkit for Python projects."
readme = "README.md"
requires-python = ">=3.10"
license = { text = "MIT" }
authors = [
    { name = "Ada Lovelace", email = "ada@example.com" },
]
keywords = ["utilities", "cli"]
classifiers = [
    "Programming Language :: Python :: 3",
    "License :: OSI Approved :: MIT License",
    "Operating System :: OS Independent",
]

# Runtime dependencies
dependencies = [
    "httpx>=0.27,<1.0",
    "click>=8.1",
]

# Optional dependency groups (installed via: pip install "awesome-toolkit[dev,docs]")
[project.optional-dependencies]
dev = [
    "pytest>=8.0",
    "ruff>=0.6",
]
docs = [
    "mkdocs>=1.6",
]

# CLI entry points (creates an executable command `awesome` on PATH)
[project.scripts]
awesome = "awesome_toolkit.cli:main"

[project.urls]
Homepage = "https://github.com/example/awesome-toolkit"
Repository = "https://github.com/example/awesome-toolkit"

# 3. Tool-Specific Settings
[tool.pytest.ini_options]
testpaths = ["tests"]
addopts = "-q"

[tool.ruff]
line-length = 100
target-version = "py310"
```

### Popular PEP 517 Build Backends

- **`setuptools`**: Traditional standard; now supports full `pyproject.toml` config.
- **`hatchling`**: Fast, modern default used by Hatch.
- **`flit_core`**: Minimalist backend for pure-Python packages.
- **`poetry-core`**: Lightweight backend for Poetry-managed projects.
- **`maturin`**: Standard backend for Python packages backed by Rust (`PyO3`).

---

## 12. Publishing Packages to PyPI

Publishing a Python package allows anyone to install it via `pip install <your-package>`.

### Step 1: Structure Your Project (`src` Layout)

The `src/` layout prevents accidental imports from the current working directory during testing:

```text
awesome-toolkit/
├── pyproject.toml
├── README.md
├── LICENSE
├── src/
│   └── awesome_toolkit/
│       ├── __init__.py
│       └── cli.py
└── tests/
    └── test_cli.py
```

### Step 2: Build Distributions (`sdist` and `wheel`)

Use the official `build` frontend tool (`python -m build`):

```bash
python -m pip install --upgrade build twine
python -m build
```

This creates `dist/`:

```text
dist/
├── awesome_toolkit-0.2.0-py3-none-any.whl
└── awesome_toolkit-0.2.0.tar.gz
```

### Step 3: Validate Package Metadata

```bash
python -m twine check dist/*
```

### Step 4: Upload to TestPyPI First

Always test your package on [TestPyPI](https://test.pypi.org/) before publishing to production PyPI (versions cannot be overwritten once uploaded to PyPI).

```bash
# Upload to TestPyPI (will prompt for API token: username = __token__)
python -m twine upload --repository testpypi dist/*

# Verify installation from TestPyPI in a fresh virtual environment
python -m venv /tmp/test-env
/tmp/test-env/bin/pip install --index-url https://test.pypi.org/simple/ \
    --extra-index-url https://pypi.org/simple/ awesome-toolkit
```

### Step 5: Publish to Production PyPI

```bash
python -m twine upload dist/*
```

### Authentication Best Practices

- **API Tokens (`__token__`):** Generate a scoped project token in your PyPI account settings rather than using your account password. Store tokens in `~/.pypirc` or pass via environment variables (`TWINE_USERNAME=__token__`, `TWINE_PASSWORD=pypi-...`).
- **Trusted Publishers (OIDC):** When publishing from GitHub Actions, GitLab CI, or Google Cloud Build, configure PyPI *Trusted Publishing* so CI authenticates with short-lived OIDC tokens—no long-lived secrets needed.

---

## 13. Version Specifiers and Dependency Resolution

Python package versions follow **PEP 440** (typically semantic versioning `MAJOR.MINOR.PATCH`, plus pre-releases like `1.0.0a1`, `1.0.0rc1`, or post-releases `1.0.0.post1`).

### Common PEP 440 Version Specifiers

| Operator | Meaning | Example | Matches |
| :--- | :--- | :--- | :--- |
| `==` | Exact version match | `==2.1.3` | `2.1.3` only |
| `==` with `*` | Prefix wildcard match | `==2.1.*` | `2.1.0`, `2.1.9`, not `2.2.0` |
| `!=` | Exclude specific version(s) | `>=2.0, !=2.1.3` | All `>=2.0` except `2.1.3` |
| `>=`, `<=`, `>`, `<` | Inclusive / exclusive bounds | `>=1.4, <2.0` | `1.4.0` up to `<2.0.0` |
| `~=` | Compatible release | `~=2.1.3` | `>=2.1.3, ==2.1.*` (`<2.2.0`) |
| `~=` (2-part) | Compatible minor release | `~=2.1` | `>=2.1, ==2.*` (`<3.0.0`) |
| `===` | Arbitrary string equality | `===custom-tag` | Exact legacy/non-PEP440 string |

> **Poetry-specific shorthand:** Poetry also supports `^1.2.3` (`>=1.2.3, <2.0.0`) and `~1.2.3` (`>=1.2.3, <1.3.0`) inside its own constraints syntax.

### Environment Markers (PEP 508)

You can conditionally require packages based on OS, Python version, or platform:

```toml
dependencies = [
    "pywin32>=306; sys_platform == 'win32'",
    "tomli>=2.0; python_version < '3.11'",
    "uvloop>=0.20; sys_platform != 'win32' and implementation_name == 'cpython'",
]
```

### How Dependency Resolution Works

When you install packages `A` and `B`, both may depend on a shared library `C`:
- `A` requires `C >= 1.5, < 3.0`
- `B` requires `C >= 2.0, < 2.5`

`pip`'s backtracking resolver (introduced in `pip` 20.3) searches for a version of `C` satisfying all constraints simultaneously (e.g., `C==2.4.0`).

**Resolution Conflicts:**
If `A` requires `C<2.0` and `B` requires `C>=2.0`, no single version of `C` satisfies both. `pip` aborts with a `ResolutionImpossible` error instead of silently installing a broken combination.

**Debugging Conflicts:**
- Inspect requirements with `pip show <pkg>` or `pipdeptree` (`pip install pipdeptree && pipdeptree`).
- Loosen overly strict pins in libraries; resolve conflicts by updating incompatible packages or isolating tools into separate environments.

---

## 14. Best Practices for Dependency Management

1. **Always use a virtual environment per project:** Never install project dependencies into the global system Python. Keep `.venv/` inside the project directory (and add `.venv/` to `.gitignore`).
2. **Distinguish between applications and libraries:**
   - **Libraries** (packages published to PyPI): Use **loose, lower-bounded** version ranges in `pyproject.toml` (e.g., `requests>=2.31,<3.0`) so consumers don't hit version conflicts.
   - **Applications & services** (APIs, web apps, data pipelines): Commit a **fully pinned lockfile** (`requirements.txt` from `pip-compile`, `poetry.lock`, `uv.lock`, or `Pipfile.lock`) so staging, CI, and production run identical code.
3. **Separate runtime, dev, and test dependencies:** Keep linters (`ruff`), type checkers (`mypy`), and test runners (`pytest`) out of production containers to minimize image size and attack surface.
4. **Never commit secrets or tokens:** Keep PyPI tokens and private index credentials in environment variables, credential helpers, or CI secret stores—never inside `requirements.txt` or `pyproject.toml`.
5. **Audit dependencies regularly:**
   - Check for known security vulnerabilities with `pip-audit` (`pip install pip-audit && pip-audit`).
   - Use Dependabot or Renovate to keep dependencies updated cleanly via automated pull requests.
6. **Pin Python version explicitly:** Document the supported Python version in `pyproject.toml` (`requires-python = ">=3.11"`), `.python-version`, or your `Dockerfile`.
7. **Prefer CLI tool isolation with `pipx`:** For global developer CLIs (`black`, `ruff`, `poetry`, `httpie`), install via `pipx` rather than polluting project environments or system Python.
8. **Consider modern fast tooling (`uv`):** Tools like Astral's `uv` provide drop-in replacements for `venv`, `pip`, and `pip-tools` (`uv venv`, `uv pip install`, `uv lock`, `uv sync`) at orders of magnitude faster speeds while staying 100% compatible with Python standards (`pyproject.toml`, PEP 508, PEP 440).

---

## Quick Reference Cheat Sheet

| Task | `venv` + `pip` | `poetry` | `conda` |
| :--- | :--- | :--- | :--- |
| **Create env** | `python -m venv .venv` | `poetry install` | `conda create -n myenv python=3.12` |
| **Activate env** | `source .venv/bin/activate` | `poetry shell` (or `poetry run`) | `conda activate myenv` |
| **Install pkg** | `pip install requests` | `poetry add requests` | `conda install requests` |
| **Install dev pkg** | `pip install -r requirements-dev.txt` | `poetry add -G dev pytest` | `conda install pytest` |
| **Export / Lock** | `pip freeze > requirements.txt` | `poetry lock` | `conda env export > environment.yml` |
| **Reproduce env** | `pip install -r requirements.txt` | `poetry install` | `conda env create -f environment.yml` |
| **Deactivate** | `deactivate` | `exit` | `conda deactivate` |
