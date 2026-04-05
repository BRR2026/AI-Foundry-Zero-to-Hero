# Module 8 — Deploying & Comparing Models — Including Multimodal

## Training Guide

**Series:** Azure AI Foundry — Zero to Hero (Module 8 of 45)  
**Arc:** ARC 2 · Prompt Engineering & Models  
**Duration:** 90 minutes (lecture + hands-on)  
**Date:** April 2026

---

## Learning Objectives

By the end of this module, learners will be able to:

1. Explain the three model deployment types in Azure AI Foundry: Managed Compute, Serverless API, and Provisioned Throughput.
2. Deploy GPT-4o as a multimodal endpoint from the AI Foundry portal.
3. Send multimodal payloads (text + images) to GPT-4o using the Chat Completions API.
4. Encode and send images via URL reference and base64 data URI.
5. Describe audio workflows using Whisper (speech-to-text) and TTS (text-to-speech).
6. Use GPT-4o vision to extract information from documents rendered as images.
7. Benchmark and compare models using AI Foundry's evaluation framework.
8. Estimate costs for different deployment types based on workload characteristics.

---

## Prerequisites

- Completed Modules 1–7 (especially Module 6: Exploring the Model Catalog and Module 7: Deploying Your First Model)
- Active Azure subscription with AI Foundry project
- Python 3.10+ with `openai` SDK installed
- Basic understanding of REST APIs and JSON

---

## Session Outline

| Time | Activity | Format |
|------|----------|--------|
| 0:00–0:10 | Welcome & Recap of Module 7 | Lecture |
| 0:10–0:25 | Deployment Types Deep Dive | Lecture + Slides |
| 0:25–0:35 | Live Demo: Deploying GPT-4o | Demo |
| 0:35–0:50 | Multimodal API Walkthrough | Lecture + Code |
| 0:50–1:05 | Hands-On Lab: Image Analysis App | Lab |
| 1:05–1:15 | Benchmarking & A/B Testing | Lecture |
| 1:15–1:25 | Cost Considerations Discussion | Discussion |
| 1:25–1:30 | Key Takeaways & Module 9 Preview | Wrap-up |

---

## Key Concepts to Cover

### Deployment Types
- **Managed Compute** — Dedicated VMs, full control, custom models
- **Serverless API** — Pay-per-token, zero infrastructure, instant provisioning
- **Provisioned Throughput** — Reserved PTUs, guaranteed latency, best at scale

### Multimodal Capabilities
- GPT-4o processes text, images, and audio in a unified architecture
- The `content` array supports mixed `text` and `image_url` blocks
- `detail` parameter controls image resolution: `low`, `high`, `auto`

### Image Handling
- URL reference: simple, requires public/SAS-protected URLs
- Base64 encoding: works with private images, larger payload size
- Multiple images per request for comparison tasks

### Audio & Speech
- Whisper model for speech-to-text transcription
- TTS model for text-to-speech generation
- Chain pattern: Whisper → GPT-4o → TTS for voice assistants

### Document Understanding
- Render PDF pages as images for GPT-4o vision processing
- Use structured output (JSON schema) for consistent extraction
- Pair with Azure AI Document Intelligence for high-volume scenarios

### Benchmarking
- Build evaluation datasets from real workload samples
- Use AI Foundry evaluation metrics: groundedness, relevance, coherence, fluency
- A/B test in production with traffic splitting

---

## Instructor Notes

1. **Demo Setup:** Pre-deploy a GPT-4o serverless endpoint before the session. Have 2-3 sample images ready (product photo, receipt, chart).
2. **Common Confusion:** Students may confuse Managed Compute with Azure ML managed endpoints. Emphasize this is AI Foundry-specific.
3. **Cost Discussion:** Use concrete numbers. Walk through the cost calculation example in the blog to make pricing tangible.
4. **Lab Timing:** The Mini Hack takes ~15 minutes. Have the starter code ready to paste for students who fall behind.
5. **Featured Video:** Play the embedded video (YouTube ID: NbResO8QcSM) during the intro or assign as pre-work.

---

## Assessment

- Quiz: 10-question assessment covering deployment types, multimodal API patterns, and cost trade-offs (see `quiz/assessment.md`)
- Mini Hack: Image analysis app must return valid JSON with all four fields

---

## Resources

- [AI Foundry Portal](https://ai.azure.com)
- [GPT-4o Vision Docs](https://learn.microsoft.com/azure/ai-services/openai/how-to/gpt-with-vision)
- [AI Foundry Evaluation Guide](https://learn.microsoft.com/azure/ai-studio/how-to/evaluate-generative-ai-app)
- [Featured Video — Fine-Tuning Models in Microsoft Foundry](https://www.youtube.com/watch?v=NbResO8QcSM)
