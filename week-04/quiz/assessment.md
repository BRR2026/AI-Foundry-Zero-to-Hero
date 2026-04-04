# Week 4: Responsible AI Principles in Foundry — Assessment

## Azure AI Foundry — Zero to Hero Training Course

**Arc 1: Foundations | Week 4 of 45**

**Total Points: 100 | Time Limit: 45 minutes**

---

## Section 1: Multiple Choice (40 points — 4 points each)

### Question 1
Which of the following is **NOT** one of Microsoft's six Responsible AI principles?

A) Fairness
B) Profitability
C) Transparency
D) Accountability

---

### Question 2
Azure AI Content Safety evaluates content across how many primary harm categories?

A) 2
B) 4
C) 6
D) 8

---

### Question 3
What is the default content filter threshold in Azure AI Foundry for production deployments?

A) Low severity and above are blocked
B) Medium severity and above are blocked
C) Only High severity is blocked
D) All content is blocked by default

---

### Question 4
Which Responsible AI tool in Azure AI Foundry is specifically designed to prevent models from generating fabricated information not supported by source documents?

A) Content Safety Filters
B) Jailbreak Detection
C) Groundedness Detection
D) Protected Material Detection

---

### Question 5
Under the EU AI Act, which risk category requires an AI system to undergo a conformity assessment before deployment?

A) Minimal Risk
B) Limited Risk
C) High Risk
D) All risk categories

---

### Question 6
In Azure AI Foundry's content safety pipeline, at what stages are content filters applied?

A) Only on user input (prompt)
B) Only on model output (response)
C) On both user input and model output
D) Only during model training

---

### Question 7
Which type of jailbreak attack involves embedding malicious instructions within documents that the AI model retrieves and processes?

A) Direct Injection
B) Role-Playing Attack
C) Encoding Attack
D) Indirect Injection

---

### Question 8
What does a "Model Card" provide?

A) A credit card for model API usage billing
B) A standardized document describing a model's capabilities, limitations, and intended uses
C) A hardware specification for running the model
D) A license key to access the model

---

### Question 9
When configuring content filters in Azure AI Foundry, what does "annotate-only" mode do?

A) Blocks all content and logs the reason
B) Returns severity information in the response metadata without blocking content
C) Adds warning annotations visible to the end user
D) Sends content to a human reviewer for annotation

---

### Question 10
Which NIST AI RMF function focuses on establishing an organizational culture for AI risk management?

A) MAP
B) MEASURE
C) MANAGE
D) GOVERN

---

## Section 2: True or False (20 points — 4 points each)

### Question 11
**True or False**: Groundedness detection verifies that a model's response is absolutely factually correct.

---

### Question 12
**True or False**: Azure AI Foundry's jailbreak protection can detect multi-turn prompt injection attempts where harmful intent is built up gradually across messages.

---

### Question 13
**True or False**: The EU AI Act classifies all AI chatbots as "High Risk" systems requiring conformity assessment.

---

### Question 14
**True or False**: Protected Material Detection in Azure AI Foundry can identify both copyrighted text and licensed source code in model outputs.

---

### Question 15
**True or False**: Microsoft's Responsible AI principles state that AI systems should always make fully autonomous decisions without human oversight for maximum efficiency.

---

## Section 3: Short Answer (20 points — 5 points each)

### Question 16
List the four content safety harm categories used in Azure AI Content Safety, and briefly describe what each one covers.

---

### Question 17
Explain the difference between "groundedness" and "accuracy" in the context of AI evaluation. Why is this distinction important?

---

### Question 18
Describe two specific actions an organization can take to build a Responsible AI culture beyond just implementing technical tools.

---

### Question 19
What are the four severity levels used in Azure AI Content Safety filters, and what score ranges do they correspond to?

---

## Section 4: Scenario-Based Questions (20 points — 10 points each)

### Question 20
**Scenario**: Your company is deploying an AI-powered customer service chatbot for a healthcare insurance company using Azure AI Foundry. The chatbot will answer questions about policy coverage, claims status, and general health information.

**Part A** (5 points): Identify at least three specific Responsible AI risks associated with this deployment. Reference specific Microsoft Responsible AI principles for each risk.

**Part B** (5 points): Describe how you would configure Azure AI Foundry's content safety tools to mitigate these risks. Be specific about which tools you would use and how you would configure them.

---

### Bonus Question (5 bonus points)

**Scenario**: A colleague argues that strict content filters slow down their AI application and reduce the quality of responses, so they want to disable all content safety filters for their production deployment.

Write a brief response (3-5 sentences) explaining why this is a bad idea. Reference specific risks, real-world examples, and regulatory requirements in your answer.

---

## Answer Key

---

### Section 1: Multiple Choice

**Question 1: B) Profitability**
> Microsoft's six Responsible AI principles are: Fairness, Reliability & Safety, Privacy & Security, Inclusiveness, Transparency, and Accountability. Profitability is a business goal, not a Responsible AI principle.

**Question 2: B) 4**
> Azure AI Content Safety evaluates content across four primary harm categories: Violence, Hate, Sexual, and Self-Harm. Each category has multiple severity levels from Safe to High.

**Question 3: B) Medium severity and above are blocked**
> By default, Azure AI Foundry content filters are set to block content at Medium severity and above. This means Safe and Low severity content is allowed through, while Medium and High severity content is blocked.

**Question 4: C) Groundedness Detection**
> Groundedness Detection specifically checks whether a model's response is supported by the provided source documents. It's designed to catch "hallucinations" — content the model generates that isn't backed by actual sources.

**Question 5: C) High Risk**
> Under the EU AI Act, High Risk AI systems must undergo a conformity assessment, maintain technical documentation, implement risk management systems, and ensure human oversight. Examples include hiring tools, credit scoring, and medical devices.

**Question 6: C) On both user input and model output**
> Azure AI Foundry applies content filters at two stages: first on the user's input (prompt) and then on the model's output (response). This dual-layer approach catches harmful content regardless of whether it originates from the user or the model.

**Question 7: D) Indirect Injection**
> Indirect injection (also called indirect prompt injection) occurs when malicious instructions are embedded in external data sources (web pages, documents, emails) that the AI model retrieves and processes. The model then follows these hidden instructions as if they were from the user.

**Question 8: B) A standardized document describing a model's capabilities, limitations, and intended uses**
> A Model Card is like a "nutrition label" for AI models. It provides standardized documentation including model details, intended uses, out-of-scope uses, training data, performance metrics, limitations, and ethical considerations.

**Question 9: B) Returns severity information in the response metadata without blocking content**
> Annotate-only mode is useful for development and testing. It evaluates content against safety filters and returns the severity scores in the response metadata, but does not block any content. This allows developers to understand how filters would behave without affecting functionality.

**Question 10: D) GOVERN**
> The GOVERN function of the NIST AI RMF focuses on establishing and maintaining an organizational culture, policies, and processes for AI risk management. The other functions are MAP (identify risks), MEASURE (analyze risks), and MANAGE (treat risks).

---

### Section 2: True or False

**Question 11: FALSE**
> Groundedness detection verifies that a model's response is *consistent with and supported by the provided source documents*. It does NOT verify absolute factual correctness. A response can be grounded (all claims appear in the source) but still inaccurate if the source documents themselves contain errors. This distinction between groundedness and accuracy is crucial.

**Question 12: TRUE**
> Azure AI Foundry's jailbreak protection includes detection capabilities for various attack types, including multi-turn attacks where malicious intent is gradually escalated across conversation turns. The system analyzes patterns across the conversation to identify suspicious escalation.

**Question 13: FALSE**
> Under the EU AI Act, AI chatbots are generally classified as "Limited Risk" systems, which require transparency obligations (users must be informed they're interacting with AI). They are NOT classified as High Risk. However, chatbots used in high-risk domains (healthcare, finance) may face additional requirements based on their application context.

**Question 14: TRUE**
> Protected Material Detection in Azure AI Foundry operates in two modes: Text mode (detecting known copyrighted text like books, articles, and lyrics) and Code mode (detecting licensed source code from known repositories). Both modes can be enabled in content filter configurations.

**Question 15: FALSE**
> Microsoft's Accountability principle emphasizes that humans should maintain meaningful oversight of AI systems. The principles explicitly call for human oversight mechanisms, appeal processes, and clear accountability chains. Fully autonomous AI decision-making without human oversight violates this principle, especially for high-stakes decisions.

---

### Section 3: Short Answer

**Question 16: Expected Answer**

The four content safety harm categories are:

1. **Violence**: Content depicting physical harm, threats, or weapons — including graphic violence, threats to harm others, and instructions for causing physical harm.

2. **Hate**: Content that attacks or discriminates against people based on identity characteristics — including slurs, stereotyping, dehumanization, and discriminatory language targeting protected groups.

3. **Sexual**: Sexually explicit or suggestive content — including explicit descriptions of sexual acts, sexual solicitation, and objectification.

4. **Self-Harm**: Content related to self-injury or suicide — including instructions for self-harm, suicide promotion or glorification, and content promoting eating disorders.

**Question 17: Expected Answer**

**Groundedness** measures whether a model's claims are supported by the provided source documents. It checks: "Did the model make this up, or is it in the source material?"

**Accuracy** measures whether the information is actually true in the real world.

This distinction matters because a response can be:
- **Grounded but inaccurate**: The model faithfully reports what's in the source document, but the source document itself contains errors.
- **Ungrounded but accurate**: The model states something true but not found in the provided sources (a "lucky hallucination").

This distinction is important because groundedness detection can only verify consistency with sources, not absolute truth. Organizations need both groundedness detection AND source quality verification for reliable AI outputs.

**Question 18: Expected Answer**

Two actions beyond technical tools:

1. **Establish a cross-functional AI Ethics Board/Committee**: Create a diverse committee including engineers, ethicists, legal experts, domain specialists, and community representatives who review high-risk AI deployments, set organizational AI ethics policies, and serve as an escalation point for ethical concerns. This ensures diverse perspectives are considered and that decisions about AI deployment aren't made solely by technical teams.

2. **Implement mandatory Responsible AI training and safe reporting channels**: Require regular Responsible AI training for all employees (not just technical staff) to build AI literacy across the organization. Simultaneously, create confidential channels for employees to report ethical concerns about AI projects without fear of retaliation. This builds a culture where responsible AI is everyone's responsibility and concerns are surfaced early.

*(Other valid answers include: conducting regular impact assessments, publishing transparency reports, engaging with affected communities, implementing red teaming programs, creating clear AI use policies, requiring sign-off processes for AI deployments, etc.)*

**Question 19: Expected Answer**

| Severity Level | Score Range | Description |
|---|---|---|
| **Safe** | 0-1 | Content is benign with no harmful elements |
| **Low** | 2-3 | Mildly concerning content that may or may not need filtering |
| **Medium** | 4-5 | Moderately harmful content typically filtered in production |
| **High** | 6-7 | Severely harmful content that should always be filtered |

---

### Section 4: Scenario-Based Questions

**Question 20: Expected Answer**

**Part A — Responsible AI Risks:**

1. **Medical misinformation risk (Reliability & Safety)**: The chatbot might generate inaccurate health information or claim coverage that doesn't exist, leading users to make harmful health decisions based on incorrect AI advice.

2. **Bias in service quality (Fairness)**: The chatbot might perform differently across demographic groups — for example, providing less helpful responses to users who speak English as a second language or using jargon that disadvantages users with lower health literacy.

3. **Privacy violation risk (Privacy & Security)**: The chatbot handles sensitive health information (PHI) and insurance details. Improper handling could violate HIPAA regulations and expose personal health data through model memorization or logging.

4. **Lack of transparency (Transparency)**: Users might not realize they're interacting with an AI and might trust AI-provided health information as if it came from a medical professional, leading to inappropriate reliance on AI guidance.

5. **Self-harm content risk (Reliability & Safety)**: Users discussing health concerns might express thoughts related to self-harm. The chatbot must handle these situations appropriately and provide crisis resources.

**Part B — Content Safety Configuration:**

1. **Content Safety Filters**: Set all four categories (Violence, Hate, Sexual, Self-Harm) to **Medium** threshold for both input and output. Self-Harm should be set to **Low** threshold to be extra cautious, given the healthcare context where users might discuss mental health.

2. **Jailbreak Detection**: Enable for input to prevent manipulation of the chatbot into providing unauthorized medical or coverage information.

3. **Groundedness Detection**: Enable for output and pair with a RAG implementation that retrieves from verified policy documents and approved health information sources. This prevents the chatbot from hallucinating coverage details or health information.

4. **Custom Blocklists**: Create blocklists that include:
   - Medical diagnosis terms (the bot should not diagnose)
   - Prescription drug recommendations
   - Specific phrases like "I guarantee coverage" or "you're definitely covered"
   - Competitor insurance company names

5. **Protected Material Detection (Text)**: Enable to prevent the chatbot from reproducing copyrighted health information verbatim.

6. **Human escalation**: Configure the system to escalate to a human agent when the chatbot detects high-sensitivity topics (mental health, coverage disputes, urgent medical situations).

---

### Bonus Question: Expected Answer

Disabling all content safety filters in production is extremely risky and should not be done. First, without filters, the chatbot could generate harmful, offensive, or discriminatory content that damages your brand and harms users — as demonstrated by incidents like Microsoft Tay, which went from friendly chatbot to generating offensive content in under 16 hours. Second, the EU AI Act and other regulations increasingly require AI safety measures, and operating without them could expose the company to fines of up to €35 million or 7% of global revenue. Third, the performance impact of content filters is minimal (typically milliseconds of latency) compared to the catastrophic cost of a single safety incident, which can run into millions of dollars in legal fees, settlements, and lost business. The correct approach is to tune filter sensitivity, not remove filters entirely.

---

## Grading Rubric Summary

| Section | Points | Questions |
|---|---|---|
| Multiple Choice | 40 | 10 questions × 4 points |
| True/False | 20 | 5 questions × 4 points |
| Short Answer | 20 | 4 questions × 5 points |
| Scenario-Based | 20 | 2 questions × 10 points |
| **Total** | **100** | **20 questions + 1 bonus** |
| Bonus | +5 | 1 bonus question |

### Grade Scale

| Grade | Score Range |
|---|---|
| A (Excellent) | 90-100+ |
| B (Good) | 80-89 |
| C (Satisfactory) | 70-79 |
| D (Needs Improvement) | 60-69 |
| F (Unsatisfactory) | Below 60 |

---

*© 2025 Azure AI Foundry Training Course. Week 4 of 45. Arc 1: Foundations.*
