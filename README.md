# gemini-swe-code
Gemini software code assistant for your terminal, powered by Gemini Pro with support for other LLM models.

## Features

- Interactive chat sessions in your terminal
- Multiple model support (Gemini 2.5 Pro, Gemini 1.5 Pro, and more)
- Basic history management (prevents excessive length)
- Markdown rendering in the terminal
- Automatic tool usage by the assistant:
  - File operations (view, edit, list, grep, glob)
  - Directory operations (ls, tree, create_directory)
  - System commands (bash)
  - Quality checks (linting, formatting)
  - Test running capabilities (pytest, etc.)

## Installation

### Method 1: Install from PyPI (Recommended)

```bash
# Install directly from PyPI
pip install gemini-code
```

### Method 2: Install from Source

```bash
# Clone the repository
git clone https://github.com/FermiQ/gemini-swe-code.git
cd gemini-code

# Install the package
pip install -e .
```

## Setup

Before using Gemini CLI, you need to set up your API keys:

```bash
# Set up Google API key for Gemini models
gemini setup YOUR_GOOGLE_API_KEY
```

## Usage

```bash
# Start an interactive session with the default model
gemini

# Start a session with a specific model
gemini --model models/gemini-2.5-pro-exp-03-25

# Set default model
gemini set-default-model models/gemini-2.5-pro-exp-03-25

# List all available models
gemini list-models
```

## Interactive Commands

During an interactive session, you can use these commands:

- `/exit` - Exit the chat session
- `/help` - Display help information

## How It Works

### Tool Usage

Unlike direct command-line tools, the Gemini CLI's tools are used automatically by the assistant to help answer your questions. For example:

1. You ask: "What files are in the current directory?"
2. The assistant uses the `ls` tool behind the scenes
3. The assistant provides you with a formatted response

This approach makes the interaction more natural and similar to how Claude Code works.

## Development

This project is under active development. More models and features will be added soon!

## License

MIT

