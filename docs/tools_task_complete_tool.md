# `src/gemini_cli/tools/task_complete_tool.py`

## Overview

This file defines the `TaskCompleteTool`, which is a special tool used by the Gemini model to signal that it has finished processing the current user request or task. It is intended to be the final tool called in an agentic sequence.

## Key Components

-   **`TaskCompleteTool(BaseTool)` class**:
    -   **`name`**: `"task_complete"`
    -   **`description`**: "Signals task completion. MUST be called as the final step, providing a user-friendly summary."
    -   **`execute(self, summary: str) -> str`**:
        *   This method is called by the Gemini model when it believes a task is fully resolved.
        *   It takes a `summary` string as input, which should be a concise, user-friendly description of the actions taken and the final outcome of the task.
        *   The method performs some cleaning on the input `summary`:
            *   It logs the original summary.
            *   It repeatedly strips common leading/trailing characters (quotes, spaces, newlines, tabs) from the summary.
            *   If the input `summary` is not a string, it attempts to convert it to a string and then strips it.
        *   It logs the cleaned summary.
        *   If the cleaned summary is missing or very short (less than 5 characters), it logs a warning and returns a default message: "Task marked as complete, but the provided summary was insufficient."
        *   Otherwise, it returns the cleaned `summary` string.
        *   The primary purpose of this tool call is to act as a signal to the controlling agent loop (e.g., in `GeminiModel`) that the current interaction for a given user prompt can be concluded. The returned summary is then typically displayed to the user.
        *   **Arguments**:
            *   `summary` (str): A concise, user-friendly summary of what was done and the final outcome.
        *   **Returns**: (str) The cleaned summary string, or a default message if the summary is inadequate.

## Important Variables/Constants

-   **`log` (Logger)**: A standard Python logger instance for this module.

## Usage Examples

The `TaskCompleteTool` is invoked by the Gemini model as the very last step of its thought process for a given user request.

**Scenario:** User asks the AI to create a file and write content to it.
1.  AI uses `EditTool` to create and write to the file.
2.  AI, believing the task is done, then calls `TaskCompleteTool`.
    *   Model call: `task_complete(summary="Successfully created 'example.txt' and wrote the requested content into it.")`

The `GeminiModel`'s agent loop would detect that `task_complete` was called, stop further iterations for the current prompt, and use the returned summary as the final response to the user.

```python
# Conceptual Python usage (how the model's call might be translated):
from gemini_cli.tools.task_complete_tool import TaskCompleteTool

task_complete_tool = TaskCompleteTool()

# Example 1: Good summary
summary_from_llm = "   \"The script 'run.sh' was created and made executable as requested.\"   "
final_output = task_complete_tool.execute(summary=summary_from_llm)
print(f"Final Output to User: {final_output}")
# Expected: Final Output to User: The script 'run.sh' was created and made executable as requested.

# Example 2: Insufficient summary
short_summary = "Done"
final_output_short = task_complete_tool.execute(summary=short_summary)
print(f"Final Output (short summary): {final_output_short}")
# Expected: Final Output (short summary): Task marked as complete, but the provided summary was insufficient.

# Example 3: Non-string summary (less likely from LLM but handled)
non_string_summary = 123
final_output_non_string = task_complete_tool.execute(summary=non_string_summary)
print(f"Final Output (non-string): {final_output_non_string}")
# Expected: Final Output (non-string): Task marked as complete, but the provided summary was insufficient. (or "123" if it passes length check)
```

## Dependencies and Interactions

-   **`logging`**: Used for logging the signaling of task completion and the provided summaries.
-   **`.base.BaseTool`**: The parent class from which `TaskCompleteTool` inherits.
-   **`GeminiModel` (or similar orchestrator)**: This tool's primary interaction is with the agent loop in `GeminiModel`. The loop is designed to recognize when this tool is called and to terminate the current interaction sequence, using the returned summary as the final output to the user.

This tool is critical for the structured completion of tasks within the agentic workflow, providing a clear signal for the AI to indicate it has finished its work on a particular request.
