
###  **Tools**

**Tools** empower your agents to take actions beyond just processing text. They allow agents to interact with the outside world, performing tasks like fetching data from the internet, running code, calling external APIs, and even controlling a computer. The Agent SDK categorizes these capabilities into three main types:


1.  **Hosted tools:**
   
    These are tools provided by the LLM service (like OpenAI) and run on their infrastructure alongside the AI models. OpenAI offers tools for web search, file retrieval from their vector stores, and computer use.

    ```python
    from agents import Agent, FileSearchTool, Runner, WebSearchTool

    agent = Agent(
        name="Assistant",
        tools=[
            WebSearchTool(),
            FileSearchTool(
                max_num_results=3,
                vector_store_ids=["VECTOR_STORE_ID"],
            ),
        ],
    )

    async def main():
        result = await Runner.run(agent, "Which coffee shop should I go to, taking into account my preferences and the weather today in SF?")
        print(result.final_output)
    ```

  

**Function tools:**

This capability allows you to integrate any Python function as a tool that your agent can utilize. The Agent SDK streamlines this process by automatically handling much of the setup:

* The tool's name defaults to the name of your Python function, although you have the option to specify a different name.
* The tool's description is automatically extracted from the docstring of your function. You can also provide a custom description if needed.
* The schema for the input parameters of your function tool is automatically generated based on the function's arguments and their type hints.
* Descriptions for each individual input parameter are also derived from the function's docstring, unless you choose to disable this feature.

The SDK leverages Python's `inspect` module to analyze the function signature, `griffe` to parse docstrings (supporting Google, Sphinx, and NumPy formats), and `pydantic` to construct the parameter schema. The `@function_tool` decorator significantly simplifies the creation of function tools.

```python
import json
from typing_extensions import TypedDict, Any
from agents import Agent, FunctionTool, RunContextWrapper, function_tool

class Location(TypedDict):
    lat: float
    long: float

@function_tool
async def fetch_weather(location: Location) -> str:
    """Fetch the weather for a given location.

    Args:
        location: The location to fetch the weather for.
    """
    # In real life, we'd fetch the weather from a weather API
    return "sunny"

@function_tool(name_override="fetch_data")
def read_file(ctx: RunContextWrapper[Any], path: str, directory: str | None = None) -> str:
    """Read the contents of a file.

    Args:
        path: The path to the file to read.
        directory: The directory to read the file from.
    """
    # In real life, we'd read the file from the file system
    return "<file contents>"

agent = Agent(
    name="Assistant",
    tools=[fetch_weather, read_file],
)

for tool in agent.tools:
    if isinstance(tool, FunctionTool):
        print(tool.name)
        print(tool.description)
        print(json.dumps(tool.params_json_schema, indent=2))
        print()
```

Alternatively, you can directly instantiate `FunctionTool` objects for more manual control. This requires you to explicitly provide the `name`, `description`, `params_json_schema` (the JSON schema for the arguments), and an asynchronous `on_invoke_tool` function that will be executed when the tool is called.

```python
from typing import Any
from pydantic import BaseModel
from agents import RunContextWrapper, FunctionTool

def do_some_work(data: str) -> str:
    return "done"

class FunctionArgs(BaseModel):
    username: str
    age: int

async def run_function(ctx: RunContextWrapper[Any], args: str) -> str:
    parsed = FunctionArgs.model_validate_json(args)
    return do_some_work(data=f"{parsed.username} is {parsed.age} years old")

tool = FunctionTool(
    name="process_user",
    description="Processes extracted user data",
    params_json_schema=FunctionArgs.model_json_schema(),
    on_invoke_tool=run_function,
)
```

The SDK's automatic parsing of function signatures and docstrings greatly simplifies the creation of function tools. You can manage docstring parsing using the `use_docstring_info` parameter within the `@function_tool` decorator.

**Agents as tools:**

This is a powerful feature that enables you to use one agent as a tool for another agent. This allows for the creation of sophisticated multi-agent systems where a central "orchestrator" agent can delegate specific tasks to specialized agents.

```python
from agents import Agent, Runner
import asyncio

spanish_agent = Agent(
    name="Spanish agent",
    instructions="You translate the user's message to Spanish",
)

french_agent = Agent(
    name="French agent",
    instructions="You translate the user's message to French",
)

orchestrator_agent = Agent(
    name="orchestrator_agent",
    instructions=(
        "You are a translation agent. You use the tools given to you to translate."
        "If asked for multiple translations, you call the relevant tools."
    ),
    tools=[
        spanish_agent.as_tool(
            tool_name="translate_to_spanish",
            tool_description="Translate the user's message to Spanish",
        ),
        french_agent.as_tool(
            tool_name="translate_to_french",
            tool_description="Translate the user's message to French",
        ),
    ],
)

async def main():
    result = await Runner.run(orchestrator_agent, input="Say 'Hello, how are you?' in Spanish.")
    print(result.final_output)
```

The `agent.as_tool()` method provides a convenient way to convert an existing `Agent` instance into a tool. For more advanced scenarios where you need finer control over the execution of the sub-agent (e.g., setting `max_turns`), you can directly use `Runner.run()` within a `@function_tool`.

```python
from agents import Agent, Runner, function_tool
from typing import Any

@function_tool
async def run_my_agent() -> str:
  """A tool that runs the agent with custom configs."""

    agent = Agent(name="My agent", instructions="...")

    result = await Runner.run(
        agent,
        input="...",
        max_turns=5,
        run_config=...
    )

    return str(result.final_output)
```

When using the `@function_tool` decorator, you can also specify a `failure_error_function` to handle errors that might occur during the execution of the tool and provide a custom error response to the LLM. By default, a `default_tool_error_function` is used. Setting `failure_error_function` to `None` will cause any errors to be re-raised. If you are manually creating a `FunctionTool` object, you are responsible for handling any potential errors within your `on_invoke_tool` function.

