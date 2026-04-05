# Module 34 — Assessment: Responsible AI Dashboards & Content Safety at Scale

## Azure AI Foundry: Zero to Hero | Arc 7: Security, Governance & Compliance

**Instructions:** Select the best answer for each question. Each question has exactly one correct answer. Score ≥ 80% (8/10) to pass.

---

### Question 1

What is the primary purpose of the Responsible AI Dashboard in Azure AI Foundry?

- A) To replace Azure Monitor for all AI-related monitoring tasks
- B) To provide a centralized, interactive view of fairness, safety, and quality metrics across deployed models
- C) To automatically retrain models when bias is detected
- D) To generate compliance reports for submission to the EU AI Act regulatory body

<details>
<summary>Show Answer</summary>

**Correct Answer: B**

The Responsible AI Dashboard provides a centralized, interactive view of how AI systems perform across Microsoft's Responsible AI principles, including fairness, safety, and quality metrics. It does not replace Azure Monitor (A), automatically retrain models (C), or generate regulatory submission reports directly (D).

</details>

---

### Question 2

When configuring custom blocklists in Azure AI Content Safety, what are the maximum limits per resource?

- A) 10 blocklists with 1,000 items each
- B) 50 blocklists with 5,000 items each
- C) 100 blocklists with 10,000 items each
- D) Unlimited blocklists with 100,000 items each

<details>
<summary>Show Answer</summary>

**Correct Answer: C**

Azure AI Content Safety supports up to 100 blocklists per resource, with up to 10,000 items per blocklist. This provides sufficient capacity for most enterprise deployments, but organizations with larger vocabularies should consider combining multiple targeted blocklists.

</details>

---

### Question 3

In the multi-layer content safety pipeline architecture, what is the correct order of processing?

- A) Model Inference → Input Filter → Output Filter → Custom Blocklists
- B) Custom Blocklists → Input Filter → Model Inference → Output Filter
- C) Input Filter → Custom Blocklists → Model Inference → Output Filter
- D) Input Filter → Model Inference → Custom Blocklists → Output Filter

<details>
<summary>Show Answer</summary>

**Correct Answer: C**

The correct enterprise pipeline order is: (1) Input safety filter checks for built-in category violations, (2) Custom blocklists check for domain-specific blocked terms, (3) Model inference processes the validated input, (4) Output safety filter checks the model's response before delivery. This layered approach provides defense-in-depth.

</details>

---

### Question 4

Which fairness metric uses the "80% rule" as its threshold guidance?

- A) Equalized Odds
- B) Quality Disparity
- C) Demographic Parity
- D) Stereotyping Score

<details>
<summary>Show Answer</summary>

**Correct Answer: C**

Demographic Parity measures the equal probability of positive outcomes across groups and uses a ratio of ≥ 0.8 (the "80% rule") as its threshold guidance. This means the least-favored group should receive positive outcomes at least 80% as often as the most-favored group.

</details>

---

### Question 5

For a Critical severity AI safety incident (e.g., self-harm instructions being generated), what is the recommended response SLA?

- A) 1 hour
- B) 30 minutes
- C) 15 minutes
- D) 4 hours

<details>
<summary>Show Answer</summary>

**Correct Answer: C**

Critical severity AI safety incidents—such as self-harm instructions, PII leakage at scale, or CSAM generation—require a 15-minute response SLA. This includes alert validation, severity confirmation, and initiation of containment measures. The urgency reflects the immediate threat to user safety.

</details>

---

### Question 6

What is the recommended approach for monitoring AI fairness in production without collecting protected demographic data from end users?

- A) Infer demographics from user behavior patterns and apply real-time fairness corrections
- B) Use controlled evaluation datasets with known demographic attributes and monitor aggregate output patterns
- C) Ask users to voluntarily self-report their demographics at sign-up
- D) Rely solely on pre-deployment bias testing and skip production monitoring

<details>
<summary>Show Answer</summary>

**Correct Answer: B**

The recommended approach is to use controlled evaluation datasets with known demographic attributes for offline fairness testing, and monitor production systems through aggregate statistical tests on anonymized output patterns. Inferring demographics (A) raises privacy concerns, voluntary self-reporting (C) introduces selection bias, and skipping production monitoring (D) misses real-world fairness drift.

</details>

---

### Question 7

In the incident response playbook, what is the primary action during the Containment phase for a high-severity jailbreak success?

- A) Retrain the model with adversarial examples
- B) Add the offending pattern to the hotfix blocklist and reduce model temperature
- C) Shut down the entire AI service permanently
- D) Notify the media and issue a public statement

<details>
<summary>Show Answer</summary>

**Correct Answer: B**

During the Containment phase, the primary actions are to add the offending pattern to the hotfix/emergency blocklist and reduce model temperature or switch to a more restrictive safety configuration. This stops the immediate threat while maintaining service availability. Retraining (A) is a remediation-phase activity, permanent shutdown (C) is disproportionate, and media notification (D) occurs only if required by regulation after investigation.

</details>

---

### Question 8

Which Azure AI Content Safety category uses a binary detection model (Detected / Not Detected) rather than a severity scale?

- A) Hate & Fairness
- B) Violence
- C) Jailbreak Attacks
- D) Sexual Content

<details>
<summary>Show Answer</summary>

**Correct Answer: C**

Jailbreak Attacks and Protected Material use a binary detection model (Detected / Not Detected) rather than the 0–6 severity scale used by the four main content categories (Hate & Fairness, Sexual, Violence, and Self-Harm). When a jailbreak attempt is detected, the recommended action is to always block.

</details>

---

### Question 9

What does the "Dashboard-as-Code" approach to Responsible AI governance enable?

- A) Automatic deployment of AI models to production without human review
- B) Version-controlled dashboard configurations that provide audit trails of policy changes over time
- C) AI-generated dashboard layouts that optimize for visual appeal
- D) Code-free dashboard creation through a drag-and-drop portal interface

<details>
<summary>Show Answer</summary>

**Correct Answer: B**

Dashboard-as-Code means storing RAI dashboard configurations in version control alongside model deployment configs. This ensures governance evolves with models, provides a complete audit trail of policy changes over time, enables peer review of governance decisions, and supports infrastructure-as-code practices. It is about governance traceability, not model automation (A), aesthetics (C), or no-code interfaces (D).

</details>

---

### Question 10

An enterprise deploys an AI copilot for financial advisors. Which combination of content safety measures is MOST appropriate?

- A) Default content safety filters only, with no custom blocklists
- B) Custom financial compliance blocklist, zero-tolerance self-harm threshold, PII protection blocklist, and output-layer groundedness checks
- C) Maximum severity thresholds on all categories to minimize false positives, with a single generic blocklist
- D) Content safety disabled for internal users, enabled only for external-facing applications

<details>
<summary>Show Answer</summary>

**Correct Answer: B**

For a financial services AI copilot, the most appropriate approach combines: (1) custom financial compliance blocklist to prevent unauthorized investment advice, (2) zero-tolerance self-harm threshold given the sensitive nature of financial stress, (3) PII protection blocklist to prevent leakage of client financial data, and (4) output-layer groundedness checks to ensure advice is based on verified information. Using only defaults (A) misses industry-specific risks, maximizing thresholds (C) would miss harmful content, and disabling safety for internal users (D) still exposes the organization to compliance risk.

</details>

---

## 📊 Scoring Guide

| Score | Rating | Recommendation |
|---|---|---|
| 10/10 | ⭐ Excellent | Ready for Module 35 |
| 8-9/10 | ✅ Pass | Review missed topics, proceed to Module 35 |
| 6-7/10 | ⚠️ Needs Review | Re-read relevant sections before proceeding |
| ≤ 5/10 | ❌ Retry | Review the full module and lab before retaking |

---

## 🔗 Review Resources

- **Section 2**: Responsible AI Dashboard Configuration → Questions 1, 9
- **Section 3**: Content Safety at Enterprise Scale → Questions 2, 3, 8, 10
- **Section 4**: Monitoring for Bias & Fairness → Questions 4, 6
- **Section 5**: Incident Response → Questions 5, 7

---

*Azure AI Foundry: Zero to Hero — Module 34 Assessment*
*Arc 7: Security, Governance & Compliance*
*© 2026 Azure AI Foundry Training*
