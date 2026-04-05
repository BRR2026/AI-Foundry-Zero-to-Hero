# Module 24 — Hands-on Lab

## Mini Hack: Create a M365 Copilot Plugin That Queries Your Knowledge Base

> **Duration:** 60–90 minutes
> **Difficulty:** Intermediate
> **Arc:** 5 — Orchestration & Integration

---

## 🎯 Lab Objective

Build a **declarative agent** with an **API plugin** for Microsoft 365 Copilot that queries an Azure AI Foundry knowledge base. When complete, users will be able to ask questions in M365 Copilot and receive grounded answers from your enterprise documents — complete with source citations.

---

## ✅ Prerequisites

Before starting, ensure you have the following:

| Requirement | Details |
|---|---|
| **Microsoft 365 License** | E3/E5 with Microsoft 365 Copilot license enabled |
| **Azure Subscription** | Active subscription with Contributor access |
| **AI Foundry Project** | A deployed Azure AI Foundry project with a RAG endpoint (from Module 18–20) |
| **Azure AI Search** | An index populated with your knowledge base documents |
| **Node.js** | Version 20 or later |
| **VS Code** | Latest version with Teams Toolkit extension installed |
| **Teams Toolkit CLI** | `npm install -g @microsoft/teamsapp-cli` |
| **Azure Functions Core Tools** | Version 4.x (`npm install -g azure-functions-core-tools@4`) |
| **M365 Developer Tenant** | (Optional) For testing without affecting production |

---

## 🏗️ Architecture Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                     Microsoft 365 Copilot                       │
│  ┌─────────────────┐    ┌──────────────────────────────────┐    │
│  │ Declarative Agent│───▶│ API Plugin (OpenAPI)             │    │
│  │ • Instructions   │    │ POST /api/knowledge/search       │    │
│  │ • Starters       │    └──────────────┬───────────────────┘    │
│  └─────────────────┘                    │                        │
└─────────────────────────────────────────┼────────────────────────┘
                                          │ HTTPS
                                          ▼
                              ┌──────────────────────┐
                              │  Azure Functions      │
                              │  (API Backend)        │
                              │  • Managed Identity   │
                              └──────────┬───────────┘
                                         │
                          ┌──────────────┴──────────────┐
                          ▼                              ▼
                 ┌─────────────────┐          ┌─────────────────┐
                 │ Azure OpenAI    │          │ Azure AI Search │
                 │ (GPT-4o)       │          │ (Vector Index)  │
                 │ AI Foundry     │          │ Knowledge Base  │
                 └─────────────────┘          └─────────────────┘
```

---

## 📝 Step-by-Step Instructions

### Step 1: Scaffold the Project (10 min)

#### 1.1 Create a New Project with Teams Toolkit

Open VS Code and use the Teams Toolkit to create a new project:

1. Open the **Command Palette** (`Ctrl+Shift+P`)
2. Type **"Teams: Create a New App"**
3. Select **"Declarative Agent"**
4. Choose **"Add plugin → Start with an OpenAPI Description Document"**
5. Select **"Start with a new API"**
6. Name the project: `kb-copilot-plugin`
7. Choose TypeScript as the language

#### 1.2 Review the Scaffolded Structure

```
kb-copilot-plugin/
├── appPackage/
│   ├── manifest.json              # Teams app manifest
│   ├── declarativeAgent.json      # Agent instructions & config
│   ├── ai-plugin.json             # API plugin definition
│   └── apiSpecificationFile/
│       └── openapi.yaml           # OpenAPI spec for the backend
├── src/
│   └── functions/
│       └── knowledgeSearch.ts     # Azure Function handler
├── env/
│   ├── .env.local                 # Local development settings
│   └── .env.dev                   # Dev environment settings
├── teamsapp.yml                   # Teams Toolkit config
├── teamsapp.local.yml             # Local debug config
├── package.json
└── tsconfig.json
```

---

### Step 2: Configure the Azure Function Backend (15 min)

#### 2.1 Install Dependencies

```bash
cd kb-copilot-plugin
npm install
npm install @azure/identity @azure/search-documents openai
```

#### 2.2 Create the Knowledge Search Function

Replace the contents of `src/functions/knowledgeSearch.ts`:

```typescript
import { app, HttpRequest, HttpResponseInit, InvocationContext } from '@azure/functions';
import { DefaultAzureCredential } from '@azure/identity';
import { SearchClient } from '@azure/search-documents';
import { AzureOpenAI } from 'openai';

interface SearchRequest {
  query: string;
  top_k?: number;
}

interface SearchResult {
  answer: string;
  citations: Array<{
    title: string;
    content: string;
    url: string;
  }>;
}

app.http('knowledgeSearch', {
  methods: ['POST'],
  authLevel: 'anonymous',
  handler: async (request: HttpRequest, context: InvocationContext): Promise<HttpResponseInit> => {
    context.log('Knowledge search invoked');

    try {
      const body = await request.json() as SearchRequest;
      const { query, top_k = 5 } = body;

      if (!query || query.trim().length === 0) {
        return {
          status: 400,
          jsonBody: { error: 'Query parameter is required' }
        };
      }

      const credential = new DefaultAzureCredential();

      // Step 1: Search the knowledge base index
      const searchClient = new SearchClient(
        process.env.AZURE_SEARCH_ENDPOINT!,
        process.env.AZURE_SEARCH_INDEX!,
        credential
      );

      const searchResults = await searchClient.search(query, {
        queryType: 'semantic',
        semanticSearchOptions: {
          configurationName: 'default'
        },
        top: top_k,
        select: ['title', 'content', 'source_url']
      });

      // Step 2: Build context from search results
      const documents: Array<{ title: string; content: string; url: string }> = [];
      for await (const result of searchResults.results) {
        documents.push({
          title: (result.document as any).title || 'Untitled',
          content: (result.document as any).content || '',
          url: (result.document as any).source_url || '#'
        });
      }

      if (documents.length === 0) {
        return {
          status: 200,
          jsonBody: {
            answer: 'No relevant documents were found for your query. Please try rephrasing your question.',
            citations: []
          }
        };
      }

      // Step 3: Generate grounded response using Azure OpenAI
      const contextText = documents
        .map((doc, i) => `[Source ${i + 1}: ${doc.title}]\n${doc.content}`)
        .join('\n\n---\n\n');

      const openaiClient = new AzureOpenAI({
        endpoint: process.env.AZURE_OPENAI_ENDPOINT!,
        apiVersion: '2025-01-01-preview',
        azureADTokenProvider: async () => {
          const token = await credential.getToken('https://cognitiveservices.azure.com/.default');
          return token.token;
        }
      });

      const completion = await openaiClient.chat.completions.create({
        model: process.env.AZURE_OPENAI_DEPLOYMENT || 'gpt-4o',
        messages: [
          {
            role: 'system',
            content: `You are a knowledge base assistant. Answer the user's question using ONLY
              the provided context. Always cite your sources using [Source N] notation.
              If the context doesn't contain enough information, say so honestly.

              Context:
              ${contextText}`
          },
          { role: 'user', content: query }
        ],
        temperature: 0.3,
        max_tokens: 1000
      });

      const response: SearchResult = {
        answer: completion.choices[0].message.content || 'Unable to generate a response.',
        citations: documents.map(doc => ({
          title: doc.title,
          content: doc.content.substring(0, 200) + '...',
          url: doc.url
        }))
      };

      return {
        status: 200,
        jsonBody: response
      };

    } catch (error: any) {
      context.error('Knowledge search error:', error);
      return {
        status: 500,
        jsonBody: {
          error: 'An error occurred while searching the knowledge base',
          details: error.message
        }
      };
    }
  }
});
```

#### 2.3 Configure Environment Variables

Update `env/.env.local`:

```env
# Azure AI Search
AZURE_SEARCH_ENDPOINT=https://your-search-service.search.windows.net
AZURE_SEARCH_INDEX=knowledge-index

# Azure OpenAI (via AI Foundry)
AZURE_OPENAI_ENDPOINT=https://your-openai.openai.azure.com
AZURE_OPENAI_DEPLOYMENT=gpt-4o

# Teams App Config (auto-populated by Teams Toolkit)
TEAMSFX_ENV=local
```

---

### Step 3: Define the OpenAPI Specification (10 min)

Replace `appPackage/apiSpecificationFile/openapi.yaml`:

```yaml
openapi: 3.0.3
info:
  title: Enterprise Knowledge Base API
  description: |
    Search the enterprise knowledge base powered by Azure AI Foundry.
    Returns AI-generated answers grounded in indexed documents with citations.
  version: 1.0.0

servers:
  - url: ${{OPENAPI_SERVER_URL}}
    description: API server

paths:
  /api/knowledgeSearch:
    post:
      operationId: searchKnowledgeBase
      summary: Search the enterprise knowledge base for answers
      description: |
        Searches indexed enterprise documents using semantic search and generates
        a grounded answer with source citations. Use this when users ask questions
        about company policies, product documentation, technical guides, or any
        information that might be in the knowledge base.
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              required:
                - query
              properties:
                query:
                  type: string
                  description: |
                    The user's natural language question to search for in the
                    knowledge base. Should be a clear, specific question.
                top_k:
                  type: integer
                  description: Number of source documents to retrieve (1-10)
                  default: 5
                  minimum: 1
                  maximum: 10
      responses:
        '200':
          description: Search results with AI-generated answer and citations
          content:
            application/json:
              schema:
                type: object
                properties:
                  answer:
                    type: string
                    description: AI-generated answer grounded in source documents
                  citations:
                    type: array
                    description: Source documents used to generate the answer
                    items:
                      type: object
                      properties:
                        title:
                          type: string
                          description: Title of the source document
                        content:
                          type: string
                          description: Relevant excerpt from the document
                        url:
                          type: string
                          description: URL to the full source document
        '400':
          description: Invalid request - query parameter missing
        '500':
          description: Server error during search
```

---

### Step 4: Configure the Declarative Agent (10 min)

#### 4.1 Update `appPackage/declarativeAgent.json`

```json
{
  "$schema": "https://developer.microsoft.com/json-schemas/copilot/declarative-agent/v1.2/schema.json",
  "version": "v1.2",
  "name": "Knowledge Base Assistant",
  "description": "An AI assistant that answers questions using the enterprise knowledge base powered by Azure AI Foundry. Ask about company policies, product documentation, technical guides, and more.",
  "instructions": "You are the Knowledge Base Assistant. Your job is to help users find information from the enterprise knowledge base.\n\nGuidelines:\n1. Always use the searchKnowledgeBase action to find relevant information before answering.\n2. Provide clear, concise answers based on the search results.\n3. Always cite your sources - mention the document title and provide the link.\n4. If the search returns no results, suggest the user rephrase their question or contact support.\n5. Never make up information - only use what the knowledge base returns.\n6. If a question is outside the scope of the knowledge base, politely say so.",
  "conversation_starters": [
    { "text": "What is our company's return policy?" },
    { "text": "How do I configure SSO for our application?" },
    { "text": "Find documentation about the API rate limits" },
    { "text": "What are the steps for onboarding a new employee?" },
    { "text": "Summarize our data privacy guidelines" }
  ],
  "actions": [
    {
      "id": "knowledgeSearchPlugin",
      "file": "ai-plugin.json"
    }
  ]
}
```

#### 4.2 Update `appPackage/ai-plugin.json`

```json
{
  "$schema": "https://developer.microsoft.com/json-schemas/copilot/plugin/v2.2/schema.json",
  "schema_version": "v2.2",
  "name_for_human": "Knowledge Base Search",
  "description_for_human": "Search the enterprise knowledge base for answers to your questions",
  "namespace": "knowledgeBase",
  "functions": [
    {
      "name": "searchKnowledgeBase",
      "description": "Search the enterprise knowledge base for information about company policies, product documentation, technical guides, and other organizational knowledge. Returns an AI-generated answer with source citations.",
      "capabilities": {
        "response_semantics": {
          "data_path": "$.answer",
          "properties": {
            "title": "$.citations[0].title",
            "subtitle": "$.citations[0].content",
            "url": "$.citations[0].url"
          },
          "static_template": {
            "$schema": "http://adaptivecards.io/schemas/adaptive-card.json",
            "type": "AdaptiveCard",
            "version": "1.5",
            "body": [
              {
                "type": "TextBlock",
                "text": "Knowledge Base Result",
                "weight": "Bolder",
                "size": "Medium"
              },
              {
                "type": "TextBlock",
                "text": "${answer}",
                "wrap": true
              }
            ]
          }
        }
      }
    }
  ],
  "runtimes": [
    {
      "type": "OpenApi",
      "auth": { "type": "None" },
      "spec": {
        "url": "apiSpecificationFile/openapi.yaml"
      },
      "run_for_functions": ["searchKnowledgeBase"]
    }
  ]
}
```

#### 4.3 Update `appPackage/manifest.json`

Ensure the manifest includes the Copilot extension:

```json
{
  "$schema": "https://developer.microsoft.com/json-schemas/teams/v1.19/MicrosoftTeams.schema.json",
  "manifestVersion": "1.19",
  "version": "1.0.0",
  "id": "${{TEAMS_APP_ID}}",
  "developer": {
    "name": "Contoso",
    "websiteUrl": "https://contoso.com",
    "privacyUrl": "https://contoso.com/privacy",
    "termsOfUseUrl": "https://contoso.com/terms"
  },
  "name": {
    "short": "KB Assistant",
    "full": "Knowledge Base Assistant powered by AI Foundry"
  },
  "description": {
    "short": "Ask questions about enterprise knowledge",
    "full": "An AI-powered assistant that searches the enterprise knowledge base and provides grounded answers with source citations. Powered by Azure AI Foundry."
  },
  "icons": {
    "color": "color.png",
    "outline": "outline.png"
  },
  "copilotExtensions": {
    "declarativeCopilots": [
      {
        "id": "knowledgeBaseAgent",
        "file": "declarativeAgent.json"
      }
    ]
  },
  "permissions": ["identity"],
  "validDomains": ["${{OPENAPI_SERVER_DOMAIN}}"]
}
```

---

### Step 5: Test Locally (15 min)

#### 5.1 Start Local Development

1. Press **F5** in VS Code (or run `teamsapp provision --env local && teamsapp deploy --env local`)
2. Teams Toolkit will:
   - Start the Azure Functions backend locally
   - Create a dev tunnel for external access
   - Package and sideload the Teams app
   - Open M365 Copilot in the browser

#### 5.2 Test the Plugin

1. In M365 Copilot, look for your **"KB Assistant"** agent in the right panel
2. Select it to start a scoped conversation
3. Try the conversation starters:
   - *"What is our company's return policy?"*
   - *"How do I configure SSO for our application?"*
4. Verify that:
   - ✅ Copilot invokes the `searchKnowledgeBase` action
   - ✅ Results include grounded answers with citations
   - ✅ Source document links are provided
   - ✅ The agent stays within knowledge base scope

#### 5.3 Debug Common Issues

| Issue | Cause | Solution |
|---|---|---|
| Plugin not appearing | Manifest error | Check Teams Toolkit output for validation errors |
| "No results found" | Search index empty | Verify Azure AI Search index has documents |
| Auth error to Azure | Credential issue | Run `az login` and ensure DefaultAzureCredential works |
| Timeout errors | Slow RAG pipeline | Increase function timeout in `host.json`, optimize search |
| Copilot ignores plugin | Poor descriptions | Improve `operationId` and `description` in OpenAPI spec |

---

### Step 6: Deploy to Azure (Optional — 15 min)

#### 6.1 Provision Azure Resources

```bash
teamsapp provision --env dev
```

This creates:
- Azure Functions App (with managed identity)
- Azure Bot registration
- Teams app registration in Entra ID

#### 6.2 Configure Managed Identity Access

Grant the Function App's managed identity access to your AI resources:

```bash
# Get the Function App's managed identity principal ID
PRINCIPAL_ID=$(az functionapp identity show \
  --name your-function-app \
  --resource-group your-rg \
  --query principalId -o tsv)

# Grant Cognitive Services User role for Azure OpenAI
az role assignment create \
  --assignee $PRINCIPAL_ID \
  --role "Cognitive Services User" \
  --scope /subscriptions/{sub}/resourceGroups/{rg}/providers/Microsoft.CognitiveServices/accounts/{aoai}

# Grant Search Index Data Reader role for Azure AI Search
az role assignment create \
  --assignee $PRINCIPAL_ID \
  --role "Search Index Data Reader" \
  --scope /subscriptions/{sub}/resourceGroups/{rg}/providers/Microsoft.Search/searchServices/{search}
```

#### 6.3 Deploy

```bash
teamsapp deploy --env dev
teamsapp publish --env dev
```

---

## ✅ Success Criteria

Your lab is complete when you can demonstrate:

- [ ] **Plugin Loads:** The KB Assistant appears in M365 Copilot
- [ ] **Search Works:** User queries trigger the `searchKnowledgeBase` action
- [ ] **Grounded Answers:** Responses are based on indexed documents, not hallucinated
- [ ] **Citations Included:** Each answer includes source document references
- [ ] **Error Handling:** Graceful responses for empty queries or no results
- [ ] **Scope Maintained:** The agent stays within the knowledge base domain

---

## 🏆 Bonus Challenges

### Challenge 1: Add Conversation Memory
Modify the Azure Function to accept a `conversation_history` parameter and pass it to Azure OpenAI for multi-turn conversations.

### Challenge 2: Add User-Context Filtering
Implement OBO authentication so the search results are filtered based on the calling user's permissions (SharePoint ACLs).

### Challenge 3: Add a Second Action
Add a `submitFeedback` action that lets users rate the quality of answers, storing feedback in Cosmos DB.

### Challenge 4: Adaptive Card Response
Create a rich Adaptive Card template that displays the answer, citations with expandable sections, and a feedback button.

---

## 🧹 Cleanup

If using a dev/test environment, clean up resources:

```bash
# Remove Azure resources
teamsapp provision --env dev --clean

# Or manually
az group delete --name your-rg --yes --no-wait

# Remove sideloaded app from M365
# Go to Teams Admin Center → Manage apps → Delete the test app
```

---

## 📚 References

- [Declarative Agents Documentation](https://learn.microsoft.com/microsoft-365-copilot/extensibility/build-declarative-agents)
- [API Plugins for M365 Copilot](https://learn.microsoft.com/microsoft-365-copilot/extensibility/build-api-plugins)
- [Teams Toolkit Overview](https://learn.microsoft.com/microsoftteams/platform/toolkit/teams-toolkit-fundamentals)
- [Azure Functions with Managed Identity](https://learn.microsoft.com/azure/azure-functions/functions-identity-based-connections-tutorial)
- [Azure AI Search Semantic Ranking](https://learn.microsoft.com/azure/search/semantic-search-overview)

---

*Module 24 Lab · Azure AI Foundry: Zero to Hero · Arc 5: Orchestration & Integration*
