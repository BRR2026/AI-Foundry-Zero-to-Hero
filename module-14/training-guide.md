# Module 14: Advanced RAG Patterns — Training Guide

## Course Information

| Field | Details |
|-------|---------|
| **Course** | Azure AI Foundry: Zero to Hero |
| **Module** | 14 of 45 |
| **Arc** | ARC 3 — DATA & RAG |
| **Title** | Advanced RAG Patterns |
| **Date** | April 2026 |
| **Duration** | 3–4 hours (lecture + lab) |
| **Prerequisites** | Module 13 — Data Ingestion & Indexing |

---

## Learning Objectives

By the end of this module, learners will be able to:

1. Compare and implement multiple chunking strategies (fixed-size, semantic, sliding window, recursive) in Azure AI Foundry
2. Configure reranking models and tune relevance scoring for improved retrieval quality
3. Design multi-index RAG architectures that query across heterogeneous data sources
4. Handle complex documents containing tables, images, and structured data
5. Implement agentic RAG patterns where AI agents dynamically orchestrate retrieval

---

## Module Outline

### 1. Introduction — Beyond Basic RAG (15 min)
- Limitations of naive RAG: context loss, hallucination, poor relevance
- The RAG quality spectrum: basic → advanced → agentic
- How Azure AI Foundry supports advanced RAG natively

### 2. Chunking Strategies Deep Dive (30 min)
- **Fixed-size chunking**: token/character limits, overlap configuration
- **Semantic chunking**: sentence-boundary detection, topic segmentation
- **Sliding window**: overlapping windows for context preservation
- **Recursive chunking**: hierarchical splitting by document structure
- Choosing the right strategy for your content type

### 3. Embedding Model Selection (15 min)
- Azure OpenAI embedding models (text-embedding-3-large, text-embedding-3-small)
- Dimensionality trade-offs and performance implications
- Custom embedding fine-tuning in AI Foundry

### 4. Reranking & Relevance Tuning (30 min)
- Semantic ranker in Azure AI Search
- Cross-encoder reranking models
- Score fusion strategies (RRF — Reciprocal Rank Fusion)
- Tuning relevance thresholds and top-k parameters

### 5. Multi-Index RAG & Cross-Source Retrieval (25 min)
- Querying multiple Azure AI Search indexes simultaneously
- Cross-source retrieval: blending structured and unstructured data
- Federation patterns and query routing

### 6. Handling Tables & Structured Data (20 min)
- Table extraction with Azure Document Intelligence
- Converting tables to text-friendly formats
- Hybrid retrieval: combining SQL queries with vector search

### 7. Agentic RAG Patterns (30 min)
- Agent-driven query decomposition
- Iterative retrieval with self-reflection
- Tool-use patterns: agents deciding when and how to retrieve
- Multi-step reasoning over retrieved context

### 8. Performance Optimization (15 min)
- Caching strategies for repeated queries
- Batch processing and async retrieval
- Monitoring RAG quality metrics in AI Foundry

---

## Instructor Notes

### Key Emphasis Points
- **Chunking is foundational**: Poor chunking undermines everything downstream. Spend extra time on semantic vs. fixed-size comparisons.
- **Reranking is the highest-ROI improvement**: For most production RAG apps, adding a reranker delivers the biggest quality boost.
- **Agentic RAG is the future**: Position this as where the industry is heading — agents that decide their own retrieval strategy.

### Common Misconceptions
- "Bigger chunks are better" — Explain the precision vs. context trade-off
- "One index is enough" — Show when multi-index is essential
- "RAG replaces fine-tuning" — Clarify they are complementary

### Demo Recommendations
1. Side-by-side comparison of chunking strategies on the same document
2. Live reranking demo showing relevance score improvements
3. Agentic RAG with an AI Foundry agent using tool-based retrieval

---

## Lab Summary

**Mini Hack: Optimize Your RAG App with Semantic Chunking + Reranking**

Students will take a baseline RAG application and iteratively improve it by:
1. Switching from fixed-size to semantic chunking
2. Adding a reranking layer with Azure AI Search semantic ranker
3. Measuring quality improvements with relevance metrics

Estimated time: 60–90 minutes

---

## Assessment

- 10 multiple-choice questions covering chunking, reranking, multi-index RAG, and agentic patterns
- Passing score: 80% (8/10)
- See `quiz/assessment.md`

---

## Featured Video

**Building RAG Solutions with Azure AI Foundry** (Microsoft Reactor)
- Focus on advanced segments covering chunking, reranking, and multi-index patterns
- YouTube: https://www.youtube.com/watch?v=IXGrwWcLJuE

---

## Resources

- [Azure AI Foundry Documentation](https://learn.microsoft.com/azure/ai-studio/)
- [Azure AI Search — Semantic Ranking](https://learn.microsoft.com/azure/search/semantic-search-overview)
- [Chunking Best Practices](https://learn.microsoft.com/azure/ai-studio/concepts/retrieval-augmented-generation)
- [Azure Document Intelligence](https://learn.microsoft.com/azure/ai-services/document-intelligence/)

---

## Navigation

| Previous | Next |
|----------|------|
| [Module 13 — Data Ingestion & Indexing](../module-13/blog.html) | [Module 15 — Evaluation & Quality Metrics](../module-15/blog.html) |
