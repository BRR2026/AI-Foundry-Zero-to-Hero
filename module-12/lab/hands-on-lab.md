# Module 12 — Hands-On Lab

## Create a Vector Index and Run Hybrid Search Queries

| Field            | Details                                       |
|------------------|-----------------------------------------------|
| **Module**       | 12 — Azure AI Search & Vector Indexes         |
| **Arc**          | ARC 3 — DATA & RAG                           |
| **Duration**     | 45–60 minutes                                 |
| **Difficulty**   | Intermediate                                  |
| **Goal**         | Build a vector index, ingest documents with embeddings, and execute hybrid search queries |

---

## Prerequisites

Before starting this lab, ensure you have:

- [ ] An active Azure subscription
- [ ] An **Azure AI Search** resource (Basic tier or higher recommended; Free tier works for small datasets)
- [ ] An **Azure OpenAI** resource with a deployed `text-embedding-ada-002` or `text-embedding-3-large` model
- [ ] Python 3.10+ installed locally
- [ ] The following Python packages installed:

```bash
pip install azure-search-documents==11.6.0 azure-identity openai python-dotenv
```

- [ ] Access to the Azure AI Foundry portal at [https://ai.azure.com](https://ai.azure.com)

---

## Lab Architecture

```
┌──────────────────────┐
│   Sample Documents   │
│   (JSON / PDF)       │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐     ┌───────────────────────┐
│  Embedding Model     │────▶│  Azure AI Search      │
│  (Azure OpenAI)      │     │  Vector Index          │
└──────────────────────┘     └──────────┬────────────┘
                                        │
                                        ▼
                             ┌───────────────────────┐
                             │  Hybrid Search Query   │
                             │  keyword + semantic    │
                             │  + vector              │
                             └───────────────────────┘
```

---

## Exercise 1 — Set Up Your Environment

### Step 1.1 — Create a `.env` File

Create a file named `.env` in your working directory:

```env
AZURE_SEARCH_ENDPOINT=https://<your-search-service>.search.windows.net
AZURE_SEARCH_ADMIN_KEY=<your-admin-key>
AZURE_SEARCH_INDEX_NAME=module12-vector-lab
AZURE_OPENAI_ENDPOINT=https://<your-openai-resource>.openai.azure.com
AZURE_OPENAI_API_KEY=<your-openai-key>
AZURE_OPENAI_EMBEDDING_DEPLOYMENT=text-embedding-ada-002
```

### Step 1.2 — Create the Python Configuration

Create `config.py`:

```python
import os
from dotenv import load_dotenv

load_dotenv()

SEARCH_ENDPOINT = os.getenv("AZURE_SEARCH_ENDPOINT")
SEARCH_ADMIN_KEY = os.getenv("AZURE_SEARCH_ADMIN_KEY")
INDEX_NAME = os.getenv("AZURE_SEARCH_INDEX_NAME", "module12-vector-lab")
OPENAI_ENDPOINT = os.getenv("AZURE_OPENAI_ENDPOINT")
OPENAI_API_KEY = os.getenv("AZURE_OPENAI_API_KEY")
EMBEDDING_DEPLOYMENT = os.getenv("AZURE_OPENAI_EMBEDDING_DEPLOYMENT")
```

---

## Exercise 2 — Create the Vector Index

### Step 2.1 — Define the Index Schema

Create `create_index.py`:

```python
from azure.search.documents.indexes import SearchIndexClient
from azure.search.documents.indexes.models import (
    SearchIndex,
    SearchField,
    SearchFieldDataType,
    SimpleField,
    SearchableField,
    VectorSearch,
    HnswAlgorithmConfiguration,
    VectorSearchProfile,
    SemanticConfiguration,
    SemanticSearch,
    SemanticPrioritizedFields,
    SemanticField,
)
from azure.core.credentials import AzureKeyCredential
from config import SEARCH_ENDPOINT, SEARCH_ADMIN_KEY, INDEX_NAME


def create_index():
    client = SearchIndexClient(
        endpoint=SEARCH_ENDPOINT,
        credential=AzureKeyCredential(SEARCH_ADMIN_KEY),
    )

    fields = [
        SimpleField(name="id", type=SearchFieldDataType.String, key=True, filterable=True),
        SearchableField(name="title", type=SearchFieldDataType.String, analyzer_name="en.microsoft"),
        SearchableField(name="content", type=SearchFieldDataType.String, analyzer_name="en.microsoft"),
        SearchableField(name="category", type=SearchFieldDataType.String, filterable=True, facetable=True),
        SimpleField(name="source_url", type=SearchFieldDataType.String),
        SearchField(
            name="content_vector",
            type=SearchFieldDataType.Collection(SearchFieldDataType.Single),
            searchable=True,
            vector_search_dimensions=1536,
            vector_search_profile_name="vector-profile",
        ),
    ]

    vector_search = VectorSearch(
        algorithms=[
            HnswAlgorithmConfiguration(name="hnsw-config"),
        ],
        profiles=[
            VectorSearchProfile(
                name="vector-profile",
                algorithm_configuration_name="hnsw-config",
            ),
        ],
    )

    semantic_config = SemanticConfiguration(
        name="semantic-config",
        prioritized_fields=SemanticPrioritizedFields(
            title_field=SemanticField(field_name="title"),
            content_fields=[SemanticField(field_name="content")],
        ),
    )

    semantic_search = SemanticSearch(configurations=[semantic_config])

    index = SearchIndex(
        name=INDEX_NAME,
        fields=fields,
        vector_search=vector_search,
        semantic_search=semantic_search,
    )

    result = client.create_or_update_index(index)
    print(f"✅ Index '{result.name}' created successfully.")


if __name__ == "__main__":
    create_index()
```

### Step 2.2 — Run the Script

```bash
python create_index.py
```

**Expected output:**
```
✅ Index 'module12-vector-lab' created successfully.
```

### ✅ Checkpoint 1

Open the Azure portal → your AI Search resource → **Indexes** tab. Confirm you see `module12-vector-lab` with 6 fields including `content_vector`.

---

## Exercise 3 — Generate Embeddings and Upload Documents

### Step 3.1 — Prepare Sample Documents

Create `sample_data.py`:

```python
DOCUMENTS = [
    {
        "id": "1",
        "title": "Introduction to Retrieval-Augmented Generation",
        "content": "RAG combines information retrieval with language model generation. By grounding LLM responses in retrieved documents, RAG reduces hallucinations and provides more accurate, up-to-date answers. Azure AI Search serves as the retrieval backbone for RAG architectures in Azure AI Foundry.",
        "category": "RAG",
        "source_url": "https://learn.microsoft.com/azure/ai-studio/concepts/retrieval-augmented-generation",
    },
    {
        "id": "2",
        "title": "Vector Search Fundamentals",
        "content": "Vector search finds documents by comparing mathematical representations called embeddings. Each document is converted into a high-dimensional vector using an embedding model. At query time, the search query is also embedded and compared using similarity metrics like cosine similarity.",
        "category": "Search",
        "source_url": "https://learn.microsoft.com/azure/search/vector-search-overview",
    },
    {
        "id": "3",
        "title": "Hybrid Search in Azure AI Search",
        "content": "Hybrid search combines keyword-based BM25 scoring with vector similarity search. Azure AI Search uses Reciprocal Rank Fusion (RRF) to merge results from multiple ranking methods. Adding semantic ranking provides a third layer of intelligence with transformer-based re-ranking.",
        "category": "Search",
        "source_url": "https://learn.microsoft.com/azure/search/hybrid-search-overview",
    },
    {
        "id": "4",
        "title": "Chunking Strategies for Document Processing",
        "content": "Effective chunking is critical for RAG systems. Documents should be split into semantically meaningful segments. Common strategies include fixed-size chunking with overlap, sentence-based splitting, and recursive character splitting. Each chunk should be small enough to be relevant but large enough to carry context.",
        "category": "Data Processing",
        "source_url": "https://learn.microsoft.com/azure/search/cognitive-search-skill-textsplit",
    },
    {
        "id": "5",
        "title": "Azure AI Foundry and Search Integration",
        "content": "Azure AI Foundry integrates with Azure AI Search to provide end-to-end RAG workflows. You can connect your search index to a Foundry project and use it as a data source for prompt flow, agents, and evaluations. The integration supports both manual and automated indexing pipelines.",
        "category": "Integration",
        "source_url": "https://ai.azure.com",
    },
    {
        "id": "6",
        "title": "Semantic Ranking Deep Dive",
        "content": "Semantic ranking uses transformer models to re-rank search results based on deep language understanding. It extracts captions and answers from matching documents, going beyond keyword matching to understand meaning and intent. Semantic ranking works on top of existing BM25 or vector search results.",
        "category": "Search",
        "source_url": "https://learn.microsoft.com/azure/search/semantic-search-overview",
    },
    {
        "id": "7",
        "title": "Index Schema Design Patterns",
        "content": "A well-designed index schema balances retrieval quality with storage efficiency. Use SearchableField for full-text search, SimpleField for filters and sorting, and vector fields for embedding-based retrieval. Consider using multiple vector fields for different embedding models or content representations.",
        "category": "Best Practices",
        "source_url": "https://learn.microsoft.com/azure/search/search-what-is-an-index",
    },
    {
        "id": "8",
        "title": "Integrated Vectorization in Azure AI Search",
        "content": "Integrated vectorization automatically generates embeddings at both indexing and query time. By defining a vectorizer in the index, Azure AI Search calls your embedding model on your behalf. This eliminates the need for a separate embedding pipeline and simplifies the end-to-end architecture.",
        "category": "Search",
        "source_url": "https://learn.microsoft.com/azure/search/vector-search-integrated-vectorization",
    },
]
```

### Step 3.2 — Generate Embeddings and Upload

Create `upload_documents.py`:

```python
from openai import AzureOpenAI
from azure.search.documents import SearchClient
from azure.core.credentials import AzureKeyCredential
from config import (
    SEARCH_ENDPOINT, SEARCH_ADMIN_KEY, INDEX_NAME,
    OPENAI_ENDPOINT, OPENAI_API_KEY, EMBEDDING_DEPLOYMENT,
)
from sample_data import DOCUMENTS


def get_embedding(client: AzureOpenAI, text: str) -> list[float]:
    response = client.embeddings.create(
        input=text,
        model=EMBEDDING_DEPLOYMENT,
    )
    return response.data[0].embedding


def upload_documents():
    openai_client = AzureOpenAI(
        azure_endpoint=OPENAI_ENDPOINT,
        api_key=OPENAI_API_KEY,
        api_version="2024-10-21",
    )

    search_client = SearchClient(
        endpoint=SEARCH_ENDPOINT,
        index_name=INDEX_NAME,
        credential=AzureKeyCredential(SEARCH_ADMIN_KEY),
    )

    for doc in DOCUMENTS:
        text_to_embed = f"{doc['title']} {doc['content']}"
        doc["content_vector"] = get_embedding(openai_client, text_to_embed)
        print(f"  ✅ Embedded: {doc['title']}")

    result = search_client.upload_documents(documents=DOCUMENTS)
    succeeded = sum(1 for r in result if r.succeeded)
    print(f"\n✅ Uploaded {succeeded}/{len(DOCUMENTS)} documents successfully.")


if __name__ == "__main__":
    upload_documents()
```

### Step 3.3 — Run the Upload

```bash
python upload_documents.py
```

### ✅ Checkpoint 2

In the Azure portal, navigate to your index and verify the document count shows **8 documents**.

---

## Exercise 4 — Run Hybrid Search Queries

### Step 4.1 — Create the Search Script

Create `hybrid_search.py`:

```python
from openai import AzureOpenAI
from azure.search.documents import SearchClient
from azure.search.documents.models import VectorizableTextQuery
from azure.core.credentials import AzureKeyCredential
from config import (
    SEARCH_ENDPOINT, SEARCH_ADMIN_KEY, INDEX_NAME,
    OPENAI_ENDPOINT, OPENAI_API_KEY, EMBEDDING_DEPLOYMENT,
)


def get_embedding(client: AzureOpenAI, text: str) -> list[float]:
    response = client.embeddings.create(input=text, model=EMBEDDING_DEPLOYMENT)
    return response.data[0].embedding


def hybrid_search(query: str):
    openai_client = AzureOpenAI(
        azure_endpoint=OPENAI_ENDPOINT,
        api_key=OPENAI_API_KEY,
        api_version="2024-10-21",
    )

    search_client = SearchClient(
        endpoint=SEARCH_ENDPOINT,
        index_name=INDEX_NAME,
        credential=AzureKeyCredential(SEARCH_ADMIN_KEY),
    )

    query_vector = get_embedding(openai_client, query)

    results = search_client.search(
        search_text=query,                        # Keyword search
        vector_queries=[
            VectorizableTextQuery(
                text=query,
                k_nearest_neighbors=3,
                fields="content_vector",
            )
        ] if False else [],                        # Use raw vector below
        query_type="semantic",                     # Semantic ranking
        semantic_configuration_name="semantic-config",
        top=5,
    )

    # For raw vector hybrid search:
    from azure.search.documents.models import VectorizedQuery
    results = search_client.search(
        search_text=query,
        vector_queries=[
            VectorizedQuery(
                vector=query_vector,
                k_nearest_neighbors=3,
                fields="content_vector",
            )
        ],
        query_type="semantic",
        semantic_configuration_name="semantic-config",
        top=5,
    )

    print(f"\n🔍 Hybrid Search Results for: '{query}'\n")
    print("-" * 70)
    for i, result in enumerate(results, 1):
        score = result.get("@search.score", "N/A")
        reranker = result.get("@search.reranker_score", "N/A")
        print(f"  {i}. {result['title']}")
        print(f"     Category : {result['category']}")
        print(f"     Score    : {score}")
        print(f"     Reranker : {reranker}")
        print(f"     Snippet  : {result['content'][:120]}...")
        print()


if __name__ == "__main__":
    queries = [
        "How does RAG reduce hallucinations?",
        "What is the best way to split documents into chunks?",
        "How do I combine keyword and vector search?",
    ]
    for q in queries:
        hybrid_search(q)
```

### Step 4.2 — Run Hybrid Queries

```bash
python hybrid_search.py
```

### ✅ Checkpoint 3

Verify that:
- Each query returns ranked results with both `@search.score` and `@search.reranker_score`.
- The most semantically relevant document appears at or near the top.
- Results combine keyword matches and vector similarity.

---

## Exercise 5 — Experiment with Search Modes

### Step 5.1 — Compare Search Approaches

Modify `hybrid_search.py` to run the same query three ways:

| Mode | `search_text` | `vector_queries` | `query_type` |
|------|--------------|-------------------|--------------|
| Keyword only | ✅ | ❌ | `"simple"` |
| Vector only | ❌ | ✅ | `"simple"` |
| Hybrid + Semantic | ✅ | ✅ | `"semantic"` |

### Step 5.2 — Observe Differences

For the query **"How does chunking affect retrieval quality?"**, note:

1. **Keyword only** — May miss documents that use different terminology.
2. **Vector only** — Finds semantically similar content regardless of exact words.
3. **Hybrid + Semantic** — Combines both and re-ranks for best relevance.

### ✅ Checkpoint 4

Document your observations about which search mode produces the best results for different query types.

---

## Exercise 6 — Apply Filters

### Step 6.1 — Add a Category Filter

```python
results = search_client.search(
    search_text="search techniques",
    vector_queries=[
        VectorizedQuery(
            vector=query_vector,
            k_nearest_neighbors=3,
            fields="content_vector",
        )
    ],
    filter="category eq 'Search'",
    query_type="semantic",
    semantic_configuration_name="semantic-config",
    top=5,
)
```

### ✅ Checkpoint 5

Verify that filtered results only contain documents with `category == "Search"`.

---

## Cleanup

When finished, delete the resources to avoid ongoing charges:

```bash
# Delete the index (keeps the search service)
az search index delete --name module12-vector-lab \
    --service-name <your-search-service> \
    --resource-group <your-resource-group>
```

Or delete via the Azure portal under your AI Search resource → Indexes.

---

## Summary

In this lab you:

| Task | Status |
|------|--------|
| Created a vector index with HNSW configuration | ✅ |
| Defined a schema with text, vector, and filterable fields | ✅ |
| Generated embeddings with Azure OpenAI | ✅ |
| Uploaded documents with vector embeddings | ✅ |
| Ran hybrid search queries (keyword + vector + semantic) | ✅ |
| Compared search modes and applied filters | ✅ |

**Next up → Module 13: Retrieval-Augmented Generation (RAG) Patterns** — Use your search index as the retrieval layer for a full RAG pipeline.
