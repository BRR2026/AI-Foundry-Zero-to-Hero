# Module 21 Assessment: Prompt Flow — Visual Orchestration

| | |
|---|---|
| **Total Questions** | 10 |
| **Question Types** | Multiple Choice |
| **Estimated Time** | 15–20 minutes |
| **Passing Score** | 80% (8 out of 10) |
| **Difficulty** | ⭐⭐ Intermediate |

> **Instructions:** Select the single best answer for each question. All questions relate to Prompt Flow in Azure AI Foundry as covered in Module 21. Read each question carefully — some options are intentionally similar.

---

### Question 1

**What is Prompt Flow in Azure AI Foundry?** *(Easy)*

- A) A machine learning model training pipeline for fine-tuning LLMs
- B) A visual orchestration tool for building LLM-based applications using directed acyclic graphs (DAGs)
- C) A data processing framework for ETL operations on Azure Blob Storage
- D) A monitoring dashboard for tracking Azure OpenAI token usage

<details>
<summary>Answer</summary>

**B) A visual orchestration tool for building LLM-based applications using directed acyclic graphs (DAGs)**

Prompt Flow is a development and runtime tool that enables you to design, test, debug, and deploy LLM-based application workflows using a visual DAG-based approach. It is not an ML training pipeline (A), a data processing framework (C), or a monitoring dashboard (D).

</details>

---

### Question 2

**Which three flow types are available in Prompt Flow?** *(Easy)*

- A) Standard Flow, Chat Flow, Evaluation Flow
- B) Standard Flow, Batch Flow, Pipeline Flow
- C) Chat Flow, Training Flow, Deployment Flow
- D) Input Flow, Processing Flow, Output Flow

<details>
<summary>Answer</summary>

**A) Standard Flow, Chat Flow, Evaluation Flow**

Prompt Flow supports three flow types: **Standard Flow** for general LLM applications, **Chat Flow** for conversation-oriented apps with built-in chat history, and **Evaluation Flow** for assessing the quality of other flows' outputs.

</details>

---

### Question 3

**What decorator must Python nodes use in Prompt Flow?** *(Easy)*

- A) `@function`
- B) `@node`
- C) `@tool`
- D) `@prompt`

<details>
<summary>Answer</summary>

**C) `@tool`**

Python nodes in Prompt Flow must use the `@tool` decorator from the `promptflow` package. This decorator registers the function as a Prompt Flow tool that can be used as a node in the flow graph. The function must also include type hints for all parameters.

</details>

---

### Question 4

**How do nodes pass data to each other in a Prompt Flow?** *(Medium)*

- A) Through shared global variables accessible to all nodes
- B) By writing data to Azure Blob Storage between nodes
- C) Using typed references with the `${node_name.output}` syntax
- D) Through environment variables set by each node at runtime

<details>
<summary>Answer</summary>

**C) Using typed references with the `${node_name.output}` syntax**

Prompt Flow nodes connect through typed references. When configuring a node's inputs, you use the `${node_name.output}` syntax to reference the output of another node. This creates a directed edge in the DAG and ensures data flows correctly between nodes.

</details>

---

### Question 5

**Which template language do LLM nodes use for prompt construction?** *(Easy)*

- A) Handlebars
- B) Mustache
- C) Jinja2
- D) EJS (Embedded JavaScript)

<details>
<summary>Answer</summary>

**C) Jinja2**

LLM nodes in Prompt Flow use **Jinja2** templates for prompt construction. Variables are substituted using the `{{variable_name}}` syntax, and control flow (loops, conditionals) is available with `{% %}` blocks. This allows dynamic prompt assembly based on flow inputs and upstream node outputs.

</details>

---

### Question 6

**What is the primary advantage of using variants in Prompt Flow?** *(Medium)*

- A) Variants allow you to run the same flow in multiple Azure regions simultaneously
- B) Variants enable side-by-side comparison of different prompt templates, models, or parameters
- C) Variants create backup copies of your flow for disaster recovery
- D) Variants automatically scale the number of compute instances based on load

<details>
<summary>Answer</summary>

**B) Variants enable side-by-side comparison of different prompt templates, models, or parameters**

Variants in Prompt Flow allow you to create multiple versions of a node (typically LLM nodes) with different prompts, model deployments, or parameter settings. You can then run batch evaluations to quantitatively compare variant performance, which is essential for data-driven prompt optimization.

</details>

---

### Question 7

**When debugging a Prompt Flow, a node shows a red outline. What does this typically indicate?** *(Medium)*

- A) The node's output exceeds the maximum size limit
- B) The node has a missing or misconfigured connection
- C) The node is currently executing and processing data
- D) The node has been deprecated and needs to be replaced

<details>
<summary>Answer</summary>

**B) The node has a missing or misconfigured connection**

A red outline on a node typically indicates a configuration error, most commonly a missing connection (e.g., the Azure OpenAI connection isn't set up). To fix this, configure the required connection in the project settings and select it in the node's configuration panel.

</details>

---

### Question 8

**In the customer support Mini Hack, why is GPT-4o-mini used for intent classification instead of GPT-4o?** *(Medium)*

- A) GPT-4o-mini supports more languages than GPT-4o
- B) GPT-4o-mini is faster and more cost-effective for simple classification tasks
- C) GPT-4o cannot perform classification tasks
- D) GPT-4o-mini has a larger context window than GPT-4o

<details>
<summary>Answer</summary>

**B) GPT-4o-mini is faster and more cost-effective for simple classification tasks**

Intent classification is a straightforward task that doesn't require the full capabilities of GPT-4o. Using GPT-4o-mini for classification reduces both latency and cost, while reserving the more powerful GPT-4o model for the response generation step where quality matters most. This is a common pattern in production flows — use smaller models for simple tasks and larger models for complex generation.

</details>

---

### Question 9

**What authentication method should be used for production Prompt Flow endpoint deployments?** *(Medium)*

- A) API key authentication with keys rotated weekly
- B) Basic HTTP authentication with username and password
- C) Azure Active Directory (Azure AD) token-based authentication
- D) OAuth 2.0 with third-party identity providers only

<details>
<summary>Answer</summary>

**C) Azure Active Directory (Azure AD) token-based authentication**

For production deployments, Azure AD token-based authentication is recommended. It provides stronger security through managed identity integration, role-based access control (RBAC), token expiration, and audit logging. API keys (A) are acceptable for development and testing but lack the security controls needed for production environments.

</details>

---

### Question 10

**Which of the following is an anti-pattern when building Prompt Flow applications?** *(Hard)*

- A) Creating variants of LLM nodes to test different prompt strategies
- B) Using Python validation nodes after LLM calls to check output quality
- C) Embedding API keys directly in Python node source code instead of using connections
- D) Splitting large flows into smaller, focused sub-flows

<details>
<summary>Answer</summary>

**C) Embedding API keys directly in Python node source code instead of using connections**

Hardcoding API keys in Python nodes is a serious anti-pattern that creates security vulnerabilities and makes key rotation difficult. Prompt Flow provides a **connections** system specifically for managing secrets securely. Connections store API keys and endpoints in Azure Key Vault-backed storage and are referenced by nodes through configuration — never through code. Options A, B, and D are all recommended best practices.

</details>

---

## Scoring Guide

| Score | Rating | Recommendation |
|---|---|---|
| 10/10 | ⭐⭐⭐ Excellent | Ready for Module 22 |
| 8–9/10 | ⭐⭐ Good | Review missed topics, proceed to Module 22 |
| 6–7/10 | ⭐ Needs Review | Re-read sections 4–7 of the training guide, redo the lab |
| Below 6 | Retry | Complete the full module again before proceeding |

---

## Answer Key (Quick Reference)

| Question | Answer | Difficulty |
|---|---|---|
| 1 | B | Easy |
| 2 | A | Easy |
| 3 | C | Easy |
| 4 | C | Medium |
| 5 | C | Easy |
| 6 | B | Medium |
| 7 | B | Medium |
| 8 | B | Medium |
| 9 | C | Medium |
| 10 | C | Hard |

---

*Module 21 Assessment — Zero to Hero: Azure AI Foundry Training · ARC 5: Orchestration & Integration · April 2026*
