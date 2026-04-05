# Module 42 Assessment: Build the Data + Model + RAG Pipeline

## Azure AI Foundry: Zero to Hero — Quiz (10 Questions)

**Arc 9: Capstone & Mastery** | **Passing Score:** 80% (8/10) | **Time Limit:** 15 minutes

---

### Question 1

**What is the recommended chunk overlap percentage for optimal retrieval quality in a RAG pipeline?**

- A) 0% — overlapping chunks wastes storage and increases index size
- B) 5% — minimal overlap is sufficient for most use cases
- C) 10–15% — this preserves context across chunk boundaries
- D) 50% — maximum overlap ensures no context is lost

<details>
<summary>✅ Show Answer</summary>

**C) 10–15% — this preserves context across chunk boundaries**

A 10–15% chunk overlap is the recommended best practice. This ensures that context is preserved at chunk boundaries, significantly improving retrieval accuracy when queries span multiple chunks. For example, with 800-token chunks, an overlap of 80–120 tokens is optimal.

</details>

---

### Question 2

**Which search approach provides the best retrieval recall for production RAG pipelines in Azure AI Foundry?**

- A) Pure keyword search using BM25 only
- B) Pure vector search using cosine similarity only
- C) Hybrid search combining vector similarity with BM25 keyword matching
- D) Full-text search with regex pattern matching

<details>
<summary>✅ Show Answer</summary>

**C) Hybrid search combining vector similarity with BM25 keyword matching**

Hybrid search combines the semantic understanding of vector search with the precision of keyword matching. Research shows hybrid search improves recall by 10–15% compared to vector-only search, especially for queries containing specific technical terms, identifiers, or proper nouns that benefit from exact matching.

</details>

---

### Question 3

**When building a RAG agent in Azure AI Foundry, which tool enables automatic retrieval from a vector store?**

- A) `code_interpreter`
- B) `function_call`
- C) `file_search`
- D) `web_search`

<details>
<summary>✅ Show Answer</summary>

**C) `file_search`**

The `file_search` tool in the Azure AI Foundry Agents API provides automatic retrieval from vector stores. When attached to an agent via `FileSearchTool().definitions` and configured with `FileSearchToolResource(vector_store_ids=[...])`, the agent automatically embeds user queries, searches the vector store, re-ranks results, and injects relevant context into the generation prompt.

</details>

---

### Question 4

**What temperature setting is recommended for factual accuracy in RAG pipeline generation?**

- A) 0.0 — always use zero temperature for deterministic output
- B) 0.1–0.3 — low temperature for factual, grounded responses
- C) 0.7–0.9 — moderate temperature for balanced creativity
- D) 1.5–2.0 — high temperature for maximum diversity

<details>
<summary>✅ Show Answer</summary>

**B) 0.1–0.3 — low temperature for factual, grounded responses**

For RAG pipelines where factual accuracy and groundedness are critical, a temperature of 0.1–0.3 is recommended. This produces responses that closely follow the retrieved context while still allowing natural language variation. Zero temperature (0.0) can sometimes produce overly rigid or repetitive text, while higher temperatures introduce unnecessary hallucination risk.

</details>

---

### Question 5

**Which evaluation metric measures whether a RAG pipeline's response is supported by the retrieved context?**

- A) Fluency
- B) Relevance
- C) Groundedness
- D) Coherence

<details>
<summary>✅ Show Answer</summary>

**C) Groundedness**

Groundedness measures the degree to which a response is factually supported by the retrieved context. This is the most critical metric for RAG pipelines because it directly measures whether the model is generating responses based on the provided documents (grounded) or fabricating information (hallucinating). The target threshold is typically ≥ 4.0 / 5.0.

</details>

---

### Question 6

**What is the correct format for preparing evaluation datasets for the Azure AI Evaluation SDK?**

- A) CSV with columns: question, answer, score
- B) JSONL with fields: query, context, response
- C) XML with elements: input, output, reference
- D) YAML with keys: prompt, completion, rating

<details>
<summary>✅ Show Answer</summary>

**B) JSONL with fields: query, context, response**

The Azure AI Evaluation SDK expects evaluation datasets in JSONL (JSON Lines) format. Each line is a JSON object containing at minimum `query`, `context`, and `response` fields. An optional `ground_truth` field can be included for reference-based evaluations. The evaluator config maps these fields using the `${data.field_name}` syntax.

</details>

---

### Question 7

**In Azure AI Foundry, which deployment type is recommended for production RAG pipelines that require guaranteed tokens-per-minute throughput?**

- A) Standard deployment with shared capacity
- B) Serverless deployment for on-demand scaling
- C) Provisioned throughput deployment with reserved capacity
- D) Global deployment for multi-region access

<details>
<summary>✅ Show Answer</summary>

**C) Provisioned throughput deployment with reserved capacity**

Provisioned throughput deployments reserve dedicated capacity for your model, providing guaranteed and consistent tokens-per-minute (TPM) throughput. This is essential for production RAG pipelines that need predictable performance. Standard deployments use shared capacity and may throttle under load, while serverless is for non-OpenAI models.

</details>

---

### Question 8

**When creating a vector store in Azure AI Foundry using the SDK, what does the `create_vector_store_and_poll` method do?**

- A) Creates the vector store and immediately returns, requiring manual polling
- B) Creates the vector store, processes all documents, and returns when indexing is complete
- C) Creates an empty vector store that must be populated separately
- D) Creates a vector store configuration template without processing data

<details>
<summary>✅ Show Answer</summary>

**B) Creates the vector store, processes all documents, and returns when indexing is complete**

The `create_vector_store_and_poll` method is a convenience method that creates a vector store, begins processing all specified data sources (document cracking, chunking, embedding generation, indexing), and polls until the operation reaches a terminal state (completed or failed). This simplifies the workflow by handling the asynchronous indexing process automatically.

</details>

---

### Question 9

**What is the primary advantage of using the Azure AI Foundry Agents API for RAG over building a custom pipeline?**

- A) It is the only way to use GPT-4o in Azure AI Foundry
- B) It automatically handles retrieval, re-ranking, context injection, and citation generation
- C) It does not require an Azure AI Search resource
- D) It supports unlimited context window sizes

<details>
<summary>✅ Show Answer</summary>

**B) It automatically handles retrieval, re-ranking, context injection, and citation generation**

The Agents API with the `file_search` tool automates the entire RAG workflow: it embeds the user query, searches the vector store, re-ranks results for relevance, injects the most relevant context into the generation prompt, and includes source citations in the response. This significantly reduces the code and configuration needed compared to a custom pipeline, while still allowing customization through agent instructions.

</details>

---

### Question 10

**Why is it critical to use the same embedding model for both document indexing and query embedding in a RAG pipeline?**

- A) Different models have different licensing requirements
- B) Azure AI Search only supports one embedding model per index
- C) Mismatched models produce vectors in different semantic spaces, degrading retrieval quality
- D) Using different models doubles the Azure compute costs

<details>
<summary>✅ Show Answer</summary>

**C) Mismatched models produce vectors in different semantic spaces, degrading retrieval quality**

Each embedding model maps text into its own learned vector space. When you index documents with one model and embed queries with a different model, the resulting vectors exist in different semantic spaces and cannot be meaningfully compared using similarity metrics. This leads to poor retrieval results, even if both models individually produce high-quality embeddings. Always use the same model (e.g., `text-embedding-3-large`) for both indexing and querying.

</details>

---

## 📊 Scoring Guide

| Score | Grade | Status |
|---|---|---|
| 10/10 | ⭐ Excellent | Mastery demonstrated |
| 8–9/10 | ✅ Pass | Strong understanding |
| 6–7/10 | ⚠️ Review | Revisit weak areas |
| ≤ 5/10 | ❌ Retry | Re-study module content |

---

## 🔑 Answer Key

| # | Answer | Topic |
|---|---|---|
| 1 | C | Chunking — Overlap |
| 2 | C | Search — Hybrid |
| 3 | C | Agents — file_search |
| 4 | B | Models — Temperature |
| 5 | C | Evaluation — Groundedness |
| 6 | B | Evaluation — JSONL Format |
| 7 | C | Deployment — Provisioned |
| 8 | B | SDK — Vector Store Creation |
| 9 | B | Agents API — Advantages |
| 10 | C | Embeddings — Consistency |

---

*Module 42 Assessment — Azure AI Foundry: Zero to Hero | Arc 9: Capstone & Mastery | April 2026*
