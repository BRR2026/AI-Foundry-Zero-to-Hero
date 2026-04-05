# Module 11 — Hands-on Lab: Upload and Index PDF Documents

## Azure AI Foundry: Zero to Hero Training Course

**ARC 3: DATA & RAG** | Mini Hack | April 2026

---

## Lab Overview

In this hands-on lab, you will upload a collection of PDF documents to Azure Blob Storage, connect that storage to Azure AI Foundry, and create a searchable vector index. By the end of this lab, you will have a fully functional data pipeline that can serve as the knowledge base for a RAG-powered AI application.

**Estimated Duration:** 60 minutes

## Learning Objectives

- Upload documents to Azure Blob Storage
- Create a data connection in Azure AI Foundry
- Configure and run a data indexing pipeline
- Validate search results against indexed content
- Apply basic data cleaning and chunking strategies

## Prerequisites

- Active Azure subscription
- Azure AI Foundry project (created in earlier modules)
- Azure Blob Storage account
- 5–10 sample PDF documents (provided or your own)
- Web browser with access to https://ai.azure.com

---

## Part 1: Prepare Your Data (15 minutes)

### Step 1.1 — Gather Sample Documents

For this lab, prepare 5–10 PDF documents on a related topic (e.g., Azure documentation pages, product manuals, or research papers). Each document should be:
- Between 2–50 pages
- Text-based (not scanned images without OCR)
- In English (or your preferred language)

> **💡 Tip:** You can download Azure documentation as PDFs from Microsoft Learn, or use any set of business documents you have available.

### Step 1.2 — Create a Blob Storage Container

1. Navigate to the [Azure Portal](https://portal.azure.com)
2. Open your existing Storage Account (or create a new one)
3. Go to **Containers** under **Data storage**
4. Click **+ Container**
5. Set the container name: `module11-pdf-docs`
6. Set access level: **Private (no anonymous access)**
7. Click **Create**

### Step 1.3 — Upload Documents

1. Open the `module11-pdf-docs` container
2. Click **Upload**
3. Select your 5–10 PDF files
4. Click **Upload** and wait for completion
5. Verify all files appear in the container listing

```
📁 module11-pdf-docs/
├── document-01.pdf
├── document-02.pdf
├── document-03.pdf
├── document-04.pdf
├── document-05.pdf
└── ... (additional PDFs)
```

---

## Part 2: Connect Data Source in Azure AI Foundry (15 minutes)

### Step 2.1 — Open Azure AI Foundry

1. Navigate to [https://ai.azure.com](https://ai.azure.com)
2. Select your AI Foundry project
3. In the left navigation, click **Data** (under Management)

### Step 2.2 — Create a New Data Connection

1. Click **+ New connection**
2. Select **Azure Blob Storage** as the data source type
3. Configure the connection:
   - **Connection name:** `pdf-training-docs`
   - **Storage account:** Select your storage account
   - **Container:** `module11-pdf-docs`
   - **Authentication:** Choose your preferred method
     - **Managed Identity** (recommended for production)
     - **Account Key** (simpler for lab purposes)
4. Click **Create connection**

> **⚠️ Important:** If using Managed Identity, ensure the AI Foundry project's managed identity has the **Storage Blob Data Reader** role on the storage account.

### Step 2.3 — Verify the Connection

1. Once created, click on the `pdf-training-docs` connection
2. You should see the list of PDF files in the container
3. Confirm the file count matches what you uploaded

---

## Part 3: Create a Vector Index (20 minutes)

### Step 3.1 — Start the Indexing Process

1. In your AI Foundry project, navigate to **Indexes** (under Management)
2. Click **+ New index**
3. Configure the index:
   - **Index name:** `module11-pdf-index`
   - **Data source:** Select `pdf-training-docs` (the connection you just created)
   - **Files:** Select all PDF documents

### Step 3.2 — Configure Index Settings

1. **Chunking settings:**
   - **Chunk size:** 1024 tokens (recommended starting point)
   - **Chunk overlap:** 128 tokens
   - **Chunking strategy:** Sentence-based
2. **Embedding model:**
   - Select your deployed embedding model (e.g., `text-embedding-ada-002` or `text-embedding-3-small`)
3. **Search type:**
   - Select **Hybrid (vector + keyword)** for best results

> **💡 Tip:** Hybrid search combines the precision of keyword matching with the semantic understanding of vector search, typically delivering the best retrieval quality.

### Step 3.3 — Configure Advanced Options

1. **Metadata fields to extract:**
   - ☑ File name
   - ☑ Page number
   - ☑ File path
2. **Schedule:** One-time (for this lab)
3. Review your configuration and click **Create**

### Step 3.4 — Monitor Indexing Progress

1. The indexing process will begin automatically
2. Monitor the status on the Indexes page
3. Wait for status to show **Completed** (typically 5–15 minutes depending on document count and size)
4. Note the statistics:
   - Total documents processed
   - Total chunks created
   - Any errors or warnings

---

## Part 4: Validate Search Results (10 minutes)

### Step 4.1 — Test with Keyword Search

1. Navigate to the **Indexes** section
2. Click on `module11-pdf-index`
3. Use the **Search** tool
4. Enter a keyword query related to your document content
5. Review the results:
   - Are relevant chunks returned?
   - Do the source file references look correct?
   - Check the relevance scores

### Step 4.2 — Test with Semantic Search

1. Switch to **Vector search** mode
2. Enter a natural language question about your document content
3. Compare results with the keyword search:
   - Does vector search find semantically related content?
   - Are results more or less relevant than keyword search?

### Step 4.3 — Test with Hybrid Search

1. Switch to **Hybrid** mode
2. Enter the same question
3. Observe how hybrid search combines both approaches
4. Document your observations:

| Search Type | Query | Top Result | Relevance Score | Notes |
|------------|-------|------------|-----------------|-------|
| Keyword | | | | |
| Vector | | | | |
| Hybrid | | | | |

---

## Bonus Challenges

If you finish early, try these additional exercises:

### Challenge 1: Adjust Chunking Strategy
- Create a second index with different chunk sizes (512 vs. 2048 tokens)
- Compare search quality between the two indexes
- Document which chunk size works better for your content

### Challenge 2: Add a Second Data Source
- Create an Azure SQL Database with sample structured data
- Add it as an additional data connection in AI Foundry
- Compare the experience of indexing structured vs. unstructured data

### Challenge 3: Test with a Playground Chat
- Navigate to the **Playground** in Azure AI Foundry
- Configure a chat session using your index as a data source
- Ask questions and evaluate the quality of grounded responses
- Note how the AI cites specific chunks from your documents

---

## Troubleshooting

| Issue | Solution |
|-------|----------|
| Connection fails with 403 error | Verify RBAC permissions — ensure Storage Blob Data Reader role is assigned |
| PDFs not appearing in connection | Check that files were uploaded to the correct container |
| Indexing shows errors | Check that PDFs are text-based, not scanned images without OCR |
| Search returns no results | Verify index status is "Completed" and try broader search terms |
| Low relevance scores | Try adjusting chunk size or switching to hybrid search mode |
| Embedding model not found | Ensure you have a deployed embedding model in your AI Foundry project |

---

## Clean-up (Optional)

If you want to conserve costs after the lab:

1. Delete the index: **Indexes** → `module11-pdf-index` → **Delete**
2. Delete the data connection: **Data** → `pdf-training-docs` → **Delete**
3. Optionally delete the blob container if no longer needed

> **Note:** Keep these resources if you plan to use them in Module 12 and later RAG modules.

---

## Lab Summary

In this lab, you successfully:

- ✅ Uploaded PDF documents to Azure Blob Storage
- ✅ Created a data connection in Azure AI Foundry
- ✅ Configured and executed a vector indexing pipeline
- ✅ Validated search results using keyword, vector, and hybrid search
- ✅ Applied chunking and metadata extraction strategies

## What's Next

In **Module 12**, you will build on this indexed data to implement full RAG pipelines, connecting your data index to AI models for grounded, context-aware responses.

---

*Azure AI Foundry: Zero to Hero Training Course — © 2026 Zero to Hero Training Team*
