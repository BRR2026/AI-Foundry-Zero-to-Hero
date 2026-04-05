# Module 13: Hands-On Lab

## Mini Hack — Build a RAG Chatbot Over Your Company Docs

**Course:** Azure AI Foundry — Zero to Hero (Module 13 of 45)
**Arc:** ARC 3 · DATA & RAG
**Duration:** 90 minutes
**Difficulty:** Intermediate

---

## Lab Overview

In this hands-on lab you will build a complete RAG (Retrieval-Augmented Generation) chatbot that answers questions grounded in your own company documents. You will set up an Azure AI Search index, connect it to a deployed model in Azure AI Foundry, and build a Python application with proper citation handling.

## Objectives

By the end of this lab you will have:

- ✅ Created and populated an Azure AI Search index with vectorized document chunks
- ✅ Tested the "On Your Data" feature in the AI Foundry portal
- ✅ Built a Python RAG chatbot using the Azure AI Foundry SDK
- ✅ Implemented citation parsing and source attribution
- ✅ Designed a system prompt that keeps responses grounded

## Prerequisites

| Requirement | Details |
|-------------|---------|
| Azure Subscription | Active subscription with Contributor access |
| AI Foundry Project | Hub + project from Module 3 |
| Deployed Model | GPT-4o (or equivalent chat model) deployed in AI Foundry |
| Azure AI Search | Basic tier or above, provisioned in the same region |
| Python Environment | Python 3.10+ with pip |
| Sample Documents | 5–10 PDF/Markdown files (company policies, FAQs, or product docs) |

---

## Part 1: Prepare Your Document Collection (15 min)

### Step 1.1 — Organize Sample Documents

Create a local folder with 5–10 documents that represent your "company knowledge base." If you do not have corporate documents, use the provided sample set.

```
company-docs/
├── employee-handbook.md
├── it-security-policy.pdf
├── product-faq.md
├── expense-policy.md
└── onboarding-guide.md
```

### Step 1.2 — Install Required Python Packages

```bash
pip install azure-ai-projects azure-search-documents azure-identity openai python-dotenv
```

### Step 1.3 — Create Environment Configuration

Create a `.env` file in your project root:

```env
AZURE_AI_PROJECT_CONNECTION_STRING=your_project_connection_string
AZURE_SEARCH_ENDPOINT=https://your-search-service.search.windows.net
AZURE_SEARCH_INDEX_NAME=company-docs-index
AZURE_SEARCH_API_KEY=your_search_admin_key
AZURE_OPENAI_ENDPOINT=https://your-openai-resource.openai.azure.com
AZURE_OPENAI_DEPLOYMENT=gpt-4o
AZURE_OPENAI_API_KEY=your_openai_key
AZURE_OPENAI_EMBEDDING_DEPLOYMENT=text-embedding-ada-002
```

> ⚠️ **Security Note:** Never commit `.env` files to source control. Add `.env` to your `.gitignore`.

---

## Part 2: Create & Populate an Azure AI Search Index (25 min)

### Step 2.1 — Document Chunking Script

Create `index_documents.py`:

```python
import os
from dotenv import load_dotenv
from azure.search.documents import SearchClient
from azure.search.documents.indexes import SearchIndexClient
from azure.search.documents.indexes.models import (
    SearchIndex,
    SimpleField,
    SearchableField,
    SearchField,
    SearchFieldDataType,
    VectorSearch,
    HnswAlgorithmConfiguration,
    VectorSearchProfile,
)
from azure.core.credentials import AzureKeyCredential
from openai import AzureOpenAI
import hashlib
import json

load_dotenv()

# Initialize clients
search_index_client = SearchIndexClient(
    endpoint=os.getenv("AZURE_SEARCH_ENDPOINT"),
    credential=AzureKeyCredential(os.getenv("AZURE_SEARCH_API_KEY")),
)

openai_client = AzureOpenAI(
    azure_endpoint=os.getenv("AZURE_OPENAI_ENDPOINT"),
    api_key=os.getenv("AZURE_OPENAI_API_KEY"),
    api_version="2024-12-01-preview",
)

INDEX_NAME = os.getenv("AZURE_SEARCH_INDEX_NAME")


def create_search_index():
    """Create an Azure AI Search index with vector search support."""
    fields = [
        SimpleField(name="id", type=SearchFieldDataType.String, key=True),
        SearchableField(name="content", type=SearchFieldDataType.String),
        SimpleField(name="source_file", type=SearchFieldDataType.String, filterable=True),
        SimpleField(name="page_number", type=SearchFieldDataType.Int32, filterable=True),
        SimpleField(name="chunk_index", type=SearchFieldDataType.Int32),
        SearchField(
            name="content_vector",
            type=SearchFieldDataType.Collection(SearchFieldDataType.Single),
            searchable=True,
            vector_search_dimensions=1536,
            vector_search_profile_name="vector-profile",
        ),
    ]

    vector_search = VectorSearch(
        algorithms=[HnswAlgorithmConfiguration(name="hnsw-config")],
        profiles=[VectorSearchProfile(name="vector-profile", algorithm_configuration_name="hnsw-config")],
    )

    index = SearchIndex(name=INDEX_NAME, fields=fields, vector_search=vector_search)
    search_index_client.create_or_update_index(index)
    print(f"Index '{INDEX_NAME}' created successfully.")


def chunk_document(text, chunk_size=500, overlap=50):
    """Split document text into overlapping chunks."""
    words = text.split()
    chunks = []
    for i in range(0, len(words), chunk_size - overlap):
        chunk = " ".join(words[i : i + chunk_size])
        if chunk.strip():
            chunks.append(chunk)
    return chunks


def get_embedding(text):
    """Generate embedding vector for a text chunk."""
    response = openai_client.embeddings.create(
        input=text,
        model=os.getenv("AZURE_OPENAI_EMBEDDING_DEPLOYMENT"),
    )
    return response.data[0].embedding


def index_documents(docs_folder="company-docs"):
    """Read, chunk, embed, and index all documents."""
    search_client = SearchClient(
        endpoint=os.getenv("AZURE_SEARCH_ENDPOINT"),
        index_name=INDEX_NAME,
        credential=AzureKeyCredential(os.getenv("AZURE_SEARCH_API_KEY")),
    )

    documents = []
    for filename in os.listdir(docs_folder):
        filepath = os.path.join(docs_folder, filename)
        if not os.path.isfile(filepath):
            continue

        with open(filepath, "r", encoding="utf-8", errors="ignore") as f:
            text = f.read()

        chunks = chunk_document(text)
        for idx, chunk in enumerate(chunks):
            doc_id = hashlib.md5(f"{filename}-{idx}".encode()).hexdigest()
            embedding = get_embedding(chunk)
            documents.append({
                "id": doc_id,
                "content": chunk,
                "source_file": filename,
                "page_number": idx + 1,
                "chunk_index": idx,
                "content_vector": embedding,
            })
            print(f"  Processed chunk {idx + 1}/{len(chunks)} from {filename}")

    result = search_client.upload_documents(documents=documents)
    succeeded = sum(1 for r in result if r.succeeded)
    print(f"\nIndexed {succeeded}/{len(documents)} document chunks.")


if __name__ == "__main__":
    create_search_index()
    index_documents()
```

### Step 2.2 — Run the Indexing Script

```bash
python index_documents.py
```

**Expected Output:**
```
Index 'company-docs-index' created successfully.
  Processed chunk 1/4 from employee-handbook.md
  Processed chunk 2/4 from employee-handbook.md
  ...
Indexed 23/23 document chunks.
```

### Step 2.3 — Verify in the Azure Portal

1. Navigate to your Azure AI Search resource in the Azure Portal
2. Select **Indexes** → `company-docs-index`
3. Click **Search explorer** and run an empty search to confirm documents are indexed

> ✅ **Checkpoint:** You should see your indexed documents with content, source file, and vector fields populated.

---

## Part 3: Test "On Your Data" in AI Foundry Portal (15 min)

### Step 3.1 — Open AI Foundry Playground

1. Go to [https://ai.azure.com](https://ai.azure.com)
2. Select your project → **Playgrounds** → **Chat**
3. Select your deployed chat model (e.g., GPT-4o)

### Step 3.2 — Add Your Data

1. In the Chat playground, click **Add your data**
2. Select **Azure AI Search** as the data source
3. Choose your search service and index (`company-docs-index`)
4. Configure field mappings:
   - **Content field:** `content`
   - **Title field:** `source_file`
   - **Vector field:** `content_vector`
5. Select search type: **Hybrid (vector + keyword)**
6. Click **Save and close**

### Step 3.3 — Test Grounded Responses

Ask questions that can be answered from your indexed documents:

```
"What is the company policy on remote work?"
"How do I submit an expense report?"
"What are the IT security requirements for new employees?"
```

> ✅ **Checkpoint:** Responses should include citations with `[doc1]`, `[doc2]` markers referencing your source documents.

---

## Part 4: Build the RAG Chatbot with Python SDK (25 min)

### Step 4.1 — Create the RAG Chatbot Application

Create `rag_chatbot.py`:

```python
import os
from dotenv import load_dotenv
from openai import AzureOpenAI
from azure.identity import DefaultAzureCredential

load_dotenv()

client = AzureOpenAI(
    azure_endpoint=os.getenv("AZURE_OPENAI_ENDPOINT"),
    api_key=os.getenv("AZURE_OPENAI_API_KEY"),
    api_version="2024-12-01-preview",
)

SYSTEM_PROMPT = """You are a helpful company assistant that answers questions
based ONLY on the provided documents. Follow these rules:

1. Only use information from the retrieved documents to answer questions.
2. If the documents do not contain relevant information, say:
   "I don't have enough information in the available documents to answer that question."
3. Always cite your sources using [doc1], [doc2], etc. notation.
4. When multiple documents are relevant, synthesize the information and cite all sources.
5. Be concise but thorough. Prefer bullet points for multi-part answers.
6. Never make up information or use knowledge outside the provided documents.
"""


def chat_with_data(user_message, conversation_history=None):
    """Send a message to the RAG-enabled chat model."""
    if conversation_history is None:
        conversation_history = []

    messages = [{"role": "system", "content": SYSTEM_PROMPT}]
    messages.extend(conversation_history)
    messages.append({"role": "user", "content": user_message})

    response = client.chat.completions.create(
        model=os.getenv("AZURE_OPENAI_DEPLOYMENT"),
        messages=messages,
        extra_body={
            "data_sources": [
                {
                    "type": "azure_search",
                    "parameters": {
                        "endpoint": os.getenv("AZURE_SEARCH_ENDPOINT"),
                        "index_name": os.getenv("AZURE_SEARCH_INDEX_NAME"),
                        "authentication": {
                            "type": "api_key",
                            "key": os.getenv("AZURE_SEARCH_API_KEY"),
                        },
                        "query_type": "vector_simple_hybrid",
                        "embedding_dependency": {
                            "type": "deployment_name",
                            "deployment_name": os.getenv(
                                "AZURE_OPENAI_EMBEDDING_DEPLOYMENT"
                            ),
                        },
                        "in_scope": True,
                        "top_n_documents": 5,
                    },
                }
            ]
        },
        temperature=0.3,
        max_tokens=800,
    )

    return response


def parse_citations(response):
    """Extract and display citations from the response."""
    message = response.choices[0].message
    answer_text = message.content

    citations = []
    if hasattr(message, "context") and message.context:
        for idx, citation in enumerate(message.context.get("citations", [])):
            citations.append({
                "index": idx + 1,
                "source": citation.get("filepath", "Unknown"),
                "title": citation.get("title", "Untitled"),
                "content_snippet": citation.get("content", "")[:150] + "...",
            })

    return answer_text, citations


def main():
    """Run the interactive RAG chatbot."""
    print("=" * 60)
    print("  RAG Chatbot — Powered by Azure AI Foundry")
    print("  Type 'quit' to exit | Type 'clear' to reset history")
    print("=" * 60)

    conversation_history = []

    while True:
        user_input = input("\nYou: ").strip()
        if not user_input:
            continue
        if user_input.lower() == "quit":
            print("Goodbye!")
            break
        if user_input.lower() == "clear":
            conversation_history = []
            print("Conversation history cleared.")
            continue

        try:
            response = chat_with_data(user_input, conversation_history)
            answer, citations = parse_citations(response)

            print(f"\nAssistant: {answer}")

            if citations:
                print("\n📚 Sources:")
                for cite in citations:
                    print(f"  [{cite['index']}] {cite['source']} — {cite['title']}")

            conversation_history.append({"role": "user", "content": user_input})
            conversation_history.append({"role": "assistant", "content": answer})

        except Exception as e:
            print(f"\n❌ Error: {e}")
            print("Please check your configuration and try again.")


if __name__ == "__main__":
    main()
```

### Step 4.2 — Run the Chatbot

```bash
python rag_chatbot.py
```

### Step 4.3 — Test with Sample Questions

Test the following interactions:

1. **In-scope question:** *"What is the vacation policy?"*
   - Should return a grounded answer with citations
2. **Out-of-scope question:** *"What is the capital of France?"*
   - Should respond with "I don't have enough information..."
3. **Multi-document question:** *"Compare the remote work policy with the expense policy for home office equipment."*
   - Should synthesize from multiple sources and cite both

> ✅ **Checkpoint:** Your chatbot should respond with grounded answers and visible source citations.

---

## Part 5: Enhance Citation Handling (10 min)

### Step 5.1 — Add Formatted Citation Display

Create a helper function in your chatbot that formats citations as a Markdown-style reference list:

```python
def format_citation_report(answer, citations):
    """Format the response with a clean citation report."""
    report = f"\n{'─' * 50}\n"
    report += f"📝 Answer:\n{answer}\n"

    if citations:
        report += f"\n{'─' * 50}\n"
        report += "📚 References:\n"
        for cite in citations:
            report += f"\n  [{cite['index']}] {cite['source']}\n"
            report += f"      Title: {cite['title']}\n"
            report += f"      Preview: {cite['content_snippet']}\n"

    report += f"{'─' * 50}\n"
    return report
```

### Step 5.2 — Verify Citation Accuracy

For each response, manually verify:

- [ ] Citations reference the correct source documents
- [ ] Quoted content matches the original document text
- [ ] Citation indices in the answer text match the reference list

---

## Lab Completion Checklist

- [ ] Azure AI Search index created and populated with document chunks
- [ ] "On Your Data" tested successfully in AI Foundry portal playground
- [ ] Python RAG chatbot runs and returns grounded responses
- [ ] Citations are parsed and displayed correctly
- [ ] Out-of-scope questions are handled gracefully
- [ ] System prompt prevents hallucination beyond retrieved context

## Stretch Goals (Optional)

- 🔥 Add streaming responses using the `stream=True` parameter
- 🔥 Implement a Gradio or Streamlit web UI for the chatbot
- 🔥 Add semantic ranking to improve retrieval quality
- 🔥 Implement conversation summarization for long chat sessions
- 🔥 Deploy the chatbot as an Azure Container App

## Troubleshooting

| Issue | Solution |
|-------|----------|
| Empty search results | Verify index has documents; check field mappings |
| No citations in response | Ensure `in_scope` is set to `True` in data source config |
| Authentication errors | Confirm API keys in `.env` match your Azure resources |
| Embedding dimension mismatch | Ensure embedding model matches the `vector_search_dimensions` in your index |
| Rate limiting errors | Add retry logic with exponential backoff, or reduce `top_n_documents` |

---

## Clean-Up

After completing the lab, you may optionally clean up resources:

```bash
# Delete the search index (keep the search service for future modules)
az search index delete --name company-docs-index \
    --service-name your-search-service \
    --resource-group your-rg
```

> 💡 **Recommendation:** Keep your Azure AI Search service active — you will use it again in Modules 14 and 15.

---

*Azure AI Foundry: Zero to Hero Training — © 2026 Zero to Hero Training Team*
