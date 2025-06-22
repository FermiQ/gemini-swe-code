# `src/gemini_cli/tools/summarizer_tool.py`

## Overview

This file defines the `SummarizeCodeTool`, which is responsible for providing summaries of code files. For small files, it returns the full content. For larger files, it uses an internal call to a Gemini model to generate a concise summary. This helps in understanding the purpose and structure of a file without needing to read its entire content, especially useful for large codebases.

## Key Components

-   **`MAX_LINES_FOR_FULL_CONTENT` (int)**: Constant defining the maximum number of lines a file can have to be considered "small" enough to return its full content instead of a summary. Default: `1000`.
-   **`MAX_CHARS_FOR_FULL_CONTENT` (int)**: Constant defining the maximum number of characters (file size in bytes) a file can have to be considered "small". Default: `50 * 1024` (50 KB).
-   **`SUMMARIZATION_SYSTEM_PROMPT` (str)**: A system prompt used when calling the Gemini model internally to generate a summary. It instructs the model to focus on the file's main purpose, key classes/functions, dependencies, and overall structure.

-   **`SummarizeCodeTool(BaseTool)` class**:
    -   **`name`**: `"summarize_code"`
    -   **`description`**: "Provides a summary of a code file's purpose, key functions/classes, and structure. Use for large files or when only an overview is needed."
    -   **`__init__(self, model_instance: genai.GenerativeModel | None = None)`**:
        *   Initializes the tool. It can optionally take an instance of `genai.GenerativeModel`. This model instance is used for generating summaries of large files.
        *   **Note**: This creates a dependency where the tool needs access to the main model instance used by the CLI.
    -   **`execute(self, file_path: str | None = None, directory_path: str | None = None, query: str | None = None, glob_pattern: str | None = None) -> str`**:
        *   The main method to get a summary or full content of a file.
        *   **(Note**: The `directory_path`, `query`, and `glob_pattern` parameters are defined in the signature but not currently used in the provided code logic, which focuses on `file_path`.)
        *   Checks if the `model_instance` was provided during initialization; returns an error if not.
        *   Performs basic path safety for `file_path`.
        *   Checks if the file exists and is a file.
        *   Determines if the file is "small" based on `MAX_LINES_FOR_FULL_CONTENT` and `MAX_CHARS_FOR_FULL_CONTENT`.
        *   If the file is small, it reads and returns its full content, prefixed with `--- Full Content of {file_path} ---`.
        *   If the file is large, it reads the file content (up to a certain limit for the summarization prompt, currently 20000 chars) and makes an internal call to the `self.model.generate_content` method using `SUMMARIZATION_SYSTEM_PROMPT` and the file content.
        *   The summary is then returned, prefixed with `--- Summary of {file_path} ---`.
        *   Handles errors during file reading or summary generation.
        *   **Arguments**:
            *   `file_path` (str | None): The path to the code file to summarize.
            *   `directory_path` (str | None, optional): (Currently unused by the logic).
            *   `query` (str | None, optional): (Currently unused by the logic).
            *   `glob_pattern` (str | None, optional): (Currently unused by the logic).
        *   **Returns**: (str) Either the full content of the file or a generated summary, or an error message.
    -   **`_extract_text_from_summary_response(self, response)`**:
        *   A helper method to safely extract the generated text from the `generate_content` response object received from the internal summarization call.
        *   Checks `response.candidates` and `finish_reason`.

## Important Variables/Constants

-   **`log` (Logger)**: Standard Python logger.
-   **`MAX_LINES_FOR_FULL_CONTENT`**: Threshold for returning full content based on line count.
-   **`MAX_CHARS_FOR_FULL_CONTENT`**: Threshold for returning full content based on character count (file size). This constant is also used by `ViewTool`.
-   **`SUMMARIZATION_SYSTEM_PROMPT`**: The specific instructions given to the LLM for generating summaries.

## Usage Examples

This tool is intended to be called by the Gemini model itself, particularly when it needs to understand a file that might be too large to process directly in its main context window.

Model: "The file `src/complex_module.py` is very large. I should summarize it first."
Model calls `summarize_code` with `file_path="src/complex_module.py"`.

The tool then either returns the full content (if `complex_module.py` is small) or uses its internal `genai.GenerativeModel` instance to generate and return a summary.

## Dependencies and Interactions

-   **`google.generativeai` (SDK)**: Specifically `genai.GenerativeModel` and `genai.types.GenerationConfig`. The tool instance may hold a reference to a `GenerativeModel` to make internal LLM calls for summarization.
-   **`logging`**: For logging tool activities and errors.
-   **`os`**: For path operations (`os.path.abspath`, `os.path.expanduser`, `os.path.exists`, `os.path.isfile`, `os.path.getsize`, `os.getcwd`).
-   **`.base.BaseTool`**: The parent class from which `SummarizeCodeTool` inherits.
-   **Interaction with `ViewTool`**: The `MAX_CHARS_FOR_FULL_CONTENT` constant is shared conceptually (and through import in `ViewTool`) to maintain consistency in how "large" files are defined for direct viewing versus summarization.

The `SummarizeCodeTool` is crucial for enabling the AI to handle large files efficiently by first getting a high-level understanding through summaries.
