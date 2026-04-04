# Week 5 Assessment: Setting Up Your Environment + First "Hello World" Model

| | |
|---|---|
| **Total Questions** | 20 |
| **Question Types** | Multiple Choice · True/False · Code Reading · Short Answer |
| **Estimated Time** | 20–25 minutes |
| **Passing Score** | 70% (14 out of 20) |
| **Difficulty** | ⭐ Beginner (Easy → Medium → Hard progression) |

> **Instructions:** Answer all 20 questions. For multiple choice, select the single best answer. For true/false, write "True" or "False" and a brief justification if indicated. For short answer, write 2–4 sentences. No external resources — this tests what you remember!

---

## Section A: Multiple Choice (8 Questions)

*Select the single best answer for each question.*

---

**Question 1** *(Easy)*

Which command creates a Python virtual environment?

- A) `python create-venv .venv`
- B) `python -m venv .venv`
- C) `pip install virtualenv`
- D) `venv create .venv`

---

**Question 2** *(Easy)*

What Azure CLI command is used to sign in to Azure?

- A) `az connect`
- B) `az authenticate`
- C) `az login`
- D) `az signin`

---

**Question 3** *(Easy)*

Which model is recommended for development and testing due to its low cost?

- A) GPT-4o
- B) GPT-4o-mini
- C) GPT-4.1
- D) Claude 3.5

---

**Question 4** *(Medium)*

In the AI Foundry resource hierarchy, what is the relationship between Hub and Project?

- A) A Project contains multiple Hubs
- B) Hub and Project are the same thing
- C) A Hub contains multiple Projects
- D) They are unrelated resources

---

**Question 5** *(Medium)*

What does `finish_reason: "length"` indicate in an API response?

- A) The model finished generating naturally
- B) The response hit the max_tokens limit and was truncated
- C) The content was filtered for safety
- D) The model encountered an error

---

**Question 6** *(Medium)*

Which Python package provides the `ChatCompletionsClient` class?

- A) `openai`
- B) `azure-ai-projects`
- C) `azure-ai-inference`
- D) `azure-openai`

---

**Question 7** *(Medium)*

What is the effect of setting `temperature=0` on a model's output?

- A) The model refuses to respond
- B) The output is maximally random
- C) The output is nearly deterministic/consistent
- D) The model generates longer responses

---

**Question 8** *(Hard)*

What is the approximate cost of processing 1 million input tokens with GPT-4o-mini?

- A) $0.015
- B) $0.15
- C) $1.50
- D) $15.00

---

## Section B: True/False (4 Questions)

*Write "True" or "False" for each statement. Where indicated, provide a brief justification.*

---

**Question 9** *(Easy)*

True or False: The `.env` file containing API keys should be committed to Git.

---

**Question 10** *(Easy)*

True or False: You must deploy a model before you can use it in the Playground.

---

**Question 11** *(Medium)*

True or False: The system message counts toward your token usage.

---

**Question 12** *(Medium)*

True or False: Setting `top_p=0.1` and `temperature=1.5` at the same time is a recommended practice.

---

## Section C: Code Reading / Output Prediction (4 Questions)

*Read the code carefully and answer the question that follows.*

---

**Question 13** *(Medium)*

Given this code:

```python
response = client.complete(
    model="gpt-4o-mini",
    messages=[
        SystemMessage(content="Respond with exactly one word."),
        UserMessage(content="What color is the sky?")
    ],
    temperature=0,
    max_tokens=5
)
print(response.usage.prompt_tokens > response.usage.completion_tokens)
```

What will the `print` statement most likely output?

- A) `True`
- B) `False`
- C) `Error`
- D) `None`

---

**Question 14** *(Medium)*

What is wrong with this code?

```python
from azure.ai.inference import ChatCompletionsClient
from azure.core.credentials import AzureKeyCredential

client = ChatCompletionsClient(
    endpoint="https://my-endpoint.openai.azure.com/",
    credential=AzureKeyCredential("")  # Empty key
)
response = client.complete(
    model="gpt-4o-mini",
    messages=[{"role": "user", "content": "Hello"}]
)
```

- A) Wrong import statement
- B) The API key credential is empty
- C) Messages should use `SystemMessage`/`UserMessage` objects
- D) Both B and C

---

**Question 15** *(Hard)*

Examine the following conversation history:

```python
messages = [
    SystemMessage(content="You are helpful."),
    UserMessage(content="Hi!"),
    AssistantMessage(content="Hello! How can I help?"),
    UserMessage(content="What did I just say?")
]
```

How many messages are in the conversation history sent to the model?

- A) 2
- B) 3
- C) 4
- D) 1

---

**Question 16** *(Hard)*

A `curl` request to the Azure AI endpoint returns HTTP status code **429**. What should you do?

- A) Change the API key
- B) Wait and retry, or request a quota increase
- C) Use a different endpoint URL
- D) Switch to a different programming language

---

## Section D: Short Answer (4 Questions)

*Write 2–4 sentences for each answer.*

---

**Question 17** *(Medium)*

Explain the difference between the `temperature` and `top_p` parameters. When would you adjust each one?

---

**Question 18** *(Medium)*

Why is it important to maintain conversation history (previous messages) when building a chatbot? What happens if you only send the latest user message?

---

**Question 19** *(Medium)*

List three strategies to minimize costs when using Azure AI Foundry during development.

---

**Question 20** *(Hard)*

You run your Python "Hello World" script and get the error: `ResourceNotFoundError: (DeploymentNotFound)`. Describe what likely caused this error and the steps you would take to fix it.

---

## Answer Key

*Detailed explanations for every question.*

---

### Section A: Multiple Choice

**Question 1:** B) `python -m venv .venv`

*Explanation:* The `python -m venv` command uses Python's built-in `venv` module to create a lightweight virtual environment. The `-m` flag tells Python to run the `venv` module as a script, and `.venv` is the conventional directory name for the environment. Option C (`pip install virtualenv`) installs a third-party package but does not itself create an environment, and options A and D use invalid syntax.

---

**Question 2:** C) `az login`

*Explanation:* The `az login` command is the standard Azure CLI command that opens a browser window for interactive authentication against your Azure tenant. After successful login, it returns a list of your available subscriptions. The other options (`az connect`, `az authenticate`, `az signin`) are not valid Azure CLI commands.

---

**Question 3:** B) GPT-4o-mini

*Explanation:* GPT-4o-mini is Microsoft's recommended model for development and testing because it offers a strong balance of capability and cost-effectiveness. It is significantly cheaper per token than GPT-4o while still being highly capable for most development tasks. This makes it ideal for iterating on prompts and testing application logic without incurring high costs.

---

**Question 4:** C) A Hub contains multiple Projects

*Explanation:* In the Azure AI Foundry resource hierarchy, a Hub acts as a top-level organizational container that provides shared infrastructure such as networking, identity, and storage. Individual Projects live inside a Hub and serve as isolated workspaces for specific applications or teams. This parent-child relationship allows centralized governance at the Hub level while giving each Project its own deployments, data, and configurations.

---

**Question 5:** B) The response hit the max_tokens limit and was truncated

*Explanation:* When the API returns `finish_reason: "length"`, it means the model's response was cut off because it reached the `max_tokens` limit you specified (or the model's maximum context window). This is distinct from `finish_reason: "stop"`, which indicates the model completed its response naturally, and `finish_reason: "content_filter"`, which means the response was blocked by Azure's content safety filters.

---

**Question 6:** C) `azure-ai-inference`

*Explanation:* The `ChatCompletionsClient` class is provided by the `azure-ai-inference` SDK package. This is the recommended client library for interacting with chat completion models deployed in Azure AI Foundry. The `openai` package provides its own `OpenAI` client, `azure-ai-projects` is used for project-level management operations, and `azure-openai` is not a real package name.

---

**Question 7:** C) The output is nearly deterministic/consistent

*Explanation:* Setting `temperature=0` minimizes the randomness in the model's token selection process, making it choose the highest-probability token at each step. This results in nearly identical output each time you send the same prompt, which is ideal for testing, evaluation, and scenarios requiring reproducibility. Higher temperature values (e.g., 0.7–1.0) introduce more randomness and creativity into the responses.

---

**Question 8:** B) $0.15

*Explanation:* GPT-4o-mini is priced at approximately $0.15 per 1 million input tokens, making it one of the most cost-effective models available in Azure AI Foundry. For comparison, GPT-4o costs roughly $2.50–$5.00 per 1 million input tokens — significantly more expensive. Understanding token pricing is essential for budgeting and choosing the right model for your workload.

---

### Section B: True/False

**Question 9:** **False**

*Explanation:* The `.env` file should **never** be committed to Git because it contains sensitive credentials such as API keys and endpoint URLs. Instead, you should add `.env` to your `.gitignore` file to prevent accidental exposure. Committing secrets to version control is a serious security risk, as anyone with access to the repository — including its history — could extract and misuse those credentials.

---

**Question 10:** **True**

*Explanation:* In Azure AI Foundry, a model must be deployed to an endpoint before it can be used in the Playground or called via API. The deployment process provisions the compute resources needed to serve inference requests and assigns a deployment name that you reference in your code. Simply having access to a model in the model catalog is not sufficient — an active deployment with a "Succeeded" status is required.

---

**Question 11:** **True**

*Explanation:* The system message is included as part of the prompt sent to the model and therefore counts toward your input token usage. Every token in the system message is billed just like tokens in user and assistant messages. This is why it is important to keep system messages concise and purposeful — a lengthy system prompt adds cost to every single API call your application makes.

---

**Question 12:** **False**

*Explanation:* It is **not** recommended to set both `top_p` and `temperature` to extreme values simultaneously, as they both control the randomness of token selection and can interact unpredictably. Microsoft's and OpenAI's documentation advises adjusting one parameter or the other, not both at the same time. Setting `temperature=1.5` (very high randomness) with `top_p=0.1` (very restrictive sampling) sends contradictory signals to the model and can produce inconsistent results.

---

### Section C: Code Reading / Output Prediction

**Question 13:** A) `True`

*Explanation:* The system message ("Respond with exactly one word.") and user message ("What color is the sky?") together comprise the prompt tokens, which will be considerably more than the completion tokens. With `temperature=0` and the instruction to respond with one word, the model will likely reply with just "Blue" — a single token. Since the prompt tokens (roughly 15–20 tokens for both messages combined) far exceed the completion tokens (1–2 tokens), the expression `prompt_tokens > completion_tokens` evaluates to `True`.

---

**Question 14:** D) Both B and C

*Explanation:* This code has two issues. First, the `AzureKeyCredential("")` is initialized with an empty string, which means no valid API key is being provided — this will cause an authentication failure at runtime. Second, the messages are passed as raw dictionaries (`{"role": "user", "content": "Hello"}`) instead of using the typed `UserMessage` and `SystemMessage` objects from the `azure-ai-inference` SDK. While dictionaries may work in some SDK versions, the best practice with `azure-ai-inference` is to use the strongly-typed message classes.

---

**Question 15:** C) 4

*Explanation:* All four message objects in the list are sent to the model as part of the conversation history. The list contains one `SystemMessage`, two `UserMessage` objects, and one `AssistantMessage` — totaling exactly 4 messages. The model receives the full conversation context so it can understand the flow of the dialogue. In this case, when the user asks "What did I just say?", the model can look back at the prior `UserMessage("Hi!")` to provide a contextually accurate response.

---

**Question 16:** B) Wait and retry, or request a quota increase

*Explanation:* HTTP status code 429 means "Too Many Requests" — you have exceeded the rate limit or token-per-minute quota assigned to your deployment. The correct response is to implement a retry strategy with exponential backoff (waiting progressively longer between retries) and, if the issue persists, request a quota increase through the Azure portal. Changing the API key, switching the endpoint URL, or changing programming languages would not resolve a rate-limiting issue.

---

### Section D: Short Answer

**Question 17:**

*Expected Answer:* The `temperature` parameter controls the overall randomness of the model's output by scaling the probability distribution over tokens — lower values (e.g., 0) make output more deterministic, while higher values (e.g., 1.0+) increase creativity and variation. The `top_p` parameter (nucleus sampling) restricts the model to only consider tokens within a cumulative probability threshold — for example, `top_p=0.1` means only the top 10% most likely tokens are considered. You would lower `temperature` when you need consistent, factual responses (like code generation or data extraction), and adjust `top_p` when you want finer control over the diversity of word choices. The general best practice is to adjust one or the other, not both simultaneously, to avoid unpredictable interactions.

---

**Question 18:**

*Expected Answer:* Maintaining conversation history is essential because large language models are stateless — they have no built-in memory of previous interactions. By including prior user and assistant messages in each API call, you provide the model with the context it needs to give coherent, relevant responses that reference earlier parts of the conversation. If you only send the latest user message, the model will treat every request as a brand-new conversation, losing all context about what was previously discussed. This would result in a chatbot that cannot follow up on topics, answer clarifying questions, or maintain a coherent dialogue across multiple turns.

---

**Question 19:**

*Expected Answer:* Three effective strategies to minimize costs during development include: **(1)** Use GPT-4o-mini instead of GPT-4o for testing and iteration, as it is roughly 10–20× cheaper per token while still being highly capable for most development tasks. **(2)** Set appropriate `max_tokens` limits on your API calls to prevent the model from generating unnecessarily long responses, which directly reduces output token costs. **(3)** Cache API responses locally during development so that repeated tests with the same prompts don't incur additional charges — this is especially useful when debugging prompt logic or UI rendering. Additional strategies include using `temperature=0` for deterministic testing (avoiding the need to re-run prompts for consistency) and batching your testing sessions to minimize idle deployment time.

---

**Question 20:**

*Expected Answer:* The `ResourceNotFoundError: (DeploymentNotFound)` error means the deployment name specified in your code does not match any active deployment in your Azure AI Foundry project. This typically happens when the model name in your `.env` file or code (e.g., `model="gpt-4o-mini"`) does not exactly match the deployment name you configured in the Azure portal — deployment names are case-sensitive and may differ from the base model name. To fix this: **(1)** Open the Azure AI Foundry portal and navigate to your project's **Deployments** section, **(2)** verify the exact deployment name and confirm its status shows "Succeeded", **(3)** update your `.env` file or code to use the exact deployment name as listed in the portal, and **(4)** ensure your endpoint URL corresponds to the correct project and region. If no deployment exists, you need to create one by deploying the model from the model catalog first.

---

## Scoring Guide

| Score | Rating | Recommendation |
|-------|--------|----------------|
| 18–20 | ⭐⭐⭐ Excellent | Ready for Week 6! |
| 14–17 | ⭐⭐ Good | Review weak areas, then proceed |
| 10–13 | ⭐ Needs Work | Re-read training guide, redo lab exercises |
| 0–9 | ❌ Not Ready | Repeat Week 5 before continuing |

---

### Point Values

| Section | Questions | Points Each | Total |
|---------|-----------|-------------|-------|
| A: Multiple Choice | 8 | 1 | 8 |
| B: True/False | 4 | 1 | 4 |
| C: Code Reading | 4 | 1 | 4 |
| D: Short Answer | 4 | 1 | 4 |
| **Total** | **20** | — | **20** |

> **Note:** Short answer questions (Section D) are graded on completeness and accuracy. Partial credit (0.5 points) may be awarded for answers that demonstrate understanding but miss key details.

---

*End of Week 5 Assessment*
