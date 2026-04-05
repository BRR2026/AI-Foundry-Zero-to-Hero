# Module 19: Custom Tools & Function Calling

> **Azure AI Foundry: Zero to Hero** · Arc 4: AI Agents · April 2026  
> Module 19 of 45

---

## 🎯 Module Overview

This module teaches you to extend AI agents with custom tools using function calling in Azure AI Foundry. You will define tool schemas with JSON Schema, implement function calling patterns, connect agents to live APIs and databases, and build robust error handling for production deployments.

**Duration:** 3–4 hours (lecture + hands-on lab)  
**Level:** Intermediate to Advanced  
**Prerequisites:** Module 18 — Agent Architecture Fundamentals

---

## 📋 Learning Objectives

By the end of this module, learners will be able to:

1. **Define** custom function tools with precise JSON Schema parameter specifications
2. **Implement** sequential, parallel, and chained function calling patterns
3. **Connect** AI agents to external REST APIs, databases, and microservices securely
4. **Build** error handling strategies including retries, circuit breakers, and fallback responses
5. **Construct** a multi-tool agent that checks weather and books meetings (Mini Hack)

---

## 🗂️ Module Structure

| Section | Topic | Duration |
|---------|-------|----------|
| 1 | Introduction to Function Calling | 20 min |
| 2 | Defining Custom Functions for Agents | 30 min |
| 3 | Function Calling Patterns & JSON Schema | 35 min |
| 4 | Connecting to External APIs & Databases | 35 min |
| 5 | Error Handling & Fallback Strategies | 30 min |
| 6 | Mini Hack: Weather + Meeting Booker Agent | 60 min |
| 7 | Review & Assessment | 20 min |

---

## 📖 Section 1: Introduction to Function Calling

### What Is Function Calling?

Function calling is the mechanism that allows an LLM to request the execution of external functions during a conversation. The model does **not** execute code directly — it outputs a structured JSON object indicating which function to call and with what arguments. Your application code then:

1. Receives the tool call request from the model
2. Executes the specified function with the provided arguments
3. Returns the result to the model for further reasoning

### Why Function Calling Matters

- **Real-world actions:** Agents can book meetings, send emails, update databases
- **Live data:** Agents can fetch current weather, stock prices, system status
- **Business logic:** Agents can apply complex rules and calculations
- **System integration:** Agents become orchestrators connecting disparate services

### Function Calling in Azure AI Foundry

Azure AI Foundry uses the OpenAI-compatible tool specification format. Key concepts:

- **Tools** are registered when creating an agent
- **Tool definitions** use JSON Schema to describe function parameters
- **The agent runtime** manages the function calling loop automatically (with `create_and_process_run()`)
- **Manual mode** (`create_run()`) gives you control over each step for custom logic

---

## 📖 Section 2: Defining Custom Functions for Agents

### Anatomy of a Tool Definition

Every tool definition has three required components:

```python
tool_definition = {
    "type": "function",
    "function": {
        "name": "function_name",           # Unique identifier (snake_case)
        "description": "What this does",    # When/why to use it
        "parameters": {                     # JSON Schema for inputs
            "type": "object",
            "properties": { ... },
            "required": [ ... ],
            "additionalProperties": false
        }
    }
}
```

### Writing Effective Descriptions

The `description` field is critical — the model uses it to decide **when** to call the tool.

**Good description:**
> "Get current weather conditions including temperature, humidity, and wind speed for a specific city. Returns data from OpenWeatherMap API. Use when the user asks about weather, temperature, or outdoor conditions."

**Bad description:**
> "Gets weather"

### JSON Schema Best Practices

| Practice | Reason | Example |
|----------|--------|---------|
| Use `enum` | Constrains values | `"enum": ["celsius", "fahrenheit"]` |
| Add `pattern` | Validates format | `"pattern": "^\\d{4}-\\d{2}-\\d{2}$"` |
| Set boundaries | Prevents invalid numbers | `"minimum": 1, "maximum": 100` |
| Mark `required` | Ensures essential fields | `"required": ["city"]` |
| Disable extras | Prevents hallucinated fields | `"additionalProperties": false` |
| Keep flat | Deep nesting reduces accuracy | Max 2 levels of nesting |

### Example Tool Definitions

#### Weather Lookup Tool

```python
weather_tool = {
    "type": "function",
    "function": {
        "name": "get_weather",
        "description": "Get current weather conditions and forecast for a city. Returns temperature, humidity, wind speed, and conditions.",
        "parameters": {
            "type": "object",
            "properties": {
                "city": {
                    "type": "string",
                    "description": "City name, e.g., 'Seattle' or 'London'"
                },
                "units": {
                    "type": "string",
                    "enum": ["celsius", "fahrenheit"],
                    "description": "Temperature unit preference"
                }
            },
            "required": ["city"],
            "additionalProperties": false
        }
    }
}
```

#### Meeting Booking Tool

```python
meeting_tool = {
    "type": "function",
    "function": {
        "name": "book_meeting",
        "description": "Schedule a calendar meeting. Checks for conflicts and sends invitations to attendees.",
        "parameters": {
            "type": "object",
            "properties": {
                "title": {"type": "string", "description": "Meeting subject"},
                "date": {"type": "string", "pattern": "^\\d{4}-\\d{2}-\\d{2}$", "description": "ISO date (YYYY-MM-DD)"},
                "start_time": {"type": "string", "description": "24-hour format (HH:MM)"},
                "duration_minutes": {"type": "integer", "minimum": 15, "maximum": 480},
                "attendees": {
                    "type": "array",
                    "items": {"type": "string", "format": "email"},
                    "description": "Attendee email addresses"
                }
            },
            "required": ["title", "date", "start_time", "duration_minutes"],
            "additionalProperties": false
        }
    }
}
```

---

## 📖 Section 3: Function Calling Patterns & JSON Schema

### Pattern 1: Sequential Calling

The model calls one function, receives the result, then decides whether to call another.

**Use case:** Weather check → decide → book meeting

```python
from azure.ai.projects import AIProjectClient
from azure.identity import DefaultAzureCredential

client = AIProjectClient(
    credential=DefaultAzureCredential(),
    project_endpoint="https://<your-project>.services.ai.azure.com"
)

agent = client.agents.create_agent(
    model="gpt-4o",
    name="scheduling-assistant",
    instructions="You are a scheduling assistant. Check weather before suggesting outdoor activities.",
    tools=[weather_tool, meeting_tool]
)

thread = client.agents.create_thread()
client.agents.create_message(
    thread_id=thread.id,
    role="user",
    content="What's the weather in Seattle? If nice, book a lunch meeting."
)

run = client.agents.create_and_process_run(
    thread_id=thread.id,
    agent_id=agent.id
)
```

### Pattern 2: Parallel Calling

The model requests multiple independent functions simultaneously.

**Use case:** "What's the weather in Seattle, Portland, and Vancouver?"

```python
import asyncio

async def handle_tool_calls(tool_calls):
    tasks = [dispatch_call(call) for call in tool_calls]
    results = await asyncio.gather(*tasks, return_exceptions=True)
    return results
```

### Pattern 3: Chained (Multi-Turn) Calling

Results from one function inform the next call. The model reasons over intermediate results.

**Use case:** Check weather → Check calendar availability → Book meeting → Send notification

### Manual Function Calling Loop

For fine-grained control (logging, approval gates, custom routing):

```python
import json

run = client.agents.create_run(thread_id=thread.id, agent_id=agent.id)

while run.status in ["queued", "in_progress", "requires_action"]:
    if run.status == "requires_action":
        tool_outputs = []
        for call in run.required_action.submit_tool_outputs.tool_calls:
            result = await router.dispatch(call)
            tool_outputs.append({
                "tool_call_id": call.id,
                "output": result
            })
        run = client.agents.submit_tool_outputs_and_process(
            thread_id=thread.id,
            run_id=run.id,
            tool_outputs=tool_outputs
        )
    else:
        import time
        time.sleep(1)
        run = client.agents.get_run(thread_id=thread.id, run_id=run.id)
```

---

## 📖 Section 4: Connecting to External APIs & Databases

### REST API Integration

```python
import httpx

async def get_weather(city: str, units: str = "celsius") -> dict:
    api_key = get_secret_from_keyvault("weather-api-key")
    async with httpx.AsyncClient(timeout=10.0) as http_client:
        response = await http_client.get(
            "https://api.weatherservice.com/v2/current",
            params={"q": city, "units": "metric" if units == "celsius" else "imperial", "appid": api_key}
        )
        response.raise_for_status()
        data = response.json()
    return {
        "city": city,
        "temperature": data["main"]["temp"],
        "humidity": data["main"]["humidity"],
        "conditions": data["weather"][0]["description"],
        "units": units
    }
```

### Database Integration

```python
import aioodbc

async def query_customer_orders(customer_id: str, status: str = "all") -> dict:
    conn_str = get_secret_from_keyvault("sql-connection-string")
    query = "SELECT order_id, product_name, quantity, total_price, status FROM orders WHERE customer_id = ?"
    params = [customer_id]
    if status != "all":
        query += " AND status = ?"
        params.append(status)
    
    async with aioodbc.connect(dsn=conn_str) as conn:
        async with conn.cursor() as cursor:
            await cursor.execute(query, params)
            rows = await cursor.fetchall()
    
    return {"customer_id": customer_id, "order_count": len(rows), "orders": [...]}
```

### Security Best Practices

- **Azure Key Vault** for all secrets (API keys, connection strings)
- **Parameterized queries** to prevent SQL injection
- **Least-privilege access** — read-only DB access where possible
- **Input validation** before executing any function
- **Rate limiting** to prevent abuse through tool calls

### Function Router Pattern

```python
class FunctionRouter:
    def __init__(self):
        self._registry = {}
    
    def register(self, name: str, func):
        self._registry[name] = func
    
    async def dispatch(self, tool_call) -> str:
        func_name = tool_call.function.name
        args = json.loads(tool_call.function.arguments)
        if func_name not in self._registry:
            return json.dumps({"error": f"Unknown function: {func_name}"})
        try:
            result = await self._registry[func_name](**args)
            return json.dumps(result)
        except Exception as e:
            return json.dumps({"error": str(e), "function": func_name})
```

---

## 📖 Section 5: Error Handling & Fallback Strategies

### The Error Handling Pyramid

1. **Input Validation** — Validate parameters before calling external services
2. **Timeouts & Retries** — Handle transient network failures
3. **Circuit Breaker** — Prevent cascade failures when services are down
4. **Fallback & Degradation** — Provide alternative paths
5. **Structured Error Responses** — Give the model context about what failed

### Retry with Exponential Backoff

```python
import asyncio, random
from functools import wraps

def retry_with_backoff(max_retries=3, base_delay=1.0, max_delay=30.0):
    def decorator(func):
        @wraps(func)
        async def wrapper(*args, **kwargs):
            for attempt in range(max_retries + 1):
                try:
                    return await func(*args, **kwargs)
                except (httpx.TimeoutException, httpx.HTTPStatusError):
                    if attempt == max_retries:
                        raise
                    delay = min(base_delay * (2 ** attempt) + random.uniform(0, 1), max_delay)
                    await asyncio.sleep(delay)
        return wrapper
    return decorator
```

### Circuit Breaker Pattern

```python
class CircuitBreaker:
    def __init__(self, failure_threshold=5, recovery_timeout=60):
        self.failure_threshold = failure_threshold
        self.recovery_timeout = recovery_timeout
        self.failure_count = 0
        self.last_failure_time = None
        self.state = "closed"  # closed=healthy, open=failing
    
    def can_execute(self) -> bool:
        if self.state == "closed":
            return True
        if time.time() - self.last_failure_time > self.recovery_timeout:
            self.state = "half-open"
            return True
        return False
    
    def record_success(self):
        self.failure_count = 0
        self.state = "closed"
    
    def record_failure(self):
        self.failure_count += 1
        self.last_failure_time = time.time()
        if self.failure_count >= self.failure_threshold:
            self.state = "open"
```

### Structured Error Responses

Always return structured errors so the model can reason about failures:

```json
{
    "error": "service_unavailable",
    "message": "Weather API returned HTTP 503",
    "retry_after_seconds": 30,
    "fallback_data": {
        "last_known_temperature": 18,
        "data_age_minutes": 45
    }
}
```

---

## 🛠️ Mini Hack: Weather + Meeting Booker Agent

See the **Hands-on Lab** (`lab/hands-on-lab.md`) for the complete step-by-step walkthrough.

**Summary:** Build an agent with two custom tools (`get_weather` and `book_meeting`), a function router, and error handling. Test with multi-step queries like "Check the weather in Seattle and book a rooftop lunch if it's sunny."

---

## 📺 Video Resource

**Build An AI Agent From Scratch Using Microsoft Foundry**  
🔗 [Watch on YouTube](https://www.youtube.com/watch?v=wL8H0RySf6I)

---

## 🔗 Key Resources

- [Azure AI Foundry — Function Calling Guide](https://learn.microsoft.com/azure/ai-services/agents/how-to/tools/function-calling)
- [Agent Tools Overview — Concepts](https://learn.microsoft.com/azure/ai-services/agents/concepts/tools)
- [Understanding JSON Schema](https://json-schema.org/understanding-json-schema/)
- [Azure AI Projects SDK — Python Reference](https://learn.microsoft.com/python/api/azure-ai-projects/)

---

## 🧭 Module Navigation

| Previous | Current | Next |
|----------|---------|------|
| [← Module 18: Agent Architecture Fundamentals](../module-18/blog.html) | **Module 19: Custom Tools & Function Calling** | [Module 20: Agent Memory & State Management →](../module-20/blog.html) |

---

*Azure AI Foundry: Zero to Hero — Module 19 of 45 · Arc 4: AI Agents · April 2026*
