
### Output types

By default, agents produce plain text (i.e. str) outputs. If you want the agent to produce a particular type of output, you can use the output_type parameter. A common choice is to use Pydantic objects, but we support any type that can be wrapped in a Pydantic TypeAdapter - dataclasses, lists, TypedDict, etc.

https://openai.github.io/openai-agents-python/agents/#output-types

```python
from pydantic import BaseModel
from agents import Agent


class CalendarEvent(BaseModel):
    name: str
    date: str
    participants: list[str]

agent = Agent(
    name="Calendar extractor",
    instructions="Extract calendar events from text",
    output_type=CalendarEvent,
)
```

**Output Types**

By default, agents produce plain text (i.e., `str`) outputs. If you want the agent to produce a particular type of output, you can use the `output_type` parameter when creating your agent.

A common and powerful choice for `output_type` is to use Pydantic objects, like the `CalendarEvent` class defined above. However, the `output_type` parameter is flexible and supports any Python type that can be adapted by Pydantic's `TypeAdapter`. This includes:

* **Dataclasses:** Simple classes for holding data.
* **Lists (`list[ElementType]`):** When you expect the agent to return a collection of items of a specific type.
* **`typing.TypedDict`:** For creating dictionary-like structures with type hints.

**Note:**

When you pass an `output_type` to your `Agent`, you are instructing the underlying language model to generate structured outputs instead of the regular plain text responses. This allows you to receive the information back in a more organized and programmatically accessible format, making it easier to work with the agent's results in your code.

