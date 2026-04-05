# Hands-On Lab: Publish Your RAG Agent as a Copilot Studio Bot

> **Module 23** — Connecting AI Foundry to Copilot Studio  
> **Azure AI Foundry: Zero to Hero** | Arc 5: Orchestration & Integration  
> **Duration:** 45–60 minutes | **Difficulty:** Intermediate

---

## Lab Overview

In this lab, you will take a RAG (Retrieval-Augmented Generation) agent deployed on Azure AI Foundry and publish it as a fully functional bot in Microsoft Copilot Studio. By the end, you'll have a working conversational agent accessible through Microsoft Teams that leverages your enterprise knowledge base.

### What You'll Build

```
┌────────────────────────────────────────────────────────────┐
│                    END-TO-END SOLUTION                      │
│                                                            │
│  Microsoft Teams → Copilot Studio → Plugin → AI Foundry   │
│       (User)        (Conversation)   (API)    (RAG Agent)  │
│                                                            │
│  User asks a question → Copilot Studio routes it →         │
│  Plugin calls AI Foundry → RAG agent searches knowledge    │
│  base → Grounded answer with citations returned            │
└────────────────────────────────────────────────────────────┘
```

### Learning Objectives

- Verify and test an AI Foundry inference endpoint
- Create an OpenAPI 3.0 specification for an AI Foundry model
- Register a custom plugin in Copilot Studio
- Build a conversation topic with plugin invocation and response formatting
- Enable generative orchestration for intelligent routing
- Deploy a copilot to Microsoft Teams

---

## Prerequisites

Before starting this lab, ensure you have:

- [ ] **Azure AI Foundry project** with a deployed RAG agent (from Module 14 or later)
- [ ] **AI Foundry endpoint URL** and **API key** for the deployed model
- [ ] **Microsoft Copilot Studio** access (trial or licensed — https://copilotstudio.microsoft.com)
- [ ] **Microsoft Teams** desktop or web client
- [ ] **Azure CLI** installed and authenticated (`az login`)
- [ ] A text editor (VS Code recommended)

### Gather Your Endpoint Information

Before starting, collect this information from your AI Foundry project:

| Parameter | Where to Find | Your Value |
|---|---|---|
| Endpoint URL | AI Foundry → Project → Deployments → Endpoint | `https://__________.inference.ml.azure.com` |
| API Key | AI Foundry → Project → Deployments → Keys | `__________________________` |
| Model Name | AI Foundry → Project → Deployments | `__________________________` |
| Input Format | Model documentation or test panel | JSON schema |
| Output Format | Model documentation or test panel | JSON schema |

---

## Exercise 1: Verify Your AI Foundry Endpoint (5 minutes)

### Step 1.1: Test the Endpoint with cURL

Open a terminal and test your AI Foundry RAG agent endpoint:

```bash
# Replace with your actual endpoint URL and API key
curl -X POST "https://<your-project>.<region>.inference.ml.azure.com/score" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer <your-api-key>" \
  -d '{
    "question": "What is the company leave policy?",
    "chat_history": []
  }'
```

### Step 1.2: Verify the Response Format

You should receive a JSON response similar to:

```json
{
  "answer": "According to our employee handbook, the company offers...",
  "citations": [
    {
      "title": "Employee Handbook - Leave Policies",
      "url": "https://docs.contoso.com/hr/leave-policy",
      "content": "Section 4.2: Annual Leave Entitlement..."
    }
  ],
  "confidence": 0.92
}
```

### Step 1.3: Document the Input/Output Schema

Record the exact field names and types:

**Input Schema:**
| Field | Type | Required | Description |
|---|---|---|---|
| `question` | string | Yes | The user's question |
| `chat_history` | array | No | Previous conversation turns |

**Output Schema:**
| Field | Type | Description |
|---|---|---|
| `answer` | string | The grounded answer text |
| `citations` | array | Source documents with title, url, content |
| `confidence` | number | Confidence score (0–1) |

> ✅ **Checkpoint:** Your endpoint returns a valid JSON response with answer and citations.

---

## Exercise 2: Create the OpenAPI Specification (10 minutes)

### Step 2.1: Create the Spec File

Create a new file called `ai-foundry-plugin.yaml` with the following content. **Replace the placeholders** with your actual values:

```yaml
openapi: "3.0.1"
info:
  title: "AI Foundry RAG Agent"
  description: |
    Queries a RAG (Retrieval-Augmented Generation) agent deployed on
    Azure AI Foundry. This agent searches the enterprise knowledge base
    to provide grounded, accurate answers with source citations.
    
    Use this plugin when users ask questions about:
    - Company policies and procedures
    - Product documentation and specifications
    - Internal knowledge base content
    
    The agent returns answers grounded in actual documents with
    citation references for verification.
  version: "1.0.0"

servers:
  - url: "https://<your-project>.<region>.inference.ml.azure.com"
    description: "AI Foundry Inference Endpoint"

paths:
  /score:
    post:
      operationId: queryRAGAgent
      summary: "Query the enterprise knowledge base RAG agent"
      description: |
        Send a natural language question to the RAG agent. The agent
        searches the enterprise knowledge base, retrieves relevant
        documents, and generates a grounded answer with citations.
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              properties:
                question:
                  type: string
                  description: "The user's natural language question"
                chat_history:
                  type: array
                  description: "Previous conversation turns for context"
                  items:
                    type: object
                    properties:
                      role:
                        type: string
                        enum: [user, assistant]
                      content:
                        type: string
              required:
                - question
      responses:
        "200":
          description: "Successful RAG response with grounded answer"
          content:
            application/json:
              schema:
                type: object
                properties:
                  answer:
                    type: string
                    description: "The grounded answer text"
                  citations:
                    type: array
                    description: "Source documents referenced"
                    items:
                      type: object
                      properties:
                        title:
                          type: string
                        url:
                          type: string
                        content:
                          type: string
                  confidence:
                    type: number
                    description: "Confidence score from 0 to 1"
        "401":
          description: "Unauthorized — invalid or missing API key"
        "429":
          description: "Rate limited — too many requests"
        "500":
          description: "Internal server error"

security:
  - apiKey: []

components:
  securitySchemes:
    apiKey:
      type: apiKey
      name: Authorization
      in: header
```

### Step 2.2: Validate the Spec

Validate your OpenAPI spec using an online validator or local tool:

```bash
# Option 1: Using npm swagger-cli
npx @apidevtools/swagger-cli validate ai-foundry-plugin.yaml

# Option 2: Paste into https://editor.swagger.io/ and check for errors
```

### Step 2.3: Customize the Description

> ⚠️ **Critical Step:** The `description` fields are the most important part of the spec for Copilot Studio integration. The AI orchestrator reads these descriptions to decide when to invoke your plugin.

**Good Description Example:**
```
"Answers questions about Contoso Corp's HR policies, including
benefits enrollment, leave procedures, compensation bands, and
onboarding checklists, using the official HR document library."
```

**Bad Description Example:**
```
"Answers questions using AI."
```

> ✅ **Checkpoint:** You have a valid OpenAPI YAML spec file with descriptive text.

---

## Exercise 3: Register the Plugin in Copilot Studio (10 minutes)

### Step 3.1: Create a New Copilot

1. Navigate to [Copilot Studio](https://copilotstudio.microsoft.com)
2. Click **Create** → **New copilot**
3. Name it: `Enterprise Knowledge Assistant`
4. Description: `An AI-powered assistant that answers questions using the enterprise knowledge base`
5. Click **Create**

### Step 3.2: Add the Custom Plugin

1. In your copilot, go to **Settings** (⚙️ gear icon)
2. Select **Plugins** from the left menu
3. Click **Add a plugin** → **Custom connector**
4. Choose **Upload an OpenAPI file**
5. Upload your `ai-foundry-plugin.yaml` file
6. Wait for Copilot Studio to parse the specification

### Step 3.3: Configure Authentication

1. In the plugin configuration, find the **Authentication** section
2. Select **API Key** as the authentication type
3. Set the **Header name** to `Authorization`
4. Set the **Header value prefix** to `Bearer `
5. Enter your AI Foundry API key in the **Key** field
6. Click **Save**

### Step 3.4: Test the Plugin Connection

1. Click **Test plugin** in the configuration panel
2. Enter a sample question: `What is the company leave policy?`
3. Verify the response includes:
   - An answer text
   - At least one citation
   - No error messages

> ✅ **Checkpoint:** Plugin is registered and returning responses in the test panel.

---

## Exercise 4: Build a Conversation Topic (10 minutes)

### Step 4.1: Create a New Topic

1. Go to **Topics** in the left navigation
2. Click **Add a topic** → **From blank**
3. Name it: `Knowledge Base Query`
4. Add trigger phrases:
   - "Search the knowledge base"
   - "What does our documentation say about"
   - "Find information about"
   - "Look up our policy on"
   - "Help me understand our process for"

### Step 4.2: Add the Plugin Action

1. In the topic canvas, add a new **Plugin action** node
2. Select your **AI Foundry RAG Agent** plugin
3. Select the **queryRAGAgent** action
4. Map the input:
   - **question** → `System.Activity.Text` (the user's message)
   - **chat_history** → Leave empty for now (we'll add this in the advanced section)

### Step 4.3: Format the Response

Add a **Message** node after the plugin action with the following content:

```
{Topic.answer}

📚 **Sources:**
{Topic.citations}
```

For a more polished response, use an Adaptive Card:

1. Add a **Send message** node
2. Switch to **Adaptive Card** mode
3. Paste this template:

```json
{
  "$schema": "http://adaptivecards.io/schemas/adaptive-card.json",
  "type": "AdaptiveCard",
  "version": "1.5",
  "body": [
    {
      "type": "TextBlock",
      "text": "${Topic.answer}",
      "wrap": true,
      "size": "medium"
    },
    {
      "type": "TextBlock",
      "text": "📚 Sources",
      "weight": "bolder",
      "spacing": "medium"
    },
    {
      "type": "TextBlock",
      "text": "${Topic.citations}",
      "wrap": true,
      "size": "small",
      "color": "accent"
    }
  ]
}
```

### Step 4.4: Add Error Handling

1. Add a **Condition** node after the plugin action
2. Set the condition: `If Topic.answer is blank`
3. **True path:** Add a message: "I'm sorry, I couldn't find an answer to your question. Would you like me to connect you with a support agent?"
4. **False path:** Continue to the response message node

> ✅ **Checkpoint:** Topic is configured with plugin action, formatted response, and error handling.

---

## Exercise 5: Enable Generative Orchestration (5 minutes)

### Step 5.1: Turn On Generative Mode

1. Go to **Settings** → **Generative AI**
2. Enable **Generative orchestration**
3. Set the orchestration mode to **Generative (preview)** or **Classic + Generative**

### Step 5.2: Configure Orchestration Behavior

1. Under Generative AI settings, find **Plugin behavior**
2. Ensure your AI Foundry plugin is listed and enabled
3. Review the plugin description — this is what the AI uses for routing decisions

### Step 5.3: Test Natural Language Routing

In the **Test** panel, try these queries:
1. "What is our vacation policy?" → Should invoke the RAG plugin
2. "Hello, how are you?" → Should NOT invoke the plugin (general greeting)
3. "Tell me about the onboarding process" → Should invoke the RAG plugin
4. "What's the weather?" → Should NOT invoke the plugin (out of scope)

Verify that the orchestrator correctly routes relevant queries to your plugin and handles unrelated queries with default responses.

> ✅ **Checkpoint:** Generative orchestration correctly routes queries to your AI Foundry plugin.

---

## Exercise 6: Publish to Microsoft Teams (10 minutes)

### Step 6.1: Configure the Teams Channel

1. Go to **Settings** → **Channels**
2. Click **Microsoft Teams**
3. Follow the prompts to connect your copilot to Teams
4. Select the Teams environment and availability settings

### Step 6.2: Publish the Copilot

1. Click **Publish** in the top-right corner
2. Review the publish summary — ensure all topics and plugins are included
3. Click **Publish** to make the copilot available

### Step 6.3: Test in Teams

1. Open Microsoft Teams
2. Go to **Apps** → Search for your copilot name
3. Install the app and open a chat
4. Test with the following queries:

```
Test Query 1: "What is the company leave policy?"
Expected: Grounded answer from knowledge base with citations

Test Query 2: "How do I submit an expense report?"
Expected: Grounded answer with relevant process documentation

Test Query 3: "Tell me about benefits enrollment"
Expected: Grounded answer about benefits with source links
```

### Step 6.4: Share and Get Feedback

1. Share the Teams app link with a colleague
2. Ask them to test with 3 questions related to your knowledge base
3. Collect feedback on:
   - Answer accuracy
   - Response time
   - Citation quality
   - Overall conversational experience

> ✅ **Checkpoint:** Your copilot is live in Teams and responding with AI Foundry-powered answers.

---

## Bonus Challenges

### Challenge A: Multi-Turn Context (15 min)
Modify your topic to pass `chat_history` to the AI Foundry endpoint:
1. Create a conversation variable to store previous turns
2. Append each user message and assistant response to the history
3. Pass the full history array to the plugin's `chat_history` parameter
4. Test with follow-up questions that require context

### Challenge B: Feedback Collection (10 min)
Add a feedback mechanism after each response:
1. Add a **Quick reply** node with "👍 Helpful" and "👎 Not helpful" options
2. Store feedback in a Dataverse table
3. Use feedback data to identify knowledge gaps

### Challenge C: Upgrade to Entra ID Auth (15 min)
Replace API key authentication with Entra ID:
1. Create an Entra ID app registration
2. Assign the Cognitive Services User role
3. Reconfigure the plugin to use OAuth 2.0
4. Test the authenticated flow

---

## Troubleshooting Guide

| Symptom | Likely Cause | Solution |
|---|---|---|
| Plugin returns 401 | Invalid API key | Verify key in AI Foundry portal; check "Bearer " prefix |
| Plugin never invoked | Poor description or orchestration off | Improve plugin description; enable generative orchestration |
| Slow responses (>10s) | Endpoint cold start or low capacity | Enable auto-scale; test with warm-up requests |
| Empty answer field | Schema mismatch | Compare OpenAPI spec field names to actual API response |
| Citations not rendering | Template binding error | Verify `Topic.citations` maps to the correct output field |
| Teams install fails | Permissions issue | Check Copilot Studio environment permissions; contact admin |
| Orchestrator routes incorrectly | Description too vague | Make description specific about domains and query types |

---

## Lab Completion Checklist

- [ ] AI Foundry endpoint verified and returning valid responses
- [ ] OpenAPI specification created with descriptive text
- [ ] Plugin registered in Copilot Studio with API key authentication
- [ ] Conversation topic built with plugin action and response formatting
- [ ] Error handling configured for unavailable endpoint
- [ ] Generative orchestration enabled and routing correctly
- [ ] Copilot published and accessible in Microsoft Teams
- [ ] At least 3 test queries produce grounded, accurate responses
- [ ] Feedback collected from at least 1 colleague

---

## Clean-Up (Optional)

If this is a training environment, clean up resources:

```bash
# Remove the Entra ID app registration (if created)
az ad app delete --id <app-id>

# Note: Do NOT delete your AI Foundry endpoint if used by other modules
# Note: Copilot Studio bots can be unpublished via Settings → Channels
```

---

## Next Steps

- **Module 24:** Advanced Orchestration Patterns — chaining multiple AI Foundry endpoints
- **Module 25:** Monitoring and Analytics for AI-Powered Copilots
- Review the [Copilot Studio documentation](https://learn.microsoft.com/microsoft-copilot-studio/) for advanced features

---

*Azure AI Foundry: Zero to Hero — Module 23 Lab*  
*Arc 5: Orchestration & Integration | April 2026*
