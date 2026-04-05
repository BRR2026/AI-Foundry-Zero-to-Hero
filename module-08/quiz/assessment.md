# Module 8 — Assessment

## Deploying & Comparing Models — Including Multimodal

**Total Questions:** 10  
**Passing Score:** 70% (7/10)  
**Time Limit:** 15 minutes

---

### Question 1

**Which deployment type in Azure AI Foundry provides pay-per-token billing with no infrastructure to manage?**

- A) Managed Compute
- B) Serverless API
- C) Provisioned Throughput
- D) Dedicated Endpoint

<details>
<summary>Answer</summary>

**B) Serverless API**

Serverless API deployments offer pay-per-token billing with instant provisioning and no VM management. Managed Compute charges per-hour for dedicated VMs, and Provisioned Throughput uses monthly PTU reservations.
</details>

---

### Question 2

**You need guaranteed low latency and predictable throughput for a production workload handling 500K requests per day. Which deployment type is most appropriate?**

- A) Serverless API
- B) Managed Compute
- C) Provisioned Throughput
- D) Any type — they all perform the same

<details>
<summary>Answer</summary>

**C) Provisioned Throughput**

Provisioned Throughput (PTUs) provides reserved capacity with guaranteed latency, making it the best choice for high-volume production workloads with predictable demand.
</details>

---

### Question 3

**In the GPT-4o Chat Completions API, how do you include an image in a request?**

- A) Use a separate `/images` endpoint
- B) Add an `image_url` content block in the `messages` array
- C) Upload the image to a storage account and reference the blob ID
- D) Use the `attachments` parameter in the request body

<details>
<summary>Answer</summary>

**B) Add an `image_url` content block in the `messages` array**

GPT-4o uses the standard Chat Completions API. Images are included as content blocks with `"type": "image_url"` alongside text blocks in the `content` array of a message.
</details>

---

### Question 4

**What does the `detail` parameter control when sending an image to GPT-4o?**

- A) The output format of the response
- B) The image resolution the model processes, affecting quality and token consumption
- C) Whether the image is stored in Azure for future use
- D) The color depth of the analyzed image

<details>
<summary>Answer</summary>

**B) The image resolution the model processes, affecting quality and token consumption**

Setting `"detail": "high"` provides better analysis of complex images but consumes more tokens. `"low"` reduces token usage and is suitable for simple images. `"auto"` lets the model decide.
</details>

---

### Question 5

**Which two methods can be used to send images to GPT-4o? (Select two)**

- A) Base64-encoded data URI
- B) Azure Blob Storage container name
- C) Publicly accessible URL
- D) File system path on the server

<details>
<summary>Answer</summary>

**A) Base64-encoded data URI** and **C) Publicly accessible URL**

Images are sent either as a URL (public or SAS-protected) or as a base64-encoded data URI embedded in the request payload. Server file paths and blob container names are not directly supported.
</details>

---

### Question 6

**What is the recommended audio pipeline pattern for building a voice AI assistant with Azure AI Foundry?**

- A) GPT-4o handles audio natively with no additional models
- B) TTS → GPT-4o → Whisper
- C) Whisper → GPT-4o → TTS
- D) Azure Speech Service only, no GPT-4o needed

<details>
<summary>Answer</summary>

**C) Whisper → GPT-4o → TTS**

The standard pattern chains Whisper (speech-to-text) for transcription, GPT-4o for reasoning and generating a response, and TTS (text-to-speech) for speaking the response back to the user.
</details>

---

### Question 7

**How can GPT-4o process a multi-page PDF document?**

- A) Upload the PDF directly to the Chat Completions API
- B) Convert each PDF page to an image and send the images to GPT-4o vision
- C) Use the built-in PDF parser in GPT-4o
- D) PDFs are not supported by GPT-4o

<details>
<summary>Answer</summary>

**B) Convert each PDF page to an image and send the images to GPT-4o vision**

GPT-4o's vision capability processes images, not PDF files directly. The recommended approach is to render each page as an image (e.g., using pdf2image), encode as base64, and send as image content blocks.
</details>

---

### Question 8

**When benchmarking models in AI Foundry, which of the following is a built-in evaluation metric?**

- A) Popularity score
- B) Groundedness
- C) Download count
- D) Star rating

<details>
<summary>Answer</summary>

**B) Groundedness**

AI Foundry's evaluation framework includes metrics such as groundedness, relevance, coherence, and fluency. These measure the quality and accuracy of model responses against provided context and expected outputs.
</details>

---

### Question 9

**Why is it important to benchmark models with your own data rather than relying on public benchmarks?**

- A) Public benchmarks are inaccurate
- B) Public benchmarks test general capabilities, not your specific domain and workload
- C) Azure AI Foundry doesn't support public benchmark datasets
- D) Public benchmarks only measure speed, not quality

<details>
<summary>Answer</summary>

**B) Public benchmarks test general capabilities, not your specific domain and workload**

A model that ranks highest on MMLU or HumanEval may not perform best for your particular use case. Evaluating with representative samples from your actual workload gives you the most actionable signal for model selection.
</details>

---

### Question 10

**A single high-detail image sent to GPT-4o can consume approximately how many tokens?**

- A) 10–50 tokens
- B) 100–200 tokens
- C) 1,000+ tokens
- D) Images don't consume tokens

<details>
<summary>Answer</summary>

**C) 1,000+ tokens**

High-detail images can consume over 1,000 tokens depending on their resolution. This significantly impacts cost calculations for multimodal workloads. Using `"detail": "low"` reduces token consumption but may affect analysis quality.
</details>

---

## Scoring Guide

| Score | Result |
|-------|--------|
| 9–10 | ⭐ Excellent — Ready for Module 9 |
| 7–8 | ✅ Pass — Review missed topics before continuing |
| 5–6 | ⚠️ Below passing — Re-read the blog and retry |
| 0–4 | ❌ Needs review — Complete the lab exercises and study the material |
