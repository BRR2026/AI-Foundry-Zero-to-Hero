# Module 40 — Assessment

## Enterprise Case Studies & Partner Success Patterns

**Total Questions:** 10  
**Time Limit:** 20 minutes  
**Passing Score:** 80% (8 of 10 correct)  

---

### Question 1

In the financial services customer support case study, which architectural pattern enabled a 73% automated resolution rate while minimizing hallucination risk?

- A) Single monolithic agent with comprehensive system prompt
- B) Multi-agent triage with routing agent delegating to specialist agents
- C) Direct API integration without any AI agent layer
- D) Fine-tuned model deployed as a standalone service

**Correct Answer:** B

**Explanation:** The multi-agent triage pattern uses a lightweight routing agent to classify intent and delegate to specialist agents with constrained instructions and tools. By limiting each specialist's scope, hallucination risk is reduced because each agent only needs to reason within a narrow domain. Option A (monolithic agent) is explicitly identified as a pitfall in Module 40.

---

### Question 2

In the insurance document processing case study, what confidence threshold was used for straight-through processing (auto-approval)?

- A) 0.70
- B) 0.85
- C) 0.95
- D) 0.99

**Correct Answer:** C

**Explanation:** The case study used a confidence threshold of 0.95 for auto-approval of claims. Claims scoring below this threshold were routed to human adjusters for review. This threshold was calibrated through 10,000+ evaluation runs in Azure AI Foundry, balancing automation throughput against accuracy requirements in the regulated insurance industry.

---

### Question 3

Which of the following is NOT one of the six common enterprise AI patterns identified in Module 40?

- A) Multi-Agent Triage & Routing
- B) Extract → Reason → Act Pipeline
- C) Autonomous Self-Improving Agent
- D) Compliance-First Agent Design

**Correct Answer:** C

**Explanation:** The six patterns identified are: (1) Multi-Agent Triage & Routing, (2) Extract → Reason → Act Pipeline, (3) RAG + Agent Hybrid, (4) Compliance-First Agent Design, (5) Human-in-the-Loop Escalation, and (6) Conversational Analytics Interface. "Autonomous Self-Improving Agent" is not one of the patterns and could represent a risky anti-pattern in regulated enterprise environments.

---

### Question 4

The healthcare compliance case study achieved 100% HIPAA audit pass rate. Which combination of measures made this possible?

- A) Public endpoints with API key authentication and basic logging
- B) Private endpoints, customer-managed encryption keys, PHI detection guardrails, and automated evaluation pipelines
- C) Standard Azure security defaults with manual compliance reviews
- D) Third-party compliance tools running alongside Azure AI Foundry

**Correct Answer:** B

**Explanation:** The healthcare case study deployed all AI Foundry resources within a VNet with private endpoints (no public internet data transit), customer-managed encryption keys via Azure Key Vault with HSM backing, custom content safety filters for PHI detection and redaction, and automated evaluation pipelines running nightly against 500+ compliance test cases. This defense-in-depth approach ensured continuous compliance.

---

### Question 5

What percentage of project effort should be allocated to knowledge base curation to avoid the "under-investing in knowledge quality" pitfall?

- A) 5-10%
- B) 15-20%
- C) 30-40%
- D) 50-60%

**Correct Answer:** C

**Explanation:** Module 40 recommends allocating 30-40% of project effort to knowledge base curation. This includes implementing freshness policies, de-duplication, chunk-quality evaluation, and content structure optimization before connecting to Azure AI Search. Teams that under-invest in knowledge quality often find that their RAG answers are technically correct but unhelpful due to outdated, duplicated, or poorly structured content.

---

### Question 6

In the AI practice roadmap, what is the recommended activity for Phase 2 (Pilot)?

- A) Set up the Azure AI Foundry environment and train the team
- B) Deploy the pilot to 50-100 internal users, measure everything, and build an internal case study
- C) Establish an AI governance board and create an internal AI playbook
- D) Experiment with autonomous agent orchestration for complex workflows

**Correct Answer:** B

**Explanation:** Phase 2 (Pilot, Months 3-4) focuses on delivering the first production win with a focused scope. Key activities include deploying to 50-100 internal users for feedback, running weekly evaluation cycles, documenting architecture decisions and lessons learned, and building an internal case study for stakeholder buy-in. Phase 1 covers environment setup (A), Phase 4 covers governance (C), and Phase 5 covers innovation experiments (D).

---

### Question 7

The manufacturing case study demonstrates which key architectural concept?

- A) Replacing the entire IoT infrastructure with AI Foundry
- B) Using AI Foundry agents as a "reasoning layer" on top of existing data infrastructure
- C) Building custom machine learning models from scratch in AI Foundry
- D) Migrating all sensor data storage to AI Foundry's built-in database

**Correct Answer:** B

**Explanation:** The manufacturing case study demonstrates using AI Foundry agents as a "reasoning layer" over existing data infrastructure. The IoT platform (Azure IoT Hub, Azure Data Explorer) continues to handle data ingestion and storage, while AI Foundry agents provide natural-language interfaces for plant managers who aren't data scientists. This approach maximizes the value of existing infrastructure investments while adding AI capabilities.

---

### Question 8

Which pitfall is described as "an impressive demo built in 2 days that takes 6 months to productionize"?

- A) Token economics surprise
- B) Single-agent monolith
- C) Demo-to-production gap
- D) Under-investing in adversarial testing

**Correct Answer:** C

**Explanation:** The "demo-to-production gap" pitfall occurs when demos skip critical production concerns: error handling, authentication, rate limiting, content safety, and evaluation. The mitigation is to use Azure AI Foundry's production-ready agent infrastructure from day one — including managed identity, content safety filters, and evaluation pipelines — even during prototyping, so that the path to production is incremental rather than a complete rewrite.

---

### Question 9

According to the pattern selection decision matrix, which pattern has the shortest time to production and smallest minimum team size?

- A) Multi-Agent Triage & Routing (4-6 weeks, 3-4 engineers)
- B) Extract → Reason → Act Pipeline (8-12 weeks, 4-6 engineers)
- C) RAG + Agent Hybrid (3-5 weeks, 2-3 engineers)
- D) Compliance-First Agent Design (10-16 weeks, 5-8 engineers)

**Correct Answer:** C

**Explanation:** The RAG + Agent Hybrid pattern has the shortest time to production (3-5 weeks) and the smallest minimum team size (2-3 engineers). This makes it ideal for organizations seeking the fastest time-to-value. It combines retrieval-augmented generation via Azure AI Search with agentic tool use, allowing the agent to decide when to search vs. when to use function tools based on query complexity. The typical ROI timeline is also the shortest at 2-4 months.

---

### Question 10

What is the recommended healthy escalation rate range for the Human-in-the-Loop pattern, and why?

- A) 0-5% — Lower is always better for cost efficiency
- B) 5-10% — Minimizes human involvement
- C) 15-25% — Lower suggests overconfidence in the AI
- D) 40-50% — Ensures human oversight on most interactions

**Correct Answer:** C

**Explanation:** Module 40 recommends a human escalation rate between 15-25% for the Human-in-the-Loop pattern. A rate lower than 15% may suggest the AI is overconfident and potentially making decisions it shouldn't. A rate higher than 25% may indicate the AI isn't providing enough value to justify the deployment. The 15-25% sweet spot ensures that complex, ambiguous, or high-stakes interactions receive appropriate human oversight while routine queries are handled efficiently by the AI.

---

## Answer Key

| # | Answer | Topic |
|---|--------|-------|
| 1 | B | Multi-agent triage pattern |
| 2 | C | Confidence-gated automation threshold |
| 3 | C | Six enterprise AI patterns |
| 4 | B | HIPAA compliance architecture |
| 5 | C | Knowledge curation effort allocation |
| 6 | B | AI practice roadmap Phase 2 |
| 7 | B | AI as reasoning layer pattern |
| 8 | C | Demo-to-production gap pitfall |
| 9 | C | RAG + Agent pattern advantages |
| 10 | C | Human-in-the-loop escalation rate |

---

*Module 40 of 45 — Azure AI Foundry: Zero to Hero Training Series*  
*Arc 8: Advanced Patterns & Enterprise*
