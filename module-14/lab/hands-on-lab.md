# Module 14: Advanced RAG Patterns — Hands-On Lab

## Mini Hack: Optimize Your RAG App with Semantic Chunking + Reranking

**Course:** Azure AI Foundry: Zero to Hero
**Module:** 14 of 45 | ARC 3 — DATA & RAG
**Duration:** 60–90 minutes
**Difficulty:** Intermediate to Advanced

---

## Overview

In this lab, you will take a baseline RAG application and iteratively improve its retrieval quality by implementing semantic chunking, adding a reranking layer, and configuring hybrid search. You will measure improvements at each stage using relevance metrics.

### What You'll Build

A production-grade RAG pipeline in Azure AI Foundry that:
- Uses semantic chunking to create context-aware document segments
- Applies Azure AI Search semantic ranker for reranking
- Combines keyword and vector search with hybrid search
- Measures and compares quality across configurations

### Prerequisites

- Completed Module 13 (Data Ingestion & Indexing)
- Azure subscription with AI Foundry hub and project
- Azure AI Search resource (Standard tier or higher for semantic ranking)
- Azure OpenAI resource with `gpt-4o` and `text-embedding-3-large` deployed
- Python 3.10+ with VS Code
- Sample documents (provided in lab assets)

---

## Lab Setup

### Step 1: Environment Configuration

```bash
# Clone the lab repository (if not already done)
cd AI-Foundry-Training/module-14/lab

# Create and activate virtual environment
python -m venv .venv
# Windows
.venv\Scripts\activate
# macOS/Linux
source .venv/bin/activate

# Install dependencies
pip install azure-ai-projects azure-search-documents azure-ai-ml \
    openai langchain-text-splitters tiktoken pandas
```

### Step 2: Configure Environment Variables

Create a `.env` file in the lab directory:

```env
AZURE_AI_FOUNDRY_PROJECT_CONNECTION=<your-project-connection-string>
AZURE_SEARCH_ENDPOINT=https://<your-search-service>.search.windows.net
AZURE_SEARCH_API_KEY=<your-search-admin-key>
AZURE_OPENAI_ENDPOINT=https://<your-openai-resource>.openai.azure.com
AZURE_OPENAI_API_KEY=<your-openai-key>
AZURE_OPENAI_EMBEDDING_DEPLOYMENT=text-embedding-3-large
AZURE_OPENAI_CHAT_DEPLOYMENT=gpt-4o
```

---

## Exercise 1: Baseline RAG with Fixed-Size Chunking

**Objective:** Establish a baseline RAG pipeline using fixed-size chunks.

### Step 1: Load and Chunk Documents

```python
# baseline_chunking.py
from langchain_text_splitters import CharacterTextSplitter
import tiktoken

def fixed_size_chunk(text: str, chunk_size: int = 512, overlap: int = 50) -> list[str]:
    """Split text into fixed-size chunks with overlap."""
    splitter = CharacterTextSplitter(
        separator="\n",
        chunk_size=chunk_size,
        chunk_overlap=overlap,
        length_function=lambda t: len(tiktoken.encoding_for_model("gpt-4o").encode(t))
    )
    return splitter.split_text(text)

# Load sample document
with open("sample-docs/azure-ai-foundry-overview.md", "r") as f:
    document_text = f.read()

chunks = fixed_size_chunk(document_text)
print(f"Fixed-size chunking produced {len(chunks)} chunks")
print(f"Average chunk length: {sum(len(c) for c in chunks) // len(chunks)} chars")
```

### Step 2: Create Search Index and Upload

```python
# create_baseline_index.py
from azure.search.documents.indexes import SearchIndexClient
from azure.search.documents.indexes.models import (
    SearchIndex, SearchField, SearchFieldDataType,
    VectorSearch, HnswAlgorithmConfiguration,
    VectorSearchProfile, SearchableField, SimpleField
)
from azure.core.credentials import AzureKeyCredential

index_name = "module14-baseline"

# Define index schema with vector field
fields = [
    SimpleField(name="id", type=SearchFieldDataType.String, key=True),
    SearchableField(name="content", type=SearchFieldDataType.String),
    SearchField(
        name="content_vector",
        type=SearchFieldDataType.Collection(SearchFieldDataType.Single),
        searchable=True,
        vector_search_dimensions=3072,
        vector_search_profile_name="myHnswProfile"
    ),
    SimpleField(name="chunk_index", type=SearchFieldDataType.Int32),
    SearchableField(name="source", type=SearchFieldDataType.String, filterable=True)
]

vector_search = VectorSearch(
    algorithms=[HnswAlgorithmConfiguration(name="myHnsw")],
    profiles=[VectorSearchProfile(name="myHnswProfile", algorithm_configuration_name="myHnsw")]
)

index = SearchIndex(name=index_name, fields=fields, vector_search=vector_search)

# Create the index
index_client = SearchIndexClient(endpoint=SEARCH_ENDPOINT, credential=AzureKeyCredential(SEARCH_KEY))
index_client.create_or_update_index(index)
print(f"Index '{index_name}' created successfully")
```

### Step 3: Query and Record Baseline Metrics

```python
# query_baseline.py
test_queries = [
    "How does Azure AI Foundry handle model deployment?",
    "What are the security features in AI Foundry?",
    "How to monitor AI model performance?",
    "What data sources can be connected to AI Foundry?",
    "How does prompt flow work in AI Foundry?"
]

# Run queries and record relevance scores
for query in test_queries:
    results = search_client.search(
        search_text=query,
        vector_queries=[vector_query],
        top=5
    )
    # Record scores for comparison
    print(f"Query: {query[:50]}...")
    for r in results:
        print(f"  Score: {r['@search.score']:.4f} | {r['content'][:80]}...")
```

📝 **Record your baseline scores** — you will compare against these after each optimization.

---

## Exercise 2: Implement Semantic Chunking

**Objective:** Replace fixed-size chunking with semantic chunking and measure improvement.

### Step 1: Implement Semantic Chunker

```python
# semantic_chunking.py
from langchain_text_splitters import RecursiveCharacterTextSplitter
import re

def semantic_chunk(text: str, max_chunk_size: int = 800) -> list[str]:
    """Split text using semantic boundaries (headers, paragraphs, sentences)."""
    splitter = RecursiveCharacterTextSplitter(
        separators=[
            "\n## ",       # H2 headers
            "\n### ",      # H3 headers
            "\n#### ",     # H4 headers
            "\n\n",        # Paragraph breaks
            "\n",          # Line breaks
            ". ",          # Sentence endings
            " ",           # Word boundaries
        ],
        chunk_size=max_chunk_size,
        chunk_overlap=100,
        length_function=len,
        is_separator_regex=False
    )

    chunks = splitter.split_text(text)

    # Post-process: add header context to each chunk
    enriched_chunks = []
    current_header = ""
    for chunk in chunks:
        header_match = re.match(r'^(#{1,4}\s+.+)', chunk)
        if header_match:
            current_header = header_match.group(1)
        elif current_header and not chunk.startswith("#"):
            chunk = f"{current_header}\n\n{chunk}"
        enriched_chunks.append(chunk)

    return enriched_chunks

semantic_chunks = semantic_chunk(document_text)
print(f"Semantic chunking produced {len(semantic_chunks)} chunks")
```

### Step 2: Create Semantic Index and Re-Index

```python
# Create a new index "module14-semantic" with the same schema
# Upload semantic chunks with embeddings
# (Use the same index creation code with index_name = "module14-semantic")
```

### Step 3: Compare Results

Run the same test queries against the semantic index and compare scores.

📊 **Expected improvement:** 10–25% better relevance scores on structured documents.

---

## Exercise 3: Add Reranking with Semantic Ranker

**Objective:** Enable Azure AI Search semantic ranker for second-stage reranking.

### Step 1: Enable Semantic Configuration

```python
# enable_reranking.py
from azure.search.documents.indexes.models import (
    SemanticConfiguration, SemanticSearch,
    SemanticPrioritizedFields, SemanticField
)

semantic_config = SemanticConfiguration(
    name="my-semantic-config",
    prioritized_fields=SemanticPrioritizedFields(
        content_fields=[SemanticField(field_name="content")]
    )
)

semantic_search = SemanticSearch(configurations=[semantic_config])

# Update the index with semantic configuration
index.semantic_search = semantic_search
index_client.create_or_update_index(index)
print("Semantic ranker enabled on index")
```

### Step 2: Query with Reranking

```python
# query_with_reranking.py
from azure.search.documents.models import QueryType

results = search_client.search(
    search_text=query,
    vector_queries=[vector_query],
    query_type=QueryType.SEMANTIC,
    semantic_configuration_name="my-semantic-config",
    top=5
)

for r in results:
    print(f"  Reranker Score: {r['@search.reranker_score']:.4f}")
    print(f"  Content: {r['content'][:80]}...")
```

### Step 3: Measure Improvement

Compare reranker scores with baseline scores for all test queries.

📊 **Expected improvement:** 15–35% better top-result relevance with reranking.

---

## Exercise 4: Configure Hybrid Search

**Objective:** Combine keyword search with vector search using RRF.

### Step 1: Implement Hybrid Query

```python
# hybrid_search.py
from azure.search.documents.models import VectorizedQuery

def hybrid_search(query: str, search_client, embedding_client):
    """Execute hybrid search combining keyword and vector retrieval."""
    # Generate query embedding
    query_embedding = embedding_client.embeddings.create(
        input=query,
        model=EMBEDDING_DEPLOYMENT
    ).data[0].embedding

    vector_query = VectorizedQuery(
        vector=query_embedding,
        k_nearest_neighbors=10,
        fields="content_vector"
    )

    # Hybrid search: keyword + vector + semantic reranking
    results = search_client.search(
        search_text=query,              # Keyword component
        vector_queries=[vector_query],   # Vector component
        query_type=QueryType.SEMANTIC,   # Reranking
        semantic_configuration_name="my-semantic-config",
        top=5
    )

    return list(results)
```

### Step 2: Compare All Approaches

Create a comparison table:

| Query | Baseline | Semantic Chunks | + Reranking | + Hybrid |
|-------|----------|-----------------|-------------|----------|
| Q1    | _score_  | _score_         | _score_     | _score_  |
| Q2    | _score_  | _score_         | _score_     | _score_  |
| ...   | ...      | ...             | ...         | ...      |

---

## Exercise 5: Multi-Index Query (Bonus)

**Objective:** Query across two indexes and merge results.

```python
# multi_index_rag.py
from azure.search.documents import SearchClient

# Create clients for two different indexes
docs_client = SearchClient(endpoint=SEARCH_ENDPOINT,
    index_name="module14-semantic",
    credential=AzureKeyCredential(SEARCH_KEY))

faq_client = SearchClient(endpoint=SEARCH_ENDPOINT,
    index_name="module14-faq",
    credential=AzureKeyCredential(SEARCH_KEY))

def multi_index_search(query: str) -> list:
    """Search across multiple indexes and merge results by score."""
    docs_results = list(docs_client.search(search_text=query, top=5))
    faq_results = list(faq_client.search(search_text=query, top=5))

    # Merge and sort by score (simple interleaving)
    all_results = [(r, "docs") for r in docs_results] + [(r, "faq") for r in faq_results]
    all_results.sort(key=lambda x: x[0]["@search.score"], reverse=True)

    return all_results[:5]
```

---

## Validation Checklist

Before completing the lab, verify:

- [ ] Baseline RAG pipeline works with fixed-size chunks
- [ ] Semantic chunking produces contextually meaningful chunks
- [ ] Semantic ranker is enabled and reranker scores are visible
- [ ] Hybrid search combines keyword + vector + reranking
- [ ] You have a comparison table showing improvements across approaches
- [ ] (Bonus) Multi-index query returns merged results

---

## Clean Up Resources

```bash
# Delete search indexes to avoid charges
python -c "
from azure.search.documents.indexes import SearchIndexClient
from azure.core.credentials import AzureKeyCredential
client = SearchIndexClient(endpoint=SEARCH_ENDPOINT, credential=AzureKeyCredential(SEARCH_KEY))
for idx in ['module14-baseline', 'module14-semantic', 'module14-faq']:
    try:
        client.delete_index(idx)
        print(f'Deleted index: {idx}')
    except: pass
"
```

---

## Key Takeaways

1. **Semantic chunking** preserves document structure and meaning, improving retrieval over fixed-size approaches
2. **Reranking** with Azure AI Search semantic ranker provides significant quality improvements with minimal code changes
3. **Hybrid search** combines the precision of keyword search with the understanding of vector search
4. **Iterative optimization** is key — measure at each stage to understand what improves quality

---

## Navigation

| Previous | Next |
|----------|------|
| [Module 13 — Data Ingestion & Indexing](../module-13/blog.html) | [Module 15 — Evaluation & Quality Metrics](../module-15/blog.html) |
