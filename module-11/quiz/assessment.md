# Module 11: Connecting Data Sources — Assessment

## Azure AI Foundry: Zero to Hero Training Course

**ARC 3: DATA & RAG** | Module 11 of 45 | April 2026

---

## Instructions

- 10 multiple-choice questions
- Select the single best answer for each question
- Passing score: 80% (8 out of 10 correct)
- Answers are provided at the end of this document

---

## Questions

### Question 1

Which of the following are supported data sources in Azure AI Foundry?

- A) Azure Blob Storage, AWS S3, Google Cloud Storage
- B) Azure Blob Storage, Azure SQL, SharePoint, OneLake, Cosmos DB
- C) Azure Blob Storage, Azure SQL, MongoDB Atlas, Snowflake
- D) Azure Data Lake only

---

### Question 2

When connecting Azure Blob Storage to Azure AI Foundry, which authentication method is recommended for production workloads?

- A) Shared Access Signature (SAS) tokens with full permissions
- B) Storage Account access keys stored in application code
- C) Managed Identity with Storage Blob Data Reader role
- D) Anonymous public access to the container

---

### Question 3

What is "chunking" in the context of data preparation for AI indexing?

- A) Compressing files to reduce storage costs
- B) Splitting large documents into smaller, overlapping segments for indexing
- C) Encrypting data at rest in Azure storage
- D) Converting all documents to a single file format

---

### Question 4

Which search type combines keyword-based matching with vector-based semantic retrieval?

- A) Full-text search
- B) Fuzzy search
- C) Hybrid search
- D) Exact match search

---

### Question 5

In Azure AI Foundry, what role does an embedding model play in the indexing pipeline?

- A) It translates documents from one language to another
- B) It converts text chunks into numerical vector representations for semantic search
- C) It compresses documents to save storage space
- D) It generates summaries of each document

---

### Question 6

Which data ingestion pattern is best suited for processing large volumes of historical documents that do not change frequently?

- A) Real-time streaming ingestion
- B) Event-driven ingestion with webhooks
- C) Batch processing with scheduled refresh
- D) Manual file upload each time

---

### Question 7

When configuring chunk overlap in an indexing pipeline, what is the primary benefit?

- A) It reduces the total number of chunks created
- B) It ensures context is preserved across chunk boundaries, preventing information loss
- C) It increases indexing speed significantly
- D) It eliminates the need for an embedding model

---

### Question 8

You need to connect enterprise documents stored in Microsoft 365 to Azure AI Foundry. Which data source should you use?

- A) Azure Blob Storage
- B) Azure Cosmos DB
- C) SharePoint Online
- D) Azure SQL Database

---

### Question 9

What is the recommended starting chunk size when creating a vector index for PDF documents in Azure AI Foundry?

- A) 64 tokens
- B) 256 tokens
- C) 1024 tokens
- D) 10,000 tokens

---

### Question 10

Which of the following is a key benefit of using Microsoft OneLake as a data source for Azure AI Foundry?

- A) OneLake is the cheapest storage option available
- B) OneLake provides a unified analytics data layer across Microsoft Fabric workloads
- C) OneLake automatically translates all documents to English
- D) OneLake replaces the need for Azure AI Search entirely

---

## Answer Key

| Question | Correct Answer | Explanation |
|----------|---------------|-------------|
| 1 | **B** | Azure AI Foundry supports Azure Blob Storage, Azure SQL, SharePoint, OneLake, and Cosmos DB as data sources. AWS and Google Cloud sources are not natively supported. |
| 2 | **C** | Managed Identity with the Storage Blob Data Reader RBAC role is the recommended secure authentication method for production. Access keys and SAS tokens are less secure, and anonymous access should never be used. |
| 3 | **B** | Chunking is the process of splitting large documents into smaller, manageable segments (often with overlap) so they can be effectively indexed and retrieved by AI search systems. |
| 4 | **C** | Hybrid search combines traditional keyword matching with vector-based semantic search to deliver more relevant results than either approach alone. |
| 5 | **B** | Embedding models convert text into dense numerical vectors that capture semantic meaning, enabling similarity-based search across indexed content. |
| 6 | **C** | Batch processing with scheduled refresh is ideal for large volumes of static or infrequently changing documents. Real-time streaming is better suited for continuously generated data. |
| 7 | **B** | Chunk overlap ensures that content near the boundary of one chunk is also captured in the adjacent chunk, preserving context that might otherwise be split across chunks. |
| 8 | **C** | SharePoint Online is the correct connector for Microsoft 365 enterprise documents. Blob Storage is for unstructured files uploaded directly, and SQL/Cosmos DB are for structured or NoSQL data. |
| 9 | **C** | A chunk size of 1024 tokens is a recommended starting point that balances context preservation with retrieval precision. Too small loses context; too large reduces search specificity. |
| 10 | **B** | OneLake is Microsoft Fabric's unified data lake that provides a single data layer across all Fabric workloads, making it ideal for organizations already using the Fabric ecosystem. |

---

## Scoring Guide

| Score | Result |
|-------|--------|
| 10/10 | Excellent — Full mastery of data source connectivity |
| 8–9/10 | Pass — Strong understanding with minor gaps |
| 6–7/10 | Review Recommended — Revisit key sections before proceeding |
| Below 6 | Retake — Complete the module again before moving to Module 12 |

---

*Azure AI Foundry: Zero to Hero Training Course — © 2026 Zero to Hero Training Team*
