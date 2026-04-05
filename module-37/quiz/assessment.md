# 📝 Module 37 Assessment — Foundry Local: Edge & On-Device AI

> **Azure AI Foundry: Zero to Hero** · ARC 8: Advanced Patterns & Enterprise  
> 10 Multiple-Choice Questions · Passing Score: 80% (8/10) · Time Limit: 15 minutes

---

## Instructions

- Select the **single best answer** for each question.
- All questions reference concepts covered in Module 37.
- Answers and explanations are provided at the end of this document.

---

### Question 1

**What is Microsoft Foundry Local?**

A) A cloud-based Azure Machine Learning deployment service  
B) A lightweight local runtime for on-device AI inference using ONNX Runtime GenAI  
C) An Azure subscription management tool for edge devices  
D) A Visual Studio extension for debugging machine learning models  

---

### Question 2

**What is the default execution provider priority order in Foundry Local?**

A) CPU → GPU → NPU  
B) GPU → NPU → CPU  
C) NPU → GPU → CPU  
D) NPU → CPU → GPU  

---

### Question 3

**Which API standard does Foundry Local expose for inference requests?**

A) gRPC with Protocol Buffers  
B) Azure Machine Learning REST API  
C) OpenAI-compatible REST API  
D) GraphQL with subscriptions  

---

### Question 4

**What is the default port for the Foundry Local inference server?**

A) 8080  
B) 3000  
C) 5272  
D) 443  

---

### Question 5

**A hospital needs to summarize patient records locally to comply with HIPAA regulations. Which is the PRIMARY reason to use Foundry Local for this scenario?**

A) Lower cost compared to cloud inference  
B) Higher model accuracy than cloud models  
C) Data never leaves the device boundary  
D) Faster model training on local hardware  

---

### Question 6

**Which quantization format is recommended for deploying a model on a resource-constrained edge device with only 4 GB of available memory?**

A) FP32 (full precision)  
B) FP16 (half precision)  
C) INT8  
D) INT4  

---

### Question 7

**How do you deploy a Foundry Local model to an air-gapped (no internet) environment?**

A) Configure the air-gapped device to use Azure Private Link  
B) Download the model on a connected machine, export the cache, transfer via physical media, and import on the target device  
C) Foundry Local cannot operate without an internet connection  
D) Use Azure DevOps pipelines to push models through a VPN tunnel  

---

### Question 8

**Which hardware accelerator offers the best power efficiency for always-on background AI inference tasks?**

A) Discrete GPU (dGPU)  
B) Integrated GPU (iGPU)  
C) NPU (Neural Processing Unit)  
D) CPU with AVX-512  

---

### Question 9

**A developer wants to switch their Python application from using an Azure AI Foundry cloud endpoint to Foundry Local. What is the minimum code change required?**

A) Rewrite the application to use the ONNX Runtime Python API  
B) Change the `base_url` parameter in the OpenAI client constructor to `http://localhost:5272/v1`  
C) Install a completely different SDK and rewrite all API calls  
D) Deploy a Docker container and modify the network configuration  

---

### Question 10

**When containerizing Foundry Local for edge IoT deployment, what is the recommended approach?**

A) Mount the model directory as a volume from the host machine only  
B) Build a Docker image that includes Foundry Local and the pre-downloaded model, then expose port 5272  
C) Run the Foundry Local installer inside the container at runtime each time  
D) Use Azure IoT Edge modules exclusively — Foundry Local cannot run in containers  

---

## ✅ Answer Key

| # | Answer | Explanation |
|---|--------|-------------|
| 1 | **B** | Foundry Local is a standalone local runtime built on ONNX Runtime GenAI for on-device AI inference. It is NOT Azure Machine Learning — it is part of the Azure AI Foundry product family and does not require an Azure subscription. |
| 2 | **C** | Foundry Local's default execution priority is NPU → GPU → CPU, preferring the most power-efficient specialized hardware first, then falling back to GPU and finally CPU. |
| 3 | **C** | Foundry Local exposes an OpenAI-compatible REST API, allowing developers to use the standard OpenAI SDK with just a base URL change. |
| 4 | **C** | The default port for Foundry Local's inference server is 5272. The full endpoint is `http://localhost:5272/v1/chat/completions`. |
| 5 | **C** | The primary reason for HIPAA-regulated environments is data sovereignty — with Foundry Local, patient data never leaves the device boundary. While cost may be a secondary benefit, data privacy is the primary driver. |
| 6 | **D** | INT4 quantization provides the smallest memory footprint (approximately parameters × 0.6 GB). For a 3.8B model, INT4 uses ~2.4 GB, fitting within the 4 GB constraint. FP32 would require ~15 GB, FP16 ~7.6 GB, and INT8 ~4.2 GB. |
| 7 | **B** | Air-gapped deployment requires downloading the model on a connected machine, exporting the model cache, transferring it via physical media (USB, etc.), and importing it on the target device. Foundry Local then operates without any internet connection. |
| 8 | **C** | NPUs (Neural Processing Units) are designed for always-on, low-power AI workloads, consuming less than 5 watts while delivering 30-40 tokens/second for a Phi-3.5-mini model. |
| 9 | **B** | Because Foundry Local uses an OpenAI-compatible API, the only change needed is updating the `base_url` to point to `http://localhost:5272/v1`. The same OpenAI SDK and code structure work unchanged. |
| 10 | **B** | The recommended approach is to build a Docker image that includes both the Foundry Local runtime and the pre-downloaded model, then expose port 5272. This creates a self-contained, reproducible deployment artifact for edge scenarios. |

---

## 📊 Score Interpretation

| Score | Rating | Recommendation |
|---|---|---|
| 10/10 | ⭐ Excellent | Ready for Module 38 — Hybrid Cloud-Edge Orchestration |
| 8-9/10 | ✅ Proficient | Review missed topics, then proceed to Module 38 |
| 6-7/10 | 📖 Developing | Re-read the relevant sections and retry the lab exercises |
| < 6/10 | 🔄 Needs Review | Complete Module 37 materials again before proceeding |

---

## 🔗 Review Resources

- **Question 1-3:** See blog.html sections 1 & 7 (What Is Foundry Local, Architecture)
- **Question 4:** See blog.html section 7.1 (API Compatibility)
- **Question 5-6:** See blog.html sections 2 & 3 (When to Use, Hardware Targets)
- **Question 7:** See blog.html section 4 (Offline Scenarios)
- **Question 8:** See blog.html section 3.2 (Performance Comparison)
- **Question 9:** See blog.html section 7.1 (API Compatibility)
- **Question 10:** See blog.html section 5.2 (Containerized Edge Deployment)
