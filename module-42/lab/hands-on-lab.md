# Hands-On Lab: Build the Data + Model + RAG Pipeline

## Module 42 — Azure AI Foundry: Zero to Hero

**Arc 9: Capstone & Mastery** | **Duration:** 90–120 minutes | **Level:** Advanced

---

## 🎯 Lab Objectives

In this lab, you will build a complete Retrieval-Augmented Generation (RAG) pipeline from scratch using Azure AI Foundry. By the end, you will have:

1. Connected data sources and created a vector index
2. Deployed embedding and generation models
3. Built a RAG agent using the file_search tool
4. Implemented a custom RAG pipeline with hybrid search
5. Evaluated pipeline quality with built-in evaluators

> **Platform Note:** This lab uses **Azure AI Foundry** (ai.azure.com). Do not use Azure Machine Learning studio.

---

## 📋 Prerequisites

- [ ] Active Azure subscription with Contributor access
- [ ] Azure AI Foundry Hub and Project created
- [ ] Azure AI Search resource (Standard tier S1+)
- [ ] Azure OpenAI resource with GPT-4o and text-embedding-3-large deployed
- [ ] Python 3.10+ installed locally
- [ ] VS Code with Python extension

### Environment Setup

```bash
# Create a virtual environment
python -m venv .venv

# Activate the virtual environment
# Windows:
.venv\Scripts\activate
# macOS/Linux:
source .venv/bin/activate

# Install required packages
pip install azure-ai-projects azure-ai-evaluation azure-search-documents azure-identity
```

### Azure Authentication

```bash
# Login to Azure
az login

# Verify your subscription
az account show --query "{name:name, id:id}" -o table
```

---

## 📁 Lab Files Structure

```
module-42-lab/
├── data/
│   ├── sample-doc-1.md
│   ├── sample-doc-2.md
│   ├── sample-doc-3.md
│   ├── sample-doc-4.txt
│   └── sample-doc-5.pdf
├── src/
│   ├── 01_create_vector_store.py
│   ├── 02_deploy_models.py
│   ├── 03_rag_agent.py
│   ├── 04_custom_rag.py
│   └── 05_evaluate_pipeline.py
├── eval/
│   └── test_dataset.jsonl
├── requirements.txt
└── .env.example
```

---

## Exercise 1: Set Up Data Sources & Vector Index (25 minutes)

### Step 1.1: Prepare Sample Documents

Create a `data/` directory and add at least 5 documents. For this lab, create Markdown files with technical content:

```markdown
<!-- data/sample-doc-1.md -->
# Azure AI Foundry Overview

Azure AI Foundry is a unified platform for building, evaluating, and deploying
AI solutions. It provides a comprehensive set of tools for working with foundation
models, building AI agents, and managing the complete AI lifecycle.

## Key Features
- Model Catalog with 1,800+ models from Microsoft, OpenAI, Meta, and more
- Built-in evaluation tools for measuring AI quality
- Agents API for building tool-augmented AI assistants
- Vector store management for RAG pipelines
- Enterprise-grade security with managed identity support
```

```markdown
<!-- data/sample-doc-2.md -->
# RAG Pipeline Best Practices

Retrieval-Augmented Generation (RAG) combines information retrieval with
language model generation to produce grounded, accurate responses.

## Chunking Best Practices
- Use 512–1024 token chunks for optimal retrieval
- Always configure 10–15% chunk overlap
- Choose semantic chunking for complex documents
- Use fixed-size chunking for uniform content like logs

## Search Best Practices
- Always use hybrid search (vector + keyword) for best recall
- Configure k=5 for most use cases
- Use re-ranking to improve precision after initial retrieval
```

```markdown
<!-- data/sample-doc-3.md -->
# Model Deployment in Azure AI Foundry

Azure AI Foundry supports multiple deployment types for foundation models.

## Deployment Types
- **Standard**: Shared capacity, pay-per-token, best for development
- **Provisioned Throughput**: Reserved capacity, consistent TPM, best for production
- **Global**: Multi-region deployment for highest availability
- **Serverless**: For non-OpenAI models (Meta, Mistral, etc.)

## Configuration Guidelines
- Set temperature to 0.1–0.3 for factual/RAG use cases
- Reserve 30% of context window for retrieved context
- Configure fallback models for high-availability scenarios
```

Create two more documents covering topics like evaluation metrics and Azure AI Search configuration.

### Step 1.2: Create the Vector Store

Create `src/01_create_vector_store.py`:

```python
import os
from azure.identity import DefaultAzureCredential
from azure.ai.projects import AIProjectClient
from azure.ai.projects.models import (
    VectorStoreDataSource,
    VectorStoreDataSourceAssetType,
    VectorStoreStaticChunkingStrategyRequest,
    VectorStoreStaticChunkingStrategyOptions,
)

# Configuration — replace with your values
SUBSCRIPTION_ID = os.environ.get("AZURE_SUBSCRIPTION_ID", "<your-subscription-id>")
RESOURCE_GROUP = os.environ.get("AZURE_RESOURCE_GROUP", "rg-ai-foundry-capstone")
PROJECT_NAME = os.environ.get("AI_FOUNDRY_PROJECT", "rag-pipeline-project")

# Initialize the project client
credential = DefaultAzureCredential()
project_client = AIProjectClient(
    credential=credential,
    subscription_id=SUBSCRIPTION_ID,
    resource_group_name=RESOURCE_GROUP,
    project_name=PROJECT_NAME,
)

print(f"✅ Connected to project: {PROJECT_NAME}")

# Upload files to the project (if using local files)
# In production, use data assets from Azure Blob Storage
uploaded_files = []
data_dir = os.path.join(os.path.dirname(__file__), "..", "data")

for filename in os.listdir(data_dir):
    filepath = os.path.join(data_dir, filename)
    if os.path.isfile(filepath):
        uploaded_file = project_client.agents.upload_file_and_poll(
            file_path=filepath,
            purpose="assistants",
        )
        uploaded_files.append(uploaded_file)
        print(f"  📄 Uploaded: {filename} → {uploaded_file.id}")

print(f"\n📁 Total files uploaded: {len(uploaded_files)}")

# Create vector store with optimal chunking configuration
vector_store = project_client.agents.create_vector_store_and_poll(
    name="capstone-knowledge-base",
    file_ids=[f.id for f in uploaded_files],
    chunking_strategy=VectorStoreStaticChunkingStrategyRequest(
        static=VectorStoreStaticChunkingStrategyOptions(
            max_chunk_size_tokens=800,
            chunk_overlap_tokens=100,  # ~12.5% overlap
        )
    ),
)

print(f"\n✅ Vector store created!")
print(f"  ID: {vector_store.id}")
print(f"  Name: {vector_store.name}")
print(f"  Status: {vector_store.status}")
print(f"  File counts: {vector_store.file_counts}")

# Save vector store ID for later exercises
with open(os.path.join(os.path.dirname(__file__), "..", "vector_store_id.txt"), "w") as f:
    f.write(vector_store.id)

print(f"\n💾 Vector store ID saved to vector_store_id.txt")
```

### Step 1.3: Run and Verify

```bash
python src/01_create_vector_store.py
```

**Expected Output:**
```
✅ Connected to project: rag-pipeline-project
  📄 Uploaded: sample-doc-1.md → file_abc123
  📄 Uploaded: sample-doc-2.md → file_def456
  ...
📁 Total files uploaded: 5
✅ Vector store created!
  ID: vs_xyz789
  Name: capstone-knowledge-base
  Status: completed
  File counts: {'completed': 5, 'in_progress': 0, 'failed': 0}
```

### ✅ Checkpoint 1

- [ ] All 5 documents uploaded successfully
- [ ] Vector store created with status "completed"
- [ ] Vector store ID saved for later use

---

## Exercise 2: Deploy & Configure Models (20 minutes)

### Step 2.1: Verify Model Deployments

Create `src/02_deploy_models.py`:

```python
import os
from azure.identity import DefaultAzureCredential
from azure.ai.projects import AIProjectClient

SUBSCRIPTION_ID = os.environ.get("AZURE_SUBSCRIPTION_ID", "<your-subscription-id>")
RESOURCE_GROUP = os.environ.get("AZURE_RESOURCE_GROUP", "rg-ai-foundry-capstone")
PROJECT_NAME = os.environ.get("AI_FOUNDRY_PROJECT", "rag-pipeline-project")

credential = DefaultAzureCredential()
project_client = AIProjectClient(
    credential=credential,
    subscription_id=SUBSCRIPTION_ID,
    resource_group_name=RESOURCE_GROUP,
    project_name=PROJECT_NAME,
)

# Test the chat completions model
print("🤖 Testing GPT-4o deployment...")
chat_client = project_client.inference.get_chat_completions_client()

response = chat_client.complete(
    model="gpt-4o",
    messages=[
        {"role": "system", "content": "You are a helpful assistant. Respond briefly."},
        {"role": "user", "content": "What is RAG in AI? One sentence."},
    ],
    temperature=0.3,
    max_tokens=100,
)
print(f"  ✅ GPT-4o response: {response.choices[0].message.content}")

# Test the embeddings model
print("\n🔢 Testing text-embedding-3-large deployment...")
embeddings_client = project_client.inference.get_embeddings_client()

embedding_response = embeddings_client.embed(
    model="text-embedding-3-large",
    input=["Test query for embeddings"],
)
embedding_dim = len(embedding_response.data[0].embedding)
print(f"  ✅ Embedding dimension: {embedding_dim}")
print(f"  ✅ Model: text-embedding-3-large")

print("\n🎉 All models verified successfully!")
```

### Step 2.2: Run and Verify

```bash
python src/02_deploy_models.py
```

### ✅ Checkpoint 2

- [ ] GPT-4o responds to test prompt
- [ ] text-embedding-3-large returns 3072-dimension embeddings
- [ ] Both models accessible via the project client

---

## Exercise 3: Build the RAG Agent (25 minutes)

### Step 3.1: Create the Agent-Based RAG Pipeline

Create `src/03_rag_agent.py`:

```python
import os
from azure.identity import DefaultAzureCredential
from azure.ai.projects import AIProjectClient
from azure.ai.projects.models import (
    FileSearchTool,
    FileSearchToolResource,
)

SUBSCRIPTION_ID = os.environ.get("AZURE_SUBSCRIPTION_ID", "<your-subscription-id>")
RESOURCE_GROUP = os.environ.get("AZURE_RESOURCE_GROUP", "rg-ai-foundry-capstone")
PROJECT_NAME = os.environ.get("AI_FOUNDRY_PROJECT", "rag-pipeline-project")

credential = DefaultAzureCredential()
project_client = AIProjectClient(
    credential=credential,
    subscription_id=SUBSCRIPTION_ID,
    resource_group_name=RESOURCE_GROUP,
    project_name=PROJECT_NAME,
)

# Read the vector store ID from Exercise 1
vs_id_path = os.path.join(os.path.dirname(__file__), "..", "vector_store_id.txt")
with open(vs_id_path, "r") as f:
    vector_store_id = f.read().strip()

print(f"📦 Using vector store: {vector_store_id}")

# Create the RAG agent
agent = project_client.agents.create_agent(
    model="gpt-4o",
    name="rag-pipeline-agent",
    instructions="""You are a knowledgeable technical assistant that answers questions
using ONLY the provided knowledge base. Follow these rules:
1. Always base your answer on the retrieved documents
2. Cite your sources using [Source: filename] format
3. If the answer is not in the knowledge base, say "I don't have enough information"
4. Be concise but thorough
5. Use bullet points for lists""",
    tools=FileSearchTool().definitions,
    tool_resources={
        "file_search": FileSearchToolResource(
            vector_store_ids=[vector_store_id]
        )
    },
)
print(f"✅ Agent created: {agent.id}")

# Test queries
test_queries = [
    "What are the best practices for chunking in a RAG pipeline?",
    "What deployment types are available in Azure AI Foundry?",
    "How should I configure temperature for RAG use cases?",
]

for i, query in enumerate(test_queries, 1):
    print(f"\n{'='*60}")
    print(f"Query {i}: {query}")
    print(f"{'='*60}")

    # Create a thread for each query
    thread = project_client.agents.create_thread()
    project_client.agents.create_message(
        thread_id=thread.id,
        role="user",
        content=query,
    )

    # Run the agent
    run = project_client.agents.create_and_process_run(
        thread_id=thread.id,
        agent_id=agent.id,
    )

    # Get the response
    messages = project_client.agents.list_messages(thread_id=thread.id)
    for msg in messages:
        if msg.role == "assistant":
            for content in msg.content:
                print(f"\n📝 Response:\n{content.text.value}")
                if content.text.annotations:
                    print(f"\n📎 Citations:")
                    for ann in content.text.annotations:
                        print(f"  - {ann.file_citation}")

    # Clean up thread
    project_client.agents.delete_thread(thread.id)

# Clean up agent
project_client.agents.delete_agent(agent.id)
print(f"\n🧹 Cleaned up agent and threads")
```

### Step 3.2: Run and Verify

```bash
python src/03_rag_agent.py
```

### ✅ Checkpoint 3

- [ ] Agent created successfully with file_search tool
- [ ] All 3 test queries return grounded responses
- [ ] Responses include citations from the knowledge base
- [ ] Agent handles "not found" cases appropriately

---

## Exercise 4: Build the Custom RAG Pipeline (25 minutes)

### Step 4.1: Implement Custom Hybrid Search RAG

Create `src/04_custom_rag.py`:

```python
import os
from azure.identity import DefaultAzureCredential
from azure.ai.projects import AIProjectClient
from azure.search.documents import SearchClient
from azure.search.documents.models import VectorizableTextQuery

SUBSCRIPTION_ID = os.environ.get("AZURE_SUBSCRIPTION_ID", "<your-subscription-id>")
RESOURCE_GROUP = os.environ.get("AZURE_RESOURCE_GROUP", "rg-ai-foundry-capstone")
PROJECT_NAME = os.environ.get("AI_FOUNDRY_PROJECT", "rag-pipeline-project")
SEARCH_ENDPOINT = os.environ.get("AZURE_SEARCH_ENDPOINT", "https://<your-search>.search.windows.net")
SEARCH_INDEX = os.environ.get("AZURE_SEARCH_INDEX", "rag-index")

credential = DefaultAzureCredential()

# Initialize clients
project_client = AIProjectClient(
    credential=credential,
    subscription_id=SUBSCRIPTION_ID,
    resource_group_name=RESOURCE_GROUP,
    project_name=PROJECT_NAME,
)

search_client = SearchClient(
    endpoint=SEARCH_ENDPOINT,
    index_name=SEARCH_INDEX,
    credential=credential,
)

chat_client = project_client.inference.get_chat_completions_client()


def custom_rag_query(user_query: str) -> dict:
    """Execute a complete custom RAG pipeline."""

    # Step 1: Hybrid search (vector + keyword)
    print(f"  🔍 Searching for: {user_query[:50]}...")
    results = search_client.search(
        search_text=user_query,
        vector_queries=[
            VectorizableTextQuery(
                text=user_query,
                k_nearest_neighbors=5,
                fields="content_vector",
            )
        ],
        select=["title", "content", "source"],
        top=5,
    )

    # Step 2: Build context from search results
    context_chunks = []
    sources = []
    for result in results:
        source = result.get("source", "unknown")
        content = result.get("content", "")
        context_chunks.append(f"[Source: {source}]\n{content}")
        sources.append(source)

    context = "\n\n---\n\n".join(context_chunks)
    print(f"  📄 Retrieved {len(context_chunks)} chunks from {len(set(sources))} sources")

    # Step 3: Construct the augmented prompt
    system_prompt = f"""You are a technical assistant. Answer the user's question
using ONLY the context provided below. Follow these rules:
1. Base your answer strictly on the provided context
2. Cite sources using [Source: filename] format
3. If the context doesn't contain the answer, say "I don't have enough information"
4. Be concise and use bullet points where appropriate

CONTEXT:
{context}"""

    # Step 4: Generate the response
    print(f"  🤖 Generating response...")
    response = chat_client.complete(
        model="gpt-4o",
        messages=[
            {"role": "system", "content": system_prompt},
            {"role": "user", "content": user_query},
        ],
        temperature=0.3,
        max_tokens=800,
    )

    return {
        "query": user_query,
        "response": response.choices[0].message.content,
        "context": context,
        "sources": list(set(sources)),
        "chunks_retrieved": len(context_chunks),
    }


# Test the custom RAG pipeline
test_queries = [
    "What are the best practices for chunking in a RAG pipeline?",
    "What deployment types are available in Azure AI Foundry?",
    "How should I configure temperature for RAG use cases?",
]

results_log = []
for i, query in enumerate(test_queries, 1):
    print(f"\n{'='*60}")
    print(f"Query {i}: {query}")
    print(f"{'='*60}")

    result = custom_rag_query(query)
    results_log.append(result)

    print(f"\n📝 Response:\n{result['response']}")
    print(f"\n📎 Sources: {', '.join(result['sources'])}")

print(f"\n\n{'='*60}")
print(f"✅ Custom RAG pipeline completed: {len(results_log)} queries processed")
```

### Step 4.2: Run and Verify

```bash
python src/04_custom_rag.py
```

### ✅ Checkpoint 4

- [ ] Hybrid search returns relevant results
- [ ] Context is properly assembled from search results
- [ ] GPT-4o generates grounded responses with citations
- [ ] Custom pipeline handles all test queries

---

## Exercise 5: Evaluate the Pipeline (20 minutes)

### Step 5.1: Create the Evaluation Dataset

Create `eval/test_dataset.jsonl`:

```jsonl
{"query": "What is Azure AI Foundry?", "context": "Azure AI Foundry is a unified platform for building, evaluating, and deploying AI solutions.", "response": "Azure AI Foundry is a unified platform for building, evaluating, and deploying AI solutions. It provides tools for working with foundation models, building AI agents, and managing the AI lifecycle.", "ground_truth": "Azure AI Foundry is a unified platform for building, evaluating, and deploying AI solutions."}
{"query": "What chunking strategies are recommended for RAG?", "context": "Use 512-1024 token chunks for optimal retrieval. Always configure 10-15% chunk overlap. Choose semantic chunking for complex documents.", "response": "For RAG pipelines, use 512-1024 token chunks with 10-15% overlap. Semantic chunking is recommended for complex documents, while fixed-size chunking works well for uniform content.", "ground_truth": "Use 512-1024 token chunks with 10-15% overlap. Semantic chunking for complex docs, fixed-size for uniform content."}
{"query": "What deployment types does Azure AI Foundry support?", "context": "Azure AI Foundry supports Standard (shared capacity), Provisioned Throughput (reserved), Global (multi-region), and Serverless deployments.", "response": "Azure AI Foundry supports four deployment types: Standard for development, Provisioned Throughput for production, Global for high availability, and Serverless for non-OpenAI models.", "ground_truth": "Standard, Provisioned Throughput, Global, and Serverless deployment types."}
{"query": "What temperature should I use for RAG?", "context": "Set temperature to 0.1-0.3 for factual/RAG use cases.", "response": "For RAG use cases, set the temperature between 0.1 and 0.3 to ensure factual, grounded responses.", "ground_truth": "Temperature 0.1-0.3 for factual RAG use cases."}
{"query": "How does hybrid search work?", "context": "Hybrid search combines vector similarity with BM25 keyword matching for better recall. Always use hybrid search for production RAG.", "response": "Hybrid search combines vector similarity search with BM25 keyword matching. This approach improves recall by 10-15% compared to vector-only search and is recommended for production RAG pipelines.", "ground_truth": "Hybrid search combines vector and keyword search for 10-15% better recall."}
{"query": "What is the recommended chunk overlap?", "context": "Always configure 10-15% chunk overlap to preserve context across chunk boundaries.", "response": "The recommended chunk overlap is 10-15% of the chunk size. This preserves context across chunk boundaries and significantly improves retrieval accuracy.", "ground_truth": "10-15% chunk overlap is recommended."}
{"query": "What embedding model should I use?", "context": "text-embedding-3-large provides 3072 dimensions for best quality. text-embedding-3-small provides 1536 dimensions.", "response": "Use text-embedding-3-large (3072 dimensions) for best quality. text-embedding-3-small (1536 dimensions) is a good alternative for cost-sensitive scenarios.", "ground_truth": "text-embedding-3-large for best quality, text-embedding-3-small for cost efficiency."}
{"query": "What are the key features of Azure AI Foundry?", "context": "Key features include Model Catalog with 1800+ models, built-in evaluation tools, Agents API, vector store management, and enterprise security.", "response": "Azure AI Foundry's key features include a Model Catalog with 1,800+ models, built-in evaluation tools, the Agents API for building AI assistants, vector store management for RAG, and enterprise-grade security with managed identities.", "ground_truth": "Model Catalog, evaluation tools, Agents API, vector stores, enterprise security."}
{"query": "How do I use the file_search tool?", "context": "The file_search tool enables automatic retrieval from vector stores. Create an agent with FileSearchTool and FileSearchToolResource pointing to your vector store.", "response": "To use file_search, create an agent with FileSearchTool().definitions as tools and FileSearchToolResource(vector_store_ids=[vs_id]) as tool_resources. The agent automatically handles retrieval, re-ranking, and citation.", "ground_truth": "Create agent with FileSearchTool and FileSearchToolResource pointing to vector store IDs."}
{"query": "What evaluation metrics should I track for RAG?", "context": "Key metrics include Groundedness (grounded in context), Relevance (answers the question), Coherence (well-structured), Fluency (readable), and Retrieval Score.", "response": "Track these RAG evaluation metrics: Groundedness (response grounded in context), Relevance (answers the query), Coherence (logical structure), Fluency (natural language), and Retrieval Score (document relevance). Target scores above 4.0/5.0.", "ground_truth": "Groundedness, Relevance, Coherence, Fluency, and Retrieval Score. Target above 4.0."}
```

### Step 5.2: Run the Evaluation

Create `src/05_evaluate_pipeline.py`:

```python
import os
from azure.identity import DefaultAzureCredential
from azure.ai.projects import AIProjectClient
from azure.ai.evaluation import (
    GroundednessEvaluator,
    RelevanceEvaluator,
    CoherenceEvaluator,
    FluencyEvaluator,
    evaluate,
)

SUBSCRIPTION_ID = os.environ.get("AZURE_SUBSCRIPTION_ID", "<your-subscription-id>")
RESOURCE_GROUP = os.environ.get("AZURE_RESOURCE_GROUP", "rg-ai-foundry-capstone")
PROJECT_NAME = os.environ.get("AI_FOUNDRY_PROJECT", "rag-pipeline-project")

credential = DefaultAzureCredential()
project_client = AIProjectClient(
    credential=credential,
    subscription_id=SUBSCRIPTION_ID,
    resource_group_name=RESOURCE_GROUP,
    project_name=PROJECT_NAME,
)

# Get the model configuration for evaluators
model_config = project_client.scope

# Initialize evaluators
groundedness_eval = GroundednessEvaluator(model_config)
relevance_eval = RelevanceEvaluator(model_config)
coherence_eval = CoherenceEvaluator(model_config)
fluency_eval = FluencyEvaluator(model_config)

# Path to evaluation dataset
eval_data_path = os.path.join(
    os.path.dirname(__file__), "..", "eval", "test_dataset.jsonl"
)

print("🧪 Running pipeline evaluation...")
print(f"📄 Dataset: {eval_data_path}\n")

# Run batch evaluation
eval_results = evaluate(
    data=eval_data_path,
    evaluators={
        "groundedness": groundedness_eval,
        "relevance": relevance_eval,
        "coherence": coherence_eval,
        "fluency": fluency_eval,
    },
    evaluator_config={
        "default": {
            "query": "${data.query}",
            "context": "${data.context}",
            "response": "${data.response}",
        }
    },
    azure_ai_project=project_client.scope,
)

# Display results
print("=" * 60)
print("📊 EVALUATION RESULTS")
print("=" * 60)

quality_gate_passed = True
thresholds = {
    "groundedness": 4.0,
    "relevance": 4.0,
    "coherence": 4.0,
    "fluency": 4.0,
}

for metric, score in eval_results["metrics"].items():
    threshold = thresholds.get(metric.split(".")[0], 0)
    status = "✅" if score >= threshold else "❌"
    if score < threshold:
        quality_gate_passed = False
    print(f"  {status} {metric}: {score:.2f} (threshold: {threshold})")

print(f"\n{'='*60}")
if quality_gate_passed:
    print("🎉 ALL QUALITY GATES PASSED!")
else:
    print("⚠️ Some quality gates failed. Review and improve your pipeline.")
print(f"{'='*60}")
```

### Step 5.3: Run and Verify

```bash
python src/05_evaluate_pipeline.py
```

### ✅ Checkpoint 5

- [ ] Evaluation runs without errors
- [ ] All metrics display with scores
- [ ] Groundedness ≥ 4.0
- [ ] Relevance ≥ 4.0
- [ ] Coherence ≥ 4.0
- [ ] Fluency ≥ 4.0

---

## 🏆 Lab Completion Checklist

- [ ] **Exercise 1:** Vector store created with 5+ documents and semantic chunking
- [ ] **Exercise 2:** Both GPT-4o and text-embedding-3-large verified
- [ ] **Exercise 3:** RAG agent returns grounded responses with citations
- [ ] **Exercise 4:** Custom hybrid search RAG pipeline working
- [ ] **Exercise 5:** All evaluation metrics ≥ 4.0

---

## 🧹 Cleanup

After completing the lab, clean up resources to avoid charges:

```python
# Delete the vector store
project_client.agents.delete_vector_store(vector_store_id)

# Delete uploaded files
for file_id in uploaded_file_ids:
    project_client.agents.delete_file(file_id)

print("🧹 Cleanup complete!")
```

> **Note:** Keep your Azure AI Foundry project and model deployments if you plan to continue with Module 43.

---

## 🔧 Troubleshooting

| Issue | Solution |
|---|---|
| `AuthenticationError` | Run `az login` and verify subscription with `az account show` |
| Vector store stuck "in_progress" | Wait 2–3 minutes; large files take longer to process |
| Search returns no results | Verify the index name and that documents were indexed successfully |
| Model returns generic responses | Lower temperature to 0.1; ensure context is being passed correctly |
| Evaluation scores low | Check that context matches the response; improve system prompt |

---

## 📚 References

- [Azure AI Foundry Agents Documentation](https://learn.microsoft.com/azure/ai-services/agents/)
- [Azure AI Search Vector Search](https://learn.microsoft.com/azure/search/vector-search-overview)
- [Azure AI Evaluation SDK](https://learn.microsoft.com/azure/ai-studio/how-to/evaluate-generative-ai-app)
- [RAG Best Practices](https://learn.microsoft.com/azure/search/retrieval-augmented-generation-overview)

---

*Module 42 — Azure AI Foundry: Zero to Hero | Arc 9: Capstone & Mastery | April 2026*
