# Module 6 — Assessment

## Prompt Engineering Fundamentals — Quiz

**Instructions:** Select the single best answer for each question. Answers are provided at the end.

---

### Question 1

What is the primary purpose of the **system message** in a chat-completion API call?

A) To provide the user's question to the model  
B) To set the model's persona, constraints, and output format  
C) To store previous conversation history  
D) To configure the API endpoint and authentication  

---

### Question 2

A developer sends the following prompt with no examples:  
*"Classify this movie review as positive, negative, or neutral: 'The cinematography was stunning but the plot was predictable.'"*

Which prompting technique is being used?

A) Few-shot prompting  
B) Chain-of-thought prompting  
C) Zero-shot prompting  
D) Meta-prompting  

---

### Question 3

When using **few-shot prompting**, what is the recommended number of examples to include?

A) Exactly 1 example  
B) 2–5 well-chosen examples  
C) At least 10 examples for any task  
D) As many examples as the context window allows  

---

### Question 4

Which parameter should you set to **0.0** when you need deterministic, repeatable outputs for a classification task?

A) top_p  
B) max_tokens  
C) temperature  
D) presence_penalty  

---

### Question 5

A prompt includes the instruction: *"Let's solve this step by step. First, identify the key variables. Then, set up the equation. Finally, solve for x."*

Which technique does this demonstrate?

A) Zero-shot prompting  
B) Few-shot prompting  
C) Chain-of-thought (CoT) prompting  
D) Prompt chaining  

---

### Question 6

In the Azure AI Foundry Playground, what is the effect of increasing the **top_p** parameter from 0.1 to 0.95?

A) The model generates longer responses  
B) The model considers a wider range of token choices, increasing output diversity  
C) The model responds faster due to fewer calculations  
D) The model ignores the system message  

---

### Question 7

Which of the following is an **anti-pattern** in prompt engineering?

A) Specifying the desired output format explicitly  
B) Including relevant examples in the prompt  
C) Writing vague instructions like "Do something useful with this data"  
D) Setting temperature to 0 for factual tasks  

---

### Question 8

A developer uses this prompt template:

```
Summarise the following {{language}} article in {{num_points}} bullet points:
{{article_text}}
```

What is the main benefit of using prompt templates with placeholders?

A) They make the prompt longer, which improves accuracy  
B) They separate logic from data, enabling reuse across different inputs  
C) They automatically tune the model's parameters  
D) They bypass the need for a system message  

---

### Question 9

You are building a customer feedback classifier. The system prompt specifies JSON output, but occasionally the model returns plain text instead. Which approach is MOST likely to fix this?

A) Increase the temperature to 1.5  
B) Add few-shot examples showing the exact JSON format expected  
C) Remove the system message entirely  
D) Set max_tokens to 1  

---

### Question 10

Which combination of parameters is recommended for a **creative writing** task such as generating poetry?

A) temperature: 0.0, top_p: 0.1  
B) temperature: 0.9, top_p: 0.95  
C) temperature: 0.0, top_p: 0.95  
D) temperature: 2.0, top_p: 0.1  

---

## Answers

| # | Answer | Explanation |
|---|--------|-------------|
| 1 | **B** | The system message defines the model's behaviour, persona, constraints, and output format. It is processed before user messages and shapes all subsequent responses. |
| 2 | **C** | Zero-shot prompting provides the task instruction without any examples. The model must infer the classification criteria from the instruction alone. |
| 3 | **B** | 2–5 well-chosen, diverse examples typically provide the best balance between clarity and token efficiency. Too few may not establish a pattern; too many waste context window space. |
| 4 | **C** | Temperature 0.0 makes the model select the highest-probability token at each step, producing deterministic (identical) outputs for the same input. |
| 5 | **C** | Chain-of-thought prompting explicitly instructs the model to break down its reasoning into sequential steps, improving accuracy on complex problems. |
| 6 | **B** | top_p (nucleus sampling) controls the cumulative probability threshold for token selection. A higher value includes more candidate tokens, producing more diverse outputs. |
| 7 | **C** | Vague, ambiguous instructions are a common anti-pattern. Effective prompts include clear, specific instructions about the task, format, and constraints. |
| 8 | **B** | Prompt templates with placeholders decouple the prompt structure from specific input data, making the same template reusable across different inputs and scenarios. |
| 9 | **B** | Few-shot examples demonstrating the exact expected JSON format are the most reliable way to ensure consistent output formatting. The model learns the pattern from examples. |
| 10 | **B** | Creative tasks benefit from higher temperature (0.7–1.0) and high top_p (0.9–1.0) to encourage diverse, imaginative word choices. Option D's temperature of 2.0 would produce incoherent output. |

---

**Passing Score:** 7 out of 10 (70%)

---

*Azure AI Foundry — Zero to Hero Training Team · April 2026*
