
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

  
   2. **Function tools:** 
   
   This allows you to turn any Python function into a tool that your agent can use. The SDK automatically handles the setup:

    * The tool's name defaults to the Python function's name (or you can specify one).
    * The tool's description is taken from the function's docstring (or you can provide one).
    * The schema for the function's input parameters is automatically generated from the function's arguments and their type hints.
    * Descriptions for each input parameter are also extracted from the docstring (unless disabled).

    The SDK uses Python's `inspect` module for signature extraction, `griffe` for parsing docstrings (supporting Google, Sphinx, and NumPy formats), and `pydantic` for creating the schema. The `@function_tool` decorator simplifies this process.

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

    You can also create `FunctionTool` objects directly if you prefer more manual control, requiring you to provide the `name`, `description`, `params_json_schema`, and an asynchronous `on_invoke_tool` function.

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

    The SDK automatically parses function signatures and docstrings to simplify the creation of function tools. You can control docstring parsing with the `use_docstring_info` parameter in `@function_tool`.


  3 **Agents as tools:** 
  
  This powerful feature allows you to use one agent as a tool for another agent. This enables the creation of sophisticated agent workflows where a central orchestrator agent can call upon specialized agents to perform specific tasks.

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

    The `agent.as_tool()` method conveniently turns an agent into a tool. For more advanced control, you can directly use `Runner.run()` within a `@function_tool`.

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

    When using `@function_tool`, you can also provide a `failure_error_function` to handle errors during tool execution and provide a custom error response to the LLM. By default, a `default_tool_error_function` is used. Setting it to `None` will re-raise any errors. If you manually create a `FunctionTool`, you are responsible for error handling within the `on_invoke_tool` function.

