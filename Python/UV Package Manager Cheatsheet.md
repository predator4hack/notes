

## Overview

UV is a blazingly fast Python package installer and resolver written in Rust. Think of it as a drop-in replacement for pip, but significantly faster - often 10-100x faster for most operations.

## Installation

```bash
# Linux/macOS
curl -LsSf https://astral.sh/uv/install.sh | sh

# Windows
powershell -c "irm https://astral.sh/uv/install.ps1 | iex"

# With pip
pip install uv
```

## Core Philosophy

UV combines the functionality of multiple tools:

- **pip** (package installation)
- **pip-tools** (dependency resolution)
- **virtualenv** (environment creation)
- **pyenv** (Python version management)

## Essential Commands

### Package Installation

```bash
# Install a package (like pip install)
uv pip install requests

# Install from requirements.txt
uv pip install -r requirements.txt

# Install specific version
uv pip install "django>=4.0,<5.0"

# Install with extras
uv pip install "fastapi[all]"
```

**Key difference from pip**: UV resolves dependencies much faster by using a more efficient algorithm and caching aggressively.

### Virtual Environments

```bash
# Create a virtual environment
uv venv

# Create with specific Python version
uv venv --python 3.11

# Create in custom location
uv venv myenv

# Activate (same as standard venv)
source .venv/bin/activate  # Linux/macOS
.venv\Scripts\activate     # Windows
```

**Intuitive example**: If you're starting a new Flask project:

```bash
mkdir my-flask-app && cd my-flask-app
uv venv
source .venv/bin/activate
uv pip install flask
```

### Dependency Management

```bash
# Compile requirements (like pip-compile)
uv pip compile requirements.in -o requirements.txt

# Sync environment to match requirements exactly
uv pip sync requirements.txt

# Upgrade all packages
uv pip compile requirements.in --upgrade
```

**Practical scenario**: You have a `requirements.in` file:

```
# requirements.in
flask
requests>=2.28
```

Running `uv pip compile requirements.in` generates a fully pinned `requirements.txt` with all transitive dependencies locked to specific versions - ensuring reproducible builds.

### Python Version Management

```bash
# Install a Python version
uv python install 3.12

# List available Python versions
uv python list

# Use specific Python for a project
uv venv --python 3.11
```

## Project Initialization (UV's Newer Feature)

```bash
# Initialize a new project with pyproject.toml
uv init my-project

# Add a dependency
uv add requests

# Add a dev dependency
uv add --dev pytest

# Run a command in the project environment
uv run python script.py

# Run tests
uv run pytest
```

**Complete workflow example**:

```bash
# Start a new data science project
uv init data-analysis
cd data-analysis

# Add dependencies
uv add pandas numpy matplotlib
uv add --dev jupyter pytest

# Run Jupyter
uv run jupyter notebook

# Your dependencies are automatically tracked in pyproject.toml
```

## Key Features & Why They Matter

### 1. Speed

UV uses Rust and aggressive caching. Installing numpy might take 30 seconds with pip but 2 seconds with uv.

### 2. Resolution Cache

UV caches dependency resolution results. If you've resolved `requests==2.31.0` once, it never needs to do it again.

### 3. Universal Lock Files

When you use `uv pip compile`, it creates lock files that work across platforms (unlike pip-tools which can have platform-specific issues).

### 4. Built-in Virtual Environments

No need for separate `virtualenv` or `venv` - it's integrated.

## Common Workflows

### Starting a new Python project

```bash
uv init myproject
cd myproject
uv add flask sqlalchemy
uv run python app.py
```

### Migrating from pip

```bash
# Instead of:
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt

# Use:
uv venv
source .venv/bin/activate
uv pip install -r requirements.txt
```

### Updating dependencies

```bash
# Update specific package
uv add --upgrade requests

# Update all packages in requirements.in
uv pip compile requirements.in --upgrade -o requirements.txt
uv pip sync requirements.txt
```

## Advanced Commands

```bash
# Install package without dependencies
uv pip install --no-deps mypackage

# Show what would be installed (dry run)
uv pip install --dry-run requests

# Clear cache
uv cache clean

# Install from local wheel
uv pip install ./dist/mypackage-1.0.0-py3-none-any.whl
```

## Tips & Best Practices

1. **Use `pyproject.toml`**: Modern projects should use UV's project features with `uv init` rather than manual requirements files
    
2. **Pin transitive dependencies**: Use `uv pip compile` to generate fully locked requirements for production
    
3. **Leverage caching**: UV's cache is safe to keep - it dramatically speeds up repeated installs
    
4. **Mix with pip if needed**: UV is mostly compatible - you can fall back to pip for edge cases
    
5. **CI/CD optimization**: UV can reduce CI build times significantly. In GitHub Actions:
    
    ```yaml
    - name: Install dependencies
      run: |
        curl -LsSf https://astral.sh/uv/install.sh | sh
        uv pip install -r requirements.txt
    ```
    

## Quick Reference

|Task|UV Command|
|---|---|
|Install package|`uv pip install <pkg>`|
|Create venv|`uv venv`|
|Activate venv|`source .venv/bin/activate`|
|Lock dependencies|`uv pip compile requirements.in`|
|Sync to lockfile|`uv pip sync requirements.txt`|
|Add to project|`uv add <pkg>`|
|Run script|`uv run python script.py`|
|Install Python|`uv python install 3.12`|

## When to Use UV

UV is particularly valuable when:

- Working with larger projects where installation speed matters
- Setting up CI/CD pipelines
- Managing multiple Python versions across projects
- You want modern Python project management without cobbling together multiple tools
- You need reproducible builds with locked dependencies

## Additional Resources

- [Official UV Documentation](https://docs.astral.sh/uv/)
- [GitHub Repository](https://github.com/astral-sh/uv)

---

**Tags**: #python #package-management #tools #development **Created**: 2026-01-02