# `src/gemini_cli/tools/test_runner.py`

## Overview

This file defines the `TestRunnerTool`, which allows the Gemini model to execute automated tests using a configurable test runner command (like `pytest`). It's designed to be used after code modifications to help verify correctness.

## Key Components

-   **`TestRunnerTool(BaseTool)` class**:
    -   **`name`**: `"test_runner"`
    -   **`description`**: "Runs automated tests using the project's test runner (defaults to trying 'pytest'). Use after making code changes to verify correctness."
    -   **`execute(self, test_path: str | None = None, options: str | None = None, runner_command: str = "pytest") -> str`**:
        *   Constructs and executes a test command.
        *   The base command is `runner_command` (defaulting to `"pytest"`).
        *   If `options` (a string of additional command-line arguments like `"-k my_test -v"`) are provided, they are parsed using `shlex.split` and appended to the command.
        *   If `test_path` (a specific file or directory) is provided, it's appended to the command.
        *   Logs the command to be executed.
        *   Uses `subprocess.run` to execute the test command, capturing `stdout` and `stderr`.
        *   A timeout of 300 seconds (5 minutes) is set for the test execution.
        *   It checks the `returncode` of the process to determine success or failure.
        *   Formats a summary message that includes:
            *   The test runner command used.
            *   The exit code.
            *   Status (SUCCESS or FAILED).
            *   On success, the last 1000 characters of `stdout`.
            *   On failure, the full `stdout` and `stderr`.
            *   A specific note if using `pytest` and the exit code is 5 (often meaning no tests were found).
        *   Handles `FileNotFoundError` if the `runner_command` is not found.
        *   Handles `subprocess.TimeoutExpired` if the tests take too long.
        *   Catches other general exceptions during execution.
        *   **Arguments**:
            *   `test_path` (str | None, optional): Specific file or directory to test. If omitted, the runner's default discovery is used.
            *   `options` (str | None, optional): Additional command-line options for the test runner (e.g., `"-k my_test_function"`, `"--cov"`).
            *   `runner_command` (str, optional): The command to invoke the test runner. Defaults to `"pytest"`.
        *   **Returns**: (str) A summary of the test results, including output, especially on failure.

## Important Variables/Constants

-   **`log` (Logger)**: A standard Python logger instance for this module.

## Usage Examples

This tool is intended to be invoked by the Gemini model after it has made code changes.

**Running default tests with pytest:**
Model calls `test_runner` (no arguments).
This would attempt to run `pytest`.

**Running tests in a specific file with options:**
Model calls `test_runner` with `test_path="tests/test_utils.py", options="-v -k test_specific_function"`.
This would attempt to run `pytest -v -k test_specific_function tests/test_utils.py`.

**Using a different test runner:**
Model calls `test_runner` with `runner_command="python -m unittest discover", test_path="project/tests"`.
This would attempt to run `python -m unittest discover project/tests`.

```python
# Conceptual Python usage:
from gemini_cli.tools.test_runner import TestRunnerTool

test_tool = TestRunnerTool()

# Example 1: Run pytest on a specific file with verbose output
# Assume 'tests/my_app_tests.py' exists and pytest is installed.
results1 = test_tool.execute(
    test_path="tests/my_app_tests.py",
    options="-v"
)
print(f"Test Results 1:\n{results1}")

# Example 2: Run default tests (e.g., pytest discovering tests)
# results2 = test_tool.execute()
# print(f"Test Results 2:\n{results2}")

# Example 3: Attempt to run with a non-existent runner command
# results3 = test_tool.execute(runner_command="non_existent_runner")
# print(f"Test Results 3:\n{results3}")
# Expected output: Error: Test runner command 'non_existent_runner' not found...
```
**Note**: The specified `runner_command` (e.g., `pytest`) must be installed and accessible in the system's PATH where the `gemini` CLI is running.

## Dependencies and Interactions

-   **`subprocess`**: Used to execute the external test runner command.
-   **`logging`**: For logging information about the test execution process and its outcome.
-   **`shlex`**: Used to safely parse the `options` string into a list of arguments, handling quotes and spaces correctly.
-   **`.base.BaseTool`**: The parent class from which `TestRunnerTool` inherits.

This tool is vital for a test-driven development or verification workflow, allowing the AI to confirm that its changes haven't broken existing functionality.
