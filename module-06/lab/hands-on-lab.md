# Module 6 — Hands-On Lab

## Build a Customer Feedback Classifier with Prompt Engineering

**Estimated Time:** 30–40 minutes  
**Difficulty:** Beginner–Intermediate  
**Environment:** Azure AI Foundry Playground  

---

## Prerequisites

- [ ] An active Azure subscription
- [ ] Access to an Azure AI Foundry project ([https://ai.azure.com](https://ai.azure.com))
- [ ] At least one deployed chat model (GPT-4o, GPT-4.1, or equivalent) in your project
- [ ] Familiarity with the AI Foundry Playground interface (covered in Module 4)

---

## Scenario

You work for **Contoso Electronics**, an online retailer that receives thousands of customer feedback messages daily. Your team needs an automated way to classify each message into one of five categories:

| Category | Description |
|----------|-------------|
| **product_quality** | Issues or praise about the physical product |
| **shipping** | Delivery speed, packaging, tracking |
| **customer_service** | Interactions with support agents |
| **pricing** | Cost, discounts, value for money |
| **website_app** | Online experience, bugs, usability |

Each classification must also include a **sentiment** (positive, negative, neutral) and a **confidence** score (high, medium, low).

---

## Lab Steps

### Step 1 — Open the Playground

1. Go to [https://ai.azure.com](https://ai.azure.com) and open your project.
2. In the left menu, select **Playgrounds → Chat**.
3. In the **Deployment** dropdown at the top, select your deployed chat model.

### Step 2 — Write a Zero-Shot System Prompt

In the **System message** panel, enter:

```text
You are a customer feedback classifier for Contoso Electronics.

Given a customer feedback message, classify it into exactly ONE of these categories:
- product_quality
- shipping
- customer_service
- pricing
- website_app

Also determine the sentiment (positive, negative, or neutral) and your confidence level (high, medium, or low).

Respond ONLY with a valid JSON object in this format:
{
  "category": "<category>",
  "sentiment": "<sentiment>",
  "confidence": "<confidence>",
  "reasoning": "<one sentence explaining your classification>"
}
```

### Step 3 — Test with Zero-Shot Examples

Send each of these messages one at a time. Record the outputs.

**Test Message 1:**
```
The laptop arrived with a cracked screen. Very disappointed.
```

**Expected output (approximate):**
```json
{
  "category": "product_quality",
  "sentiment": "negative",
  "confidence": "high",
  "reasoning": "The customer reports physical damage to the product."
}
```

**Test Message 2:**
```
Delivery was super fast, got it in 2 days! Great job.
```

**Test Message 3:**
```
Your website keeps crashing when I try to checkout on mobile.
```

**Test Message 4:**
```
I think $299 is a bit steep for what you get, but the build quality is decent.
```

> 💡 **Tip:** For Test Message 4, observe whether the model picks `pricing` or `product_quality`. Ambiguous inputs are the best test of your prompt's clarity.

### Step 4 — Upgrade to Few-Shot Prompting

Improve consistency by adding examples. Update your system message to append:

```text

Here are example classifications:

User: "The headphones broke after just one week of use."
Assistant: {"category": "product_quality", "sentiment": "negative", "confidence": "high", "reasoning": "The customer reports a product defect within a short time period."}

User: "I called support three times and nobody could help me."
Assistant: {"category": "customer_service", "sentiment": "negative", "confidence": "high", "reasoning": "The customer describes repeated unsuccessful interactions with support."}

User: "Love the new app update, checkout is so much smoother now!"
Assistant: {"category": "website_app", "sentiment": "positive", "confidence": "high", "reasoning": "The customer praises the improved app experience."}
```

Now resend Test Message 4 and compare the output with your zero-shot result. Is it more consistent?

### Step 5 — Add Chain-of-Thought Reasoning

For complex or ambiguous feedback, add this instruction to the system prompt:

```text
For ambiguous messages, think step by step:
1. Identify the primary topic of the feedback.
2. Determine if a secondary topic exists.
3. Choose the category that best matches the PRIMARY concern.
4. Assess sentiment based on the overall tone.
```

Test with this deliberately ambiguous message:

```
The price was great and shipping was fast, but the product quality is terrible. I called support and they were unhelpful. Worst experience ever.
```

Observe how the model handles multiple categories and chooses a primary one.

### Step 6 — Tune Parameters

1. In the **Parameters** panel on the right side of the Playground:
   - Set **Temperature** to `0.0`
   - Set **Max tokens** to `200`
   - Leave **Top P** at `1.0`

2. Send the same message 3 times. Verify you get identical outputs (deterministic mode).

3. Now change **Temperature** to `0.8` and send the same message 3 times. Note any variation in the `reasoning` field.

> ⚠️ **For classification tasks, use temperature 0.0.** You want deterministic, repeatable results — not creative variation.

### Step 7 — Handle Edge Cases

Test your prompt with these tricky inputs:

**Empty input:**
```
 
```

**Non-English input:**
```
Die Lieferung war sehr schnell, danke!
```

**Off-topic input:**
```
What time does the store close on Sundays?
```

**Multiple feedback points:**
```
Loved the product but the shipping took 3 weeks and your app crashed twice while I was trying to track it.
```

If the model doesn't handle these gracefully, add guardrails to your system prompt:

```text
Additional rules:
- If the message is empty or unintelligible, respond with: {"category": "unclassifiable", "sentiment": "neutral", "confidence": "low", "reasoning": "Input could not be classified."}
- If the message is not in English, attempt classification based on translated content.
- If the message is off-topic (not customer feedback), respond with: {"category": "off_topic", "sentiment": "neutral", "confidence": "high", "reasoning": "Message is not customer feedback."}
```

### Step 8 — Save Your Prompt

1. Once satisfied, click **Save** in the system message panel to save this as a named configuration.
2. Name it `contoso-feedback-classifier-v1`.
3. Optionally, export the prompt for use in Prompt Flow or your application code.

---

## Expected Final System Prompt

Your final system message should be approximately 40–60 lines and include:
- Clear role definition
- Category definitions
- Output format specification (JSON schema)
- 3 few-shot examples
- Chain-of-thought instructions for ambiguous cases
- Edge case handling rules
- Parameter recommendations in comments

---

## Challenge Exercises

### Challenge 1: Add a Priority Field
Extend the JSON output to include a `"priority"` field (urgent, normal, low) based on the severity of the feedback. Update your few-shot examples accordingly.

### Challenge 2: Batch Classification
Write a prompt that accepts **multiple** feedback messages (numbered 1–5) and returns a JSON array of classifications. Test with 5 messages in a single user turn.

### Challenge 3: Confidence Calibration
Add instructions that make the model output `"confidence": "low"` when the message is ambiguous and could belong to two or more categories. Test with 3 ambiguous messages to verify behaviour.

### Challenge 4: Multi-Language Support
Modify the system prompt to handle feedback in English, Spanish, French, and German. Add a `"detected_language"` field to the output. Test with one message in each language.

---

## Clean-Up

No Azure resources were created during this lab — the Playground uses your existing model deployment. No clean-up is required.

---

## Key Takeaways

- A well-structured system prompt is the foundation of any prompt engineering task.
- Few-shot examples dramatically improve output consistency.
- Chain-of-thought reasoning helps with ambiguous inputs.
- Temperature 0.0 is essential for deterministic classification.
- Edge case handling must be explicitly addressed in the prompt — models do not infer guardrails on their own.

---

*Azure AI Foundry — Zero to Hero Training Team · April 2026*
