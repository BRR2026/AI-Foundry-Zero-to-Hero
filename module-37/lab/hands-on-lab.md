# 🛠️ Hands-On Lab: Run a Phi Model Locally with Foundry Local

> **Module 37 — Foundry Local: Edge & On-Device AI**  
> Azure AI Foundry: Zero to Hero · ARC 8: Advanced Patterns & Enterprise  
> ⏱️ Duration: 45 minutes | Level: Intermediate

---

## 🎯 Lab Objectives

By completing this lab, you will:

1. Install Microsoft Foundry Local on your machine
2. Explore the local model catalog
3. Download and run a Phi-3.5-mini model entirely on-device
4. Interact with the model via CLI and Python API
5. Test inference in offline mode (no internet)
6. Compare performance across hardware execution providers

---

## 📋 Prerequisites

| Requirement | Details |
|---|---|
| **Operating System** | Windows 11 (x64/ARM), macOS (Apple Silicon), or Linux (x64/ARM) |
| **RAM** | ≥ 8 GB (16 GB recommended) |
| **Disk Space** | ≥ 5 GB free |
| **Python** | 3.10 or later |
| **Package Manager** | `winget` (Windows), `brew` (macOS), or `curl` (Linux) |
| **Internet** | Required for initial setup; offline for later exercises |

---

## Exercise 1: Install Foundry Local (10 min)

### Step 1.1 — Install the Runtime

Choose the command for your platform:

**Windows:**
```powershell
winget install Microsoft.FoundryLocal
```

**macOS:**
```bash
brew install foundry-local
```

**Linux:**
```bash
curl -sL https://aka.ms/foundry-local | bash
```

### Step 1.2 — Verify Installation

```bash
foundry --version
```

**Expected output:**
```
Microsoft Foundry Local v1.x.x
```

### Step 1.3 — Check Hardware Detection

```bash
foundry hardware
```

**Expected output (example for a Windows laptop with NVIDIA GPU):**
```
Hardware Detection Results:
  CPU:  Intel Core Ultra 7 155H (22 cores) — AVX-512 supported
  GPU:  NVIDIA GeForce RTX 4060 Laptop (8 GB VRAM) — CUDA 12.x
  NPU:  Intel AI Boost — Supported
  RAM:  32 GB DDR5

Recommended execution provider: GPU (CUDA)
```

> 📝 **Note:** Your output will vary based on your hardware. All three providers (CPU, GPU, NPU) are valid — Foundry Local works on any of them.

---

## Exercise 2: Explore the Model Catalog (5 min)

### Step 2.1 — List Available Models

```bash
foundry model list
```

**Expected output:**
```
NAME                        PARAMETERS  SIZE (INT4)  HARDWARE         STATUS
────────────────────────────────────────────────────────────────────────────
phi-3.5-mini                3.8B        2.4 GB       CPU/GPU/NPU      Not downloaded
phi-4-mini                  3.8B        3.1 GB       CPU/GPU/NPU      Not downloaded
phi-3-vision                4.2B        4.2 GB       GPU              Not downloaded
phi-3.5-moe                 16×3.8B     8.1 GB       GPU              Not downloaded
mistral-7b-instruct         7B          4.1 GB       CPU/GPU          Not downloaded
```

### Step 2.2 — Get Model Details

```bash
foundry model info phi-3.5-mini
```

Review the output for:
- Model architecture and parameter count
- Supported quantization formats
- Compatible hardware execution providers
- License information (MIT for Phi models)

---

## Exercise 3: Download & Run the Model (10 min)

### Step 3.1 — Download Phi-3.5-mini

```bash
foundry model download phi-3.5-mini
```

This downloads the INT4-quantized version (~2.4 GB). Wait for the download to complete.

**Expected output:**
```
Downloading phi-3.5-mini (INT4, 2.4 GB)...
  [████████████████████████████████████████] 100%  2.4 GB / 2.4 GB
Model saved to: ~/.foundry/models/phi-3.5-mini/
```

### Step 3.2 — Start the Model Server

```bash
foundry model run phi-3.5-mini
```

**Expected output:**
```
Loading model: phi-3.5-mini (INT4)
Execution provider: GPU (DirectML)
Memory allocated: 2.1 GB VRAM

Model loaded successfully.
Listening on http://localhost:5272
OpenAI-compatible endpoint: http://localhost:5272/v1/chat/completions
```

> 💡 **Tip:** The server runs in the foreground. Open a new terminal for the next exercises.

### Step 3.3 — Verify the Server

Open a **new terminal** and test with curl:

```bash
curl http://localhost:5272/v1/models
```

**Expected output:**
```json
{
  "data": [
    {
      "id": "phi-3.5-mini",
      "object": "model",
      "owned_by": "microsoft"
    }
  ]
}
```

---

## Exercise 4: Interactive Chat (5 min)

### Step 4.1 — Start CLI Chat

In a new terminal:

```bash
foundry chat
```

### Step 4.2 — Test Prompts

Try these prompts in the chat interface:

**Prompt 1 — General knowledge:**
```
What are three key benefits of running AI models on edge devices?
```

**Prompt 2 — Code generation:**
```
Write a Python function that reads sensor data from a JSON file and detects anomalies using a simple z-score method.
```

**Prompt 3 — Summarization:**
```
Summarize the following in 2 sentences: Edge computing brings AI inference closer to the data source, reducing latency and bandwidth requirements while maintaining data privacy.
```

### Step 4.3 — Observe Performance

Note the following in the chat output:
- **Tokens per second** (displayed in the CLI)
- **Time to first token** (initial latency)
- **Total generation time**

> 📝 Record these numbers — you'll compare them in Exercise 6.

Type `exit` to leave the chat.

---

## Exercise 5: Python API Integration (10 min)

### Step 5.1 — Install the OpenAI SDK

```bash
pip install openai
```

### Step 5.2 — Create the Script

Create a file named `foundry_local_demo.py`:

```python
"""
Module 37 Lab — Foundry Local API Demo
Demonstrates calling a local Phi model via the OpenAI-compatible API.
"""

from openai import OpenAI
import time

# Connect to Foundry Local (no API key needed)
client = OpenAI(
    base_url="http://localhost:5272/v1",
    api_key="local"
)

# ── Test 1: Basic Chat Completion ──────────────────────
print("=" * 60)
print("TEST 1: Basic Chat Completion")
print("=" * 60)

start = time.time()
response = client.chat.completions.create(
    model="phi-3.5-mini",
    messages=[
        {"role": "system", "content": "You are a helpful edge AI assistant."},
        {"role": "user", "content": "What is Foundry Local and why would I use it?"}
    ],
    temperature=0.7,
    max_tokens=300
)
elapsed = time.time() - start

print(f"\nResponse:\n{response.choices[0].message.content}")
print(f"\nTokens used: {response.usage.total_tokens}")
print(f"Time: {elapsed:.2f}s")

# ── Test 2: Streaming Response ─────────────────────────
print("\n" + "=" * 60)
print("TEST 2: Streaming Response")
print("=" * 60 + "\n")

stream = client.chat.completions.create(
    model="phi-3.5-mini",
    messages=[
        {"role": "user", "content": "List 5 IoT edge AI use cases, one per line."}
    ],
    temperature=0.5,
    max_tokens=200,
    stream=True
)

for chunk in stream:
    if chunk.choices[0].delta.content:
        print(chunk.choices[0].delta.content, end="", flush=True)

print("\n")

# ── Test 3: Multi-Turn Conversation ────────────────────
print("=" * 60)
print("TEST 3: Multi-Turn Conversation")
print("=" * 60)

messages = [
    {"role": "system", "content": "You are an IoT architect. Be concise."},
    {"role": "user", "content": "I need to detect anomalies in factory sensor data at the edge."},
]

# First turn
response1 = client.chat.completions.create(
    model="phi-3.5-mini",
    messages=messages,
    temperature=0.7,
    max_tokens=200
)
print(f"\nAssistant: {response1.choices[0].message.content}")

# Second turn — follow up
messages.append({"role": "assistant", "content": response1.choices[0].message.content})
messages.append({"role": "user", "content": "What model size would you recommend for a device with 4 GB RAM?"})

response2 = client.chat.completions.create(
    model="phi-3.5-mini",
    messages=messages,
    temperature=0.7,
    max_tokens=200
)
print(f"\nAssistant: {response2.choices[0].message.content}")

print("\n✅ All tests completed successfully!")
```

### Step 5.3 — Run the Script

```bash
python foundry_local_demo.py
```

### Step 5.4 — Verify Results

Confirm that:
- [ ] Test 1 returns a coherent response about Foundry Local
- [ ] Test 2 streams tokens incrementally
- [ ] Test 3 maintains conversation context across turns
- [ ] No errors or connection issues

---

## Exercise 6: Hardware Comparison & Offline Test (5 min)

### Step 6.1 — Compare Execution Providers

Stop the current server (Ctrl+C in the server terminal), then restart with a different device:

```bash
# Run on CPU
foundry model run phi-3.5-mini --device cpu
```

In another terminal, run the same test:

```bash
curl -s -X POST http://localhost:5272/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "phi-3.5-mini",
    "messages": [{"role": "user", "content": "Hello, what hardware are you running on?"}],
    "max_tokens": 100
  }' | python -m json.tool
```

### Step 6.2 — Record Performance

Fill in this table with your observations:

| Hardware | Tokens/sec | First Token Latency | Memory Used |
|---|---|---|---|
| GPU | ___ | ___ ms | ___ GB |
| CPU | ___ | ___ ms | ___ GB |
| NPU (if available) | ___ | ___ ms | ___ GB |

### Step 6.3 — Offline Test

1. **Disconnect** from Wi-Fi / unplug Ethernet
2. **Restart** the Foundry Local server: `foundry model run phi-3.5-mini`
3. **Send a request** via curl or the Python script
4. **Verify** you receive a valid response with no internet connection

> 🎉 **Success!** If you get a response while fully offline, you've proven Foundry Local works in air-gapped mode.

5. **Reconnect** your internet when done

---

## 🧹 Cleanup

```bash
# Stop the Foundry Local server
foundry service stop

# (Optional) Remove the downloaded model to reclaim disk space
foundry model delete phi-3.5-mini

# (Optional) Remove the demo script
rm foundry_local_demo.py
```

---

## ✅ Completion Checklist

- [ ] Foundry Local installed and verified
- [ ] Hardware detection confirmed
- [ ] Phi-3.5-mini model downloaded
- [ ] Server running on localhost:5272
- [ ] CLI chat working
- [ ] Python API demo completed (3 tests)
- [ ] Performance comparison recorded
- [ ] Offline inference verified

---

## 🏆 Bonus Challenges

If you finish early, try these advanced exercises:

### Bonus 1: Multi-Model Serving
```bash
# Download a second model
foundry model download phi-4-mini

# Run both models simultaneously
foundry service start --models phi-3.5-mini,phi-4-mini
```

### Bonus 2: Docker Containerization
```dockerfile
FROM mcr.microsoft.com/foundry/local:latest
RUN foundry model download phi-3.5-mini
EXPOSE 5272
CMD ["foundry", "service", "start", "--host", "0.0.0.0"]
```

```bash
docker build -t my-edge-ai .
docker run -p 5272:5272 my-edge-ai
```

### Bonus 3: Build an Edge Pipeline
Write a Python script that:
1. Reads simulated sensor data (temperature, pressure, vibration)
2. Sends it to Foundry Local for anomaly classification
3. Generates a natural-language alert if anomalies are detected

---

## 📚 References

- [Foundry Local Documentation](https://learn.microsoft.com/azure/ai-foundry/foundry-local)
- [ONNX Runtime GenAI](https://github.com/microsoft/onnxruntime-genai)
- [OpenAI Python SDK](https://github.com/openai/openai-python)
- [MS Mechanics Video — Foundry Local](https://www.youtube.com/watch?v=qL3HADDI6W4)
