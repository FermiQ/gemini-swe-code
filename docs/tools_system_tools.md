# `src/gemini_cli/tools/system_tools.py`

## Overview

This file provides tools for executing system-level operations, specifically for running bash commands. It includes safety measures by disallowing certain commands.

## Key Components

-   **`BashTool(BaseTool)` class**:
    -   **`name`**: `"bash"`
    -   **`description`**: "Execute a bash command"
    -   **`BANNED_COMMANDS` (list)**: A class attribute listing strings of commands that are disallowed for security reasons (e.g., `curl`, `wget`, `ssh`).
    -   **`execute(self, command, timeout=30000)`**:
        *   Executes the given `command` string in a bash shell.
        *   First, it checks if any part of the `command` matches a command in `BANNED_COMMANDS`. If a banned command is found, it returns an error message and does not execute the command.
        *   The `timeout` argument is in milliseconds and defaults to 30,000 (30 seconds). It's converted to seconds for `subprocess.communicate`.
        *   Uses `subprocess.Popen` with `shell=True` to execute the command. stdout and stderr are captured.
        *   If the command executes successfully (return code 0), it returns the standard output.
        *   If the command exits with a non-zero status, it returns a message including the status code, stdout, and stderr.
        *   Handles `subprocess.TimeoutExpired` if the command exceeds the specified timeout, killing the process and returning a timeout error message.
        *   Catches other general exceptions during command execution and returns an error message.
        *   **Arguments**:
            *   `command` (str): The bash command string to execute.
            *   `timeout` (int, optional): Timeout in milliseconds. Defaults to 30000.
        *   **Returns**: (str) The standard output of the command if successful, or an error message detailing the failure (banned command, non-zero exit, timeout, or other exception).

## Important Variables/Constants

-   **`BANNED_COMMANDS`**: A list within the `BashTool` class defining commands that are not permitted for execution due to security concerns. This list includes network utilities like `curl`, `wget`, `ssh`, etc.

## Usage Examples

This tool is intended to be called by the Gemini model when it needs to run a shell command.

**Running a simple bash command:**
Model calls `bash` with `command="ls -l /tmp"`.

**Attempting a banned command:**
Model calls `bash` with `command="wget http://example.com/file"`.
The tool would return: `"Error: The command 'wget' is not allowed for security reasons."`

**Command with a timeout:**
Model calls `bash` with `command="sleep 45", timeout=30000`.
The tool would return: `"Error: Command timed out after 30.0 seconds"` (assuming `sleep 45` exceeds the 30s timeout).

```python
# Conceptual Python usage:
from gemini_cli.tools.system_tools import BashTool

bash_runner = BashTool()

# Example 1: Allowed command
result1 = bash_runner.execute(command="echo 'Hello from BashTool'")
print(f"Result 1:\n{result1}")

# Example 2: Banned command
result2 = bash_runner.execute(command="curl google.com")
print(f"Result 2:\n{result2}")

# Example 3: Command that might fail or produce stderr
result3 = bash_runner.execute(command="ls /non_existent_directory")
print(f"Result 3:\n{result3}")

# Example 4: Command that times out
# result4 = bash_runner.execute(command="sleep 5", timeout=2000) # 2-second timeout
# print(f"Result 4:\n{result4}")
```

## Dependencies and Interactions

-   **`os`**: (Imported but not directly used in the provided snippet of `BashTool`).
-   **`subprocess`**: Used by `BashTool` to create and manage the child process for executing the bash command (`subprocess.Popen`, `process.communicate`, `process.kill`).
-   **`tempfile`**: (Imported but not directly used in the provided snippet of `BashTool`).
-   **`.base.BaseTool`**: The parent class from which `BashTool` inherits.

The `BashTool` provides a controlled way for the AI to interact with the system shell, with some safety restrictions.
