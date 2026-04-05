# Module 11: Connecting Data Sources — Training Guide

## Azure AI Foundry: Zero to Hero Training Course

**ARC 3: DATA & RAG** | Module 11 of 45 | April 2026

---

## Module Overview

This module teaches participants how to connect various Azure data sources to Azure AI Foundry for building AI-powered applications. Learners will explore supported data connectors, data ingestion patterns, preprocessing techniques, and indexing strategies essential for Retrieval-Augmented Generation (RAG) workflows.

## Learning Objectives

By the end of this module, participants will be able to:

1. Identify and configure supported data sources in Azure AI Foundry
2. Connect Azure Blob Storage, Azure SQL, Cosmos DB, SharePoint, and OneLake
3. Implement data ingestion patterns for AI applications
4. Apply data formatting, cleaning, and preprocessing techniques
5. Design and execute indexing strategies for efficient retrieval
6. Upload and index PDF documents in Azure AI Foundry

## Prerequisites

- Completion of Modules 1–10 (ARC 1 & ARC 2)
- Active Azure subscription with AI Foundry resource provisioned
- Basic understanding of Azure storage services
- Familiarity with data formats (JSON, CSV, PDF, DOCX)

## Module Duration

| Section | Duration |
|---------|----------|
| Lecture & Discussion | 45 minutes |
| Featured Video | 30 minutes |
| Hands-on Lab (Mini Hack) | 60 minutes |
| Quiz & Review | 15 minutes |
| **Total** | **~2.5 hours** |

## Agenda

### 1. Introduction (10 min)
- Why data connectivity matters for AI applications
- Overview of the data pipeline in Azure AI Foundry
- Connection between data sources and RAG architectures

### 2. Supported Data Sources (15 min)
- **Azure Blob Storage** — Unstructured data (PDFs, images, text files)
- **Azure SQL Database** — Structured relational data
- **Azure Cosmos DB** — NoSQL document and graph data
- **SharePoint Online** — Enterprise documents and collaboration content
- **Microsoft OneLake** — Unified analytics data lake (Fabric)

### 3. Data Ingestion Patterns (10 min)
- Push vs. pull ingestion models
- Batch processing vs. real-time streaming
- Scheduled refresh and incremental updates
- Event-driven ingestion with Azure Functions

### 4. Data Preparation & Cleaning (10 min)
- Text extraction from PDFs and Office documents
- Chunking strategies for large documents
- Metadata enrichment and tagging
- Handling multilingual content
- Data quality validation

### 5. Indexing Strategies (10 min)
- Vector index creation in Azure AI Foundry
- Hybrid search (keyword + vector)
- Index schema design best practices
- Incremental indexing and refresh policies

### 6. Featured Video (30 min)
- **"Building RAG Solutions with Azure AI Foundry"** — Microsoft Reactor
- YouTube: https://www.youtube.com/watch?v=IXGrwWcLJuE

### 7. Mini Hack: Upload and Index PDFs (60 min)
- Hands-on lab where participants upload a set of PDF documents
- Configure data connection in Azure AI Foundry
- Create an index and validate search results
- See `lab/hands-on-lab.md` for detailed instructions

### 8. Quiz & Review (15 min)
- 10 multiple-choice questions covering key concepts
- See `quiz/assessment.md`

## Key Concepts

| Concept | Description |
|---------|-------------|
| Data Connector | A configured link between Azure AI Foundry and an external data source |
| Chunking | Splitting large documents into smaller segments for indexing |
| Vector Index | A searchable index using embedding vectors for semantic similarity |
| Hybrid Search | Combining keyword-based and vector-based retrieval for better results |
| Ingestion Pipeline | The automated flow from raw data source to searchable index |

## Trainer Notes

- Ensure all participants have an Azure AI Foundry project created before the lab
- Have sample PDF documents prepared and accessible via a shared storage account
- Emphasize that Azure AI Foundry is NOT Azure Machine Learning — they are distinct platforms
- Encourage participants to explore the AI Foundry portal at https://ai.azure.com
- The Mini Hack can be extended if time permits by adding SharePoint as a second source

## Resources

- [Azure AI Foundry Documentation](https://learn.microsoft.com/azure/ai-studio/)
- [Azure AI Foundry Portal](https://ai.azure.com)
- [Azure AI Search Documentation](https://learn.microsoft.com/azure/search/)
- [Data Connection in Azure AI Foundry](https://learn.microsoft.com/azure/ai-studio/how-to/data-add)

## Navigation

| Previous | Next |
|----------|------|
| [Module 10 →](../module-10/training-guide.md) | [Module 12 →](../module-12/training-guide.md) |

---

*Azure AI Foundry: Zero to Hero Training Course — © 2026 Zero to Hero Training Team*
