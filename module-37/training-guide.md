# Module 37 — Foundry Local: Edge & On-Device AI

> **Azure AI Foundry: Zero to Hero** · ARC 8: Advanced Patterns & Enterprise  
> Duration: 4 hours (instructor-led) | Level: Intermediate–Advanced | Date: April 2026

---

## 🎯 Learning Objectives

By the end of this module, participants will be able to:

1. Explain what Foundry Local is and how it fits into the Azure AI Foundry ecosystem
2. Differentiate between NPU, GPU, and CPU execution providers and select the appropriate one
3. Deploy and run AI models in offline and air-gapped environments
4. Design IoT and edge deployment architectures using Foundry Local
5. Install and operate Foundry Local across Windows, macOS, and Linux
6. Execute a Phi model locally using the OpenAI-compatible API

---

## 📋 Prerequisites

| Requirement | Details |
|---|---|
| **Prior Modules** | Modules 1–36 recommended; Modules 1–10 required |
| **Hardware** | Windows 11, macOS (Apple Silicon), or Linux machine with ≥ 8 GB RAM |
| **Software** | Python 3.10+, pip, a code editor (VS Code recommended) |
| **Accounts** | No Azure subscription required for local operations |
| **Knowledge** | Basic understanding of REST APIs, Python, and AI model concepts |

---

## 📚 Module Outline

### Session 1: Foundations (60 min)

#### 1.1 What Is Foundry Local? (20 min)

**Key Concepts:**
- Foundry Local as a standalone local AI runtime
- Part of Azure AI Foundry — NOT Azure Machine Learning
- ONNX Runtime GenAI as the inference engine
- OpenAI-compatible REST API on `localhost:5272`
- Local model catalog and cache management

**Discussion Points:**
- How does Foundry Local compare to other local inference tools (Ollama, llama.cpp)?
- What role does ONNX Runtime play in hardware abstraction?
- Why did Microsoft choose OpenAI API compatibility?

#### 1.2 When to Use Foundry Local (20 min)

**Decision Framework:**

```
Need cloud-scale models (70B+)?  → Use Azure AI Foundry (cloud)
Need real-time, low-latency?     → Use Foundry Local
Need offline capability?         → Use Foundry Local
Need data sovereignty?           → Use Foundry Local
Need multi-modal (video/audio)?  → Use Azure AI Foundry (cloud) — for now
Need ≤14B parameter models?      → Foundry Local is ideal
```

**Scenario Analysis Exercise (15 min):**

Present these scenarios and have participants decide cloud vs. local:

1. A hospital summarizing patient records (HIPAA-regulated)
2. A retail chain doing real-time inventory classification
3. A research lab running 70B model experiments
4. A military field unit needing NLP in disconnected environments
5. A startup prototyping a chatbot with limited budget

#### 1.3 Architecture Overview (20 min)

**Three-Layer Architecture:**

```
┌─────────────────────────────────┐
│  Application Layer              │
│  (Python, C#, Node.js, cURL)    │
├─────────────────────────────────┤
│  Foundry Local Runtime          │
│  ├── API Gateway (REST)         │
│  ├── Model Catalog (local)      │
│  └── ONNX Runtime GenAI         │
├─────────────────────────────────┤
│  Hardware Layer                 │
│  NPU │ GPU │ CPU │ Memory       │
└─────────────────────────────────┘
```

**Instructor Demo:** Show the Foundry Local architecture diagram and trace a request from the application through to hardware execution.

---

### Session 2: Hardware & Execution (60 min)

#### 2.1 NPU / GPU / CPU Execution Providers (30 min)

**Hardware Comparison Matrix:**

| Hardware | Throughput | Power | Latency | Best For |
|---|---|---|---|---|
| NPU | Medium (30-40 tok/s) | Very Low (< 5W) | Low | Always-on, background |
| dGPU | High (80-120 tok/s) | High (30-75W) | Low | Batch processing |
| iGPU | Medium (25-45 tok/s) | Medium (10-15W) | Low | Balanced workloads |
| CPU | Low (8-15 tok/s) | Medium (15-45W) | Higher | Universal fallback |

**Execution Provider Selection:**

```bash
# Auto-detect (recommended for most use cases)
foundry model run phi-3.5-mini

# Force specific hardware
foundry model run phi-3.5-mini --device gpu
foundry model run phi-3.5-mini --device npu
foundry model run phi-3.5-mini --device cpu
```

**Hands-On Activity (15 min):**
Have participants run the model on different devices (if available) and compare tokens/sec output.

#### 2.2 Model Quantization & Memory (15 min)

**Quantization Levels:**

| Format | Memory (3.8B model) | Quality | Speed |
|---|---|---|---|
| INT4 | ~2.4 GB | Good | Fastest |
| INT8 | ~4.2 GB | Better | Fast |
| FP16 | ~7.6 GB | Best | Moderate |
| FP32 | ~15.2 GB | Reference | Slow |

**Recommendation:** Use INT4 for edge/IoT, INT8 for desktop apps, FP16 only when accuracy is paramount.

#### 2.3 Performance Tuning (15 min)

```bash
# Check current hardware detection
foundry hardware

# Monitor resource usage during inference
foundry status --watch

# Set maximum context window to reduce memory
foundry model run phi-3.5-mini --max-tokens 2048

# Enable flash attention for supported GPUs
foundry model run phi-3.5-mini --flash-attention
```

---

### Session 3: Offline, IoT & Edge (60 min)

#### 3.1 Offline & Air-Gapped Deployment (20 min)

**Deployment Workflow:**

1. **Download** models on a connected machine
2. **Export** the model cache to portable media
3. **Transfer** to the air-gapped device
4. **Import** and start the service

```bash
# Connected machine
foundry model download phi-3.5-mini
foundry cache export --output /mnt/usb/foundry-models/

# Air-gapped machine
foundry cache import --from /mnt/usb/foundry-models/
foundry service start
```

**Discussion:** What operational procedures should organizations put in place for model updates in air-gapped environments?

#### 3.2 IoT & Edge Patterns (25 min)

**Pattern 1: Gateway Aggregation**
- Edge gateway runs Foundry Local
- IoT sensors push data to gateway
- Local model processes/classifies data
- Results synced to cloud when connected

**Pattern 2: Containerized Edge**
```dockerfile
FROM mcr.microsoft.com/foundry/local:latest
RUN foundry model download phi-3.5-mini
EXPOSE 5272
CMD ["foundry", "service", "start", "--host", "0.0.0.0"]
```

**Pattern 3: Hybrid Cloud-Edge**
- Primary inference on edge (Foundry Local)
- Complex queries routed to cloud (Azure AI Foundry)
- Model updates pulled from cloud on schedule

**Group Exercise (15 min):**
Design an edge AI architecture for one of these scenarios:
- Smart building energy management
- Autonomous vehicle telemetry processing
- Retail store customer sentiment analysis

#### 3.3 Cross-Platform Deployment (15 min)

**Installation across platforms:**

| Platform | Command |
|---|---|
| Windows 11 | `winget install Microsoft.FoundryLocal` |
| macOS | `brew install foundry-local` |
| Linux | `curl -sL https://aka.ms/foundry-local \| bash` |

**Platform-Specific Notes:**
- **Windows:** DirectML for GPU; Qualcomm QNN for Snapdragon NPU
- **macOS:** Metal for Apple GPU; ANE for Apple Neural Engine
- **Linux:** CUDA for NVIDIA; Vulkan for AMD; CPU fallback universal

---

### Session 4: Hands-On Lab & Assessment (60 min)

#### 4.1 Mini Hack: Run Phi Locally (35 min)

See the full lab instructions in `lab/hands-on-lab.md`.

**Summary of Steps:**
1. Install Foundry Local on your machine
2. Browse and download the Phi-3.5-mini model
3. Start the local inference server
4. Chat interactively via CLI
5. Call the API from Python using the OpenAI SDK
6. Compare performance across available hardware
7. Test offline mode (disconnect from internet)

#### 4.2 Knowledge Check (15 min)

Administer the quiz from `quiz/assessment.md` (10 MCQs).

#### 4.3 Wrap-Up & Q&A (10 min)

**Key Takeaways:**
- Foundry Local enables private, low-latency AI on any device
- NPU > GPU > CPU is the default execution hierarchy
- OpenAI-compatible API makes cloud-to-local migration trivial
- Containerization is the recommended approach for edge/IoT deployments
- Model right-sizing (quantization) is critical for edge success

**Preview of Module 38:**
Hybrid Cloud-Edge Orchestration — coordinating workloads between Foundry Local and Azure AI Foundry in the cloud.

---

## 📦 Required Materials

| Material | Location |
|---|---|
| Blog Article | `blog.html` |
| Slide Deck | `slides/presentation.html` |
| Hands-On Lab | `lab/hands-on-lab.md` |
| Assessment Quiz | `quiz/assessment.md` |
| Video Resource | [MS Mechanics — Foundry Local](https://www.youtube.com/watch?v=qL3HADDI6W4) |

---

## 🔗 Additional Resources

- [Foundry Local Documentation](https://learn.microsoft.com/azure/ai-foundry/foundry-local)
- [ONNX Runtime GenAI](https://github.com/microsoft/onnxruntime-genai)
- [Phi Model Family](https://azure.microsoft.com/products/phi)
- [Azure IoT Edge](https://learn.microsoft.com/azure/iot-edge/)
- [OpenAI Python SDK](https://github.com/openai/openai-python)

---

## 📝 Instructor Notes

### Timing Guidelines

| Session | Duration | Format |
|---|---|---|
| Session 1: Foundations | 60 min | Lecture + Discussion |
| Session 2: Hardware & Execution | 60 min | Lecture + Demo |
| Session 3: Offline, IoT & Edge | 60 min | Lecture + Group Exercise |
| Session 4: Lab & Assessment | 60 min | Hands-On + Quiz |
| **Total** | **4 hours** | |

### Common Student Questions

1. **"Is Foundry Local free?"** — Yes, the runtime is free. Model licenses vary (Phi models are MIT-licensed).
2. **"Can I fine-tune models with Foundry Local?"** — Foundry Local is inference-only. Fine-tuning should be done in Azure AI Foundry (cloud) or locally with other tools, then the resulting model can be loaded into Foundry Local.
3. **"How does this compare to Ollama?"** — Foundry Local uses ONNX Runtime (optimized for Windows NPU/GPU), provides OpenAI API compatibility, and integrates with the Azure AI Foundry ecosystem. Ollama uses llama.cpp and has a different API format.
4. **"What's the largest model I can run?"** — Depends on available RAM/VRAM. Rule of thumb: INT4 model size in GB ≈ parameters in billions × 0.6. A 16 GB device can typically handle up to ~14B parameters.

### Environment Setup Checklist

- [ ] Foundry Local installed on instructor machine
- [ ] Phi-3.5-mini model pre-downloaded
- [ ] Python 3.10+ with `openai` package installed
- [ ] VS Code with Python extension
- [ ] Backup slides available offline
- [ ] Projector/screen for demos
