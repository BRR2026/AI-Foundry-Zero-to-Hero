# Module 13: Building Your First RAG Application

## Training Guide

**Course:** Azure AI Foundry — Zero to Hero (Module 13 of 45)
**Arc:** ARC 3 · DATA & RAG
**Date:** April 2026
**Duration:** 3 hours (1.5 hours instruction + 1.5 hours hands-on lab)

---

## Module Overview

This module introduces Retrieval-Augmented Generation (RAG), the most impactful pattern for grounding AI responses in enterprise data. Participants will learn the full RAG architecture — retrieve, augment, generate — and build a working RAG chatbot using Azure AI Foundry and Azure AI Search.

## Prerequisites

- Completion of Modules 1–12 (especially Module 11: Data Connections & Indexing)
- An active Azure subscription with AI Foundry hub and project provisioned
- Azure AI Search resource (Basic tier or above)
- A deployed chat model (GPT-4o or equivalent) in AI Foundry
- Python 3.10+ with `azure-ai-projects` and `azure-search-documents` SDKs installed
- Sample company documents (PDF/Markdown/Word) for indexing

## Learning Objectives

By the end of this module, participants will be able to:

1. **Explain** the RAG architecture and why it solves hallucination problems
2. **Configure** an Azure AI Search index with vectorized document content
3. **Connect** Azure AI Search to a model deployment in AI Foundry
4. **Use** the "On Your Data" feature in the AI Foundry portal
5. **Build** a RAG pipeline using the Python SDK with proper citation handling
6. **Design** system prompts optimized for grounded, citation-rich responses
7. **Handle** edge cases including no-result queries and conflicting sources

## Agenda

| Time | Topic | Type |
|------|-------|------|
| 0:00–0:10 | Welcome & Module Overview | Lecture |
| 0:10–0:25 | Why RAG? The Hallucination Problem | Discussion |
| 0:25–0:45 | RAG Architecture Deep Dive | Lecture |
| 0:45–1:00 | Featured Video + Q&A | Video |
| 1:00–1:15 | Setting Up Azure AI Search Index | Demo |
| 1:15–1:30 | On Your Data Feature in AI Foundry | Demo |
| 1:30–1:45 | Break | — |
| 1:45–2:15 | Mini Hack: Build a RAG Chatbot (Part 1) | Lab |
| 2:15–2:30 | Citation Handling & Source Attribution | Lecture |
| 2:30–2:50 | Mini Hack: Build a RAG Chatbot (Part 2) | Lab |
| 2:50–3:00 | Key Takeaways & Module 14 Preview | Wrap-up |

## Key Concepts

### RAG Architecture (Retrieve → Augment → Generate)

- **Retrieve:** Query Azure AI Search to find relevant document chunks using vector/hybrid search
- **Augment:** Inject retrieved context into the model prompt alongside the user question
- **Generate:** The model produces a response grounded in the retrieved documents

### On Your Data (AI Foundry)

- No-code RAG setup directly in the AI Foundry portal
- Connects deployed models to Azure AI Search indexes
- Supports role-based access, semantic ranking, and vector search
- Ideal for prototyping and non-developer stakeholders

### Citation Handling

- Parse citation metadata from model responses
- Map citations back to source documents and page numbers
- Display inline references with clickable source links

## Instructor Notes

1. **Common Misconception:** Students may confuse RAG with fine-tuning. Emphasize that RAG retrieves external knowledge at inference time, while fine-tuning bakes knowledge into model weights.
2. **Demo Tip:** Pre-index a set of sample documents before class to avoid waiting for indexing during live demos.
3. **Troubleshooting:** If "On Your Data" returns empty results, check that the search index has at least one populated field mapped as "content."
4. **SDK Version:** Ensure students install `azure-ai-projects>=1.0.0b5` for the latest RAG pipeline support.

## Assessment

- **Quiz:** 10 multiple-choice questions in `quiz/assessment.md`
- **Lab:** Hands-on lab in `lab/hands-on-lab.md` — Build a RAG chatbot over company docs
- **Completion Criteria:** Score ≥ 70% on quiz AND successfully complete the Mini Hack

## Resources

- [Featured Video: Building RAG Solutions with Azure AI Foundry](https://www.youtube.com/watch?v=IXGrwWcLJuE)
- [Azure AI Foundry Documentation](https://learn.microsoft.com/azure/ai-studio/)
- [Azure AI Search Documentation](https://learn.microsoft.com/azure/search/)
- [RAG Pattern — Microsoft Learn](https://learn.microsoft.com/azure/ai-studio/concepts/retrieval-augmented-generation)
- [Azure AI Foundry Portal](https://ai.azure.com)

## Next Module

**Module 14** continues the RAG journey with advanced retrieval strategies, including hybrid search tuning, semantic rankers, and multi-index federation.

---

*Azure AI Foundry: Zero to Hero Training — © 2026 Zero to Hero Training Team*
