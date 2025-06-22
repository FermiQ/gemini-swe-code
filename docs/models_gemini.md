# `src/gemini_cli/models/gemini.py`

## Overview

This file provides the core integration with Google's Gemini models for the CLI application. It defines the `GeminiModel` class, which encapsulates the logic for interacting with the Gemini API, including managing chat history, handling native function calls (tools), processing model responses, and implementing an agentic loop for multi-turn interactions.

## Key Components

-   **`list_available_models(api_key)`**: A function to fetch and list all available Gemini models that support the `generateContent` method.
-   **`GeminiModel` class**: The primary class for interacting with the Gemini API.
    -   **`__init__(self, api_key: str, console: Console, model_name: str)`**: Initializes the model with API key, a `rich.console.Console` instance for output, and the desired model name. It configures the Gemini API, sets up generation parameters, safety settings, defines tools (function declarations), and creates a system prompt.
    -   **`_initialize_model_instance(self)`**: Helper method to create the `genai.GenerativeModel` instance.
    -   **`get_available_models(self)`**: Instance method to list available models.
    -   **`generate(self, prompt: str) -> str | None`**: The main method for processing a user prompt. It implements an agentic loop:
        1.  Performs mandatory orientation (`ls` tool call) to get directory context.
        2.  Prepares the initial user turn by combining orientation context and the user's prompt.
        3.  Manages the chat history and context window.
        4.  Enters a loop (up to `MAX_AGENT_ITERATIONS`):
            *   Sends the history and available tools to the LLM.
            *   Processes the LLM response, which can include text parts and/or function call requests.
            *   If a function call is requested:
                *   For potentially destructive tools (`edit`, `create_file`), it prompts the user for confirmation using `questionary`.
                *   Executes the confirmed tool via `get_tool(tool_name).execute(**tool_args)`.
                *   Adds the tool's result back to the chat history.
                *   If the tool was `task_complete`, it breaks the loop and returns the summary.
            *   If only text is returned, it's treated as the final response.
            *   Handles `ResourceExhausted` errors by attempting to switch to a `FALLBACK_MODEL`.
        5.  Returns the final summary or response.
    -   **`_manage_context_window(self)`**: A basic context window manager that truncates the chat history if it exceeds a certain number of turns to prevent overly long contexts.
    -   **`_create_tool_definitions(self) -> list[FunctionDeclaration] | None`**: Dynamically creates `FunctionDeclaration` objects for each tool defined in `AVAILABLE_TOOLS` that has a `get_function_declaration` method.
    -   **`_create_system_prompt(self) -> str`**: Constructs a detailed system prompt for the Gemini model. This prompt instructs the AI on its role, available tools (with their signatures and descriptions), the expected workflow (Analyze & Plan, Execute, Observe, Repeat, Complete), and important rules for interaction (native functions, sequential calls, context handling, explanations, precise edits, task completion).
    -   **`_extract_text_from_response(self, response) -> str | None`**: Utility to safely extract text content from a Gemini API response object.
    -   **`_find_last_model_text(self, history: list) -> str`**: Utility to find the last text message sent by the model in the chat history.

## Important Variables/Constants

-   **`MAX_AGENT_ITERATIONS` (int)**: Maximum number of iterations (LLM calls and tool executions) the agent loop will run before timing out. Default: `10`.
-   **`FALLBACK_MODEL` (str)**: The model ID to switch to if the primary model encounters a quota exhaustion error. Default: `"gemini-1.5-pro-latest"`.
-   **`CONTEXT_TRUNCATION_THRESHOLD_TOKENS` (int)**: (Currently unused, but defined) A token limit that could be used for more sophisticated context window management. Default: `800000`.
-   **`AVAILABLE_TOOLS` (dict)**: Imported from `..tools`, this dictionary maps tool names to their respective class instances.
-   **`generation_config` (genai.types.GenerationConfig)**: Configuration for the model's content generation (e.g., temperature, top_p, top_k).
-   **`safety_settings` (dict)**: Configuration for content safety filtering (e.g., blocking harmful content).
-   **`chat_history` (list)**: A list of dictionaries representing the conversation history between the user, the model, and tool interactions. It's initialized with the system prompt and a ready message.

## Usage Examples

This class is primarily used by `src/gemini_cli/main.py` to drive the interactive session.

```python
# Simplified example of how GeminiModel might be used:
from rich.console import Console
from gemini_cli.models.gemini import GeminiModel, list_available_models
from gemini_cli.config import Config # Assuming config is set up

# --- Prerequisites ---
# 1. API Key must be configured (e.g., via Config class or environment)
# config_manager = Config()
# api_key = config_manager.get_api_key("google")
api_key = "YOUR_GOOGLE_API_KEY" # Replace with your actual key for direct testing

# 2. Console object
console = Console()

# --- List available models ---
# models = list_available_models(api_key)
# if isinstance(models, list) and models and "error" not in models[0]:
#     print("Available models:")
#     for model_info in models:
#         print(f"- {model_info['name']} ({model_info.get('display_name', 'N/A')})")
# else:
#     print(f"Error listing models: {models[0].get('error') if models else 'Unknown error'}")

# --- Initialize the model ---
# Ensure a valid model_name is used, e.g., one from list_available_models()
# For this example, we'll use a common one.
# Make sure the chosen model supports native function calling if tools are used.
model_name_to_use = "gemini-1.5-flash-latest" # Or another suitable model

try:
    gemini_agent = GeminiModel(api_key=api_key, console=console, model_name=model_name_to_use)

    # --- Send a prompt to the model ---
    # The generate method will automatically handle the 'ls' tool call first.
    user_prompt = "Create a python file named 'hello.py' that prints 'Hello, World!'."

    # In a real CLI, this would run in a loop.
    # The `generate` method handles the agentic loop internally.
    # User confirmation for file creation will be requested via questionary.
    # If you run this script directly, you'll see the confirmation prompt in the terminal.

    # final_response = gemini_agent.generate(user_prompt)

    # if final_response:
    #     console.print("\n[bold green]Final Response from Agent:[/bold green]")
    #     console.print(final_response)
    # else:
    #     console.print("\n[bold red]Agent did not produce a final response.[/bold red]")

except Exception as e:
    console.print(f"[bold red]An error occurred:[/bold red] {e}")

# Note: Running this example directly requires AVAILABLE_TOOLS to be correctly
# populated and accessible, and the 'ls' tool to function in the execution environment.
# The `questionary` prompts will also require interactive terminal input.
```

## Dependencies and Interactions

-   **`google.generativeai` (SDK)**: The official Google Generative AI SDK used for all interactions with the Gemini API (configuring, listing models, generating content, defining tools).
-   **`google.generativeai.protos`**: Specific protobuf message types from the SDK used for constructing tool responses.
-   **`google.api_core.exceptions.ResourceExhausted`**: Specific exception caught for handling API quota limits.
-   **`logging`**: Standard Python library for logging information and errors.
-   **`time`**: (Not directly used in the provided snippet but often useful in such classes).
-   **`rich.console.Console`**: Used for displaying rich output (status messages, panels) to the terminal.
-   **`rich.panel.Panel`**: Used for displaying user confirmation prompts in a structured way.
-   **`questionary`**: Used for interactively asking the user for confirmation before executing file modification tools.
-   **`..utils.count_tokens`**: (Imported but not directly used in the provided snippet) Utility for counting tokens, potentially for context management.
-   **`..tools.get_tool`**: Function to retrieve a specific tool instance by name from `AVAILABLE_TOOLS`.
-   **`..tools.AVAILABLE_TOOLS`**: A dictionary mapping tool names to tool instances, which provide `FunctionDeclaration`s for the Gemini API.

This module is critical for the AI's ability to understand, plan, and execute tasks using the defined tools and interacting with the Gemini models.
