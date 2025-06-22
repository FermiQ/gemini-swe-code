# `src/gemini_cli/utils.py`

## Overview

This file contains utility functions for the Gemini CLI application. Currently, its primary utility is for token counting.

## Key Components

-   **`count_tokens(text: str) -> int`**:
    -   Estimates the number of tokens in a given text string.
    -   It primarily tries to use the `tiktoken` library with the `"gpt-4"` model's tokenizer as a proxy for estimating token count for Gemini models. This is noted as a rough estimate.
    -   If `tiktoken` fails for any reason (e.g., library not available, model name incorrect), it falls back to a simpler estimation method: dividing the length of the text by 4 (based on the heuristic that one token is approximately four characters).
    -   **Arguments**:
        *   `text` (str): The input string for which to count tokens.
    -   **Returns**: (int) The estimated number of tokens.

## Important Variables/Constants

-   None specific to this file beyond standard library imports.

## Usage Examples

This function can be used by various parts of the application that need to estimate the size of text data in terms of tokens, for example, before sending it to the Gemini API to ensure it's within context window limits or to estimate costs.

```python
from gemini_cli.utils import count_tokens

text1 = "This is a sample sentence."
tokens1 = count_tokens(text1)
print(f"'{text1}' has approximately {tokens1} tokens.")

text2 = "Another example with a few more words to count."
tokens2 = count_tokens(text2)
print(f"'{text2}' has approximately {tokens2} tokens.")

# Example demonstrating fallback (if tiktoken was unavailable or failed)
# This would typically rely on the fallback len(text)//4
# For instance, if tiktoken.encoding_for_model("gpt-4") raised an error.
# tokens_fallback = count_tokens("Some text for fallback scenario")
# print(f"Fallback token count for 'Some text for fallback scenario' might be: {tokens_fallback}")

```

## Dependencies and Interactions

-   **`tiktoken`**: An optional dependency. The `count_tokens` function attempts to use it for more accurate token counting. If not available or if it errors, a fallback estimation is used.
-   **`json`**: (Imported but not currently used in the provided snippet of `utils.py`).

This utility is helpful for managing interactions with the language model by providing a way to gauge the "size" of text data from the model's perspective.
