# Agentic AI 3.0 — Core Concepts README

A practical study guide covering:

* Pydantic
* LangChain Tools
* LangChain Messages
* Short-Term Memory
* Long-Term Memory
* Structured Output
* Middleware

---

# 1. Pydantic

## What is Pydantic?

Pydantic is a Python library used for **data validation, parsing, and structured data modelling**.

In Agentic AI systems, Pydantic is especially important because it is used for:

* Tool schemas
* Structured LLM outputs
* Agent configuration
* Response validation
* Validated pipeline results

The Agentic AI 3.0 syllabus specifically positions Pydantic as a foundation for modern agentic frameworks.

## Simple Example

```python
from pydantic import BaseModel


class User(BaseModel):
    name: str
    age: int


user = User(
    name="Mayank",
    age=25
)

print(user)
```

Pydantic validates that:

```text
name → string
age  → integer
```

---

## Why do agents need Pydantic?

LLMs normally produce text.

For example:

```text
"The customer is John and his age is 30."
```

But your application may need:

```json
{
    "name": "John",
    "age": 30
}
```

Pydantic allows us to define the expected structure.

```text
LLM
 ↓
Structured Output
 ↓
Pydantic Model
 ↓
Validated Python Object
 ↓
Application
```

---

## Pydantic + Agentic AI

Pydantic becomes especially important when:

```text
LLM
 ↓
Tool Schema
```

or:

```text
LLM
 ↓
Structured Response
 ↓
Pydantic Validation
```

The course covers:

* `BaseModel`
* Fields
* Defaults
* `field_validator`
* `model_validator`
* Required vs optional fields
* JSON parsing
* Serialization
* Nested models
* Type coercion
* Computed fields
* Model configuration
* `pydantic-settings`
* Structured LLM outputs
* Tool schemas

---

# 2. LangChain Tools

## What is a Tool?

A **tool is a capability that an agent can invoke to perform an action or retrieve information**.

An LLM itself can reason and generate text, but it cannot automatically:

* Query your database
* Call your API
* Search a file
* Perform a calculation
* Send an email

Tools give the agent access to these capabilities.

---

## Simple Architecture

```text
User
 ↓
Agent
 ↓
LLM
 ↓
"I need more information"
 ↓
Tool
 ↓
External System
 ↓
Tool Result
 ↓
LLM
 ↓
Final Answer
```

---

## Example

Imagine an agent has:

```text
Tools:
    calculator
    weather
    database_search
```

User asks:

```text
What is 25 × 40?
```

The model can decide:

```text
Use calculator
```

Then:

```text
LLM
 ↓
calculator(25, 40)
 ↓
1000
 ↓
LLM
 ↓
"The answer is 1000."
```

---

## Simple Python Tool

```python
from langchain_core.tools import tool


@tool
def add_numbers(a: int, b: int) -> int:
    """Add two numbers."""
    return a + b
```

The function becomes a tool that an agent can use.

Notice the type hints:

```python
a: int
b: int
```

These help describe the expected tool input.

This is one place where **Pydantic and tools connect**.

---

# 3. Messages

## What is a Message?

Messages represent the communication between the user, the model, and the system.

Common message roles include:

```text
System
Human
AI
Tool
```

Conceptually:

```text
SystemMessage
      ↓
HumanMessage
      ↓
AIMessage
      ↓
ToolMessage
      ↓
AIMessage
```

---

## System Message

Defines the model's behavior.

```python
from langchain_core.messages import SystemMessage

message = SystemMessage(
    content="You are a helpful Python teacher."
)
```

Think:

> "These are the rules/instructions for the model."

---

## Human Message

Represents the user's input.

```python
from langchain_core.messages import HumanMessage

message = HumanMessage(
    content="Explain Python decorators."
)
```

---

## AI Message

Represents the model's response.

```python
from langchain_core.messages import AIMessage

message = AIMessage(
    content="A decorator is a function that modifies another function."
)
```

---

## Tool Message

Represents the result returned by a tool.

```text
AI
 ↓
Tool Call
 ↓
Tool executes
 ↓
ToolMessage
 ↓
AI
```

This is extremely important in agent systems because tool execution becomes part of the conversation state.

---

# 4. Why Messages Matter in Agents

An agent isn't simply:

```text
Question → Answer
```

It can be:

```text
User
 ↓
AI
 ↓
Tool Call
 ↓
Tool Result
 ↓
AI
 ↓
Tool Call
 ↓
Tool Result
 ↓
AI
 ↓
Final Answer
```

Messages preserve this interaction history.

This becomes especially important in LangGraph, where message handling, trimming/filtering, token budgeting, and context-window management are explicit topics in the course.

---

# 5. Short-Term Memory

## What is Short-Term Memory?

Short-term memory is the information available **within the current conversation/thread/context**.

The course frames short-term memory as **in-context memory**.

Example:

```text
User:
My name is Mayank.

Agent:
Nice to meet you, Mayank.

User:
What is my name?

Agent:
Your name is Mayank.
```

The agent knows because the information is still available in its conversation context.

---

## Architecture

```text
Current Conversation
        │
        ▼
Message History
        │
        ▼
Context
        │
        ▼
LLM
```

Short-term memory is therefore closely related to:

* Message history
* Agent state
* Context windows
* Token limits
* Checkpointing

---

# 6. Why Short-Term Memory Is Needed

Without conversation history:

```text
User:
My name is Mayank.

        ↓

User:
What is my name?

        ↓

Agent:
I don't know.
```

With short-term memory:

```text
User
 ↓
Message History
 ↓
LLM
 ↓
"Your name is Mayank."
```

---

# 7. Problem with Short-Term Memory

The context window is not infinite.

Imagine:

```text
Message 1
Message 2
Message 3
...
Message 500
Message 501
...
```

Eventually, the conversation becomes expensive or exceeds the available context.

Therefore, production systems need:

* Token budgeting
* Message trimming
* Summarization
* Context compression
* Selective memory loading

## These are explicitly part of the course's context-engineering and LangGraph memory topics.

# 8. Long-Term Memory

## What is Long-Term Memory?

Long-term memory stores information **outside the current conversation context**, allowing an agent to retrieve it later.

The course describes long-term memory as **external storage**, with mem0 introduced as a persistent memory layer.

Example:

```text
Monday:

User:
I prefer Python examples.

        ↓

Long-Term Memory
        ↓
"user prefers Python examples"
```

Later:

```text
Friday:

User:
Explain decorators.

        ↓

Retrieve memory
        ↓
Python preference
        ↓
Agent gives Python-focused explanation
```

---

# 9. Short-Term vs Long-Term Memory

| Feature   | Short-Term                  | Long-Term            |
| --------- | --------------------------- | -------------------- |
| Scope     | Current conversation/thread | Across conversations |
| Storage   | Context/state               | External store       |
| Purpose   | Current task                | Persistent knowledge |
| Example   | Current messages            | User preferences     |
| Lifetime  | Usually session/thread      | Persistent           |
| Challenge | Context size                | Retrieval/storage    |

---

# 10. Memory Architecture

A production agent can have:

```text
                 AGENT
                   │
        ┌──────────┴──────────┐
        │                     │
 Short-Term              Long-Term
 Memory                   Memory
        │                     │
        ▼                     ▼
 Messages              External Store
        │                     │
        │                mem0 / DB /
        │                vector store
        │
        ▼
 Current Context
```

The course also introduces different memory types:

* Short-term
* Long-term
* Episodic
* Semantic
* Procedural

These are later connected to context engineering and persistent memory.

---

# 11. Structured Output

## The Problem

LLMs naturally generate free-form text.

Example:

```text
The candidate has 5 years of Python experience
and 3 years of machine learning experience.
```

But applications often need predictable data:

```json
{
    "python_experience": 5,
    "ml_experience": 3
}
```

This is where **structured output** becomes useful.

---

## Architecture

```text
User
 ↓
LLM
 ↓
Structured Output Schema
 ↓
Validation
 ↓
Application
```

Pydantic can define the schema.

```python
from pydantic import BaseModel


class Candidate(BaseModel):
    name: str
    python_experience: int
    ml_experience: int
```

Now the expected output has a clear structure.

---

# 12. Why Structured Output Is Important

Imagine you're building a hiring agent.

You need:

```text
candidate_name
skills
experience
education
score
```

If the LLM returns random text, your backend has to parse it.

Structured output provides a predictable interface:

```text
LLM
 ↓
Schema
 ↓
Validated Data
 ↓
Database/API/UI
```

The Agentic AI 3.0 syllabus explicitly connects Pydantic with structured LLM outputs, tool schemas, and response validation.

---

# 13. Structured Output vs Normal Output

### Normal output

```text
"The candidate is strong in Python."
```

### Structured output

```json
{
    "candidate": "John",
    "skill": "Python",
    "score": 9
}
```

The second is much easier for software to consume.

---

# 14. Middleware

## What is Middleware?

Middleware provides a mechanism to **intercept and customize agent execution**.

Instead of:

```text
Agent
 ↓
Model
```

we can have:

```text
Agent
 ↓
Middleware
 ↓
Model
 ↓
Middleware
 ↓
Response
```

Middleware can implement concerns such as:

* Logging
* Model routing
* Retry
* Tool control
* Guardrails
* PII handling
* Context management
* Human approval

---

# 15. `wrap_model_call`

One important middleware concept is:

```python
def wrap_model_call(request, handler):
    ...
```

The basic idea is:

```text
Middleware
     │
     ▼
Before Model
     │
     ▼
handler(request)
     │
     ▼
Model
     │
     ▼
Response
     │
     ▼
After Model
```

The `handler` represents the next step in execution.

---

# 16. Class-Based Middleware

For more complex middleware, a class can encapsulate the behavior.

```python
class LoggingMiddleware:

    def wrap_model_call(self, request, handler):

        print("Model call started")

        response = handler(request)

        print("Model call completed")

        return response
```

Conceptually:

```text
LoggingMiddleware
       │
       ├── Before Model
       │
       ├── Model Call
       │
       └── After Model
```

Middleware is therefore a powerful place to implement cross-cutting behavior without putting that logic directly into the agent's core reasoning logic.

---

# 17. How Everything Connects

This is the most important architecture to understand.

```text
                         USER
                           │
                           ▼
                    ┌─────────────┐
                    │    AGENT    │
                    └──────┬──────┘
                           │
                  ┌────────▼────────┐
                  │   MIDDLEWARE    │
                  └────────┬────────┘
                           │
                           ▼
                         LLM
                           │
              ┌────────────┼────────────┐
              │            │            │
              ▼            ▼            ▼
           Messages      Tools       Memory
                           │
                           ▼
                    External Systems
```

And structured output connects like this:

```text
                    LLM
                     │
                     ▼
             Structured Output
                     │
                     ▼
                 Pydantic
                     │
                     ▼
              Validated Object
```

---

# 18. Complete Agentic AI Flow

Putting all concepts together:

```text
USER
 │
 ▼
Messages
 │
 ▼
Agent
 │
 ▼
Middleware
 │
 ▼
LLM
 │
 ├───────────────┐
 │               │
 ▼               ▼
Tool           Memory
 │               │
 ▼               ├── Short-Term
External API     │
                 └── Long-Term
 │
 └───────────────┐
                 ▼
              LLM
                 │
                 ▼
        Structured Output
                 │
                 ▼
             Pydantic
                 │
                 ▼
          Validated Result
                 │
                 ▼
               USER
```

---

# 19. One Example Project

Imagine we're building a **Customer Support Agent**.

### User

```text
"Can you check the status of my order?"
```

### Messages

The user request becomes a message.

```text
HumanMessage
```

### Agent

The agent determines:

```text
I need order information.
```

### Tool

It calls:

```text
get_order_status()
```

### Middleware

Middleware can:

```text
Log request
 ↓
Check permissions
 ↓
Execute model/tool
 ↓
Track execution
```

### Short-Term Memory

The agent remembers what the user said during the current conversation.

### Long-Term Memory

The system may store persistent user information such as preferences or other application-specific memories.

### Structured Output

The model produces:

```json
{
    "order_id": "12345",
    "status": "shipped",
    "estimated_delivery": "Friday"
}
```

### Pydantic

Validates the response:

```python
class OrderStatus(BaseModel):
    order_id: str
    status: str
    estimated_delivery: str
```

Now the application receives reliable structured data.

---

# 20. The Relationship Between the Concepts

This is the mental model I recommend remembering:

```text
Pydantic
   │
   ├── Validates data
   ├── Defines schemas
   └── Structures outputs


Tools
   │
   └── Give agents capabilities


Messages
   │
   └── Carry communication/context


Short-Term Memory
   │
   └── Remembers current interaction


Long-Term Memory
   │
   └── Remembers persistent information


Structured Output
   │
   └── Makes LLM responses predictable


Middleware
   │
   └── Controls/intercepts execution
```


That project will make these concepts much easier to remember than studying each one independently.
