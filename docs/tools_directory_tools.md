# `src/gemini_cli/tools/directory_tools.py`

## Overview

This file provides tools for performing directory-related operations within the Gemini CLI application. These tools allow the AI model to interact with the file system by creating directories and listing their contents.

## Key Components

-   **`CreateDirectoryTool(BaseTool)` class**:
    -   **`name`**: `"create_directory"`
    -   **`description`**: "Creates a new directory, including any necessary parent directories."
    -   **`execute(self, dir_path: str) -> str`**:
        *   Creates the specified directory.
        *   It includes basic path safety to prevent accessing parent directories using `..`.
        *   Uses `os.makedirs(target_path, exist_ok=True)` for directory creation, which also creates parent directories if they don't exist.
        *   Returns a success message or an error message if the directory already exists as a file, or if any other `OSError` occurs.
        *   **Arguments**:
            *   `dir_path` (str): The path of the directory to create (can be relative or absolute).
        *   **Returns**: (str) A message indicating success or failure.

-   **`LsTool(BaseTool)` class**:
    -   **`name`**: `"ls"`
    -   **`description`**: "Lists the contents of a specified directory (long format, including hidden files)."
    -   **`execute(self, path: str | None = None) -> str`**:
        *   Executes the system command `ls -lA` on the specified `path` or the current directory if `path` is omitted.
        *   Includes basic path safety to prevent navigating outside the current working directory context using `..` in the `path` argument.
        *   Uses `subprocess.run` to execute the command, capturing its output and errors.
        *   Includes a timeout of 15 seconds for the command.
        *   If the command is successful, it returns the stripped standard output. The output is truncated if it exceeds 100 lines.
        *   If the command fails (e.g., directory not found), it returns an error message including details from stderr.
        *   Handles `FileNotFoundError` (if `ls` command is not found) and `subprocess.TimeoutExpired`.
        *   **Arguments**:
            *   `path` (str | None, optional): The path to the directory to list. Defaults to the current directory (`.`) if not provided.
        *   **Returns**: (str) The output of the `ls -lA` command or an error message.

## Important Variables/Constants

-   **`log` (Logger)**: A standard Python logger instance for logging messages from this module.

## Usage Examples

These tools are intended to be called by the Gemini model via its native function calling mechanism.

**Creating a directory:**
The model might decide to call `create_directory` with `dir_path="new_project/src"`.

**Listing directory contents:**
The model might call `ls` with no arguments to list the current directory or with `path="src"` to list the contents of a subdirectory named `src`.

```python
# Conceptual Python usage (how the model's call might be translated)
from gemini_cli.tools.directory_tools import CreateDirectoryTool, LsTool

# CreateDirectoryTool
create_tool = CreateDirectoryTool()
result_create = create_tool.execute(dir_path="my_new_folder/subfolder")
print(f"CreateDirectoryTool result: {result_create}")

# LsTool
ls_tool = LsTool()
result_ls_current = ls_tool.execute()
print(f"LsTool result (current dir):\n{result_ls_current}")

result_ls_specific = ls_tool.execute(path="my_new_folder") # Assuming my_new_folder was created
print(f"LsTool result (my_new_folder):\n{result_ls_specific}")
```

## Dependencies and Interactions

-   **`os`**: Used for path manipulations (`os.path.sep`, `os.path.abspath`, `os.path.expanduser`, `os.path.exists`, `os.path.isdir`, `os.makedirs`, `os.path.normpath`).
-   **`logging`**: Used for logging tool execution details and errors.
-   **`subprocess`**: Used by `LsTool` to run the external `ls` command.
-   **`.base.BaseTool`**: The parent class from which `CreateDirectoryTool` and `LsTool` inherit, providing the basic structure and `get_function_declaration` capability.

These tools are crucial for the AI to understand and interact with the project's directory structure, enabling it to navigate and organize files as needed for coding tasks. The `LsTool` is particularly important as it's used for the mandatory orientation step at the beginning of each user interaction in the `GeminiModel`.
