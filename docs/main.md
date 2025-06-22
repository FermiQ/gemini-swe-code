# `src/gemini_cli/main.py`

## Overview

This file serves as the main entry point for the Gemini CLI application. It handles command-line argument parsing, initializes the application environment (including configuration and logging), and launches the interactive chat session or executes specific commands. It uses the `click` library for CLI handling and `rich` for enhanced terminal output.

## Key Components

-   **`console` (Console)**: An instance of `rich.console.Console` for rich text and formatted output to the terminal.
-   **`config` (Config)**: An instance of `gemini_cli.config.Config` for managing application settings.
-   **`log` (Logger)**: A standard Python logger instance configured for this module.
-   **`DEFAULT_MODEL` (str)**: A constant defining the default Gemini model to be used if not specified otherwise.
-   **`GEMINI_CODE_ART` (str)**: A raw string containing ASCII art displayed at the start of an interactive session.
-   **`cli(ctx, model)` (click.group)**: The main command group for the CLI. It initializes the model and starts an interactive session if no subcommand is invoked.
    -   `@click.option('--model', '-m', ...)`: Allows specifying the model via the command line.
-   **`setup(key)` (cli.command)**: A subcommand to set and save the Google API key.
-   **`set_default_model(model_name)` (cli.command)**: A subcommand to set and save the default Gemini model.
-   **`list_models()` (cli.command)**: A subcommand to fetch and display available Gemini models.
-   **`start_interactive_session(model_name: str, console: Console)`**: Function to start an interactive chat session with the specified Gemini model. It handles user input, model interaction, and displays responses.
-   **`show_help()`**: Displays help information for interactive mode commands and available tools.

## Important Variables/Constants

-   **`DEFAULT_MODEL`**: Defines the fallback model ID (`gemini-2.5-pro-exp-03-25`) if no other model is configured or specified.
-   **`GEMINI_CODE_ART`**: ASCII art displayed as a welcome message.
-   **`CONTEXT_SETTINGS`**: A dictionary configuring help options for the `click` CLI.
-   **`log_level`**: Determined from the `LOG_LEVEL` environment variable (defaults to "WARNING").
-   **`log_format`**: String defining the format for log messages.

## Usage Examples

**Starting an interactive session with the default model:**
```bash
gemini
```

**Starting an interactive session with a specific model:**
```bash
gemini --model gemini-1.0-pro
```

**Setting up the API key:**
```bash
gemini setup YOUR_GOOGLE_API_KEY
```

**Listing available models:**
```bash
gemini list-models
```

**Setting a default model:**
```bash
gemini set-default-model gemini-1.5-flash
```

**Inside the interactive session:**
```
You: /help  # Displays help
You: /exit  # Exits the session
You: Tell me a joke.
Gemini: ...
```

## Dependencies and Interactions

-   **`os`**: Used for environment variables (e.g., `LOG_LEVEL`).
-   **`sys`**: Used for system-specific parameters and functions, like exiting the application and stream handling for logging.
-   **`click`**: The primary library for creating the command-line interface.
-   **`rich`**: Used for creating rich terminal UIs, including formatted text, panels, and markdown rendering.
-   **`pathlib.Path`**: Used for filesystem path manipulations (though less prominent in this specific file).
-   **`yaml`**: Indirectly used via the `Config` class for configuration file handling.
-   **`google.generativeai`**: The official Google AI SDK, used indirectly via `GeminiModel` for interacting with Gemini APIs.
-   **`logging`**: Standard Python logging library.
-   **`time`**: Used for `time.sleep()` for UI presentation.
-   **`.models.gemini.GeminiModel`**: The class responsible for actual communication with the Gemini API.
-   **`.models.gemini.list_available_models`**: Function to fetch model list.
-   **`.config.Config`**: Handles loading and saving of application configuration.
-   **`.utils.count_tokens`**: Utility function (though not directly called in the provided snippet, it's imported).
-   **`.tools.AVAILABLE_TOOLS`**: A dictionary of available tools that can be used by the Gemini model.

This `main.py` orchestrates the CLI application, bringing together configuration, model interaction, and user interface elements.
