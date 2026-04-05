# Module 25 — Hands-On Lab

## 🏗️ Mini Hack: Build a Teams Bot Powered by Your AI Foundry Agent

> **Arc 5: Orchestration & Integration** | Azure AI Foundry: Zero to Hero
>
> **Duration:** 45–60 minutes | **Level:** Intermediate–Advanced

---

## 🎯 Lab Objective

Build a fully functional Microsoft Teams bot that:

1. Receives messages from users in Microsoft Teams
2. Forwards each message to an Azure AI Foundry agent
3. Maintains conversation context across multiple messages
4. Returns formatted responses using Adaptive Cards
5. Handles errors gracefully with fallback messages

---

## 📋 Prerequisites

| Requirement | Details |
|---|---|
| Azure subscription | Active subscription with Contributor role |
| AI Foundry project | A deployed agent with at least one tool (from Module 23) |
| Development tools | VS Code, Python 3.10+, Azure Functions Core Tools v4 |
| Teams access | Microsoft Teams with side-loading enabled (or Bot Framework Emulator) |
| Azure CLI | Logged in (`az login`) with correct subscription set |

---

## Architecture

```
┌──────────────┐     ┌──────────────────┐     ┌───────────────────┐     ┌──────────────────┐
│  Microsoft   │────▶│   Azure Bot      │────▶│  Azure Function   │────▶│  Azure AI        │
│  Teams       │◀────│   Service        │◀────│  (Bot Handler)    │◀────│  Foundry Agent   │
└──────────────┘     └──────────────────┘     └───────────────────┘     └──────────────────┘
                                                       │
                                                       ▼
                                              ┌───────────────────┐
                                              │  Azure Table      │
                                              │  Storage           │
                                              │  (Thread Mapping) │
                                              └───────────────────┘
```

---

## Step 1: Set Up Your AI Foundry Agent

If you already have an agent from Module 23, note down these values. Otherwise, create one now.

### 1.1 Create the Agent (if needed)

```python
from azure.identity import DefaultAzureCredential
from azure.ai.projects import AIProjectClient

# Connect to your AI Foundry project
client = AIProjectClient(
    credential=DefaultAzureCredential(),
    project="<YOUR_PROJECT_CONNECTION_STRING>"
)

# Create an agent
agent = client.agents.create_agent(
    model="gpt-4o",
    name="teams-assistant",
    instructions="""You are a helpful enterprise assistant deployed in Microsoft Teams.
    You help employees with questions about company policies, project status,
    and general knowledge. Be concise and professional.
    Format your responses for readability with bullet points when appropriate."""
)

print(f"Agent ID: {agent.id}")
```

### 1.2 Record Your Configuration

Create a `.env` file in your project root (do NOT commit this to source control):

```env
FOUNDRY_PROJECT_CONN_STR=<your-project-connection-string>
FOUNDRY_AGENT_ID=<your-agent-id>
MICROSOFT_APP_ID=<from-bot-registration-step>
MICROSOFT_APP_PASSWORD=<from-bot-registration-step>
AZURE_STORAGE_CONNECTION_STRING=<from-storage-account>
```

---

## Step 2: Create the Azure Function Project

### 2.1 Initialize the Project

```bash
# Create a new directory
mkdir teams-foundry-bot
cd teams-foundry-bot

# Initialize Azure Functions project
func init --python

# Create the bot endpoint function
func new --name bot --template "HTTP trigger" --authlevel anonymous
```

### 2.2 Install Dependencies

Add these to `requirements.txt`:

```text
azure-functions
botbuilder-core>=4.15.0
botbuilder-integration-aiohttp>=4.15.0
azure-identity
azure-ai-projects
azure-data-tables
python-dotenv
aiohttp
```

Install them:

```bash
pip install -r requirements.txt
```

### 2.3 Project Structure

```
teams-foundry-bot/
├── bot/
│   ├── __init__.py          # HTTP trigger entry point
│   └── function.json        # Function binding config
├── shared/
│   ├── __init__.py
│   ├── foundry_bot.py       # Bot logic
│   ├── thread_store.py      # Thread mapping storage
│   └── card_formatter.py    # Adaptive Card builder
├── host.json
├── local.settings.json
├── requirements.txt
└── .env
```

---

## Step 3: Implement the Thread Store

Thread persistence is critical — without it, every message starts a new conversation and the agent loses all context.

### 3.1 Create `shared/thread_store.py`

```python
"""Maps Teams conversation IDs to AI Foundry thread IDs using Azure Table Storage."""

import os
from azure.data.tables import TableServiceClient, TableClient

class ThreadStore:
    def __init__(self):
        conn_str = os.environ["AZURE_STORAGE_CONNECTION_STRING"]
        service = TableServiceClient.from_connection_string(conn_str)
        self.table: TableClient = service.create_table_if_not_exists("conversationthreads")

    def get_thread_id(self, conversation_id: str) -> str | None:
        """Retrieve the AI Foundry thread ID for a Teams conversation."""
        try:
            entity = self.table.get_entity(
                partition_key="threads",
                row_key=conversation_id
            )
            return entity["foundry_thread_id"]
        except Exception:
            return None

    def set_thread_id(self, conversation_id: str, thread_id: str) -> None:
        """Store the mapping between a Teams conversation and AI Foundry thread."""
        self.table.upsert_entity({
            "PartitionKey": "threads",
            "RowKey": conversation_id,
            "foundry_thread_id": thread_id
        })
```

---

## Step 4: Implement the Adaptive Card Formatter

### 4.1 Create `shared/card_formatter.py`

```python
"""Formats AI Foundry agent responses as Teams Adaptive Cards."""

from botbuilder.core import CardFactory

def create_response_card(agent_response: str, agent_name: str = "AI Assistant") -> dict:
    """Create an Adaptive Card with the agent's response and feedback buttons."""
    card = {
        "type": "AdaptiveCard",
        "$schema": "http://adaptivecards.io/schemas/adaptive-card.json",
        "version": "1.5",
        "body": [
            {
                "type": "ColumnSet",
                "columns": [
                    {
                        "type": "Column",
                        "width": "auto",
                        "items": [
                            {
                                "type": "Image",
                                "url": "https://img.icons8.com/fluency/48/bot.png",
                                "size": "Small",
                                "style": "Person"
                            }
                        ]
                    },
                    {
                        "type": "Column",
                        "width": "stretch",
                        "items": [
                            {
                                "type": "TextBlock",
                                "text": agent_name,
                                "weight": "Bolder",
                                "size": "Small",
                                "color": "Accent"
                            }
                        ],
                        "verticalContentAlignment": "Center"
                    }
                ]
            },
            {
                "type": "TextBlock",
                "text": agent_response,
                "wrap": True,
                "spacing": "Medium"
            }
        ],
        "actions": [
            {
                "type": "Action.Submit",
                "title": "👍 Helpful",
                "data": {"feedback": "positive"}
            },
            {
                "type": "Action.Submit",
                "title": "👎 Not Helpful",
                "data": {"feedback": "negative"}
            }
        ]
    }
    return CardFactory.adaptive_card(card)


def create_error_card(error_message: str) -> dict:
    """Create an error Adaptive Card for graceful failure handling."""
    card = {
        "type": "AdaptiveCard",
        "$schema": "http://adaptivecards.io/schemas/adaptive-card.json",
        "version": "1.5",
        "body": [
            {
                "type": "TextBlock",
                "text": "⚠️ Something went wrong",
                "weight": "Bolder",
                "color": "Attention"
            },
            {
                "type": "TextBlock",
                "text": error_message,
                "wrap": True,
                "color": "Default"
            }
        ]
    }
    return CardFactory.adaptive_card(card)
```

---

## Step 5: Implement the Bot Logic

### 5.1 Create `shared/foundry_bot.py`

```python
"""Teams bot that forwards messages to an Azure AI Foundry agent."""

import os
import logging
from botbuilder.core import ActivityHandler, TurnContext, MessageFactory
from azure.identity import DefaultAzureCredential
from azure.ai.projects import AIProjectClient

from .thread_store import ThreadStore
from .card_formatter import create_response_card, create_error_card

logger = logging.getLogger(__name__)


class FoundryTeamsBot(ActivityHandler):
    def __init__(self):
        self.client = AIProjectClient(
            credential=DefaultAzureCredential(),
            project=os.environ["FOUNDRY_PROJECT_CONN_STR"]
        )
        self.agent_id = os.environ["FOUNDRY_AGENT_ID"]
        self.thread_store = ThreadStore()

    async def on_message_activity(self, turn_context: TurnContext):
        """Handle incoming messages from Teams users."""
        user_message = turn_context.activity.text
        conversation_id = turn_context.activity.conversation.id

        if not user_message:
            # Handle Adaptive Card submissions (feedback)
            if turn_context.activity.value:
                feedback = turn_context.activity.value.get("feedback")
                logger.info(f"Feedback received: {feedback}")
                await turn_context.send_activity("Thanks for your feedback! 🙏")
            return

        try:
            # 1. Show typing indicator
            await turn_context.send_activities([
                {"type": "typing"}
            ])

            # 2. Get or create thread
            thread_id = self.thread_store.get_thread_id(conversation_id)
            if not thread_id:
                thread = self.client.agents.create_thread()
                thread_id = thread.id
                self.thread_store.set_thread_id(conversation_id, thread_id)
                logger.info(f"New thread created: {thread_id}")

            # 3. Send user message to the agent
            self.client.agents.create_message(
                thread_id=thread_id,
                role="user",
                content=user_message
            )

            # 4. Run the agent
            run = self.client.agents.create_and_process_run(
                thread_id=thread_id,
                agent_id=self.agent_id
            )

            # 5. Check run status
            if run.status == "failed":
                logger.error(f"Agent run failed: {run.last_error}")
                card = create_error_card(
                    "The AI assistant encountered an error. Please try again."
                )
                await turn_context.send_activity(
                    MessageFactory.attachment(card)
                )
                return

            # 6. Get the response
            messages = self.client.agents.list_messages(thread_id=thread_id)
            assistant_reply = messages.data[0].content[0].text.value

            # 7. Send formatted response
            card = create_response_card(assistant_reply)
            await turn_context.send_activity(
                MessageFactory.attachment(card)
            )

        except Exception as e:
            logger.exception(f"Error processing message: {e}")
            card = create_error_card(
                "I'm having trouble connecting to the AI service. "
                "Please try again in a moment."
            )
            await turn_context.send_activity(
                MessageFactory.attachment(card)
            )

    async def on_members_added_activity(self, members_added, turn_context: TurnContext):
        """Welcome new members when they join the conversation."""
        for member in members_added:
            if member.id != turn_context.activity.recipient.id:
                await turn_context.send_activity(
                    "👋 Hi! I'm your AI assistant powered by Azure AI Foundry. "
                    "Ask me anything and I'll do my best to help!"
                )
```

---

## Step 6: Create the HTTP Trigger Entry Point

### 6.1 Update `bot/__init__.py`

```python
"""Azure Function HTTP trigger that handles Bot Framework messages."""

import os
import logging
import azure.functions as func
from botbuilder.core import (
    BotFrameworkAdapterSettings,
    BotFrameworkAdapter,
)
from botbuilder.core.integration import aiohttp_error_middleware

import sys
sys.path.insert(0, os.path.join(os.path.dirname(__file__), ".."))
from shared.foundry_bot import FoundryTeamsBot

# Configure adapter
settings = BotFrameworkAdapterSettings(
    app_id=os.environ.get("MICROSOFT_APP_ID", ""),
    app_password=os.environ.get("MICROSOFT_APP_PASSWORD", "")
)
adapter = BotFrameworkAdapter(settings)

# Create bot instance
bot = FoundryTeamsBot()


async def main(req: func.HttpRequest) -> func.HttpResponse:
    """Process incoming Bot Framework activities."""
    if req.method == "POST":
        if "application/json" not in (req.headers.get("Content-Type", "")):
            return func.HttpResponse(status_code=415)

        body = req.get_json()
        auth_header = req.headers.get("Authorization", "")

        from botbuilder.schema import Activity
        activity = Activity().deserialize(body)

        try:
            response = await adapter.process_activity(
                activity, auth_header, bot.on_turn
            )
            if response:
                return func.HttpResponse(
                    body=response.body,
                    status_code=response.status,
                    headers=dict(response.headers) if response.headers else {}
                )
            return func.HttpResponse(status_code=200)
        except Exception as e:
            logging.exception(f"Error processing activity: {e}")
            return func.HttpResponse(status_code=500)
    else:
        return func.HttpResponse(
            "Teams Bot endpoint is running. POST Bot Framework activities here.",
            status_code=200
        )
```

### 6.2 Update `bot/function.json`

```json
{
  "scriptFile": "__init__.py",
  "bindings": [
    {
      "authLevel": "anonymous",
      "type": "httpTrigger",
      "direction": "in",
      "name": "req",
      "methods": ["get", "post"],
      "route": "api/messages"
    },
    {
      "type": "http",
      "direction": "out",
      "name": "$return"
    }
  ]
}
```

---

## Step 7: Register the Azure Bot

### 7.1 Create Azure Resources

```bash
# Set variables
RG="rg-teams-foundry-bot"
LOCATION="eastus"
BOT_NAME="foundry-teams-bot"
STORAGE_NAME="stfoundrybot$(date +%s)"
FUNC_APP_NAME="func-foundry-bot"

# Create resource group
az group create --name $RG --location $LOCATION

# Create storage account (for Function App + Thread Store)
az storage account create \
  --name $STORAGE_NAME \
  --resource-group $RG \
  --location $LOCATION \
  --sku Standard_LRS

# Create Function App
az functionapp create \
  --name $FUNC_APP_NAME \
  --resource-group $RG \
  --storage-account $STORAGE_NAME \
  --consumption-plan-location $LOCATION \
  --runtime python \
  --runtime-version 3.10 \
  --functions-version 4 \
  --os-type Linux

# Create Bot registration
az bot create \
  --resource-group $RG \
  --name $BOT_NAME \
  --kind registration \
  --endpoint "https://$FUNC_APP_NAME.azurewebsites.net/api/messages" \
  --sku F0

# Enable Teams channel
az bot msteams create --name $BOT_NAME --resource-group $RG
```

### 7.2 Configure App Settings

```bash
# Get the storage connection string
STORAGE_CONN=$(az storage account show-connection-string \
  --name $STORAGE_NAME \
  --resource-group $RG \
  --query connectionString -o tsv)

# Set Function App configuration
az functionapp config appsettings set \
  --name $FUNC_APP_NAME \
  --resource-group $RG \
  --settings \
    "FOUNDRY_PROJECT_CONN_STR=<your-connection-string>" \
    "FOUNDRY_AGENT_ID=<your-agent-id>" \
    "MICROSOFT_APP_ID=<from-bot-registration>" \
    "MICROSOFT_APP_PASSWORD=<from-bot-registration>" \
    "AZURE_STORAGE_CONNECTION_STRING=$STORAGE_CONN"
```

---

## Step 8: Deploy and Test

### 8.1 Deploy to Azure

```bash
# Deploy the function app
func azure functionapp publish $FUNC_APP_NAME
```

### 8.2 Test with Bot Framework Emulator (Local)

1. Start the function locally: `func start`
2. Open Bot Framework Emulator
3. Connect to `http://localhost:7071/api/messages`
4. Send a test message and verify the response

### 8.3 Test in Microsoft Teams

1. Create a Teams app manifest (`manifest.json`):

```json
{
  "$schema": "https://developer.microsoft.com/en-us/json-schemas/teams/v1.16/MicrosoftTeams.schema.json",
  "manifestVersion": "1.16",
  "version": "1.0.0",
  "id": "<your-bot-app-id>",
  "developer": {
    "name": "AI Foundry Training",
    "websiteUrl": "https://example.com",
    "privacyUrl": "https://example.com/privacy",
    "termsOfUseUrl": "https://example.com/terms"
  },
  "name": {
    "short": "AI Foundry Bot",
    "full": "AI Foundry Teams Bot"
  },
  "description": {
    "short": "AI assistant powered by Azure AI Foundry",
    "full": "An intelligent assistant that uses Azure AI Foundry agents."
  },
  "icons": {
    "outline": "outline.png",
    "color": "color.png"
  },
  "accentColor": "#0078D4",
  "bots": [
    {
      "botId": "<your-bot-app-id>",
      "scopes": ["personal", "team", "groupChat"],
      "supportsFiles": false,
      "isNotificationOnly": false
    }
  ]
}
```

2. Package the manifest into a `.zip` file with two icon PNGs (32x32 outline, 192x192 color)
3. In Teams, go to **Apps → Manage your apps → Upload a custom app**
4. Upload the zip and start chatting!

---

## Step 9: Verify Success

### ✅ Completion Checklist

- [ ] Bot responds to messages in Teams (or Bot Framework Emulator)
- [ ] Agent maintains context across multiple messages in the same conversation
- [ ] Responses are formatted as Adaptive Cards
- [ ] Feedback buttons (👍/👎) work and are logged
- [ ] New conversations get a welcome message
- [ ] Errors are handled gracefully with user-friendly messages

### Test Scenarios

| # | Test | Expected Result |
|---|------|-----------------|
| 1 | Send "Hello" | Welcome or greeting response in Adaptive Card |
| 2 | Ask "What is Azure AI Foundry?" | Informative answer with details about the platform |
| 3 | Follow up with "Tell me more about agents" | Context-aware answer that builds on previous response |
| 4 | Click 👍 button | "Thanks for your feedback!" confirmation |
| 5 | Send empty message or card action | Graceful handling, no crash |

---

## 🏆 Stretch Goals

1. **Streaming Responses:** Modify the bot to stream tokens progressively using proactive messaging
2. **File Attachments:** Allow users to upload files that get passed to the agent as knowledge
3. **Command Router:** Add `/reset` command to start a new conversation thread
4. **Usage Tracking:** Log token usage per user to Application Insights
5. **Multi-Agent:** Route messages to different agents based on keywords or slash commands

---

## 🧹 Cleanup

When you're done, remove the Azure resources to avoid charges:

```bash
az group delete --name rg-teams-foundry-bot --yes --no-wait
```

---

## 🔗 Resources

- [Azure AI Foundry Agent Service — Overview](https://learn.microsoft.com/azure/ai-services/agents/overview)
- [Bot Framework SDK for Python](https://learn.microsoft.com/azure/bot-service/bot-service-quickstart-create-bot)
- [Adaptive Cards Designer](https://adaptivecards.io/designer/)
- [Teams App Manifest Schema](https://learn.microsoft.com/microsoftteams/platform/resources/schema/manifest-schema)
- [Bot Framework Emulator](https://github.com/microsoft/BotFramework-Emulator)

---

> **Note:** This lab uses **Azure AI Foundry** — not Azure Machine Learning. AI Foundry is the unified platform for building, deploying, and orchestrating AI agents.
