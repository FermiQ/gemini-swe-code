# `src/gemini_cli/tools/base.py`

## Overview

This file defines the foundational `BaseTool` abstract class, which serves as a blueprint for all tools used within the Gemini CLI application. It provides a common interface and a mechanism for generating `FunctionDeclaration` objects compatible with the Google Generative AI API, enabling native function calling by the Gemini model.

## Key Components

-   **`BaseTool(ABC)` class**: An abstract base class that all specific tool implementations must inherit from.
    -   **`name` (str)**: A class attribute that should be overridden by subclasses to define the tool's name as it will be known to the Gemini model.
    -   **`description` (str)**: A class attribute that should be overridden by subclasses to provide a description of the tool's purpose and functionality for the Gemini model.
    -   **`execute(self, *args, **kwargs)` (abstractmethod)**: An abstract method that subclasses must implement. This method contains the core logic of the tool.
    -   **`get_function_declaration(cls) -> FunctionDeclaration | None` (classmethod)**:
        *   This method dynamically generates a `FunctionDeclaration` object for the tool.
        *   It inspects the signature of the `execute` method (excluding `self`) to determine the parameters, their types (basic mapping from Python types to JSON schema types like string, integer, number, boolean, array, object), and whether they are required.
        *   The tool's `name` and `description` attributes are used directly in the declaration.
        *   Parameter descriptions are generically created (e.g., "Parameter [param_name]").
        *   If the tool's `name` or `description` is missing, or if an error occurs during generation, it logs a warning/error and returns `None`.
        *   The generated `FunctionDeclaration` is used to inform the Gemini model about how to call the tool.

## Important Variables/Constants

-   **`log` (Logger)**: A standard Python logger instance for logging messages from this module.

## Usage Examples

This class is not used directly but is inherited by all other tool classes. Here's a conceptual example of how a subclass would be defined:

```python
from gemini_cli.tools.base import BaseTool
from google.generativeai.types import FunctionDeclaration
import logging

log = logging.getLogger(__name__)

class MyCustomTool(BaseTool):
    name = "my_custom_tool"
    description = "This is a custom tool that performs a specific action."

    def execute(self, required_param: str, optional_param: int = 0) -> str:
        """
        Executes the custom action.

        Args:
            required_param: A mandatory string parameter.
            optional_param: An optional integer parameter.

        Returns:
            A string result of the action.
        """
        log.info(f"Executing MyCustomTool with {required_param=} and {optional_param=}")
        # ... tool logic here ...
        return f"Action performed with {required_param} and {optional_param}"

# To get its function declaration:
# declaration = MyCustomTool.get_function_declaration()
# if declaration:
#     print(declaration)
```

The `get_function_declaration` method would then generate a `FunctionDeclaration` object based on `MyCustomTool`'s `name`, `description`, and the parameters of its `execute` method.

## Dependencies and Interactions

-   **`shlex`**: (Imported but not directly used in the `BaseTool` class itself, may be used by subclasses).
-   **`inspect`**: Used by `get_function_declaration` to introspect the `execute` method's signature (parameters, annotations, default values).
-   **`abc.ABC`** and **`abc.abstractmethod`**: Used to define `BaseTool` as an abstract base class and `execute` as an abstract method.
-   **`google.generativeai.types.FunctionDeclaration`**: The type definition for the function declaration object that this class generates. This object is crucial for enabling native function calling with the Gemini API.
-   **`logging`**: Used for logging warnings or errors, especially during the function declaration generation process.

This `BaseTool` class is fundamental for extending the CLI's capabilities by adding new tools. It ensures that all tools adhere to a common structure and can be easily integrated with the Gemini model's function calling mechanism.
