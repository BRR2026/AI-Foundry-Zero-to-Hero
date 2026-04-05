# Module 18: Multi-Turn Conversations & Memory — Assessment

## Azure AI Foundry: Zero to Hero Training Course

**ARC 4: AI AGENTS** | Module 18 of 45 | April 2026

---

## Instructions

- 10 multiple-choice questions
- Select the single best answer for each question
- Passing score: 80% (8 out of 10 correct)
- Answers are provided at the end of this document

---

## Questions

### Question 1
What is a **thread** in Azure AI Agent Service?

- A) A background process that executes agent functions in parallel
- B) An ordered collection of messages that represents a conversation
- C) A compute instance dedicated to running a single agent
- D) A queue of pending tool calls waiting to be processed

---

### Question 2
Which SDK method is used to create a new thread for a conversation?

- A) `client.agents.start_conversation()`
- B) `client.agents.create_thread()`
- C) `client.threads.new()`
- D) `client.agents.init_thread()`

---

### Question 3
What roles are supported for messages added to a thread?

- A) system and user
- B) user and assistant
- C) user, assistant, and function
- D) system, user, assistant, and tool

---

### Question 4
What is **in-context memory** (short-term memory) in a multi-turn conversation?

- A) User preferences stored in an external database between sessions
- B) A vector index of previous conversation embeddings
- C) Conversation history stored within the thread and context window
- D) A cache of tool call results persisted to disk

---

### Question 5
What happens when a conversation exceeds the model's context window?

- A) The agent automatically creates a new thread and continues
- B) An error is raised and the conversation must be restarted
- C) A truncation strategy is applied to manage the context
- D) The model's context window is dynamically expanded to fit

---

### Question 6
What does setting `truncation_strategy` type to `"auto"` do?

- A) It compresses all messages using a summarization model before sending
- B) It automatically manages context by dropping older messages as needed
- C) It splits the conversation across multiple parallel threads
- D) It disables truncation and sends all messages regardless of length

---

### Question 7
What does the `last_messages` truncation strategy do?

- A) Sends only the final assistant response to the model
- B) Archives all messages except the system prompt
- C) Keeps only the N most recent messages in the context
- D) Removes duplicate messages from the conversation history

---

### Question 8
What does the `max_prompt_tokens` parameter control?

- A) The maximum number of tokens allowed in the model's completion output
- B) The total token budget for both the prompt and the response combined
- C) The maximum number of tokens allowed for the prompt input
- D) The maximum number of tokens a single message can contain

---

### Question 9
Which Azure service is best suited for storing user preferences as JSON documents for cross-session memory?

- A) Azure Blob Storage
- B) Azure SQL Database
- C) Azure Cosmos DB
- D) Azure Table Storage

---

### Question 10
What is the recommended pattern for implementing cross-session memory in a multi-turn agent?

- A) Embed all past conversations into the system prompt at startup
- B) Load preferences from an external store → inject into the prompt → process the conversation → extract and save updates
- C) Store the entire thread as a binary blob and reload it for every new session
- D) Use a fine-tuned model that has memorized individual user preferences

---

## Answer Key

| Question | Answer | Explanation |
|----------|--------|-------------|
| 1 | B | A thread in Azure AI Agent Service is an ordered collection of messages that represents a conversation between a user and an agent. |
| 2 | B | The `client.agents.create_thread()` method is the correct SDK call to create a new conversation thread. |
| 3 | B | Threads support two message roles: **user** (for human input) and **assistant** (for agent responses). |
| 4 | C | In-context (short-term) memory refers to the conversation history stored within the thread and kept inside the model's context window during a session. |
| 5 | C | When the conversation exceeds the context window, a truncation strategy is applied to reduce the number of messages sent to the model. |
| 6 | B | The `"auto"` truncation strategy automatically manages context by dropping older messages when the conversation grows beyond the context window. |
| 7 | C | The `last_messages` truncation strategy keeps only the N most recent messages, discarding everything older to fit within the context window. |
| 8 | C | The `max_prompt_tokens` parameter sets the maximum number of tokens allowed for the prompt (input) sent to the model, helping control cost and context usage. |
| 9 | C | Azure Cosmos DB is the best fit for storing user preferences as JSON documents because it is a fully managed NoSQL database designed for flexible, schema-free JSON data with low-latency reads. |
| 10 | B | The recommended cross-session memory pattern is: load user preferences from an external store, inject them into the prompt context, process the conversation, then extract any updates and persist them back to the store. |
