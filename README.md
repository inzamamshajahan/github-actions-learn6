# My Data Transformation Project

This project demonstrates a Python application that performs data transformations using Pandas and NumPy. It is structured with industry-standard practices, including robust logging, automated testing, code quality checks, and a CI/CD pipeline for deployment to an AWS EC2 instance.

## Table of Contents

1.  [Project Overview](#project-overview)
2.  [Features](#features)
3.  [Project Structure](#project-structure)
4.  [Core Components & Logic](#core-components--logic)
    *   [`src/main.py`](#srcmainpy)
5.  [Dependency Management](#dependency-management)
    *   [`requirements.txt`](#requirementstxt)
    *   [`requirements-dev.txt`](#requirements-devtxt)
6.  [Tooling and Configuration](#tooling-and-configuration)
    *   [`pyproject.toml`](#pyprojecttoml)
    *   [`.pre-commit-config.yaml`](#pre-commit-configyaml)
    *   [`.gitignore`](#gitignore)
7.  [Testing](#testing)
    *   [`tests/test_main.py`](#teststest_mainpy)
    *   [Code Coverage](#code-coverage)
8.  [Local Development Setup](#local-development-setup)
9.  [Running the Script Locally](#running-the-script-locally)
10. [CI/CD with GitHub Actions](#cicd-with-github-actions)
    *   [Workflow: `.github/workflows/ci-cd.yml`](#workflow-githubworkflowsci-cdyml)
    *   [GitHub Secrets](#github-secrets)
11. [EC2 Deployment Details](#ec2-deployment-details)
12. [Troubleshooting Common Issues](#troubleshooting-common-issues)

## Project Overview

The primary goal of this project is to read data (either from a CSV file or by generating a sample), perform several data transformations (e.g., adding columns, filtering rows based on conditions), and then save the processed data to a new CSV file. The script also incorporates logging to track its execution and any potential issues.

The project emphasizes a modern Python development workflow, including:
*   Separation of source code (`src`) and tests (`tests`).
*   Clear dependency management for runtime and development.
*   Automated code formatting, linting, and type checking.
*   Unit testing with code coverage.
*   Pre-commit hooks to maintain code quality locally.
*   A full CI/CD pipeline to automate testing and deployment.

## Features

*   **Data Transformation:** Script in `src/main.py` uses Pandas and NumPy for data manipulation.
*   **Logging:** Built-in Python `logging` module is used to log script activity to both the console and a file (`data/data_processing.log`).
*   **Dependency Management:**
    *   Runtime dependencies are listed in `requirements.txt`.
    *   Development dependencies are listed in `requirements-dev.txt`.
*   **Tool Configuration:** `pyproject.toml` centralizes configuration for development tools like Ruff, Mypy, Pytest, and Bandit.
*   **Code Quality:**
    *   **Ruff:** Used for extremely fast Python linting and formatting.
    *   **Mypy:** Used for static type checking to catch type errors before runtime.
    *   **Bandit:** Used for finding common security issues in Python code.
    *   **Safety:** Used for checking installed dependencies for known security vulnerabilities.
*   **Testing:**
    *   **Pytest:** Used as the testing framework for writing and running unit tests.
    *   **pytest-cov:** Used for measuring code coverage by tests.
*   **Automation:**
    *   **Pre-commit Hooks:** Configured via `.pre-commit-config.yaml` to automatically run checks (e.g., formatting, linting) before code is committed.
    *   **GitHub Actions:** Used for CI/CD to automate testing on multiple Python versions and deploy the script to an AWS EC2 instance on pushes to the `main` branch.

## Project Structure

```
my_pure_config_project/
├── .github/
│   └── workflows/
│       └── ci-cd.yml         # GitHub Actions workflow for CI/CD
├── data/
│   ├── sample_input.csv      # Sample input data (or generated if not present)
│   ├── processed_output.csv  # Output of the data transformation
│   └── data_processing.log   # Log file for script execution
├── src/
│   ├── __init__.py           # Makes 'src' a Python package
│   └── main.py               # Main data processing script
├── tests/
│   ├── __init__.py           # Makes 'tests' a Python package
│   └── test_main.py          # Unit tests for src/main.py
├── .gitignore                # Specifies intentionally untracked files for Git
├── .pre-commit-config.yaml   # Configuration for pre-commit hooks
├── LICENSE                   # Project's license file (MIT)
├── pyproject.toml            # Configuration for development tools (Ruff, Mypy, etc.)
├── README.md                 # This file
├── requirements-dev.txt      # Development dependencies
├── requirements.txt          # Runtime dependencies
└── venv/                     # Python virtual environment (typically not committed due to .gitignore)
```

*   **`.github/workflows/`**: Contains CI/CD pipeline configurations for GitHub Actions.
*   **`data/`**: Holds input data, processed output, and log files.
*   **`src/`**: Contains the main application source code (`main.py`).
*   **`tests/`**: Contains unit tests for the application.
*   **Configuration Files**: Root directory files like `.gitignore`, `.pre-commit-config.yaml`, `pyproject.toml`, `requirements.txt`, and `requirements-dev.txt` manage the project's behavior, dependencies, and tooling.

## Core Components & Logic

### `src/main.py`

This is the heart of the project. Its responsibilities include:

1.  **Project Root Determination:** Dynamically determines the project's root directory to ensure paths are correctly resolved, regardless of where the script is run from. This is crucial for finding data files and log files.
2.  **Path Management:** Uses helper functions (`get_default_input_path`, `get_default_output_path`, `get_default_log_path`) that rely on the `PROJECT_ROOT` variable. This allows tests to easily `monkeypatch` the `PROJECT_ROOT` to redirect file operations to temporary directories during testing.
3.  **Logging Setup (`setup_logging` function):**
    *   Configures Python's built-in `logging` module.
    *   Sets up two handlers:
        *   A `FileHandler` to write logs of `DEBUG` level and above to `data/data_processing.log`.
        *   A `StreamHandler` (console) to output logs of `INFO` level and above.
    *   Uses a consistent formatter for log messages (timestamp, logger name, level, message).
    *   Called once at the beginning of the script's execution (in the `if __name__ == "__main__":` block) and also within tests if needed.
4.  **Sample Data Generation (`create_sample_dataframe` function):**
    *   If an input CSV is not found, this function generates a sample Pandas DataFrame with predefined columns and random data.
5.  **Data Processing (`process_data` function):**
    *   Attempts to read data from an input CSV (defaulting to `data/sample_input.csv`). If the file doesn't exist or is empty, it calls `create_sample_dataframe()` and saves the generated data.
    *   Handles potential errors during file reading (e.g., `EmptyDataError`).
    *   Performs transformations on the DataFrame:
        *   Adds a new column `value1_plus_10`.
        *   Adds a new column `value2_div_value1`.
        *   Filters rows where `value1` is greater than 20.
        *   Adds a categorical column `value1_type` based on `value1`'s value ('High' or 'Medium').
    *   Logs various stages of the process (e.g., reading data, transformations, DataFrame heads).
    *   Returns the processed DataFrame.
6.  **Main Execution Block (`if __name__ == "__main__":`)**:
    *   This block runs when the script is executed directly.
    *   Calls `setup_logging()` to initialize logging.
    *   Calls `process_data()` to perform the data transformation.
    *   Saves the processed DataFrame to `data/processed_output.csv` if it's not empty.
    *   Includes a top-level `try...except` block to catch and log any critical unhandled errors during script execution.

## Dependency Management

This project separates runtime dependencies from development dependencies for clarity and efficiency.

### `requirements.txt`

*   **Purpose:** Lists the Python packages required for the script (`src/main.py`) to *run* in any environment (local, EC2, etc.).
*   **Contents:**
    ```txt
    pandas
    numpy
    ```
*   **Why this way?** Keeping runtime dependencies minimal ensures that deployment environments are lightweight and only install what's strictly necessary for the application to function.

### `requirements-dev.txt`

*   **Purpose:** Lists all Python packages needed for *development* tasks such as testing, linting, formatting, type checking, security scanning, and running pre-commit hooks. These are generally not needed in a production runtime environment.
*   **Contents:** (Pinned versions ensure consistent development environments)
    ```txt
    pytest==8.3.5
    pytest-cov
    mypy==1.14.0
    ruff==0.11.10
    bandit==1.7.9
    safety==3.5.1
    pre-commit==3.5.0
    types-PyYAML
    pandas-stubs
    ```
*   **Why this way?** This separation prevents development tools from being installed in production, and pinning versions (as seen in your file) helps ensure that all developers and the CI environment use the exact same versions of these tools, reducing "works on my machine" issues.

## Tooling and Configuration

### `pyproject.toml`

*   **Purpose:** In this project, `pyproject.toml` is used *exclusively* for configuring development tools. It does **not** define project metadata (like name, version, author) or build system information, as this project is not set up to be built as a distributable Python package.
*   **Key Sections & Why:**
    *   `[tool.ruff]`: Configures Ruff, the linter and formatter.
        *   `line-length`: Sets the maximum line length for code.
        *   `lint.select`: Specifies which Ruff linting rules to enable.
        *   `format`: Defines formatting preferences like quote style and indentation.
    *   `[tool.mypy]`: Configures Mypy, the static type checker.
        *   `python_version`: Specifies the target Python version for type checking.
        *   `warn_return_any`, `warn_unused_configs`: Enable stricter checks.
        *   `plugins`: Enables plugins like `numpy.typing.mypy_plugin` for better type checking with libraries like NumPy.
        *   `mypy_path = "src"`: Tells Mypy where to find your source code.
    *   `[tool.pytest.ini_options]`: Configures Pytest, the testing framework.
        *   `minversion`: Specifies the minimum required Pytest version.
        *   `addopts`: Default command-line options for Pytest. Here it includes:
            *   `-ra`: Show extra test summary information.
            *   `-q`: Quiet mode (less verbose output).
            *   `--cov=src.main`: Enable code coverage for the `src/main.py` module.
            *   `--cov-report=term-missing`: Show coverage report in the terminal, including missing lines.
            *   `--cov-fail-under=60`: Fail the test run if coverage is below 60%.
        *   `testpaths = ["tests"]`: Specifies the directory where Pytest should look for tests.
    *   `[tool.bandit]`: Configures Bandit, the security linter. You can add specific rules to skip (e.g., `skips = ["B101"]`) or adjust severity thresholds.
*   **Why this way?** Using `pyproject.toml` for tool configuration is the modern standard (PEP 518, PEP 621). By keeping it focused on tools and managing dependencies separately in `requirements.txt` files, the project remains simple if packaging isn't a primary goal.

### `.pre-commit-config.yaml`

*   **Purpose:** Configures pre-commit hooks, which are automated checks that run on your code *before* you make a Git commit. This helps catch issues early and maintain consistent code quality across all contributions.
*   **Key Sections & Why:**
    *   `repos`: A list of repositories containing pre-commit hooks.
    *   Each `repo` entry specifies:
        *   `repo`: The URL of the repository providing the hooks.
        *   `rev`: The specific version (tag or commit hash) of the hooks to use, ensuring consistency.
        *   `hooks`: A list of specific hooks to use from that repository.
            *   `id`: The identifier of the hook.
            *   `args`: Optional arguments to pass to the hook.
*   **Configured Hooks:**
    *   **`pre-commit-hooks`:** Standard general-purpose hooks.
        *   `trailing-whitespace`: Removes trailing whitespace.
        *   `end-of-file-fixer`: Ensures files end with a single newline.
        *   `check-yaml`: Checks YAML files for syntax errors.
        *   `check-added-large-files`: Prevents accidentally committing large files.
    *   **`ruff-pre-commit`:** Integrates Ruff.
        *   `ruff`: Runs Ruff linter (with `--fix` to auto-correct issues and `--exit-non-zero-on-fix` to ensure commit proceeds only if fixes are clean or no issues).
        *   `ruff-format`: Runs Ruff formatter to ensure code style consistency.
    *   **`mirrors-mypy`:** Integrates Mypy for static type checking.
        *   `mypy`: Runs Mypy. `additional_dependencies` are specified here because the Mypy hook runs in its own isolated environment and needs these to correctly type-check your code which uses Pandas, NumPy, etc.
*   **Why this way?** Pre-commit hooks automate best practices, reduce the number of trivial errors pushed to the repository, and save time in code reviews by ensuring basic quality checks are met before code even leaves the developer's machine.

### `.gitignore`

*   **Purpose:** Specifies intentionally untracked files and directories that Git should ignore. This prevents committing unnecessary or sensitive files to the repository.
*   **Key Entries & Why:**
    *   `__pycache__/`, `*.py[cod]`: Python bytecode and compiled files (generated automatically).
    *   `venv/`, `.venv/`, `env/`: Python virtual environment directories (can be large, machine-specific, and easily recreated).
    *   `.vscode/`, `.idea/`: IDE-specific setting directories.
    *   `.pytest_cache/`, `.coverage`, `htmlcov/`: Testing and coverage artifacts.
    *   `.mypy_cache/`, `.ruff_cache/`: Caches for Mypy and Ruff.
    *   `data/processed_output.csv`, `data/*.log`: Generated output and log files. It's good practice to commit sample input (`data/sample_input.csv` is *not* ignored here if you manually create and commit it) but not generated outputs or logs, which can change frequently and clutter the repository.
*   **Why this way?** A well-configured `.gitignore` keeps the repository clean, focused on source code and essential configuration, and avoids issues with platform-specific or generated files.

## Testing

### `tests/test_main.py`

*   **Purpose:** Contains unit tests for the functions in `src/main.py`. Unit tests verify that individual components of the code work correctly in isolation.
*   **Key Features & Why:**
    *   **Pytest Fixtures (`@pytest.fixture`):**
        *   `sample_df_for_test()`: Provides a reusable, predefined Pandas DataFrame for tests that need sample input. This makes tests consistent and easier to write.
        *   `temp_data_dir(monkeypatch)`: This is a crucial fixture for testing file I/O.
            *   It uses `tempfile.TemporaryDirectory()` to create a temporary directory that exists only for the duration of the test.
            *   It uses `monkeypatch.setattr(main_module, "PROJECT_ROOT", tmpdir_path)` to dynamically change the `PROJECT_ROOT` variable within the imported `main_module` (`src.main`) to this temporary directory. This means any code in `src.main.py` that uses `get_default_input_path()`, `get_default_output_path()`, or `get_default_log_path()` will now operate within this temporary directory during the test.
            *   **Why `monkeypatch`?** It allows modifying the behavior or state of modules or objects during tests without permanently altering the original code. This is essential for isolating tests and making them independent of the actual `data/` directory.
    *   **Test Functions (`test_*`):**
        *   `test_create_sample_dataframe()`: Checks if `create_sample_dataframe()` returns a DataFrame with the expected structure.
        *   `test_process_data_with_input_file()`: Tests the `process_data` function with a predefined input CSV (created in the temporary directory). It asserts that the output DataFrame has the expected columns and values after transformations.
        *   `test_process_data_generates_sample_if_no_input()`: Tests that `process_data` correctly generates and uses sample data if the specified input file doesn't exist. It also checks if the sample input file was saved to the (temporary) default location.
        *   `test_process_data_handles_empty_input_file()`: Tests error handling for empty input CSVs, expecting an empty DataFrame as output.
    *   **Logging Initialization in Tests:** The lines `if not main_module.logger.hasHandlers(): main_module.setup_logging()` are included in test functions.
        *   **Why?** When Pytest imports and runs test functions, the `if __name__ == "__main__":` block in `src/main.py` (which calls `setup_logging()`) doesn't execute. If functions within `src.main.py` that are being tested (like `process_data`) use the logger, the logger needs to be configured for the test context to avoid errors or ensure logs are captured as expected. This check ensures that logging handlers are added if they haven't been already.
    *   **Imports:** `from src import main as main_module` and `from src.main import ...` are used to correctly import and patch the module located in the `src` directory.
*   **Why this way?** Well-structured tests with fixtures make testing more robust, maintainable, and easier to understand. Testing file I/O with temporary directories ensures tests are clean and don't leave artifacts or depend on the state of the actual `data` directory.

### Code Coverage

*   **Purpose:** Measures how much of your source code (`src/main.py`) is executed by your tests (`tests/test_main.py`).
*   **Configuration:** Done in `pyproject.toml` under `[tool.pytest.ini_options]`:
    *   `--cov=src.main`: Specifies that coverage should be measured for the `src.main` module.
    *   `--cov-report=term-missing`: Displays a coverage report in the terminal after tests run, highlighting lines that were not covered.
    *   `--cov-fail-under=60`: The test suite will fail if the code coverage drops below 60%. This enforces a minimum standard of testing.
*   **Why this way?** Code coverage helps identify untested parts of your application, guiding you to write more comprehensive tests. Enforcing a minimum coverage threshold helps maintain testing discipline.

## Local Development Setup

Follow these steps to set up the project locally for development:

1.  **Clone the Repository:**
    If you haven't already, get a copy of the project code:
    ```bash
    git clone <your-repository-url> # Replace with the actual URL
    cd my_pure_config_project       # Navigate into the project directory
    ```
    *   **Why:** This downloads the project files from the Git repository to your local machine.

2.  **Create and Activate a Python Virtual Environment:**
    It's highly recommended to use a virtual environment to manage project dependencies and avoid conflicts with system-wide Python packages.
    ```bash
    python3 -m venv venv
    source venv/bin/activate  # On Linux/macOS
    # venv\Scripts\activate   # On Windows (cmd.exe)
    # venv\Scripts\Activate.ps1 # On Windows (PowerShell)
    ```
    *   **Why:** `python3 -m venv venv` creates a new directory named `venv` containing a fresh Python installation. `source venv/bin/activate` modifies your shell's environment so that `python` and `pip` commands use the versions and packages installed within this `venv` directory, isolating the project.

3.  **Install Dependencies:**
    The project uses two files for dependencies: `requirements.txt` for runtime and `requirements-dev.txt` for development.
    ```bash
    # Ensure pip is up-to-date within the virtual environment
    python -m pip install --upgrade pip

    # Install runtime dependencies
    pip install -r requirements.txt

    # Install development dependencies
    pip install -r requirements-dev.txt
    ```
    *   **Why `pip install -r ...`?** The `-r` flag tells `pip` to install all packages listed in the specified requirements file.
    *   **Why separate files?** This separation ensures that only necessary packages are installed in production environments (using `requirements.txt`), while development environments get all the tools needed for coding, testing, and quality checks (from `requirements-dev.txt`).

4.  **Install Pre-commit Hooks:**
    This step integrates the pre-commit checks defined in `.pre-commit-config.yaml` with your local Git repository.
    ```bash
    pre-commit install
    ```
    *   **Why:** This command copies the pre-commit hook script into your `.git/hooks/` directory. Now, every time you run `git commit`, the configured hooks (like Ruff formatting/linting, Mypy type checking) will run automatically on the files you've staged. This helps catch errors *before* you commit them. If a hook fails, the commit will be aborted, allowing you to fix the issues.

## Running the Script Locally

Once the setup is complete, you can run the data processing script:

1.  **Ensure your virtual environment is activated.**
    (Your shell prompt should typically show `(venv)`).
2.  **Execute the script:**
    ```bash
    python src/main.py
    ```
*   **Expected Behavior:**
    *   If `data/sample_input.csv` does not exist, the script will log a warning, generate sample data, and save it to `data/sample_input.csv`.
    *   The script will then process the data (either from the existing file or the newly generated one).
    *   The processed output will be saved to `data/processed_output.csv`.
    *   Logs detailing the script's execution will be printed to the console (INFO level and above) and also written to `data/data_processing.log` (DEBUG level and above).

## CI/CD with GitHub Actions

This project uses GitHub Actions for Continuous Integration (CI) and Continuous Deployment (CD). The workflow is defined in `.github/workflows/ci-cd.yml`.

### Workflow: `.github/workflows/ci-cd.yml`

This workflow is triggered on every `push` to the `main` branch and on every `pull_request` targeting the `main` branch. It consists of two main jobs:

1.  **`lint-test-analyze` Job:**
    *   **Purpose:** To ensure code quality, correctness, and security before deployment.
    *   **Runs on:** `ubuntu-latest` GitHub-hosted runner.
    *   **Strategy Matrix:** Runs the job across multiple Python versions (`3.8` to `3.12`) to ensure compatibility.
    *   **Steps:**
        1.  `Checkout code`: Fetches the latest version of your repository code.
        2.  `Set up Python`: Installs the specified Python version from the matrix.
        3.  `Install dependencies`:
            *   Upgrades `pip`.
            *   Installs runtime dependencies from `requirements.txt`.
            *   Installs development dependencies from `requirements-dev.txt` (needed for linting, testing tools, etc.).
        4.  `Lint and Format Check with Ruff`:
            *   `ruff check .`: Runs Ruff linter to find code style issues and potential bugs.
            *   `ruff format --check .`: Checks if the code is formatted according to Ruff rules. If not, this step fails the build, ensuring code is formatted correctly before merging/deploying.
        5.  `Static type checking with Mypy`: Runs Mypy using `pyproject.toml` for configuration to catch type errors.
        6.  `Security scan (code) with Bandit`: Scans the `src` directory for common security vulnerabilities in the Python code using `pyproject.toml` for Bandit's configuration.
        7.  `Security scan (dependencies) with Safety`:
            *   `pip freeze > current_requirements_frozen.txt`: Creates a list of all installed packages and their exact versions in the CI environment.
            *   `safety check -r current_requirements_frozen.txt`: Checks these installed packages against a database of known security vulnerabilities.
        8.  `Run tests with Pytest`: Executes the unit tests using Pytest. Pytest automatically picks up its configuration (like coverage settings) from `pyproject.toml`.
    *   **Why this job is important:** It automates the verification of code quality and correctness for every change, preventing regressions and maintaining standards. Running on multiple Python versions ensures broad compatibility.

2.  **`deploy-and-run-on-ec2` Job:**
    *   **Purpose:** To deploy and run the data processing script on a pre-configured AWS EC2 instance.
    *   **`needs: lint-test-analyze`**: This job will *only* run if the `lint-test-analyze` job completes successfully for all Python versions.
    *   **`if: github.ref == 'refs/heads/main' && github.event_name == 'push'`**: This job will *only* run for direct pushes to the `main` branch (i.e., not for pull requests or pushes to other branches). This is a common strategy to deploy only merged and fully tested code.
    *   **Runs on:** `ubuntu-latest` GitHub-hosted runner.
    *   **Steps:**
        1.  `Checkout code`: Fetches the repository code again (jobs run in fresh environments).
        2.  `Deploy to EC2 and Run Script`: Uses the `appleboy/ssh-action` to connect to your EC2 instance via SSH and execute a series of commands.
            *   **Secrets:** Uses GitHub Encrypted Secrets (`EC2_HOST`, `EC2_USERNAME`, `EC2_SSH_PRIVATE_KEY`, `EC2_PORT`) to securely store and use sensitive connection details.
            *   **Script on EC2 (explained further in [EC2 Deployment Details](#ec2-deployment-details)):**
                *   Creates an application directory (`/opt/my_data_project_src_main`).
                *   Clones or updates the repository in this directory.
                *   Sets up a Python virtual environment (`venv`) within the application directory.
                *   Installs *only runtime dependencies* from `requirements.txt` into this venv.
                *   Executes the `python src/main.py` script.
                *   Prints status messages, including the location of output files on EC2.
    *   **Why this job is important:** It automates the deployment process, making it repeatable, reliable, and reducing the chance of manual errors.

### GitHub Secrets

To allow GitHub Actions to securely connect to your EC2 instance, you must configure the following secrets in your GitHub repository settings (**Settings > Secrets and variables > Actions > New repository secret**):

*   `EC2_HOST`: The public IP address or DNS name of your EC2 instance.
*   `EC2_USERNAME`: The username for SSHing into the EC2 instance (e.g., `ubuntu`, `ec2-user`).
*   `EC2_SSH_PRIVATE_KEY`: The private SSH key (the full content of your `.pem` file or other private key) that corresponds to a public key authorized on your EC2 instance for the `EC2_USERNAME`.
*   `EC2_PORT` (Optional): The SSH port if it's not the default (22). The workflow defaults to 22 if this secret is not set.

**Why use secrets?** Directly embedding sensitive information like IP addresses, usernames, and private keys into your workflow file is a major security risk, as the workflow file is part of your repository. GitHub Secrets provide a secure way to store and use this information in your workflows.

## EC2 Deployment Details

The `script` section within the `deploy-and-run-on-ec2` job in the GitHub Actions workflow performs the following actions on your EC2 instance:

1.  **`set -e`**: Ensures that the script will exit immediately if any command fails, preventing further execution on error.
2.  **`export APP_DIR="/opt/my_data_project_src_main"`**: Defines a variable for the application directory on EC2. Using `/opt/` is a common practice for user-installed software.
3.  **Directory Creation & Ownership**:
    *   `sudo mkdir -p $APP_DIR`: Creates the application directory if it doesn't exist (uses `sudo` for permission).
    *   `sudo chown ${{ secrets.EC2_USERNAME }}:${{ secrets.EC2_USERNAME }} $APP_DIR`: Changes the ownership of the directory to the SSH user. This allows subsequent commands (like `git clone` and `python3 -m venv`) to run without `sudo` within this directory.
4.  **`cd $APP_DIR`**: Navigates into the application directory.
5.  **Repository Cloning/Updating**:
    *   Checks if a `.git` directory exists.
    *   If not (first deployment), it clones the repository: `git clone https://github.com/${{ github.repository }}.git .` (clones into the current directory `$APP_DIR`).
    *   If it exists (subsequent deployments):
        *   `git remote set-url origin ...`: Ensures the remote URL is correct.
        *   `git fetch origin main --prune`: Fetches the latest changes from the `main` branch and removes any stale remote-tracking branches.
        *   `git reset --hard origin/main`: Resets the local `main` branch to exactly match the remote `main` branch, discarding any local changes or previous commits on the EC2 instance. This ensures a clean state.
        *   `git clean -fdx`: Removes any untracked files and directories (including those ignored by `.gitignore`) to ensure a completely clean build state.
6.  **Python Virtual Environment Setup**:
    *   Checks if a `venv` directory exists.
    *   If not, it creates one: `python3 -m venv venv`.
    *   `source venv/bin/activate`: Activates the virtual environment.
7.  **Install Runtime Dependencies**:
    *   `pip install --upgrade pip`: Upgrades pip within the virtual environment.
    *   `pip install -r requirements.txt`: Installs only the runtime dependencies specified in `requirements.txt`. Development tools are not installed on the EC2 instance.
8.  **Run the Data Processing Script**:
    *   `python src/main.py`: Executes your main Python script. The script's internal `PROJECT_ROOT` logic (which determines paths relative to `src/main.py`'s location) will correctly find and create the `data` subdirectory within `$APP_DIR` (e.g., `/opt/my_data_project_src_main/data/`).
9.  **Output Confirmation**: Prints messages indicating the script has finished and where to find the output files and logs on the EC2 instance.

**Why these steps on EC2?** This sequence ensures a consistent and clean environment for each deployment. Cloning/resetting the repository ensures the latest code is used. Using a virtual environment isolates dependencies. Installing only runtime dependencies keeps the EC2 environment lean.

## Troubleshooting Common Issues

*   **Pre-commit Hooks Not Running Locally:**
    *   **Ensure `pre-commit` is installed:** Run `pip show pre-commit` in your `venv`. If not found, `pip install pre-commit`.
    *   **Run `pre-commit install`:** This command must be run once per cloned repository to activate the hooks.
    *   **Staged Files:** Pre-commit hooks run on *staged* files. Make sure you `git add` your changes before `git commit`.
    *   **YAML Validity:** Check `.pre-commit-config.yaml` for syntax errors.

*   **GitHub Actions Failing on Linting/Formatting (e.g., Ruff):**
    *   **Error Message:** The workflow log (like the one you shared for `ruff format --check .`) will indicate which files are problematic.
    *   **Fix:** Run the formatter/linter locally (e.g., `ruff format .` or `ruff check --fix .`), commit the changes, and push again.
    *   **Why:** The CI pipeline enforces these checks. If your local pre-commit hooks didn't catch it (or weren't run/configured identically), the CI acts as a safety net.

*   **GitHub Actions Failing on Tests:**
    *   **Error Message:** The workflow log for the `Run tests with Pytest` step will show which tests failed and why.
    *   **Fix:** Debug the failing tests locally, fix the code or the tests, commit, and push.

*   **GitHub Actions Failing on EC2 Deployment (`Deploy to EC2 and Run Script` step):**
    *   **SSH Connection Issues:**
        *   Verify `EC2_HOST`, `EC2_USERNAME`, `EC2_SSH_PRIVATE_KEY`, and `EC2_PORT` secrets in GitHub are correct.
        *   Ensure the public key corresponding to `EC2_SSH_PRIVATE_KEY` is in the `~/.ssh/authorized_keys` file for the `EC2_USERNAME` on the EC2 instance.
        *   Check the EC2 instance's Security Group to ensure it allows inbound SSH traffic (port 22 or your custom `EC2_PORT`) from GitHub Actions' IP ranges (for testing, you might temporarily allow from `0.0.0.0/0`, but restrict this for production).
    *   **Script Errors on EC2:** Examine the output in the GitHub Actions log for the failing command. Common issues include:
        *   `git clone` failing (permissions, repository URL).
        *   `python3` or `pip` not found (ensure Python/pip are installed on EC2 and in PATH).
        *   `pip install -r requirements.txt` failing (missing system dependencies for a Python package, network issues).
        *   `python src/main.py` failing (runtime errors in your script, incorrect paths if `PROJECT_ROOT` logic is flawed, or permissions issues accessing/creating files in `/opt/my_data_project_src_main/data/`).

---
