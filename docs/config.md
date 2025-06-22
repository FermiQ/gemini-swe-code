# `src/gemini_cli/config.py`

## Overview

This file manages configuration for the Gemini CLI application. It handles loading, saving, and providing access to configuration settings such as API keys, default models, and other operational parameters.

## Key Components

-   **`Config` class**: Manages all configuration-related operations.
    -   **`__init__(self)`**: Initializes the configuration by setting up paths and loading existing configuration or creating a default one.
    -   **`_ensure_config_exists(self)`**: Creates the configuration directory and a default configuration file if they don't already exist.
    -   **`_load_config(self)`**: Loads configuration settings from the YAML file.
    -   **`_save_config(self)`**: Saves the current configuration settings to the YAML file.
    -   **`get_api_key(self, provider)`**: Retrieves the API key for a specified provider.
    -   **`set_api_key(self, provider, key)`**: Sets and saves the API key for a specified provider.
    -   **`get_default_model(self)`**: Retrieves the default model name.
    -   **`set_default_model(self, model)`**: Sets and saves the default model name.
    -   **`get_setting(self, setting, default=None)`**: Retrieves a specific setting's value.
    -   **`set_setting(self, setting, value)`**: Sets and saves a specific setting's value.

## Important Variables/Constants

-   **`config_dir` (Path)**: Stores the path to the configuration directory (`~/.config/gemini-code`).
-   **`config_file` (Path)**: Stores the path to the configuration file (`~/.config/gemini-code/config.yaml`).
-   **`default_config` (dict)**: A dictionary holding the default configuration values used when a configuration file is first created. It includes sections for `api_keys`, `default_model`, and `settings` like `max_tokens`, `temperature`, etc.

## Usage Examples

```python
# How to initialize and use the Config class
from gemini_cli.config import Config

# Create a Config instance
config_manager = Config()

# Get an API key
api_key = config_manager.get_api_key("gemini")
print(f"Gemini API Key: {api_key}")

# Set an API key
config_manager.set_api_key("gemini", "YOUR_GEMINI_API_KEY")

# Get the default model
default_model = config_manager.get_default_model()
print(f"Default Model: {default_model}")

# Set the default model
config_manager.set_default_model("models/gemini-2.5-pro-latest")

# Get a specific setting
max_tokens = config_manager.get_setting("max_tokens")
print(f"Max Tokens: {max_tokens}")

# Set a specific setting
config_manager.set_setting("temperature", 0.8)
```

## Dependencies and Interactions

-   **`os`**: Used for path operations, though `pathlib` is primarily used.
-   **`yaml` (PyYAML)**: Used for reading and writing the configuration file in YAML format.
-   **`pathlib.Path`**: Used for object-oriented filesystem path manipulation, making it easier to handle directory and file paths.

This module is central to the application's ability to store and retrieve user-specific settings and credentials. Other modules in the `gemini_cli` package will likely interact with `Config` to access these settings.
