
---

# OpenAI Agents SDK 🚀

Build intelligent AI applications effortlessly with the **OpenAI Agents SDK**. This lightweight tool enables you to create powerful AI agents capable of performing tasks, interacting with each other, and utilizing external tools—all with minimal complexity.

### Key Features 🌟

- **Agents**: LLMs with instructions and tools to handle tasks.
- **Handoffs**: Delegate tasks between agents for specialized processing.
- **Guardrails**: Validate inputs and outputs to ensure safe and appropriate agent behavior.
- **Python-First**: Utilize Python's natural features to orchestrate agents.
- **Tracing**: Visualize and debug agent workflows.

- 
Let's break down the provided code example into smaller chunks and explain each part:

```python
from agents import Agent, Runner, handoff
from pydantic import BaseModel
```
- **Import Statements**: Here, we import essential components from the `agents` module:
  - `Agent`: Used to create AI agents with specific instructions and behaviors.
  - `Runner`: Facilitates the execution of agents.
  - `handoff`: Allows agents to delegate tasks to other agents.
- We also import `BaseModel` from the `pydantic` library, which is used for data validation and settings management.

```python
def validate_input(input_data: str) -> bool:
    if 'homework' in input_data.lower():
        return False
    return True
```
- **Guardrail Function**: This function, `validate_input`, acts as a guardrail to validate incoming data.
  - It checks if the word 'homework' appears in the `input_data` (case-insensitive).
  - If 'homework' is found, it returns `False`, indicating the input is not valid.
  - Otherwise, it returns `True`, allowing the input to proceed.

```python
class EducationalAgent(Agent):
    def __init__(self, name: str):
        super().__init__(name=name)
        self.input_guardrail = validate_input

    def run(self, input_data: str):
        if not self.input_guardrail(input_data):
            return "I'm sorry, but I can't assist with homework-related requests."
        return f"Here's some educational content about {input_data}."
```
- **EducationalAgent Class**: This class defines an agent specializing in educational content.
  - **Initialization (`__init__`)**:
    - Inherits from the `Agent` class.
    - Sets the agent's name.
    - Assigns the `validate_input` function as the input guardrail.
  - **Run Method**:
    - Accepts `input_data` as a parameter.
    - Validates the input using the `input_guardrail`.
    - If invalid, returns a message indicating it can't assist with homework-related requests.
    - If valid, provides educational content related to the input.

```python
class ResearchAgent(Agent):
    def run(self, input_data: str):
        return f"Researching information on {input_data}..."
```
- **ResearchAgent Class**: This class defines an agent focused on conducting research.
  - **Run Method**:
    - Accepts `input_data` as a parameter.
    - Simulates a research task by returning a message indicating it's researching the given topic.

```python
educational_agent = EducationalAgent(name="Educational Assistant")
research_agent = ResearchAgent(name="Research Specialist")
```
- **Agent Instances**: Here, we create two instances:
  - `educational_agent`: An instance of `EducationalAgent`, named "Educational Assistant".
  - `research_agent`: An instance of `ResearchAgent`, named "Research Specialist".

```python
handoff_obj = handoff(agent=research_agent, on_handoff=None)
```
- **Handoff Setup**: Establishes a handoff mechanism:
  - `handoff_obj` is created to facilitate transferring tasks from one agent to another.
  - It specifies that when a handoff occurs, no additional action (`on_handoff=None`) is performed.

```python
input_data = "Can you help me with my math homework?"
result = Runner.run_sync(educational_agent, input_data)
print(result.final_output)
```
- **Agent Execution**:
  - `input_data` is defined as a request for math homework assistance.
  - `Runner.run_sync` synchronously runs the `educational_agent` with the provided input.
  - The agent processes the input, applies the guardrail, and returns the appropriate response.
  - `result.final_output` is printed, displaying the agent's response.

**Output:**
```
I'm sorry, but I can't assist with homework-related requests.
```
- **Explanation**:
  - The `EducationalAgent` detects the term 'homework' in the input.
  - The input is deemed invalid based on the guardrail.
  - The agent responds with a message indicating it cannot assist with homework-related requests.

This example demonstrates how to create AI agents with specific behaviors, validate inputs using guardrails, and set up task delegation between agents using handoffs. These features enable the development of sophisticated AI systems capable of handling complex workflows and ensuring appropriate interactions. 
