# `src/gemini_cli/tools/quality_tools.py`

## Overview

This file provides tools for integrating code quality checks, such as linting and formatting, into the Gemini CLI application. These tools rely on external command-line utilities (e.g., `ruff`, `black`, `flake8`) being installed in the environment where the CLI is executed.

## Key Components

-   **`_run_quality_command(command: list[str], tool_name: str) -> str`**:
    -   A helper function that executes a given shell command for a quality tool.
    -   It uses `subprocess.run` to execute the command, capturing stdout and stderr.
    -   It has a timeout of 120 seconds (2 minutes).
    -   Logs the command execution, exit code, stdout, and stderr.
    -   Formats the output, including exit code, stdout, and stderr.
    -   Truncates the result if it exceeds `max_len` (2000 characters).
    -   Handles `FileNotFoundError` (if the command isn't found), `subprocess.TimeoutExpired`, and other general exceptions.
    -   **Arguments**:
        *   `command` (list[str]): The command and its arguments as a list of strings.
        *   `tool_name` (str): The name of the tool being run (e.g., "Linter", "Formatter") for logging and output messages.
    -   **Returns**: (str) A formatted string containing the tool's output, or an error message.

-   **`LinterCheckerTool(BaseTool)` class**:
    -   **`name`**: `"linter_checker"`
    -   **`description`**: "Runs a code linter (default: 'ruff check') on a specified path to find potential issues."
    -   **`execute(self, path: str = '.', linter_command: str = 'ruff check') -> str`**:
        *   Executes a code linter command on a given file or directory.
        *   `path` defaults to the current directory (`.`).
        *   `linter_command` defaults to `'ruff check'`. The actual path is appended to this command.
        *   Uses `shlex.split` to parse the `linter_command` string into a list for `subprocess`.
        *   Includes basic path safety to prevent `..` in `path`.
        *   Calls `_run_quality_command` to execute the linter.
        *   **Arguments**:
            *   `path` (str, optional): File or directory to lint. Defaults to `.`
            *   `linter_command` (str, optional): The linter command. Defaults to `'ruff check'`.
        *   **Returns**: (str) The output from the linter.

-   **`FormatterTool(BaseTool)` class**:
    -   **`name`**: `"formatter"`
    -   **`description`**: "Runs a code formatter (default: 'black') on a specified path to automatically fix styling."
    -   **`execute(self, path: str = '.', formatter_command: str = 'black') -> str`**:
        *   Executes a code formatter command on a given file or directory.
        *   `path` defaults to the current directory (`.`).
        *   `formatter_command` defaults to `'black'`. The actual path is appended to this command.
        *   Uses `shlex.split` to parse the `formatter_command` string.
        *   Includes basic path safety.
        *   Calls `_run_quality_command` to execute the formatter.
        *   **Arguments**:
            *   `path` (str, optional): File or directory to format. Defaults to `.`
            *   `formatter_command` (str, optional): The formatter command. Defaults to `'black'`.
        *   **Returns**: (str) The output from the formatter.

## Important Variables/Constants

-   **`log` (Logger)**: A standard Python logger instance for this module.

## Usage Examples

These tools are intended to be invoked by the Gemini model via function calling.

**Running the linter:**
Model calls `linter_checker` with `path="src/my_module.py"`.
Or, to use a different linter: `linter_checker` with `path="src/", linter_command="flake8 --select=E,W"`.

**Running the formatter:**
Model calls `formatter` with `path="src/my_module.py"`.
Or, to use a different formatter on a specific file type: `formatter` with `path=".", formatter_command="prettier --write **/*.js"`.

```python
# Conceptual Python usage:
from gemini_cli.tools.quality_tools import LinterCheckerTool, FormatterTool

# Linter
linter = LinterCheckerTool()
lint_results = linter.execute(path="src/", linter_command="ruff check --select I001") # Example: check for import sorting
print(f"Linter Results:\n{lint_results}")

# Formatter
formatter = FormatterTool()
format_results = formatter.execute(path="src/ugly_file.py", formatter_command="black --check") # Example: check formatting
print(f"Formatter Results:\n{format_results}")
```
**Note**: The actual commands (`ruff check`, `black`, `flake8`, `prettier`) must be installed and accessible in the system's PATH where the `gemini` CLI is running.

## Dependencies and Interactions

-   **`subprocess`**: Used by `_run_quality_command` to execute external linter/formatter commands.
-   **`logging`**: For logging tool execution details and errors.
-   **`shlex`**: Used to safely split the `linter_command` and `formatter_command` strings into argument lists.
-   **`os`**: Used for path manipulation (`os.path.sep`, `os.path.abspath`, `os.path.expanduser`).
-   **`.base.BaseTool`**: The parent class from which these tools inherit.

These quality tools allow the AI to help maintain code standards by identifying issues and applying automated formatting.
