# Module 14: Advanced RAG Patterns — Assessment

**Course:** Azure AI Foundry: Zero to Hero
**Module:** 14 of 45 | ARC 3 — DATA & RAG
**Passing Score:** 80% (8 out of 10)
**Time Limit:** 15 minutes

---

## Question 1

**Which chunking strategy splits text based on topic boundaries and sentence structure rather than fixed token counts?**

- A) Fixed-size chunking
- B) Sliding window chunking
- C) Semantic chunking ✅
- D) Character-level chunking

> **Explanation:** Semantic chunking uses natural language processing to detect topic shifts and sentence boundaries, creating chunks that preserve meaningful context units rather than splitting arbitrarily at fixed token limits.

---

## Question 2

**What is the primary benefit of using overlapping sliding window chunks?**

- A) Reduces the total number of chunks stored in the index
- B) Ensures context is preserved across chunk boundaries ✅
- C) Eliminates the need for reranking
- D) Decreases embedding computation costs

> **Explanation:** Sliding window chunking creates overlapping segments so that context at chunk boundaries is captured in multiple chunks, preventing information loss that occurs at hard split points.

---

## Question 3

**In Azure AI Search, what does Reciprocal Rank Fusion (RRF) accomplish?**

- A) Replaces vector search with keyword search
- B) Combines scores from multiple retrieval methods into a unified ranking ✅
- C) Reduces the number of indexes needed
- D) Automatically selects the best embedding model

> **Explanation:** RRF is a score fusion technique that merges results from different retrieval methods (e.g., vector search and keyword search) into a single ranked list, leveraging the strengths of each approach.

---

## Question 4

**When implementing reranking in a RAG pipeline, where does the reranker operate?**

- A) Before the initial retrieval step
- B) After initial retrieval, re-scoring the candidate documents ✅
- C) During the embedding generation phase
- D) At the final LLM response generation step

> **Explanation:** A reranker is a second-stage model that takes the initial set of retrieved candidates and re-scores them using a more sophisticated (typically cross-encoder) model to improve relevance ranking before passing context to the LLM.

---

## Question 5

**What is a key advantage of multi-index RAG over single-index approaches?**

- A) It requires fewer Azure resources
- B) It eliminates the need for chunking
- C) It enables retrieval across heterogeneous data sources with different schemas ✅
- D) It automatically handles authentication

> **Explanation:** Multi-index RAG allows querying across indexes with different schemas, data types, and sources (e.g., product docs in one index, support tickets in another), enabling comprehensive cross-source retrieval.

---

## Question 6

**Which Azure service is best suited for extracting tables from PDF documents before indexing them for RAG?**

- A) Azure Blob Storage
- B) Azure Document Intelligence ✅
- C) Azure Functions
- D) Azure Cosmos DB

> **Explanation:** Azure Document Intelligence (formerly Form Recognizer) provides specialized models for extracting structured data like tables, key-value pairs, and layout information from PDF and image-based documents.

---

## Question 7

**In agentic RAG, what distinguishes it from traditional RAG pipelines?**

- A) It uses larger embedding models
- B) It eliminates the need for a vector store
- C) An AI agent dynamically decides when, what, and how to retrieve information ✅
- D) It only works with structured data

> **Explanation:** Agentic RAG empowers an AI agent to autonomously decide its retrieval strategy — including query decomposition, iterative retrieval, and tool selection — rather than following a fixed retrieve-then-generate pipeline.

---

## Question 8

**When using recursive chunking, how is document text typically split?**

- A) Randomly into equal-sized pieces
- B) Hierarchically, first by major sections, then sub-sections, then paragraphs ✅
- C) By individual words
- D) Only at page boundaries

> **Explanation:** Recursive chunking uses a hierarchy of separators (sections → sub-sections → paragraphs → sentences) to progressively split text while respecting document structure, ensuring chunks align with logical content boundaries.

---

## Question 9

**What is the primary trade-off when increasing the top-k parameter (number of retrieved chunks) in a RAG query?**

- A) Higher cost but no quality change
- B) More context for the LLM but increased risk of noise and higher latency ✅
- C) Faster retrieval but lower relevance
- D) Better chunking but worse embedding quality

> **Explanation:** Increasing top-k provides the LLM with more retrieved context, which can improve recall. However, it also introduces more potentially irrelevant content (noise) and increases token costs and latency.

---

## Question 10

**In a hybrid search configuration within Azure AI Search, what two retrieval methods are typically combined?**

- A) SQL queries and GraphQL queries
- B) Keyword (full-text) search and vector (semantic) search ✅
- C) Real-time search and batch search
- D) Local search and cloud search

> **Explanation:** Hybrid search in Azure AI Search combines traditional keyword-based full-text search (BM25) with vector-based semantic search, using score fusion (RRF) to deliver results that leverage both lexical matching and semantic understanding.

---

## Answer Key

| Question | Answer | Topic |
|----------|--------|-------|
| 1 | C | Chunking Strategies |
| 2 | B | Sliding Window Chunking |
| 3 | B | Score Fusion / RRF |
| 4 | B | Reranking Pipeline |
| 5 | C | Multi-Index RAG |
| 6 | B | Document Intelligence |
| 7 | C | Agentic RAG |
| 8 | B | Recursive Chunking |
| 9 | B | Retrieval Parameters |
| 10 | B | Hybrid Search |

---

## Navigation

| Previous | Next |
|----------|------|
| [Module 13 — Data Ingestion & Indexing](../module-13/blog.html) | [Module 15 — Evaluation & Quality Metrics](../module-15/blog.html) |
