# Module 13: Building Your First RAG Application

## Quiz Assessment

**Course:** Azure AI Foundry — Zero to Hero (Module 13 of 45)
**Arc:** ARC 3 · DATA & RAG
**Total Questions:** 10 | **Passing Score:** 70% (7/10)
**Time Limit:** 15 minutes

---

### Question 1

**What does RAG stand for in the context of AI applications?**

- A) Rapid Application Generation
- B) Retrieval-Augmented Generation
- C) Real-time Analytical Grouping
- D) Response-Aware Grounding

**Correct Answer:** B

**Explanation:** RAG stands for Retrieval-Augmented Generation. It is an architecture pattern that retrieves relevant documents from a knowledge base, augments the prompt with that context, and then generates a grounded response using a language model.

---

### Question 2

**What are the three stages of the RAG architecture in the correct order?**

- A) Generate → Augment → Retrieve
- B) Augment → Retrieve → Generate
- C) Retrieve → Augment → Generate
- D) Retrieve → Generate → Augment

**Correct Answer:** C

**Explanation:** The RAG pipeline follows three sequential steps: (1) Retrieve relevant documents from a search index, (2) Augment the prompt by injecting the retrieved context alongside the user's question, and (3) Generate a response using the language model grounded in that context.

---

### Question 3

**Which Azure service provides the retrieval component for RAG applications built in Azure AI Foundry?**

- A) Azure Cosmos DB
- B) Azure Blob Storage
- C) Azure AI Search
- D) Azure SQL Database

**Correct Answer:** C

**Explanation:** Azure AI Search is the primary retrieval service used in RAG architectures within Azure AI Foundry. It supports vector search, semantic ranking, and hybrid search, making it ideal for finding relevant document chunks to ground model responses.

---

### Question 4

**What is the primary problem that RAG solves compared to using a language model alone?**

- A) It reduces the cost of API calls
- B) It eliminates the need for prompt engineering
- C) It grounds responses in factual, domain-specific data to reduce hallucinations
- D) It makes the model respond faster

**Correct Answer:** C

**Explanation:** The primary benefit of RAG is grounding model responses in your own data, which significantly reduces hallucinations. Without RAG, models rely solely on their training data and may fabricate information when asked domain-specific questions.

---

### Question 5

**In Azure AI Foundry, the "On Your Data" feature allows you to:**

- A) Fine-tune a model with your proprietary dataset
- B) Connect a deployed model to an Azure AI Search index for grounded chat without writing code
- C) Upload data directly into the model's training pipeline
- D) Export model weights to use with your data offline

**Correct Answer:** B

**Explanation:** The "On Your Data" feature in Azure AI Foundry provides a no-code way to connect a deployed chat model to an Azure AI Search index. This enables the model to ground its responses in your indexed documents directly from the portal, without writing any application code.

---

### Question 6

**When building a RAG pipeline with the Python SDK, what must you provide to the chat completion call to enable grounded responses?**

- A) A fine-tuning job ID
- B) A data source configuration pointing to your Azure AI Search index
- C) A list of all documents in plain text
- D) A connection string to Azure Blob Storage

**Correct Answer:** B

**Explanation:** When using the Azure AI Foundry Python SDK for RAG, you configure a data source (typically an `AzureSearchChatExtensionConfiguration`) that points to your Azure AI Search index. This tells the model where to retrieve context from when generating responses.

---

### Question 7

**Which search strategy combines keyword matching with vector similarity for the best retrieval results?**

- A) Full-text search
- B) Semantic search
- C) Hybrid search
- D) Fuzzy search

**Correct Answer:** C

**Explanation:** Hybrid search combines traditional keyword (BM25) matching with vector similarity search. This approach leverages the strengths of both methods — keyword precision and semantic understanding — to deliver the most relevant document chunks for RAG applications.

---

### Question 8

**What is the recommended approach for handling citations in RAG application responses?**

- A) Ignore citations since the model already knows the information
- B) Parse citation metadata from the response and map references back to source documents with page numbers or section identifiers
- C) Simply append the entire source document at the end of each response
- D) Ask the user to verify information independently

**Correct Answer:** B

**Explanation:** Proper citation handling involves parsing the citation metadata returned by the RAG pipeline, mapping each reference back to its source document, and providing users with specific page numbers, sections, or clickable links. This builds trust and allows users to verify information.

---

### Question 9

**When designing a system prompt for a RAG chatbot, which instruction helps prevent the model from answering beyond the retrieved context?**

- A) "Answer all questions using your general knowledge."
- B) "If the retrieved documents do not contain relevant information, say you don't have enough information to answer."
- C) "Always provide an answer, even if you are unsure."
- D) "Ignore the provided context and use your training data."

**Correct Answer:** B

**Explanation:** A well-designed RAG system prompt instructs the model to only answer based on the retrieved context. Including an explicit instruction to acknowledge when information is not found prevents hallucination and maintains response trustworthiness.

---

### Question 10

**How does RAG differ from fine-tuning as an approach to incorporating domain-specific knowledge?**

- A) RAG permanently modifies the model weights; fine-tuning retrieves documents at runtime
- B) RAG retrieves external knowledge at inference time; fine-tuning adjusts the model's weights during a training phase
- C) RAG requires retraining the model; fine-tuning does not
- D) There is no difference; they are the same technique

**Correct Answer:** B

**Explanation:** RAG retrieves relevant external knowledge at inference (query) time without modifying the model, while fine-tuning adjusts the model's internal weights through additional training. RAG is preferred when knowledge changes frequently, as you only need to update the search index rather than retrain the model.

---

## Answer Key

| Question | Answer |
|----------|--------|
| 1 | B |
| 2 | C |
| 3 | C |
| 4 | C |
| 5 | B |
| 6 | B |
| 7 | C |
| 8 | B |
| 9 | B |
| 10 | B |

---

*Azure AI Foundry: Zero to Hero Training — © 2026 Zero to Hero Training Team*
