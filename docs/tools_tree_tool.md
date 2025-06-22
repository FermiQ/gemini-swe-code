# `src/gemini_cli/tools/tree_tool.py`

## Overview

This file defines the `TreeTool`, which allows the Gemini model to display a directory structure in a tree-like format. It relies on the external `tree` command-line utility being installed in the system.

## Key Components

-   **`DEFAULT_TREE_DEPTH` (int)**: Constant defining the default depth for the tree display if not specified. Default: `3`.
-   **`MAX_TREE_DEPTH` (int)**: Constant defining the maximum allowed depth for the tree display to prevent overly large outputs. Default: `10`.

-   **`TreeTool(BaseTool)` class**:
    -   **`name`**: `"tree"`
    -   **`description`**: A detailed description explaining that the tool displays the directory structure, defaults to a depth of `DEFAULT_TREE_DEPTH`, allows specifying `path` and `depth`.
    -   **`args_schema` (dict)**: Defines the expected arguments for the Gemini model's function calling:
        *   `path` (string, optional): Path to a specific directory. Defaults to the current directory.
        *   `depth` (integer, optional): Maximum display depth. Defaults to `DEFAULT_TREE_DEPTH`.
    -   **`required_args` (list)**: Empty, as both `path` and `depth` are optional.
    -   **`execute(self, path: str | None = None, depth: int | None = None) -> str`**:
        *   Constructs and executes the `tree` command.
        *   Sets `depth_limit` based on the `depth` argument, clamped between 1 and `MAX_TREE_DEPTH`. Uses `DEFAULT_TREE_DEPTH` if `depth` is not provided.
        *   The base command is `['tree', f'-L {depth_limit}']`.
        *   If `path` is provided, it's appended to the command. Otherwise, it operates on the current directory (`.`).
        *   Logs the command to be executed.
        *   Uses `subprocess.run` to execute the `tree` command, capturing `stdout` and `stderr`.
        *   A timeout of 15 seconds is set for the command execution.
        *   If the command is successful (return code 0):
            *   Returns the `stdout`.
            *   The output is truncated if it exceeds 200 lines to prevent excessively large responses.
        *   If the `tree` command is not found (return code 127 or "command not found" in stderr, or `FileNotFoundError`), it returns an error message prompting the user to install `tree`.
        *   Handles `subprocess.TimeoutExpired` if the command takes too long.
        *   Catches other general exceptions and returns an error message.
        *   **Arguments**:
            *   `path` (str | None, optional): The directory path to display. Defaults to the current directory.
            *   `depth` (int | None, optional): The maximum depth of the tree. Defaults to `DEFAULT_TREE_DEPTH`.
        *   **Returns**: (str) The directory tree structure as a string, or an error message.

## Important Variables/Constants

-   **`log` (Logger)**: A standard Python logger instance for this module.
-   **`DEFAULT_TREE_DEPTH`**: Default value for the tree display depth.
-   **`MAX_TREE_DEPTH`**: Maximum allowed value for the tree display depth.

## Usage Examples

This tool is intended to be invoked by the Gemini model to understand the layout of a directory.

**Displaying the current directory tree with default depth:**
Model calls `tree`.
This would execute `tree -L 3 .` (or similar, depending on OS `tree` version).

**Displaying a specific subdirectory with a custom depth:**
Model calls `tree` with `path="src/components", depth=2`.
This would execute `tree -L 2 src/components`.

```python
# Conceptual Python usage:
from gemini_cli.tools.tree_tool import TreeTool

tree_viewer = TreeTool()

# Example 1: Tree of current directory with default depth
# Assumes 'tree' command is installed.
# result1 = tree_viewer.execute()
# print(f"Tree Output 1:\n{result1}")

# Example 2: Tree of a specific path with specified depth
# Create a dummy directory structure for this example
import os
if not os.path.exists("temp_tree_dir/subdir"):
    os.makedirs("temp_tree_dir/subdir")
with open("temp_tree_dir/file1.txt", "w") as f: f.write("content")
with open("temp_tree_dir/subdir/file2.txt", "w") as f: f.write("content")

# result2 = tree_viewer.execute(path="temp_tree_dir", depth=1)
# print(f"Tree Output 2:\n{result2}")

# Cleanup dummy directory
# import shutil
# shutil.rmtree("temp_tree_dir")
```
**Note**: The `tree` command-line utility must be installed and accessible in the system's PATH where the `gemini` CLI is running.

## Dependencies and Interactions

-   **`subprocess`**: Used to execute the external `tree` command.
-   **`logging`**: For logging information about the command execution and any errors.
-   **`google.generativeai.types.FunctionDeclaration`, `Tool`**: (Imported but not directly used in the `TreeTool` class itself; `BaseTool` handles declaration generation).
-   **`.base.BaseTool`**: The parent class from which `TreeTool` inherits.

The `TreeTool` is useful for providing the AI with a visual representation of the directory hierarchy, aiding in navigation and understanding project structure.
