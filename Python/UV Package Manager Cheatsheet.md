

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


With `uv run python -m`:

- UV checks if you're in a project directory
- UV reads `pyproject.toml` and `uv.lock`
- UV ensures the virtual environment exists and is synced
- UV activates the environment
- UV runs Python with your command
- **Startup time: ~150-250ms** (UV overhead + Python startup)

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


# Python Dependency Management Explained

## What is `pyproject.toml`?

`pyproject.toml` is the **modern, standardized configuration file** for Python projects (introduced in PEP 518). Think of it like `package.json` in Node.js - a single source of truth for your project.

### Example:

```toml
[project]
name = "my-awesome-app"
version = "1.0.0"
requires-python = ">=3.10"
dependencies = [
    "requests>=2.28.0",
    "beautifulsoup4>=4.11.0",
    "pandas>=2.0.0",
]

[project.optional-dependencies]
dev = [
    "pytest>=7.0.0",
    "black>=23.0.0",
]
```

## Why Two Different Package Lists?

Python dependency management has evolved, leading to different patterns:

### The Old Way: `requirements.txt`

- Simple text file with exact versions
- Mixes direct and transitive dependencies
- Works directly with pip
- Good for reproducibility but lacks structure

```
# requirements.txt
requests==2.31.0
beautifulsoup4==4.12.2
pandas==2.1.3
certifi==2023.7.22  # transitive dependency
# ... 50+ more dependencies
```

### The New Way: `pyproject.toml`

- Standard format (PEP 621)
- All configuration in one place
- Clear separation of direct dependencies
- Makes your project installable
- Tool configurations included

## Is It Necessary to Have Both?

**No!** Here are the recommended approaches:

### Approach 1: Modern UV Workflow (Recommended)

**Files you need:**

- `pyproject.toml` - your direct dependencies
- `uv.lock` - auto-generated lock file (like package-lock.json)

```bash
# Initialize project
uv init my-project
cd my-project

# Add dependencies (updates pyproject.toml + uv.lock automatically)
uv add requests beautifulsoup4 pandas
uv add --dev pytest black

# Run your app
uv run python app.py
```

**Both files are auto-synced by UV!**

### Approach 2: Legacy/Hybrid Workflow

**Files you need:**

- `requirements.in` - your direct dependencies
- `requirements.txt` - generated lock file

```bash
# Compile lock file from requirements.in
uv pip compile requirements.in -o requirements.txt

# Install locked dependencies
uv pip sync requirements.txt

# Update everything
uv pip compile requirements.in --upgrade -o requirements.txt
```

## How to Keep Them Synced?

### Using Modern UV Projects (Auto-sync):

```bash
# Everything is automatic!
uv add requests          # Updates pyproject.toml AND uv.lock
uv remove requests       # Updates both files
uv lock --upgrade        # Updates all dependencies
```

### Using requirements.in/txt Pattern:

```bash
# Regenerate requirements.txt from requirements.in
uv pip compile requirements.in -o requirements.txt

# Or update to latest versions
uv pip compile requirements.in --upgrade -o requirements.txt
```

### Using Both pyproject.toml and requirements.txt:

```bash
# Generate requirements.txt from pyproject.toml
uv pip compile pyproject.toml -o requirements.txt
```

## The Key Concept: Two Levels of Dependencies

You need **TWO levels** for good dependency management:

1. **High-level dependencies** (what you actually use)
    
    - In `pyproject.toml` or `requirements.in`
    - Example: `requests>=2.28.0`
2. **Locked dependencies** (exact versions of everything)
    
    - In `uv.lock` or `requirements.txt`
    - Example: `requests==2.31.0, certifi==2023.7.22, ...`

This separation lets you:

- Express flexible requirements ("I need requests 2.x")
- Ensure reproducibility (everyone gets exact same versions)

## Complete Example: Web Scraper Project

### Setup:

```bash
uv init price-scraper
cd price-scraper
uv add requests beautifulsoup4 pandas sqlalchemy
uv add --dev pytest black ruff
```

### Your `pyproject.toml`:

```toml
[project]
name = "price-scraper"
version = "0.1.0"
requires-python = ">=3.10"
dependencies = [
    "beautifulsoup4>=4.12.3",
    "pandas>=2.2.1",
    "requests>=2.31.0",
    "sqlalchemy>=2.0.28",
]

[project.optional-dependencies]
dev = [
    "black>=24.2.0",
    "pytest>=8.0.2",
    "ruff>=0.3.0",
]
```

### Daily Workflow:

```bash
# Clone and install
git clone your-repo && cd your-repo
uv sync

# Run app
uv run python scraper.py

# Add new dependency (auto-updates both files)
uv add openpyxl

# Commit both files
git add pyproject.toml uv.lock
git commit -m "Add Excel export"
```

## What Should You Use?

### For New Projects (2024+):

```
✅ pyproject.toml (direct dependencies)
✅ uv.lock (auto-generated lock file)
❌ requirements.txt (not needed)
```

### For Existing/Legacy Projects:

```
✅ requirements.in (direct dependencies)
✅ requirements.txt (generated lock file)
⚠️ pyproject.toml (optional but recommended)
```

## Quick Comparison Table

|Feature|requirements.txt|pyproject.toml + uv.lock|
|---|---|---|
|Direct dependencies only|No (mixed)|Yes (in pyproject.toml)|
|Exact version locking|Yes|Yes (in uv.lock)|
|Auto-sync|Manual|Automatic with UV|
|Project metadata|No|Yes|
|Tool configuration|No|Yes|
|Standard format|Informal|PEP 621 standard|
|Best for|Legacy projects|Modern projects|

## Key Takeaways

1. **`pyproject.toml`** is the modern standard for Python projects
2. You need both **high-level** (your deps) and **locked** (exact versions) files
3. With **UV projects**, syncing is automatic - just use `uv add/remove`
4. With **requirements.in/txt**, manually run `uv pip compile` to sync
5. For new projects, use `pyproject.toml + uv.lock` - it's simpler and automatic

---

**Tags**: #python #dependency-management #packaging #uv #pyproject **Created**: 2026-01-02 **Related**: [[uv-package-manager-cheatsheet]]