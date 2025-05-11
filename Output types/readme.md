Output types
By default, agents produce plain text (i.e. str) outputs. If you want the agent to produce a particular type of output, you can use the output_type parameter. A common choice is to use Pydantic objects, but we support any type that can be wrapped in a Pydantic TypeAdapter - dataclasses, lists, TypedDict, etc.


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
Note

When you pass an output_type, that tells the model to use structured outputs instead of regular plain text responses.
