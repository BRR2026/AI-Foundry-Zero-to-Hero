# Module 12 — Azure AI Search & Vector Indexes

## Training Guide

| Field             | Details                                      |
|-------------------|----------------------------------------------|
| **Course**        | Azure AI Foundry: Zero to Hero (Module 12/45)|
| **Arc**           | ARC 3 — DATA & RAG                          |
| **Title**         | Azure AI Search & Vector Indexes             |
| **Date**          | April 2026                                   |
| **Duration**      | 2.5 hours (lecture + lab)                    |
| **Prerequisite**  | Module 11 — Data Ingestion & Preparation     |

---

## Learning Objectives

By the end of this module, learners will be able to:

1. Explain what Azure AI Search is and how it powers AI-driven retrieval scenarios.
2. Create vector indexes using embedding models within Azure AI Foundry.
3. Design effective index schemas with fields, analyzers, and vector configurations.
4. Perform hybrid search combining keyword, semantic, and vector approaches.
5. Compare integrated vectorization against manual embedding pipelines.
6. Query vector indexes using REST APIs and the Azure SDK.
7. Apply performance and scaling best practices for production workloads.

---

## Module Outline

### Section 1 — Introduction (10 min)
- Module roadmap and position within ARC 3 (DATA & RAG).
- Why search is the backbone of Retrieval-Augmented Generation.

### Section 2 — What Is Azure AI Search? (15 min)
- Service overview: full-text, semantic, and vector search in one platform.
- Integration with Azure AI Foundry projects and AI models.
- Key concepts: indexes, indexers, skillsets, data sources.

### Section 3 — Understanding Embeddings & Vectors (15 min)
- What embeddings are and why they matter.
- Embedding models: Azure OpenAI `text-embedding-ada-002`, `text-embedding-3-large`.
- Vector similarity metrics: cosine, dot product, Euclidean.

### Section 4 — Creating Vector Indexes (20 min)
- Creating an index with vector fields in the Azure portal.
- Programmatic index creation via REST API and Python SDK.
- Configuring HNSW and exhaustive KNN algorithms.

### Section 5 — Index Schema Design Best Practices (15 min)
- Choosing field types: `Edm.String`, `Collection(Edm.Single)`, filterable, sortable.
- Designing for hybrid retrieval: combining text and vector fields.
- Managing dimensions and storage trade-offs.

### Section 6 — Hybrid Search (20 min)
- Keyword search with BM25 scoring.
- Semantic ranking and captions.
- Vector search with nearest-neighbor retrieval.
- Combining all three in a single query with Reciprocal Rank Fusion (RRF).

### Section 7 — Integrated Vectorization (15 min)
- What integrated vectorization is: automatic embedding at index and query time.
- Setting up vectorizers within the index definition.
- Comparing integrated vs. manual embedding pipelines.

### Section 8 — Querying Vector Indexes (15 min)
- REST API query syntax for vector and hybrid search.
- Python SDK (`azure-search-documents`) examples.
- Filtering, scoring profiles, and paging.

### Section 9 — Performance & Scaling (10 min)
- Partition and replica planning.
- Index size estimation and quota management.
- Monitoring with Azure Monitor metrics and logs.

### Section 10 — Mini Hack (30 min)
- Hands-on: create a vector index, ingest documents with embeddings, run hybrid queries.
- See `lab/hands-on-lab.md` for detailed instructions.

---

## Featured Video

**Build Vector Indexes Like a Pro in Azure AI Search**
- YouTube: [https://www.youtube.com/watch?v=8rX2z1HmKVk](https://www.youtube.com/watch?v=8rX2z1HmKVk)
- Show during Section 4 or as pre-class preparation.

---

## Trainer Notes

- **Common misconception**: Learners may confuse Azure AI Search with Azure Machine Learning — clarify that Azure AI Search is a dedicated search service integrated with Azure AI Foundry, not a model-training platform.
- **Demo tip**: Use the Azure portal "Import and vectorize data" wizard for a live demo before switching to code-based approaches.
- **Lab prep**: Ensure learners have an Azure AI Search resource (Free or Basic tier) and an Azure OpenAI deployment with an embedding model.

---

## Assessment

- See `quiz/assessment.md` for the 10-question multiple-choice quiz.
- Passing score: 80% (8 of 10 correct).

---

## Resources

| Resource | Link |
|----------|------|
| Azure AI Search Docs | https://learn.microsoft.com/azure/search/ |
| Vector Search Overview | https://learn.microsoft.com/azure/search/vector-search-overview |
| Integrated Vectorization | https://learn.microsoft.com/azure/search/vector-search-integrated-vectorization |
| Azure AI Foundry Portal | https://ai.azure.com |
| Python SDK Reference | https://learn.microsoft.com/python/api/azure-search-documents/ |

---

## Next Module

**Module 13** — Retrieval-Augmented Generation (RAG) Patterns → Building on the search foundation from this module.
