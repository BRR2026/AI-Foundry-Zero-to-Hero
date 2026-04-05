# Hands-On Lab: Deploy & Integrate with Agents/Copilot

## Module 43 — Azure AI Foundry: Zero to Hero

**ARC 9: Capstone & Mastery** · Estimated Duration: 3–4 hours

---

## 🎯 Lab Objectives

In this capstone lab, you will build and deploy a fully integrated AI agent solution that:

1. Uses Azure AI Foundry Agent Service with a RAG pipeline
2. Includes custom function tools for real-world actions
3. Is accessible through a FastAPI endpoint
4. Integrates with Microsoft Copilot Studio
5. Is published to Microsoft Teams
6. Is protected by Azure API Management
7. Is validated with end-to-end tests and AI quality evaluation

---

## 📋 Prerequisites

- [ ] Azure subscription with Contributor role
- [ ] Azure AI Foundry project with GPT-4o deployment
- [ ] Azure AI Search index populated with sample data (from Modules 28–35)
- [ ] Python 3.10+ installed
- [ ] Azure CLI installed and authenticated (`az login`)
- [ ] VS Code with Python and Azure extensions
- [ ] Microsoft 365 developer tenant (for Teams testing)
- [ ] Copilot Studio access

---

## 🛠️ Environment Setup

### Step 1: Create the Project Structure

```bash
mkdir agent-integration-lab
cd agent-integration-lab
python -m venv .venv

# Activate virtual environment
# Windows:
.venv\Scripts\activate
# macOS/Linux:
source .venv/bin/activate
```

### Step 2: Install Dependencies

```bash
pip install azure-ai-projects azure-ai-evaluation azure-identity \
    fastapi uvicorn httpx pytest pytest-asyncio python-dotenv
```

### Step 3: Configure Environment Variables

Create a `.env` file in the project root:

```env
AZURE_AI_FOUNDRY_ENDPOINT=https://<your-project>.services.ai.azure.com
AZURE_AI_SEARCH_CONNECTION_ID=<your-search-connection-id>
AZURE_AI_SEARCH_INDEX_NAME=product-knowledge-base
AZURE_OPENAI_MODEL=gpt-4o
```

> ⚠️ **Never commit `.env` files to source control.** Add `.env` to your `.gitignore`.

---

## Exercise 1: Build the AI Agent (45 min)

### 1.1 Create the Agent Service Module

Create `agent_service.py`:

```python
"""Agent service module for Azure AI Foundry integration."""

import os
from dotenv import load_dotenv
from azure.ai.projects import AIProjectClient
from azure.ai.projects.models import (
    AzureAISearchTool,
    AzureAISearchQueryType,
    FunctionTool,
    ToolSet,
)
from azure.identity import DefaultAzureCredential

load_dotenv()

# Initialize the AI Foundry client
client = AIProjectClient(
    credential=DefaultAzureCredential(),
    endpoint=os.getenv("AZURE_AI_FOUNDRY_ENDPOINT"),
)


# ── Custom Function Tools ──────────────────────────────

def create_support_ticket(title: str, priority: str, description: str) -> dict:
    """Create a support ticket in the ticketing system.

    Args:
        title: Brief title for the support ticket
        priority: Priority level (low, medium, high, critical)
        description: Detailed description of the issue
    """
    # In production, this would call your ticketing API
    import uuid
    ticket_id = f"TKT-{uuid.uuid4().hex[:8].upper()}"
    return {
        "ticket_id": ticket_id,
        "title": title,
        "priority": priority,
        "status": "created",
        "message": f"Support ticket {ticket_id} created successfully.",
    }


def check_order_status(order_id: str) -> dict:
    """Look up the current status of a customer order.

    Args:
        order_id: The unique order identifier (e.g., ORD-1234)
    """
    # In production, this would query your order database
    return {
        "order_id": order_id,
        "status": "shipped",
        "tracking_number": "1Z999AA10123456784",
        "estimated_delivery": "2026-04-20",
    }


# ── Agent Setup ────────────────────────────────────────

def create_agent():
    """Create and return the AI agent with RAG and function tools."""
    # Configure RAG tool
    search_tool = AzureAISearchTool(
        index_connection_id=os.getenv("AZURE_AI_SEARCH_CONNECTION_ID"),
        index_name=os.getenv("AZURE_AI_SEARCH_INDEX_NAME"),
        query_type=AzureAISearchQueryType.SEMANTIC,
        top_k=5,
    )

    # Configure function tools
    functions = FunctionTool(
        functions=[create_support_ticket, check_order_status]
    )

    # Compose toolset
    toolset = ToolSet()
    toolset.add(search_tool)
    toolset.add(functions)

    # Create the agent
    agent = client.agents.create_agent(
        model=os.getenv("AZURE_OPENAI_MODEL", "gpt-4o"),
        name="product-support-agent",
        instructions="""You are a helpful product support specialist for Contoso.

Your capabilities:
1. Search the product knowledge base to answer customer questions
2. Create support tickets when issues cannot be resolved immediately
3. Check order status for customers

Guidelines:
- Always search the knowledge base before answering product questions
- Cite your sources when providing information from the knowledge base
- If you cannot resolve an issue, offer to create a support ticket
- Be professional, friendly, and concise
- If you don't know something, say so honestly""",
        toolset=toolset,
    )

    return agent


# ── Conversation Handler ──────────────────────────────

# Thread cache (in production, use Redis or a database)
_threads: dict[str, str] = {}


async def invoke_agent(message: str, session_id: str | None = None) -> dict:
    """Invoke the agent with a user message."""
    # Get or create agent
    agent = create_agent()

    # Get or create thread
    if session_id and session_id in _threads:
        thread_id = _threads[session_id]
    else:
        thread = client.agents.create_thread()
        thread_id = thread.id
        if session_id:
            _threads[session_id] = thread_id

    # Post the user message
    client.agents.create_message(
        thread_id=thread_id,
        role="user",
        content=message,
    )

    # Run the agent
    run = client.agents.create_and_process_run(
        thread_id=thread_id,
        agent_id=agent.id,
    )

    # Extract the response
    messages = client.agents.list_messages(thread_id=thread_id)
    reply = ""
    citations = []

    for msg in messages.data:
        if msg.role == "assistant":
            for content_block in msg.content:
                if hasattr(content_block, 'text'):
                    reply = content_block.text.value
                    # Extract citations if available
                    if hasattr(content_block.text, 'annotations'):
                        for ann in content_block.text.annotations:
                            citations.append({
                                "text": ann.text if hasattr(ann, 'text') else "",
                                "url": ann.url if hasattr(ann, 'url') else "",
                            })
            break

    return {
        "reply": reply,
        "session_id": session_id or thread_id,
        "citations": citations,
        "thread_id": thread_id,
    }
```

### 1.2 Test the Agent Locally

Create `test_agent_local.py`:

```python
"""Quick local test for the agent service."""

import asyncio
from agent_service import invoke_agent


async def main():
    # Test 1: Simple knowledge query
    print("=" * 60)
    print("Test 1: Knowledge query")
    result = await invoke_agent("What is your return policy?")
    print(f"Reply: {result['reply'][:200]}...")
    print(f"Session: {result['session_id']}")

    # Test 2: Order status check
    print("\n" + "=" * 60)
    print("Test 2: Order status")
    result = await invoke_agent("Check order ORD-9921")
    print(f"Reply: {result['reply'][:200]}...")

    # Test 3: Multi-turn conversation
    print("\n" + "=" * 60)
    print("Test 3: Multi-turn")
    r1 = await invoke_agent("I'm having trouble with Product X")
    session = r1["session_id"]
    r2 = await invoke_agent("Can you create a ticket for this?", session_id=session)
    print(f"Reply: {r2['reply'][:200]}...")


if __name__ == "__main__":
    asyncio.run(main())
```

Run: `python test_agent_local.py`

✅ **Checkpoint:** Your agent responds to queries, checks order status, and can create tickets.

---

## Exercise 2: Deploy the API Endpoint (30 min)

### 2.1 Create the FastAPI Application

Create `app.py`:

```python
"""FastAPI application for the AI Agent API."""

from fastapi import FastAPI, HTTPException
from fastapi.middleware.cors import CORSMiddleware
from pydantic import BaseModel
from agent_service import invoke_agent

app = FastAPI(
    title="AI Agent API",
    description="Product support agent powered by Azure AI Foundry",
    version="1.0.0",
)

# CORS configuration
app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],  # Restrict in production
    allow_methods=["*"],
    allow_headers=["*"],
)


# ── Request / Response Models ──────────────────────────

class ChatRequest(BaseModel):
    message: str
    session_id: str | None = None

    model_config = {
        "json_schema_extra": {
            "examples": [
                {
                    "message": "What is your return policy?",
                    "session_id": None,
                }
            ]
        }
    }


class ChatResponse(BaseModel):
    reply: str
    session_id: str
    citations: list[dict] = []


class HealthResponse(BaseModel):
    status: str
    service: str
    version: str


# ── Endpoints ──────────────────────────────────────────

@app.get("/health", response_model=HealthResponse)
async def health_check():
    """Health check endpoint."""
    return HealthResponse(
        status="healthy",
        service="ai-agent-api",
        version="1.0.0",
    )


@app.post("/api/chat", response_model=ChatResponse)
async def chat(request: ChatRequest):
    """Send a message to the AI agent."""
    if not request.message or not request.message.strip():
        raise HTTPException(status_code=400, detail="'message' is required")

    try:
        result = await invoke_agent(
            message=request.message,
            session_id=request.session_id,
        )
        return ChatResponse(
            reply=result["reply"],
            session_id=result["session_id"],
            citations=result.get("citations", []),
        )
    except Exception as e:
        raise HTTPException(status_code=500, detail=f"Agent error: {str(e)}")


@app.get("/")
async def root():
    """API root — redirect to docs."""
    return {"message": "AI Agent API", "docs": "/docs"}
```

### 2.2 Run Locally

```bash
uvicorn app:app --reload --port 8000
```

### 2.3 Test the API

```bash
# Health check
curl http://localhost:8000/health

# Chat
curl -X POST http://localhost:8000/api/chat \
  -H "Content-Type: application/json" \
  -d '{"message": "What is your return policy?"}'
```

### 2.4 Deploy to Azure

Option A — Azure Container Apps:
```bash
az containerapp up \
  --name agent-api \
  --resource-group rg-agent-lab \
  --source . \
  --env-vars AZURE_AI_FOUNDRY_ENDPOINT=<endpoint> \
             AZURE_AI_SEARCH_CONNECTION_ID=<conn-id> \
             AZURE_AI_SEARCH_INDEX_NAME=product-knowledge-base
```

Option B — Azure App Service:
```bash
az webapp up \
  --name agent-api-<unique> \
  --resource-group rg-agent-lab \
  --runtime PYTHON:3.11 \
  --sku B1
```

✅ **Checkpoint:** Your API is deployed and accessible via a public URL.

---

## Exercise 3: Copilot Studio Integration (45 min)

### 3.1 Register the Custom Connector

1. Open [Copilot Studio](https://copilotstudio.microsoft.com)
2. Navigate to **Settings → Custom connectors**
3. Click **+ New custom connector**
4. Select **Import from URL** and enter your deployed API's OpenAPI spec URL:
   ```
   https://your-agent-api.azurewebsites.net/openapi.json
   ```
5. Configure the connector:
   - **Name:** AI Agent Connector
   - **Host:** `your-agent-api.azurewebsites.net`
   - **Base URL:** `/`
6. Under **Security**, select **API Key** or **OAuth 2.0** (for Entra ID)
7. Click **Create connector**

### 3.2 Create a Conversation Topic

1. In your Copilot Studio bot, go to **Topics**
2. Click **+ New topic → From blank**
3. Name it: "Product Support Agent"
4. Add trigger phrases:
   - "I need help with a product"
   - "Check my order status"
   - "I have a question about..."
   - "Create a support ticket"
5. Add an **Action** node:
   - Select your custom connector
   - Choose the **chat** action
   - Map `message` to the user's input
6. Add a **Message** node to display the agent's reply
7. Save and test

### 3.3 Test in Copilot Studio

1. Open the **Test bot** pane
2. Try these conversations:
   - "What is your return policy?"
   - "Check order ORD-1234"
   - "I need to create a support ticket for a defective product"
3. Verify the agent responds correctly via the custom connector

✅ **Checkpoint:** Your agent is accessible through Copilot Studio.

---

## Exercise 4: Teams Deployment (30 min)

### 4.1 Publish to Teams from Copilot Studio

1. In Copilot Studio, go to **Channels**
2. Select **Microsoft Teams**
3. Configure:
   - **Bot name:** Product Support Agent
   - **Short description:** AI-powered product support assistant
   - **Icon:** Upload a 192x192 icon
4. Click **Publish**
5. Select **Availability**:
   - For testing: **Show to my teammates**
   - For production: **Submit for admin approval**

### 4.2 Install in Teams

1. Open Microsoft Teams
2. Go to **Apps → Built for your org**
3. Find "Product Support Agent" and click **Add**
4. Start a conversation with the bot

### 4.3 Test in Teams

Try these conversations in the Teams chat:
1. "Hi, I need help with my recent purchase"
2. "What's the status of order ORD-5678?"
3. "Can you help me understand the warranty coverage?"
4. "Please create a ticket — my product arrived damaged"

✅ **Checkpoint:** Your agent is live in Microsoft Teams.

---

## Exercise 5: API Management Setup (30 min)

### 5.1 Create APIM Instance

```bash
# Create API Management instance (Developer tier for lab)
az apim create \
  --name apim-agent-lab-<unique> \
  --resource-group rg-agent-lab \
  --publisher-email admin@contoso.com \
  --publisher-name "Contoso" \
  --sku-name Developer \
  --location eastus
```

> ⚠️ APIM provisioning can take 20–40 minutes. Continue with Exercise 6 while waiting.

### 5.2 Import the Agent API

```bash
# Import API from OpenAPI spec
az apim api import \
  --resource-group rg-agent-lab \
  --service-name apim-agent-lab-<unique> \
  --api-id ai-agent-api \
  --path agent \
  --specification-format OpenApi \
  --specification-url https://your-agent-api.azurewebsites.net/openapi.json
```

### 5.3 Add Rate Limiting Policy

In the Azure portal, navigate to your APIM instance → APIs → AI Agent API → All operations → Inbound processing:

```xml
<policies>
    <inbound>
        <base />
        <rate-limit calls="100" renewal-period="60" />
        <set-header name="X-Request-ID" exists-action="skip">
            <value>@(context.RequestId.ToString())</value>
        </set-header>
    </inbound>
    <backend>
        <base />
    </backend>
    <outbound>
        <base />
    </outbound>
</policies>
```

### 5.4 Test via APIM

```bash
# Get subscription key
az apim subscription show \
  --resource-group rg-agent-lab \
  --service-name apim-agent-lab-<unique> \
  --subscription-id master \
  --query primaryKey -o tsv

# Test through APIM
curl -X POST https://apim-agent-lab-<unique>.azure-api.net/agent/api/chat \
  -H "Content-Type: application/json" \
  -H "Ocp-Apim-Subscription-Key: <your-key>" \
  -d '{"message": "What is your return policy?"}'
```

✅ **Checkpoint:** Your API is accessible through APIM with rate limiting.

---

## Exercise 6: End-to-End Testing (45 min)

### 6.1 Create the Test Suite

Create `tests/test_e2e.py`:

```python
"""End-to-end tests for the deployed AI agent."""

import os
import pytest
import httpx

BASE_URL = os.getenv("AGENT_API_URL", "http://localhost:8000")


@pytest.fixture
def client():
    """Create an HTTP client for testing."""
    return httpx.Client(base_url=BASE_URL, timeout=60)


class TestHealthCheck:
    """Health and readiness tests."""

    def test_health_endpoint_returns_200(self, client):
        response = client.get("/health")
        assert response.status_code == 200

    def test_health_returns_healthy_status(self, client):
        data = client.get("/health").json()
        assert data["status"] == "healthy"
        assert data["service"] == "ai-agent-api"


class TestChatEndpoint:
    """Chat functionality tests."""

    def test_simple_query_returns_response(self, client):
        response = client.post("/api/chat", json={
            "message": "What is your return policy?"
        })
        assert response.status_code == 200
        data = response.json()
        assert "reply" in data
        assert len(data["reply"]) > 20

    def test_response_includes_session_id(self, client):
        response = client.post("/api/chat", json={
            "message": "Hello, I need help."
        })
        data = response.json()
        assert "session_id" in data
        assert data["session_id"] is not None

    def test_multi_turn_conversation(self, client):
        # First turn
        r1 = client.post("/api/chat", json={
            "message": "I need help with Product X."
        })
        session_id = r1.json()["session_id"]

        # Second turn (same session)
        r2 = client.post("/api/chat", json={
            "message": "What is its warranty period?",
            "session_id": session_id,
        })
        assert r2.status_code == 200
        assert len(r2.json()["reply"]) > 10

    def test_order_status_tool_invocation(self, client):
        response = client.post("/api/chat", json={
            "message": "Check the status of order ORD-1234"
        })
        data = response.json()
        assert response.status_code == 200
        # Agent should reference the order
        assert "ORD-1234" in data["reply"] or "order" in data["reply"].lower()

    def test_ticket_creation_tool(self, client):
        response = client.post("/api/chat", json={
            "message": "Please create a high-priority support ticket. "
                       "Title: 'Defective widget'. "
                       "Description: 'Widget stopped working after 2 days.'"
        })
        data = response.json()
        assert response.status_code == 200
        assert "ticket" in data["reply"].lower() or "TKT" in data["reply"]


class TestErrorHandling:
    """Error handling and edge case tests."""

    def test_empty_message_returns_400(self, client):
        response = client.post("/api/chat", json={"message": ""})
        assert response.status_code == 400

    def test_missing_message_returns_400(self, client):
        response = client.post("/api/chat", json={})
        assert response.status_code == 422  # Pydantic validation error

    def test_invalid_content_type(self, client):
        response = client.post(
            "/api/chat",
            content="not json",
            headers={"Content-Type": "text/plain"},
        )
        assert response.status_code in [400, 415, 422]
```

### 6.2 Run the Tests

```bash
# Against local server
AGENT_API_URL=http://localhost:8000 pytest tests/test_e2e.py -v

# Against deployed server
AGENT_API_URL=https://your-agent-api.azurewebsites.net pytest tests/test_e2e.py -v
```

✅ **Checkpoint:** All E2E tests pass against your deployed agent.

---

## Exercise 7: AI Quality Evaluation (30 min)

### 7.1 Create the Evaluation Script

Create `evaluation/evaluate_agent.py`:

```python
"""AI quality evaluation for the deployed agent."""

import os
import asyncio
from dotenv import load_dotenv
from azure.ai.evaluation import (
    GroundednessEvaluator,
    RelevanceEvaluator,
    CoherenceEvaluator,
    FluencyEvaluator,
    evaluate,
)

load_dotenv()


# Test dataset with expected contexts
TEST_DATA = [
    {
        "query": "What is the return policy for electronics?",
        "context": "Our return policy allows returns within 30 days of purchase for electronics. Items must be in original packaging with receipt.",
    },
    {
        "query": "How do I track my order?",
        "context": "You can track your order by visiting our website and entering your order number, or by contacting support with your order ID.",
    },
    {
        "query": "What warranty does Product X come with?",
        "context": "Product X comes with a 2-year manufacturer warranty covering defects in materials and workmanship.",
    },
    {
        "query": "Can I cancel my subscription?",
        "context": "Subscriptions can be cancelled at any time from your account settings. Refunds are prorated based on remaining time.",
    },
    {
        "query": "What payment methods do you accept?",
        "context": "We accept Visa, Mastercard, American Express, PayPal, and bank transfers for all purchases.",
    },
]


async def collect_responses():
    """Collect agent responses for evaluation."""
    from agent_service import invoke_agent

    results = []
    for item in TEST_DATA:
        response = await invoke_agent(item["query"])
        results.append({
            "query": item["query"],
            "context": item["context"],
            "response": response["reply"],
        })
    return results


def run_evaluation():
    """Run AI quality evaluation."""
    # Collect responses
    eval_data = asyncio.run(collect_responses())

    # Configure model for evaluators
    model_config = {
        "azure_endpoint": os.getenv("AZURE_AI_FOUNDRY_ENDPOINT"),
        "azure_deployment": os.getenv("AZURE_OPENAI_MODEL", "gpt-4o"),
    }

    # Run evaluation
    results = evaluate(
        data=eval_data,
        evaluators={
            "groundedness": GroundednessEvaluator(model_config),
            "relevance": RelevanceEvaluator(model_config),
            "coherence": CoherenceEvaluator(model_config),
            "fluency": FluencyEvaluator(model_config),
        },
    )

    # Print results
    print("\n" + "=" * 60)
    print("AI QUALITY EVALUATION RESULTS")
    print("=" * 60)

    metrics = results.get("metrics", {})
    for metric, score in metrics.items():
        status = "✅" if score >= 4.0 else "❌"
        print(f"  {status} {metric}: {score:.2f} / 5.00")

    print("\n" + "-" * 60)
    all_pass = all(v >= 4.0 for v in metrics.values())
    print(f"  Overall: {'✅ PASS' if all_pass else '❌ FAIL'}")
    print("=" * 60)

    return results


if __name__ == "__main__":
    run_evaluation()
```

### 7.2 Run the Evaluation

```bash
python evaluation/evaluate_agent.py
```

Expected output:
```
============================================================
AI QUALITY EVALUATION RESULTS
============================================================
  ✅ groundedness: 4.40 / 5.00
  ✅ relevance: 4.60 / 5.00
  ✅ coherence: 4.80 / 5.00
  ✅ fluency: 4.70 / 5.00

------------------------------------------------------------
  Overall: ✅ PASS
============================================================
```

✅ **Checkpoint:** All quality metrics are ≥ 4.0/5.0.

---

## Exercise 8: Documentation & Demo (30 min)

### 8.1 Create a README

Document your solution in a `README.md` including:
- Architecture diagram (text-based is fine)
- Setup instructions
- API documentation
- Test results summary
- Evaluation scores

### 8.2 Record a Demo

Record a 3-minute demo showing:
1. **API access** — curl/Postman calling the agent API
2. **Teams chat** — Conversation with the agent in Teams
3. **Copilot Studio** — Agent responding through Copilot Studio
4. **Test results** — E2E tests passing
5. **Evaluation scores** — Quality metrics above threshold

---

## 🏁 Lab Completion Checklist

| # | Task | Status |
|---|---|---|
| 1 | Agent with RAG + 2 custom tools created | ☐ |
| 2 | FastAPI endpoint deployed to Azure | ☐ |
| 3 | Copilot Studio custom connector configured | ☐ |
| 4 | Agent published to Microsoft Teams | ☐ |
| 5 | APIM configured with rate limiting | ☐ |
| 6 | 8+ E2E tests passing | ☐ |
| 7 | All quality scores ≥ 4.0/5.0 | ☐ |
| 8 | README and demo video completed | ☐ |

---

## 🧹 Cleanup

When you're done, clean up Azure resources to avoid charges:

```bash
# Delete the resource group (removes all resources)
az group delete --name rg-agent-lab --yes --no-wait
```

---

## 🎉 Congratulations!

You've completed the Module 43 hands-on lab! You've built a production-grade AI agent solution
that integrates across multiple channels and is validated with comprehensive testing.

**Next:** Module 44 — Performance Optimization & Cost Management

---

*Module 43 of 45 — Azure AI Foundry: Zero to Hero*
*ARC 9: Capstone & Mastery*
*© 2026 Azure AI Foundry Training Series*
