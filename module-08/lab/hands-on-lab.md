# Module 8 — Hands-On Lab

## Deploying & Comparing Models — Including Multimodal

**Duration:** 45 minutes  
**Difficulty:** Intermediate  
**Prerequisites:** Azure subscription, AI Foundry project (Modules 1–7 complete), Python 3.10+

---

## Lab Overview

In this lab you will:

1. Deploy GPT-4o as a serverless API endpoint in Azure AI Foundry
2. Send a text-only request to verify the deployment
3. Send an image via URL to GPT-4o and get a description
4. Send a local image as base64 and extract structured information
5. Compare GPT-4o and GPT-4o-mini on the same image task
6. Build the Mini Hack image analysis app

---

## Setup

### Install Dependencies

```bash
pip install openai python-dotenv
```

### Create Environment File

Create a `.env` file in your lab working directory:

```env
AZURE_OPENAI_ENDPOINT=https://<your-resource>.openai.azure.com/
AZURE_OPENAI_API_KEY=<your-api-key>
GPT4O_DEPLOYMENT=gpt-4o-multimodal
```

---

## Exercise 1 — Deploy GPT-4o (10 min)

### Steps

1. Open [ai.azure.com](https://ai.azure.com) and sign in
2. Navigate to your project
3. Click **Model catalog** in the left sidebar
4. Search for **GPT-4o** and click the model card
5. Click **Deploy** → select **Serverless API**
6. Set deployment name to `gpt-4o-multimodal`
7. Review content filters (keep defaults)
8. Click **Deploy**
9. Copy the **Endpoint URL** and **API Key** to your `.env` file

### Verify Deployment

```python
import os
from openai import AzureOpenAI
from dotenv import load_dotenv

load_dotenv()

client = AzureOpenAI(
    azure_endpoint=os.environ["AZURE_OPENAI_ENDPOINT"],
    api_key=os.environ["AZURE_OPENAI_API_KEY"],
    api_version="2024-12-01-preview"
)

response = client.chat.completions.create(
    model=os.environ["GPT4O_DEPLOYMENT"],
    messages=[{"role": "user", "content": "Say hello and confirm you are GPT-4o."}],
    max_tokens=100
)

print(response.choices[0].message.content)
```

**✅ Expected:** GPT-4o responds with a greeting confirming its identity.

---

## Exercise 2 — Image Analysis via URL (10 min)

Send a publicly accessible image to GPT-4o for analysis.

```python
response = client.chat.completions.create(
    model=os.environ["GPT4O_DEPLOYMENT"],
    messages=[{
        "role": "user",
        "content": [
            {"type": "text", "text": "Describe this image in detail. What objects do you see?"},
            {
                "type": "image_url",
                "image_url": {
                    "url": "https://upload.wikimedia.org/wikipedia/commons/thumb/3/3a/Cat03.jpg/1200px-Cat03.jpg",
                    "detail": "high"
                }
            }
        ]
    }],
    max_tokens=500
)

print(response.choices[0].message.content)
print(f"\nTokens used — Input: {response.usage.prompt_tokens}, Output: {response.usage.completion_tokens}")
```

### Questions to Answer

1. How many tokens did the image consume?
2. Try changing `"detail"` from `"high"` to `"low"` — how does token count change?
3. Does the quality of description change with different detail levels?

---

## Exercise 3 — Base64 Image Encoding (10 min)

Analyze a local image by encoding it as base64.

### Step 1: Save a test image

Download or use any image file on your machine. Place it in the lab directory as `test-image.jpg`.

### Step 2: Send as base64

```python
import base64

def encode_image(path: str) -> str:
    with open(path, "rb") as f:
        return base64.b64encode(f.read()).decode("utf-8")

image_b64 = encode_image("test-image.jpg")

response = client.chat.completions.create(
    model=os.environ["GPT4O_DEPLOYMENT"],
    messages=[{
        "role": "user",
        "content": [
            {
                "type": "text",
                "text": "Analyze this image. Return a JSON object with keys: description, objects (array), extracted_text, sentiment."
            },
            {
                "type": "image_url",
                "image_url": {
                    "url": f"data:image/jpeg;base64,{image_b64}",
                    "detail": "high"
                }
            }
        ]
    }],
    max_tokens=800,
    temperature=0.2
)

print(response.choices[0].message.content)
```

**✅ Expected:** A JSON object with all four fields populated.

---

## Exercise 4 — Model Comparison (10 min)

Deploy GPT-4o-mini and compare it against GPT-4o on the same image task.

### Step 1: Deploy GPT-4o-mini

1. In AI Foundry, go to **Model catalog** → search **GPT-4o mini**
2. Deploy as Serverless API with name `gpt-4o-mini-test`

### Step 2: Compare

```python
import time, json

MODELS = [os.environ["GPT4O_DEPLOYMENT"], "gpt-4o-mini-test"]

prompt = [{
    "role": "user",
    "content": [
        {"type": "text", "text": "Describe this image in exactly 3 sentences."},
        {"type": "image_url", "image_url": {"url": "https://upload.wikimedia.org/wikipedia/commons/thumb/3/3a/Cat03.jpg/1200px-Cat03.jpg", "detail": "low"}}
    ]
}]

for model in MODELS:
    start = time.time()
    resp = client.chat.completions.create(model=model, messages=prompt, max_tokens=200)
    elapsed = time.time() - start

    print(f"\n{'='*50}")
    print(f"Model: {model}")
    print(f"Latency: {elapsed:.2f}s")
    print(f"Tokens — In: {resp.usage.prompt_tokens}, Out: {resp.usage.completion_tokens}")
    print(f"Response: {resp.choices[0].message.content}")
```

### Record Your Results

| Metric | GPT-4o | GPT-4o-mini |
|--------|--------|-------------|
| Latency (s) | | |
| Input tokens | | |
| Output tokens | | |
| Quality (1-5) | | |

---

## Exercise 5 — Mini Hack: Image Analysis App (15 min)

Build the complete image analysis application from the blog's Mini Hack section.

### Requirements

1. Create a file named `image_analyzer.py`
2. Accept an image source from command-line args (URL or local path)
3. Detect whether input is a URL or file path
4. If file, encode as base64
5. Send to GPT-4o with the analysis prompt
6. Parse and pretty-print the JSON response
7. Handle errors (file not found, API errors, invalid JSON)

### Test Your App

```bash
# Test with URL
python image_analyzer.py "https://upload.wikimedia.org/wikipedia/commons/thumb/3/3a/Cat03.jpg/1200px-Cat03.jpg"

# Test with local file
python image_analyzer.py test-image.jpg

# Test error handling
python image_analyzer.py nonexistent-file.jpg
```

**✅ Success Criteria:**
- Valid JSON returned with `description`, `objects`, `extracted_text`, `sentiment`
- URL and local file both work
- Error case handled gracefully (prints error, doesn't crash)

---

## Cleanup

When finished, delete test deployments to avoid charges:

1. In AI Foundry, go to **Deployments**
2. Delete `gpt-4o-mini-test` (keep `gpt-4o-multimodal` if continuing to Module 9)

---

## Summary

In this lab you:

- ✅ Deployed GPT-4o as a serverless endpoint
- ✅ Sent images via URL and base64 encoding
- ✅ Compared GPT-4o and GPT-4o-mini on vision tasks
- ✅ Built a complete image analysis application
- ✅ Measured latency and token consumption

**Next:** Module 9 — Prompt Engineering Foundations
