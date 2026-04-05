# Module 42: Build the Data + Model + RAG Pipeline

## Azure AI Foundry: Zero to Hero (Module 42 of 45)

**Arc 9: Capstone & Mastery** | **Duration:** 4–5 hours | **Level:** Advanced | **Date:** April 2026

---

## 🎯 Training Overview

This module is the cornerstone of the Capstone & Mastery arc. Learners will build a complete, production-ready Retrieval-Augmented Generation (RAG) pipeline using Azure AI Foundry — from raw data ingestion through vector indexing, model deployment, pipeline orchestration, and quality evaluation.

> **Important:** This module uses **Azure AI Foundry** (not Azure ML). All references, APIs, and portal instructions target the Azure AI Foundry platform at [ai.azure.com](https://ai.azure.com).

---

## 📋 Prerequisites

| Requirement | Details |
|---|---|
| Prior Modules | Modules 1–41 completed |
| Azure Subscription | Active subscription with Contributor access |
| Azure AI Foundry | Hub + Project provisioned |
| Azure AI Search | Standard tier (S1+) for vector search |
| Azure OpenAI Service | GPT-4o and text-embedding-3-large deployed |
| Python Environment | Python 3.10+ with `azure-ai-projects`, `azure-ai-evaluation`, `azure-search-documents` |
| IDE | VS Code with Python and Azure extensions |

### Required Python Packages

```bash
pip install azure-ai-projects azure-ai-evaluation azure-search-documents azure-identity
```

---

## 🗂️ Module Structure

### Session 1: Data Sources & Vector Indexes (75 minutes)

#### Learning Objectives
- Connect heterogeneous data sources to Azure AI Foundry
- Configure document processing and chunking strategies
- Create and optimize vector indexes using Azure AI Search

#### Topics Covered

1. **Understanding Data Sources in Azure AI Foundry**
   - Azure Blob Storage (unstructured: PDFs, DOCX, TXT, images)
   - Azure SQL Database / Cosmos DB (structured and semi-structured)
   - SharePoint / OneDrive (enterprise documents)
   - Web URLs and REST APIs (crawled content)

2. **Document Processing Pipeline**
   - Document cracking (PDF extraction, OCR for images)
   - Content cleaning and normalization
   - Metadata extraction and enrichment
   - Language detection and handling

3. **Chunking Strategies**
   - Fixed-size chunking (512–1024 tokens)
   - Sentence-based chunking (3–5 sentences)
   - Semantic chunking (model-driven, variable size)
   - Page/section-based chunking (for structured documents)
   - **Best practice:** Always use 10–15% chunk overlap

4. **Creating Vector Indexes**
   - Provisioning Azure AI Search from Azure AI Foundry
   - Configuring vector fields and dimensions
   - Setting up hybrid search (vector + keyword)
   - Index schema design for RAG

#### Hands-On Activity (30 minutes)
- Upload sample documents to Azure Blob Storage
- Create a data asset in Azure AI Foundry
- Build a vector store with static chunking (800 tokens, 100 overlap)
- Verify index creation in Azure AI Search

---

### Session 2: Model Deployment & Configuration (60 minutes)

#### Learning Objectives
- Deploy foundation models from the Azure AI Foundry model catalog
- Configure embedding and generation models for RAG
- Understand deployment types and capacity planning

#### Topics Covered

1. **Model Selection for RAG Pipelines**
   - Generation models: GPT-4o, GPT-4o mini, Phi-4, Mistral Large
   - Embedding models: text-embedding-3-large, text-embedding-3-small
   - Trade-offs: latency vs. quality vs. cost vs. context window

2. **Deployment Types**
   - Standard deployment (shared capacity, pay-per-token)
   - Provisioned throughput (reserved capacity, consistent TPM)
   - Global deployment (multi-region, highest availability)
   - Serverless deployment (for non-OpenAI models)

3. **Configuration Best Practices**
   - Temperature settings for RAG (0.1–0.3 for factual accuracy)
   - Max token allocation (reserve tokens for context)
   - Rate limiting and throttling strategies
   - Fallback model configuration

4. **Embedding Model Setup**
   - Choosing embedding dimensions (1536 vs. 3072)
   - Batch embedding for large document sets
   - Query vs. document embedding consistency

#### Hands-On Activity (20 minutes)
- Deploy GPT-4o and text-embedding-3-large via the SDK
- Test model endpoints with sample prompts
- Generate embeddings and verify dimensions

---

### Session 3: Building the RAG Pipeline (75 minutes)

#### Learning Objectives
- Build an end-to-end RAG pipeline using the Agents API
- Implement a custom RAG pipeline with direct SDK calls
- Compare agent-based vs. custom pipeline approaches

#### Topics Covered

1. **Agent-Based RAG (Recommended)**
   - Using the Azure AI Foundry Agents API
   - The `file_search` tool for automatic retrieval
   - Vector store integration with agents
   - Thread management and conversation context
   - Automatic citation handling

2. **Custom RAG Pipeline**
   - Manual query embedding
   - Hybrid search execution (vector + keyword)
   - Context assembly and prompt construction
   - Response generation with grounding instructions
   - Source citation formatting

3. **Pipeline Architecture Patterns**
   - Simple RAG (single retrieval, single generation)
   - Multi-step RAG (query rewriting, iterative retrieval)
   - Agentic RAG (tool-augmented retrieval)
   - Federated RAG (multiple indexes, result fusion)

4. **Prompt Engineering for RAG**
   - System prompt design for grounded responses
   - Context window management
   - Citation instruction patterns
   - Handling "I don't know" responses

#### Hands-On Activity (30 minutes)
- Build a RAG agent with `file_search` tool
- Implement a custom RAG pipeline with hybrid search
- Compare responses from both approaches

---

### Session 4: Testing & Validation (60 minutes)

#### Learning Objectives
- Create evaluation datasets for RAG pipelines
- Run built-in evaluators for quality measurement
- Interpret evaluation results and identify improvements

#### Topics Covered

1. **Evaluation Framework**
   - Groundedness: Is the response grounded in context?
   - Relevance: Does the response answer the question?
   - Coherence: Is the response well-structured?
   - Fluency: Is the response natural and readable?
   - Retrieval Score: Are retrieved documents relevant?

2. **Creating Evaluation Datasets**
   - JSONL format: query, context, response, ground_truth
   - Minimum 20–50 test cases for reliable metrics
   - Covering edge cases: ambiguous queries, multi-topic queries
   - Including negative cases (questions not in knowledge base)

3. **Running Evaluations**
   - Using the `azure-ai-evaluation` SDK
   - Batch evaluation with multiple evaluators
   - Interpreting aggregate vs. per-sample scores
   - Viewing results in Azure AI Foundry portal

4. **Continuous Evaluation**
   - Integrating evaluations into CI/CD
   - Setting quality gates (minimum scores)
   - Tracking metric trends over time
   - Alert configuration for quality degradation

#### Hands-On Activity (20 minutes)
- Create a 10-sample evaluation dataset
- Run groundedness, relevance, and coherence evaluators
- Analyze results and identify weak spots

---

### Session 5: Mini Hack — Complete Your Pipeline (45 minutes)

#### Challenge
Build a complete, working RAG pipeline from scratch that meets all quality criteria.

#### Requirements Checklist
- [ ] Azure AI Foundry project with connected resources
- [ ] 5+ documents uploaded from different formats
- [ ] Vector store with semantic chunking and overlap
- [ ] Embedding model (text-embedding-3-large) deployed
- [ ] Generation model (GPT-4o) deployed
- [ ] RAG agent with file_search tool
- [ ] Custom RAG pipeline with hybrid search
- [ ] 10+ test cases in JSONL format
- [ ] All evaluator scores above 4.0/5.0
- [ ] Documented architecture and results

#### Bonus Challenge
Set up continuous evaluation with Azure Monitor alerts.

---

## 🎬 Featured Video

**Building RAG Solutions with Azure AI Foundry** — Microsoft Reactor

- **URL:** [https://www.youtube.com/watch?v=IXGrwWcLJuE](https://www.youtube.com/watch?v=IXGrwWcLJuE)
- **Embed:** `https://www.youtube.com/embed/IXGrwWcLJuE`
- **Source:** Microsoft Reactor

Watch this session for a complete walkthrough of building RAG solutions in Azure AI Foundry, including data ingestion, index creation, and pipeline orchestration.

---

## 📊 Assessment Criteria

| Criterion | Weight | Passing Score |
|---|---|---|
| Data source setup & vector index | 20% | Complete |
| Model deployment & configuration | 20% | Complete |
| RAG pipeline (agent + custom) | 30% | Both working |
| Evaluation results | 20% | All metrics ≥ 4.0 |
| Documentation quality | 10% | Clear and complete |

---

## 🔑 Key Takeaways

1. **Data quality is the foundation** — invest in chunking strategy and overlap before building the pipeline
2. **Agents API simplifies RAG** — `file_search` handles retrieval, re-ranking, and citation automatically
3. **Hybrid search wins** — vector + keyword search improves recall by 10–15%
4. **Evaluation is mandatory** — use groundedness, relevance, and coherence as quality gates
5. **Model selection matters** — match model to latency, cost, and quality requirements

---

## 📚 Additional Resources

- [Azure AI Foundry Documentation](https://learn.microsoft.com/azure/ai-studio/)
- [RAG with Azure AI Search](https://learn.microsoft.com/azure/search/retrieval-augmented-generation-overview)
- [Azure AI Foundry Agents](https://learn.microsoft.com/azure/ai-services/agents/)
- [Azure AI Evaluation SDK](https://learn.microsoft.com/azure/ai-studio/how-to/evaluate-generative-ai-app)
- [Vector Search in Azure AI Search](https://learn.microsoft.com/azure/search/vector-search-overview)

---

## ⏭️ What's Next

- **Module 43:** Continue the Capstone & Mastery arc with advanced pipeline optimization and production deployment patterns.

---

*Module 42 of 45 — Azure AI Foundry: Zero to Hero Training Series*
*Arc 9: Capstone & Mastery — April 2026*
