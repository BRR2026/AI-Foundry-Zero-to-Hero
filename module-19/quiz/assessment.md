# 📝 Module 19 Assessment: Custom Tools & Function Calling

> **Azure AI Foundry: Zero to Hero** · Arc 4: AI Agents  
> 10 Multiple-Choice Questions · Passing Score: 80% (8/10)

---

## Instructions

- Select the **single best answer** for each question.
- Review the explanations after completing all questions.
- Revisit the training material for any topics you find challenging.

---

### Question 1

**When function calling is triggered in Azure AI Foundry, what actually happens?**

A) The LLM directly executes the function code inside its runtime  
B) The model outputs a structured JSON request specifying the function name and arguments, and your application code executes it  
C) Azure AI Foundry automatically calls the external API without any application code  
D) The function is executed inside a sandboxed Azure Functions environment managed by the agent  

---

### Question 2

**Which three components are required in every tool definition for an Azure AI Foundry agent?**

A) `name`, `endpoint`, `authentication`  
B) `name`, `description`, `parameters`  
C) `function_id`, `description`, `return_type`  
D) `type`, `handler`, `schema`  

---

### Question 3

**Why is setting `"additionalProperties": false` in a tool's JSON Schema important?**

A) It improves the API response time by reducing payload size  
B) It prevents the model from hallucinating extra parameters that your function doesn't expect  
C) It's required by the Azure AI Foundry API and will throw an error if omitted  
D) It enables parallel function calling mode  

---

### Question 4

**A user asks: "What's the weather in Seattle, Portland, and Vancouver?" Which function calling pattern is most efficient for this scenario?**

A) Sequential calling — check each city one at a time  
B) Chained calling — use each city's result to decide the next call  
C) Parallel calling — request all three weather lookups simultaneously  
D) Nested calling — embed weather calls inside a summary function  

---

### Question 5

**What is the key difference between `create_and_process_run()` and `create_run()` in the Azure AI Foundry agents SDK?**

A) `create_and_process_run()` is for testing only; `create_run()` is for production  
B) `create_and_process_run()` handles the function calling loop automatically; `create_run()` gives you manual control over each tool call step  
C) `create_run()` supports more models; `create_and_process_run()` only works with GPT-4o  
D) There is no difference — they are aliases for the same operation  

---

### Question 6

**Where should API keys and database connection strings be stored when building function-calling agents?**

A) In the tool definition's `parameters` field  
B) Hard-coded in the function handler source code  
C) In Azure Key Vault, retrieved at runtime  
D) In the agent's system instructions  

---

### Question 7

**In the circuit breaker pattern, what happens when the circuit is in the "open" state?**

A) All function calls are executed normally  
B) Function calls are blocked and a fallback response is returned immediately  
C) Function calls are queued until the service recovers  
D) The circuit breaker retries the function call with exponential backoff  

---

### Question 8

**What is the best practice for returning errors from a function handler back to the AI model?**

A) Raise a Python exception and let the SDK handle it  
B) Return a structured JSON object with error type, message, and optional fallback data  
C) Return an empty string so the model knows something went wrong  
D) Return the raw HTTP error response from the external API  

---

### Question 9

**A developer writes a tool description that says: "Does stuff with data." Why is this problematic?**

A) It's too long — descriptions should be under 10 characters  
B) The model uses the description to decide when to call the tool, so vague descriptions lead to incorrect or missed tool selections  
C) It will cause a JSON Schema validation error  
D) It's fine — the model ignores descriptions and only uses the function name  

---

### Question 10

**In the Mini Hack scenario, the agent needs to check weather and then conditionally book a meeting. Which implementation approach is correct?**

A) Define a single combined tool called `check_weather_and_book_meeting`  
B) Define separate `get_weather` and `book_meeting` tools, and let the model chain them based on the weather result and system instructions  
C) Hard-code an if/else in the agent's system instructions to handle the logic  
D) Use two separate agents — one for weather and one for booking — and orchestrate them with a script  

---

## Answer Key

<details>
<summary>Click to reveal answers</summary>

### Answers

| # | Answer | Explanation |
|---|--------|-------------|
| 1 | **B** | The model outputs a structured JSON tool call request. Your application code is responsible for executing the function and returning the result. The LLM never executes code directly. |
| 2 | **B** | Every tool definition requires a `name` (unique identifier), `description` (tells the model when to use it), and `parameters` (JSON Schema defining the input shape). |
| 3 | **B** | Setting `additionalProperties: false` prevents the model from generating extra parameters that your function doesn't handle, reducing errors and improving reliability. |
| 4 | **C** | Parallel calling is most efficient here because the three weather lookups are independent. The model can request all three simultaneously, reducing total latency. |
| 5 | **B** | `create_and_process_run()` automates the entire function calling loop. `create_run()` returns control to your code at each step, allowing custom logging, approval gates, and routing. |
| 6 | **C** | Azure Key Vault is the recommended secure storage for secrets. Never hard-code API keys or connection strings in source code or tool definitions. |
| 7 | **B** | When a circuit breaker is "open," it immediately blocks calls and returns a fallback response, preventing cascade failures to an already-failing service. |
| 8 | **B** | Structured JSON errors with error type, message, and fallback data give the model enough context to communicate the failure to the user and potentially suggest alternatives. |
| 9 | **B** | The model relies heavily on the `description` to decide when to invoke a tool. A vague description like "does stuff with data" provides no useful signal, leading to the tool being called at wrong times or not called when needed. |
| 10 | **B** | Defining separate tools and using system instructions to guide chaining behavior is the correct approach. This keeps tools atomic and reusable while leveraging the model's reasoning to chain them appropriately. |

### Scoring

| Score | Result |
|-------|--------|
| 10/10 | ⭐ Excellent — Full mastery of function calling concepts |
| 8–9/10 | ✅ Pass — Strong understanding, review missed topics |
| 6–7/10 | ⚠️ Needs Review — Revisit error handling and patterns sections |
| Below 6 | ❌ Retake — Complete the training guide and lab before retrying |

</details>

---

## 🧭 Navigation

| Previous | Current | Next |
|----------|---------|------|
| [← Module 18 Assessment](../../module-18/quiz/assessment.md) | **Module 19: Custom Tools & Function Calling** | [Module 20 Assessment →](../../module-20/quiz/assessment.md) |

---

*Azure AI Foundry: Zero to Hero — Module 19 Assessment · Arc 4: AI Agents · April 2026*
