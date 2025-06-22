# `src/gemini_cli/tools/file_tools.py`

## Overview

This file contains tools for performing various file operations. These tools enable the Gemini model to view file contents, edit or create files, search for patterns within files (grep), and find files using glob patterns.

## Key Components

-   **`ViewTool(BaseTool)` class**:
    -   **`name`**: `"view"`
    -   **`description`**: "View specific sections of a file using offset/limit, or view small files entirely. Use summarize_code for large files."
    -   **`execute(self, file_path: str, offset: int | None = None, limit: int | None = None) -> str`**:
        *   Reads and returns the content of a specified file.
        *   If `offset` (1-based line number) and `limit` (max lines) are provided, it returns only that slice of the file, prepending line numbers.
        *   If `offset` and `limit` are not provided, it checks the file size. If the file is larger than `MAX_CHARS_FOR_FULL_CONTENT` (imported from `summarizer_tool` or a fallback value), it returns an error suggesting to use `summarize_code` or `view` with offset/limit. Otherwise, it returns the full file content with line numbers.
        *   Includes basic path safety to prevent `..` in `file_path`.
        *   Handles file not found and attempts to view directories.
        *   **Arguments**:
            *   `file_path` (str): The path to the file.
            *   `offset` (int | None, optional): The 1-based line number to start reading from.
            *   `limit` (int | None, optional): The maximum number of lines to read.
        *   **Returns**: (str) The requested file content with line numbers, or an error/suggestion message.

-   **`EditTool(BaseTool)` class**:
    -   **`name`**: `"edit"`
    -   **`description`**: "Edit or create a file. Use 'content' to provide the **entire** new file content (for creation or full overwrite). Use 'old_string' and 'new_string' to replace the **first** occurrence of an exact string. For precise changes, it's best to first `view` the relevant section, then use `edit` with the exact `old_string` and `new_string`, or provide the complete, modified content using the `content` parameter."
    -   **`execute(self, file_path: str, content: str | None = None, old_string: str | None = None, new_string: str | None = None) -> str`**:
        *   Modifies or creates a file.
        *   If `content` is provided, it overwrites the file with this content. If the directory for `file_path` doesn't exist, it creates it.
        *   If `old_string` and `new_string` are provided (and `content` is not), it reads the file, replaces the first occurrence of `old_string` with `new_string`, and writes the modified content back. `new_string` can be empty to delete the `old_string`.
        *   If only `file_path` is provided (all other optional args are `None`), it creates an empty file or truncates an existing one.
        *   Prioritizes `content` over `old_string`/`new_string` if both are somehow provided.
        *   Includes basic path safety.
        *   **Arguments**:
            *   `file_path` (str): Path to the file.
            *   `content` (str | None, optional): Full content to write.
            *   `old_string` (str | None, optional): Exact string to find for replacement.
            *   `new_string` (str | None, optional): String to replace `old_string` with.
        *   **Returns**: (str) A success or error message.

-   **`GrepTool(BaseTool)` class**:
    -   **`name`**: `"grep"`
    -   **`description`**: "Search for a pattern (regex) in files within a directory."
    -   **`execute(self, pattern: str, path: str = '.', include: str | None = None) -> str`**:
        *   Searches for a regular expression `pattern` in files.
        *   `path` specifies the directory to search in (defaults to current directory).
        *   `include` is an optional glob pattern (e.g., `*.py`, `src/**/*.js`) to filter which files are searched. If `include` contains `**`, recursive search is enabled for the glob.
        *   If `include` is not provided, it walks through the `path` directory, skipping hidden directories and `__pycache__`.
        *   Compiles the `pattern` into a regex.
        *   Reads files line by line and adds matching lines to the results, prefixed with `relative_file_path:line_number:`.
        *   Limits the total number of matches to `max_matches` (currently 500).
        *   Returns a newline-separated string of matches or a "No matches found" message.
        *   **Arguments**:
            *   `pattern` (str): The regular expression to search for.
            *   `path` (str, optional): Directory to search in. Defaults to `.`
            *   `include` (str | None, optional): Glob pattern to filter files.
        *   **Returns**: (str) Matching lines or an error/status message.

-   **`GlobTool(BaseTool)` class**:
    -   **`name`**: `"glob"`
    -   **`description`**: "Find files/directories matching specific glob patterns recursively."
    -   **`execute(self, pattern: str, path: str = '.') -> str`**:
        *   Finds files and directories matching a `pattern` (glob syntax) within a given `path`.
        *   The search is recursive if the pattern implies it (e.g., using `**`).
        *   `path` defaults to the current directory.
        *   Returns a newline-separated string of sorted relative paths of matching items.
        *   Prepends `./` to matches directly in the `path` directory for clarity.
        *   **Arguments**:
            *   `pattern` (str): The glob pattern (e.g., `*.txt`, `**/*.py`).
            *   `path` (str, optional): The base directory for the glob search. Defaults to `.`
        *   **Returns**: (str) Newline-separated list of matching paths, or a message if no matches.

## Important Variables/Constants

-   **`log` (Logger)**: Standard Python logger.
-   **`MAX_CHARS_FOR_FULL_CONTENT` (int)**: Imported from `.summarizer_tool` or defaults to `50 * 1024`. Used by `ViewTool` to decide if a file is too large for direct viewing without offset/limit.

## Usage Examples

These tools are designed for the Gemini model's function calling.

**Viewing a file:**
Model calls `view` with `file_path="src/utils.py", offset=10, limit=20`.

**Editing a file (overwrite):**
Model calls `edit` with `file_path="config.json", content="{\"key\": \"new_value\"}"`.

**Editing a file (replace string):**
Model calls `edit` with `file_path="README.md", old_string="TODO", new_string="DONE"`.

**Grepping for a pattern:**
Model calls `grep` with `pattern="class\s+\w+", path="src", include="*.py"`.

**Globbing for files:**
Model calls `glob` with `pattern="**/*.test.js", path="tests"`.

## Dependencies and Interactions

-   **`.base.BaseTool`**: Parent class for all tools.
-   **`os`**: Extensive use for path operations (joining, normalizing, checking existence, etc.).
-   **`glob`**: Used by `GrepTool` (if `include` is provided) and `GlobTool` for pattern matching of file paths.
-   **`re`**: Used by `GrepTool` for compiling and searching with regular expressions.
-   **`logging`**: For logging tool activities.
-   **`pathlib.Path`**: (Imported but not directly used in the provided snippet for these tools, `os.path` is preferred).
-   **`.summarizer_tool.MAX_CHARS_FOR_FULL_CONTENT`**: Constant used by `ViewTool`.

These file tools are essential for the AI to read, understand, modify, and navigate the codebase.
