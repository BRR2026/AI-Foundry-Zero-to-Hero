# Module 30 Assessment: Cost Optimization & Quota Management

| | |
|---|---|
| **Total Questions** | 10 |
| **Question Types** | Multiple Choice |
| **Estimated Time** | 15–20 minutes |
| **Passing Score** | 80% (8 out of 10) |
| **Difficulty** | 🟡 Intermediate |

> **Instructions:** Answer all 10 questions. Select the single best answer for each question.

---

## Questions

**Question 1** *(Easy)*

What does PTU stand for in the context of Azure AI Foundry model deployments?

- A) Pay-per-Token Unit — a billing metric that charges per individual token processed
- B) Provisioned Throughput Unit — a reserved capacity unit guaranteeing dedicated model throughput
- C) Processing Time Unit — a measurement of how long a model takes to generate a response
- D) Performance Tier Upgrade — a premium licensing option that unlocks higher-quality model outputs

---

**Question 2** *(Easy)*

Which Azure service is the primary tool for monitoring and analyzing costs associated with Azure AI Foundry resources?

- A) Azure Monitor
- B) Azure Advisor
- C) Microsoft Cost Management
- D) Azure Service Health

---

**Question 3** *(Medium)*

You need to estimate token usage before sending requests to an Azure OpenAI model deployed in Azure AI Foundry. Which library is commonly used to count tokens for OpenAI models?

- A) transformers (Hugging Face tokenizer library)
- B) tiktoken (OpenAI's token counting library)
- C) sentencepiece (Google's tokenization library)
- D) nltk.tokenize (Natural Language Toolkit tokenizer)

---

**Question 4** *(Medium)*

Your Azure AI Foundry deployment has a quota limit expressed in TPM. What does TPM stand for, and why is it significant?

- A) Tokens Per Minute — it defines the maximum number of tokens your deployment can process per minute across all requests
- B) Throughput Per Model — it defines the total number of models you can deploy simultaneously in a project
- C) Tasks Per Month — it defines the maximum number of API calls allowed within a billing cycle
- D) Terabytes Per Model — it defines the maximum storage allocated for model weights and artifacts

---

**Question 5** *(Medium)*

A development team wants to reduce token consumption and lower costs when using a GPT-4o deployment in Azure AI Foundry. Which prompt optimization technique is most effective at reducing token usage?

- A) Increasing the `max_tokens` parameter to allow the model more room to generate comprehensive responses
- B) Using concise system prompts, removing redundant instructions, and requesting structured output formats like JSON
- C) Setting the `temperature` parameter to 0.0 to make responses deterministic
- D) Switching from synchronous to asynchronous API calls to batch requests together

---

**Question 6** *(Medium)*

Your team needs to receive notifications when Azure AI Foundry spending approaches a predefined threshold. How are budget alerts configured in Azure?

- A) By creating a budget in Microsoft Cost Management with alert conditions and assigning an action group to send email or webhook notifications
- B) By enabling diagnostic logging on each Azure AI Foundry resource and creating a Log Analytics query that triggers alerts
- C) By configuring spending limits directly on the Azure AI Foundry hub resource through its networking settings
- D) By submitting a support ticket to Microsoft requesting manual monitoring of your subscription costs

---

**Question 7** *(Hard)*

An application sends a request to an Azure OpenAI GPT-4o deployment. The request contains 800 input tokens and the response generates 400 output tokens. If the pay-as-you-go pricing is $2.50 per 1M input tokens and $10.00 per 1M output tokens, what is the total cost of this single request?

- A) $0.002
- B) $0.006
- C) $0.010
- D) $0.0125

---

**Question 8** *(Hard)*

A production application in Azure AI Foundry requires consistent, low-latency responses and processes a sustained volume of approximately 90,000 tokens per minute around the clock. Under which scenario is a PTU-based deployment the better choice over pay-as-you-go?

- A) When the workload is unpredictable with long idle periods and occasional traffic spikes lasting only a few minutes
- B) When the application is in early prototyping and the team is still experimenting with different models and prompt designs
- C) When the workload has sustained, predictable throughput demands where reserved capacity provides cost savings and guaranteed availability over pay-as-you-go pricing
- D) When the application only needs to process a small number of requests during business hours on weekdays

---

**Question 9** *(Medium)*

To reduce redundant API calls and lower costs in an Azure AI Foundry application, a team implements a caching strategy. What type of caching is most effective for reducing repeated identical requests to a deployed model?

- A) Browser-level caching using HTTP cache headers on the client side
- B) Semantic caching that stores and retrieves responses based on the similarity of input prompts
- C) DNS caching to speed up endpoint resolution for the Azure AI Foundry API
- D) CDN caching at the edge to distribute model responses geographically

---

**Question 10** *(Hard)*

A developer's application begins receiving HTTP 429 responses from an Azure AI Foundry model deployment. What does this indicate, and what is the recommended approach to handle it?

- A) The model deployment has been deleted; the developer should redeploy the model from the Azure AI Foundry model catalog
- B) The deployment's TPM quota has been exceeded; the developer should implement exponential backoff retry logic and consider requesting a quota increase
- C) The API key has expired; the developer should regenerate the key in the Azure AI Foundry project settings
- D) The Azure region is experiencing an outage; the developer should wait for Microsoft to resolve the service disruption

---

## Answer Key

| Question | Answer | Explanation |
|----------|--------|-------------|
| 1 | B | PTU stands for Provisioned Throughput Unit. It represents reserved, dedicated capacity for a model deployment in Azure AI Foundry, guaranteeing a specific level of throughput regardless of overall service demand. |
| 2 | C | Microsoft Cost Management is the primary Azure service for tracking, analyzing, and optimizing costs across all Azure resources, including Azure AI Foundry hubs, projects, and model deployments. |
| 3 | B | tiktoken is OpenAI's open-source library specifically designed to count tokens for OpenAI models (GPT-4o, GPT-4, GPT-3.5, etc.). It uses the same tokenization logic as the models themselves, ensuring accurate token counts before making API calls. |
| 4 | A | TPM stands for Tokens Per Minute. It is the primary rate-limiting quota for Azure OpenAI deployments in Azure AI Foundry, defining the maximum number of tokens (input + output combined) the deployment can process within a one-minute window. |
| 5 | B | Writing concise system prompts, eliminating redundant instructions, and requesting structured outputs (like JSON) directly reduces the number of tokens sent and received, lowering per-request costs. Temperature affects randomness, not token count, and `max_tokens` caps output length but doesn't reduce usage. |
| 6 | A | Azure budgets are created within Microsoft Cost Management. You define a spending threshold, configure alert conditions (e.g., 80% of budget reached), and attach an action group that can send email notifications, trigger webhooks, or invoke Azure Functions. |
| 7 | B | Input cost: (800 / 1,000,000) × $2.50 = $0.002. Output cost: (400 / 1,000,000) × $10.00 = $0.004. Total cost: $0.002 + $0.004 = $0.006. |
| 8 | C | PTU deployments are ideal for sustained, predictable workloads because the reserved capacity provides guaranteed throughput, lower latency under load, and cost savings compared to pay-as-you-go at high utilization levels. Sporadic, low-volume, or experimental workloads are better suited to pay-as-you-go. |
| 9 | B | Semantic caching stores previous prompt-response pairs and retrieves cached responses when a new prompt is sufficiently similar to a previously seen one. This avoids redundant API calls to the model, directly reducing token consumption and cost. Azure AI Foundry supports this through integrations with services like Azure Cache for Redis with vector search. |
| 10 | B | HTTP 429 (Too Many Requests) indicates the deployment's rate limit (TPM quota) has been exceeded. The recommended approach is to implement exponential backoff with retry logic to gracefully handle throttling, and if the workload genuinely requires higher throughput, request a quota increase through the Azure portal. |
