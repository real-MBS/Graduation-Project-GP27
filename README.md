# Graduation Project GP27

This project contains a Jupyter notebook demonstrating a Pandapower time-series
simulation.

## Student setup

This project uses [uv](https://docs.astral.sh/uv/) to install Python and all
required libraries. You only need to install uv once on your computer.

### 1. Install uv

#### Microsoft Windows

Open **PowerShell** and run:

```powershell
winget install --id=astral-sh.uv -e
```

If `winget` is unavailable, use the official PowerShell installer instead:

```powershell
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
```

#### Ubuntu or another Linux distribution

Open a terminal and run:

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

If `curl` is not installed on Ubuntu, install it first:

```bash
sudo apt update
sudo apt install curl
```

#### macOS

If you use Homebrew, open Terminal and run:

```bash
brew install uv
```

Without Homebrew, use the official installer:

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

After installation, close and reopen PowerShell or your terminal. Confirm that
uv is available:

```bash
uv --version
```

### 2. Set up the project

Clone or download this repository, open a terminal inside the project folder,
and run:

```bash
uv sync --locked
```

This command automatically:

- installs Python 3.12 if it is not already available;
- creates a private virtual environment in `.venv`;
- installs the exact tested library versions from `uv.lock`.

Do not edit `uv.lock`, install the project libraries manually with `pip`, or
commit the `.venv` folder.

### 3. Run the notebook

Start JupyterLab from the project folder:

```bash
uv run jupyter lab
```

In the browser, open `notebooks/PandaPower_Time_Series.ipynb`. If Jupyter asks
you to choose a kernel, select the Python kernel from this project's `.venv`.

To stop JupyterLab, return to the terminal and press `Ctrl+C`.

### Updating your copy

After receiving or pulling new project changes, run this again:

```bash
uv sync --locked
```

Project maintainers should use `uv add <package>` for runtime libraries or
`uv add --dev <package>` for development tools, then commit both
`pyproject.toml` and `uv.lock`.
