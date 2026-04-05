# 🛠️ Hands-On Lab: Custom Tools & Function Calling

> **Module 19** · Azure AI Foundry: Zero to Hero · Arc 4: AI Agents  
> **Duration:** 60–90 minutes  
> **Level:** Intermediate–Advanced

---

## 🎯 Lab Objective

Build a fully functional AI agent in Azure AI Foundry that uses two custom tools — **weather lookup** and **meeting booking** — to handle multi-step user requests. You will implement function calling patterns, a function router, and production-grade error handling.

**By the end of this lab, your agent will:**
- Check live weather for any city
- Book calendar meetings with attendees
- Chain decisions (only suggest outdoor meetings when weather is favorable)
- Handle API failures gracefully with retries and circuit breakers

---

## 📋 Prerequisites

- [x] Azure subscription with Azure AI Foundry access
- [x] Azure AI Foundry project with a deployed **GPT-4o** or **GPT-4.1** model
- [x] Python 3.10+ installed
- [x] Completed Module 18 (Agent Architecture Fundamentals)
- [x] Basic familiarity with REST APIs and JSON

---

## 🔧 Environment Setup

### Step 1: Install Required Packages

```bash
pip install azure-ai-projects azure-identity httpx python-dotenv
```

### Step 2: Configure Environment Variables

Create a `.env` file in your project directory:

```env
PROJECT_ENDPOINT=https://<your-project>.services.ai.azure.com
MODEL_DEPLOYMENT=gpt-4o
WEATHER_API_KEY=<your-openweathermap-api-key>
```

> 💡 **Tip:** Get a free API key from [OpenWeatherMap](https://openweathermap.org/api) for the weather tool, or use the mock implementation provided in this lab.

### Step 3: Create Project Structure

```
module-19-lab/
├── main.py              # Agent entry point
├── tools.py             # Tool definitions (JSON Schema)
├── handlers.py          # Function implementations
├── router.py            # Function router with error handling
├── resilience.py        # Retry & circuit breaker utilities
├── .env                 # Environment variables
└── requirements.txt     # Dependencies
```

---

## 📝 Exercise 1: Define Tool Schemas (15 minutes)

Create `tools.py` with JSON Schema tool definitions.

### Task 1.1: Weather Lookup Tool

```python
# tools.py

weather_tool = {
    "type": "function",
    "function": {
        "name": "get_weather",
        "description": (
            "Get current weather conditions for a specific city. "
            "Returns temperature, humidity, wind speed, and a short text forecast. "
            "Use when the user asks about weather, temperature, or outdoor conditions."
        ),
        "parameters": {
            "type": "object",
            "properties": {
                "city": {
                    "type": "string",
                    "description": "City name, e.g., 'Seattle', 'London', 'Tokyo'"
                },
                "units": {
                    "type": "string",
                    "enum": ["celsius", "fahrenheit"],
                    "description": "Temperature unit preference. Defaults to celsius.",
                    "default": "celsius"
                }
            },
            "required": ["city"],
            "additionalProperties": False
        }
    }
}
```

### Task 1.2: Meeting Booking Tool

```python
meeting_tool = {
    "type": "function",
    "function": {
        "name": "book_meeting",
        "description": (
            "Schedule a meeting in the user's calendar. "
            "Checks for conflicts and sends invitations to all attendees. "
            "Use when the user wants to schedule, book, or create a meeting or appointment."
        ),
        "parameters": {
            "type": "object",
            "properties": {
                "title": {
                    "type": "string",
                    "description": "Meeting title or subject line"
                },
                "date": {
                    "type": "string",
                    "description": "Meeting date in ISO 8601 format (YYYY-MM-DD)",
                    "pattern": "^\\d{4}-\\d{2}-\\d{2}$"
                },
                "start_time": {
                    "type": "string",
                    "description": "Start time in 24-hour format (HH:MM)",
                    "pattern": "^\\d{2}:\\d{2}$"
                },
                "duration_minutes": {
                    "type": "integer",
                    "description": "Meeting duration in minutes",
                    "minimum": 15,
                    "maximum": 480
                },
                "attendees": {
                    "type": "array",
                    "items": {
                        "type": "string",
                        "format": "email"
                    },
                    "description": "List of attendee email addresses (optional)"
                },
                "location": {
                    "type": "string",
                    "description": "Meeting location (optional, e.g., 'Conference Room A', 'Rooftop Patio')"
                }
            },
            "required": ["title", "date", "start_time", "duration_minutes"],
            "additionalProperties": False
        }
    }
}
```

### ✅ Checkpoint 1

Verify your tool definitions:
- [ ] Both tools have descriptive names in snake_case
- [ ] Descriptions explain **when** to use each tool
- [ ] Required fields are specified
- [ ] `additionalProperties` is set to `False`
- [ ] Enums and patterns are used for constrained fields

---

## 📝 Exercise 2: Implement Function Handlers (20 minutes)

Create `handlers.py` with the actual function implementations.

### Task 2.1: Weather Handler (with Mock Fallback)

```python
# handlers.py

import httpx
import os
import random
from datetime import datetime

# Mock weather data for testing without an API key
MOCK_WEATHER = {
    "seattle": {"temp_c": 14, "humidity": 72, "conditions": "partly cloudy", "wind_kph": 12},
    "portland": {"temp_c": 16, "humidity": 65, "conditions": "sunny", "wind_kph": 8},
    "new york": {"temp_c": 22, "humidity": 58, "conditions": "clear sky", "wind_kph": 15},
    "london": {"temp_c": 11, "humidity": 80, "conditions": "light rain", "wind_kph": 20},
    "tokyo": {"temp_c": 26, "humidity": 70, "conditions": "warm and humid", "wind_kph": 10},
}


async def get_weather(city: str, units: str = "celsius") -> dict:
    """Fetch weather data — uses live API if key is set, otherwise mock data."""
    api_key = os.getenv("WEATHER_API_KEY")
    
    if api_key and api_key != "<your-openweathermap-api-key>":
        return await _get_live_weather(city, units, api_key)
    else:
        return _get_mock_weather(city, units)


async def _get_live_weather(city: str, units: str, api_key: str) -> dict:
    """Call OpenWeatherMap API for live data."""
    unit_param = "metric" if units == "celsius" else "imperial"
    
    async with httpx.AsyncClient(timeout=10.0) as client:
        response = await client.get(
            "https://api.openweathermap.org/data/2.5/weather",
            params={"q": city, "units": unit_param, "appid": api_key}
        )
        response.raise_for_status()
        data = response.json()
    
    return {
        "city": data["name"],
        "temperature": data["main"]["temp"],
        "feels_like": data["main"]["feels_like"],
        "humidity": data["main"]["humidity"],
        "conditions": data["weather"][0]["description"],
        "wind_speed": data["wind"]["speed"],
        "units": units,
        "timestamp": datetime.utcnow().isoformat()
    }


def _get_mock_weather(city: str, units: str) -> dict:
    """Return mock weather data for testing."""
    city_lower = city.lower().strip()
    
    if city_lower in MOCK_WEATHER:
        data = MOCK_WEATHER[city_lower]
    else:
        # Generate random data for unknown cities
        data = {
            "temp_c": random.randint(5, 35),
            "humidity": random.randint(30, 90),
            "conditions": random.choice(["sunny", "cloudy", "rainy", "partly cloudy"]),
            "wind_kph": random.randint(5, 30)
        }
    
    temp = data["temp_c"] if units == "celsius" else round(data["temp_c"] * 9/5 + 32)
    
    return {
        "city": city.title(),
        "temperature": temp,
        "feels_like": temp - random.randint(0, 3),
        "humidity": data["humidity"],
        "conditions": data["conditions"],
        "wind_speed": data["wind_kph"],
        "units": units,
        "timestamp": datetime.utcnow().isoformat(),
        "source": "mock"
    }
```

### Task 2.2: Meeting Booking Handler

```python
# Add to handlers.py

from datetime import datetime, timedelta

# In-memory calendar for demo purposes
_calendar = []


async def book_meeting(
    title: str,
    date: str,
    start_time: str,
    duration_minutes: int,
    attendees: list[str] = None,
    location: str = None
) -> dict:
    """Book a meeting — checks for conflicts and adds to calendar."""
    
    # Parse and validate the date/time
    try:
        meeting_start = datetime.fromisoformat(f"{date}T{start_time}:00")
        meeting_end = meeting_start + timedelta(minutes=duration_minutes)
    except ValueError as e:
        return {
            "success": False,
            "error": "invalid_datetime",
            "message": f"Could not parse date/time: {e}"
        }
    
    # Check for conflicts
    for existing in _calendar:
        existing_start = datetime.fromisoformat(existing["start"])
        existing_end = datetime.fromisoformat(existing["end"])
        if meeting_start < existing_end and meeting_end > existing_start:
            return {
                "success": False,
                "error": "conflict",
                "message": f"Conflicts with '{existing['title']}' at {existing['start']}",
                "conflicting_meeting": existing["title"]
            }
    
    # Book the meeting
    meeting = {
        "id": f"MTG-{len(_calendar) + 1:04d}",
        "title": title,
        "date": date,
        "start": meeting_start.isoformat(),
        "end": meeting_end.isoformat(),
        "duration_minutes": duration_minutes,
        "attendees": attendees or [],
        "location": location or "Not specified",
        "status": "confirmed"
    }
    _calendar.append(meeting)
    
    return {
        "success": True,
        "meeting_id": meeting["id"],
        "title": title,
        "date": date,
        "start_time": start_time,
        "end_time": meeting_end.strftime("%H:%M"),
        "duration_minutes": duration_minutes,
        "attendees_count": len(meeting["attendees"]),
        "location": meeting["location"],
        "message": f"Meeting '{title}' booked successfully!"
    }
```

### ✅ Checkpoint 2

Test your handlers locally:
```python
import asyncio

async def test():
    weather = await get_weather("Seattle", "fahrenheit")
    print(f"Weather: {weather}")
    
    meeting = await book_meeting(
        title="Team Standup",
        date="2026-04-15",
        start_time="10:00",
        duration_minutes=30,
        attendees=["alice@contoso.com"]
    )
    print(f"Meeting: {meeting}")

asyncio.run(test())
```

- [ ] Weather returns temperature, conditions, and humidity
- [ ] Meeting returns a confirmation with meeting ID
- [ ] Conflict detection works (book two overlapping meetings)

---

## 📝 Exercise 3: Build the Function Router (10 minutes)

Create `router.py` to dispatch tool calls.

```python
# router.py

import json
from typing import Callable, Any

class FunctionRouter:
    """Routes agent tool calls to implementation functions."""
    
    def __init__(self):
        self._registry: dict[str, Callable] = {}
    
    def register(self, name: str, func: Callable) -> None:
        """Register a function handler by tool name."""
        self._registry[name] = func
        print(f"  ✅ Registered tool: {name}")
    
    async def dispatch(self, tool_call) -> str:
        """Execute the requested function and return a JSON string result."""
        func_name = tool_call.function.name
        raw_args = tool_call.function.arguments
        
        print(f"\n  🔧 Tool call: {func_name}")
        print(f"     Arguments: {raw_args}")
        
        # Check if function exists
        if func_name not in self._registry:
            error = {"error": "unknown_function", "message": f"No handler for '{func_name}'"}
            print(f"     ❌ {error['message']}")
            return json.dumps(error)
        
        # Parse arguments
        try:
            args = json.loads(raw_args)
        except json.JSONDecodeError as e:
            error = {"error": "invalid_arguments", "message": f"Could not parse arguments: {e}"}
            print(f"     ❌ {error['message']}")
            return json.dumps(error)
        
        # Execute function
        try:
            result = await self._registry[func_name](**args)
            print(f"     ✅ Success: {json.dumps(result)[:120]}...")
            return json.dumps(result, default=str)
        except TypeError as e:
            error = {"error": "argument_error", "message": f"Invalid arguments for {func_name}: {e}"}
            print(f"     ❌ {error['message']}")
            return json.dumps(error)
        except Exception as e:
            error = {"error": "execution_error", "function": func_name, "message": str(e)}
            print(f"     ❌ Execution error: {e}")
            return json.dumps(error)
```

---

## 📝 Exercise 4: Add Error Handling & Resilience (15 minutes)

Create `resilience.py` with retry and circuit breaker utilities.

```python
# resilience.py

import asyncio
import random
import time
from functools import wraps

def retry_with_backoff(max_retries: int = 3, base_delay: float = 1.0, max_delay: float = 30.0):
    """Decorator: retry async functions with exponential backoff + jitter."""
    def decorator(func):
        @wraps(func)
        async def wrapper(*args, **kwargs):
            last_exception = None
            for attempt in range(max_retries + 1):
                try:
                    return await func(*args, **kwargs)
                except Exception as e:
                    last_exception = e
                    if attempt == max_retries:
                        print(f"     ⚠️ All {max_retries + 1} attempts failed for {func.__name__}")
                        raise
                    delay = min(base_delay * (2 ** attempt) + random.uniform(0, 1), max_delay)
                    print(f"     🔄 Retry {attempt + 1}/{max_retries} for {func.__name__} in {delay:.1f}s")
                    await asyncio.sleep(delay)
        return wrapper
    return decorator


class CircuitBreaker:
    """Prevents repeated calls to a consistently failing service."""
    
    def __init__(self, name: str, failure_threshold: int = 3, recovery_timeout: float = 60.0):
        self.name = name
        self.failure_threshold = failure_threshold
        self.recovery_timeout = recovery_timeout
        self.failure_count = 0
        self.last_failure_time: float | None = None
        self.state = "closed"  # closed = healthy, open = failing, half-open = testing
    
    def can_execute(self) -> bool:
        """Check if the circuit allows execution."""
        if self.state == "closed":
            return True
        if self.state == "open" and self.last_failure_time:
            elapsed = time.time() - self.last_failure_time
            if elapsed > self.recovery_timeout:
                self.state = "half-open"
                print(f"     🔄 Circuit '{self.name}' → half-open (testing)")
                return True
        return False
    
    def record_success(self):
        """Record a successful call."""
        if self.state == "half-open":
            print(f"     ✅ Circuit '{self.name}' → closed (recovered)")
        self.failure_count = 0
        self.state = "closed"
    
    def record_failure(self):
        """Record a failed call."""
        self.failure_count += 1
        self.last_failure_time = time.time()
        if self.failure_count >= self.failure_threshold:
            self.state = "open"
            print(f"     🔴 Circuit '{self.name}' → OPEN (threshold reached)")
    
    def get_status(self) -> dict:
        """Return current circuit breaker status."""
        return {
            "name": self.name,
            "state": self.state,
            "failure_count": self.failure_count,
            "threshold": self.failure_threshold
        }


def with_circuit_breaker(breaker: CircuitBreaker, fallback_result: dict | None = None):
    """Decorator: wrap an async function with circuit breaker logic."""
    def decorator(func):
        @wraps(func)
        async def wrapper(*args, **kwargs):
            if not breaker.can_execute():
                result = fallback_result or {
                    "error": "circuit_open",
                    "message": f"Service '{breaker.name}' is temporarily unavailable",
                    "retry_after_seconds": int(breaker.recovery_timeout)
                }
                print(f"     🔴 Circuit open for '{breaker.name}' — returning fallback")
                return result
            try:
                result = await func(*args, **kwargs)
                breaker.record_success()
                return result
            except Exception as e:
                breaker.record_failure()
                return fallback_result or {
                    "error": "service_failure",
                    "message": str(e),
                    "circuit_status": breaker.get_status()
                }
        return wrapper
    return decorator
```

---

## 📝 Exercise 5: Assemble the Agent (15 minutes)

Create `main.py` — the complete agent with all tools, routing, and error handling.

```python
# main.py

import asyncio
import json
import os
from dotenv import load_dotenv
from azure.ai.projects import AIProjectClient
from azure.identity import DefaultAzureCredential

from tools import weather_tool, meeting_tool
from handlers import get_weather, book_meeting
from router import FunctionRouter
from resilience import retry_with_backoff, CircuitBreaker, with_circuit_breaker

load_dotenv()


# --- Apply resilience decorators ---
weather_breaker = CircuitBreaker("weather-api", failure_threshold=3, recovery_timeout=30)

@with_circuit_breaker(weather_breaker, fallback_result={
    "error": "weather_unavailable",
    "message": "Weather service is temporarily unavailable. Try again shortly.",
    "fallback_data": {"suggestion": "Check a weather website for current conditions."}
})
@retry_with_backoff(max_retries=2, base_delay=1.0)
async def resilient_get_weather(city: str, units: str = "celsius") -> dict:
    return await get_weather(city, units)


# --- Build function router ---
router = FunctionRouter()
router.register("get_weather", resilient_get_weather)
router.register("book_meeting", book_meeting)


async def run_agent(user_message: str):
    """Create and run an agent with function calling."""
    print(f"\n{'='*60}")
    print(f"👤 User: {user_message}")
    print(f"{'='*60}")
    
    # Initialize client
    client = AIProjectClient(
        credential=DefaultAzureCredential(),
        project_endpoint=os.getenv("PROJECT_ENDPOINT")
    )
    
    # Create agent with tools
    print("\n📦 Creating agent...")
    agent = client.agents.create_agent(
        model=os.getenv("MODEL_DEPLOYMENT", "gpt-4o"),
        name="weather-meeting-assistant",
        instructions=(
            "You are a helpful scheduling assistant that can check weather conditions "
            "and book meetings. Follow these rules:\n"
            "1. When asked about outdoor activities or meetings, check the weather first.\n"
            "2. Only suggest outdoor locations when conditions are favorable (no rain, temp > 10°C/50°F).\n"
            "3. When booking meetings, always confirm the details before proceeding.\n"
            "4. If a service is unavailable, inform the user and suggest alternatives.\n"
            "5. Always be concise and professional."
        ),
        tools=[weather_tool, meeting_tool]
    )
    print(f"  ✅ Agent created: {agent.id}")
    
    # Create thread and message
    thread = client.agents.create_thread()
    client.agents.create_message(
        thread_id=thread.id,
        role="user",
        content=user_message
    )
    
    # Process with manual function calling loop
    print("\n🔄 Processing...")
    run = client.agents.create_run(
        thread_id=thread.id,
        agent_id=agent.id
    )
    
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
            await asyncio.sleep(1)
            run = client.agents.get_run(thread_id=thread.id, run_id=run.id)
    
    # Get response
    if run.status == "completed":
        messages = client.agents.list_messages(thread_id=thread.id)
        assistant_message = messages.data[0].content[0].text.value
        print(f"\n🤖 Assistant: {assistant_message}")
    else:
        print(f"\n❌ Run failed with status: {run.status}")
    
    # Cleanup
    client.agents.delete_agent(agent.id)
    print(f"\n{'='*60}\n")


async def main():
    """Run test scenarios."""
    
    # Scenario 1: Simple weather check
    await run_agent("What's the weather like in Seattle today?")
    
    # Scenario 2: Chained weather → meeting decision
    await run_agent(
        "Check the weather in Portland. If it's sunny, book a team lunch "
        "on the rooftop for 2026-04-18 at 12:00 for 60 minutes."
    )
    
    # Scenario 3: Parallel-style multi-city weather
    await run_agent("Compare the weather in Seattle, Tokyo, and London.")
    
    # Scenario 4: Meeting booking with details
    await run_agent(
        "Book a project kickoff meeting for 2026-04-20 at 09:00, "
        "90 minutes, with alice@contoso.com and bob@contoso.com, "
        "in Conference Room B."
    )


if __name__ == "__main__":
    asyncio.run(main())
```

---

## ✅ Final Validation Checklist

### Functionality
- [ ] Weather tool returns temperature, humidity, conditions for any city
- [ ] Meeting tool books with all required fields and generates an ID
- [ ] Agent chains weather → meeting decisions based on conditions
- [ ] Conflict detection prevents double-booking

### Error Handling
- [ ] Invalid arguments return structured error JSON (not exceptions)
- [ ] Unknown function names return a clear error message
- [ ] Retry decorator retries transient failures up to 3 times
- [ ] Circuit breaker opens after 3 consecutive failures
- [ ] Fallback data is provided when circuit is open

### Code Quality
- [ ] Tool definitions use `additionalProperties: False`
- [ ] All secrets come from environment variables (not hard-coded)
- [ ] Function router handles all error types gracefully
- [ ] Logging shows tool call flow for debugging

---

## 🚀 Stretch Goals (Optional)

1. **Add a third tool** — `check_calendar_availability` that checks for open time slots before booking
2. **Implement caching** — Cache weather results for 10 minutes to reduce API calls
3. **Add input validation** — Validate date is not in the past, email format is correct
4. **Parallel execution** — Use `asyncio.gather()` to handle parallel tool calls efficiently
5. **Persistent calendar** — Store meetings in Azure Cosmos DB instead of in-memory

---

## 📺 Reference Video

**Build An AI Agent From Scratch Using Microsoft Foundry**  
🔗 [Watch on YouTube](https://www.youtube.com/watch?v=wL8H0RySf6I)

---

## 🧭 Navigation

| Previous | Current | Next |
|----------|---------|------|
| [← Module 18 Lab](../../module-18/lab/hands-on-lab.md) | **Module 19: Custom Tools & Function Calling** | [Module 20 Lab →](../../module-20/lab/hands-on-lab.md) |

---

*Azure AI Foundry: Zero to Hero — Module 19 Lab · Arc 4: AI Agents · April 2026*
