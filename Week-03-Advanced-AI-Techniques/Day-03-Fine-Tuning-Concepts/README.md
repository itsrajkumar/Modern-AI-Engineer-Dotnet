# Day 3: Fine-Tuning Concepts

> **Type:** 📖 Theory | **Time:** ~3 hours

---

## 🎯 Learning Objectives

- Understand what fine-tuning is and when to use it
- Compare Prompt Engineering vs RAG vs Fine-Tuning vs Training from Scratch
- Learn about LoRA and parameter-efficient fine-tuning
- Understand the fine-tuning workflow and data requirements
- Know how to use fine-tuned models in .NET applications

---

## 📖 What is Fine-Tuning?

Fine-tuning is **retraining a pre-trained model** on your own domain-specific dataset to improve its performance for a specific task.

```
Pre-trained Model              Fine-Tuned Model
(knows everything              (knows everything
 about everything)              + is EXPERT in YOUR domain)

"I know C#, Python,            "I know C#, Python, Java...
 Java, cooking,                 AND I specifically know
 history, etc."                 YOUR company's API style,
                                YOUR code conventions,
                                YOUR error patterns."
```

---

## 📊 When to Use What: Decision Matrix

| Strategy | Cost | Time | When |
|----------|------|------|------|
| **Prompt Engineering** | Free | Minutes | Model already knows the answer, just needs guidance |
| **Few-Shot Prompting** | Token cost | Minutes | Need specific output format or style |
| **RAG** | API + DB cost | Hours | Need access to external/private data |
| **Fine-Tuning** | Training cost | Days | Need specialized behavior or domain expertise |
| **Train from Scratch** | $$$$$ | Weeks+ | Entirely new domain, no suitable base model |

### Decision Flowchart

```
Start here: "My AI isn't performing well enough"
│
├── Is the answer in your data? ──YES──► RAG (Week 7)
│                                        Inject data into prompts
│
├── Is it a formatting/style ──YES──► Prompt Engineering (Week 1)
│   issue?                            Better system prompts
│
├── Does it need 10+ ──YES──► Few-Shot Prompting (Week 1)
│   consistent examples?              Examples in the prompt
│
├── Does it need ──YES──► Fine-Tuning (This lesson)
│   domain-specific                   Retrain on your data
│   behavior?
│
└── None of the above ──────► Custom Training
    work?                     Full model training (advanced)
```

---

## 📖 How Fine-Tuning Works

```
Step 1: Prepare Training Data
  ┌─────────────────────────────────────────┐
  │ {"messages": [                          │
  │   {"role": "system", "content": "..."},│
  │   {"role": "user", "content": "..."},  │
  │   {"role": "assistant", "content": "..."}│
  │ ]}                                      │
  │ ... (100-10,000 examples)               │
  └─────────────────────────────────────────┘

Step 2: Upload to Provider (OpenAI / Azure)

Step 3: Train (hours, provider handles GPU)

Step 4: Use fine-tuned model via same IChatClient!
  ┌─────────────────────────────────────────┐
  │ // Just change the model ID!            │
  │ .AsChatClient("ft:gpt-4o-mini:myorg")  │
  └─────────────────────────────────────────┘
```

---

## 📖 LoRA: Parameter-Efficient Fine-Tuning

**LoRA (Low-Rank Adaptation)** is the modern approach to fine-tuning:

```
Full Fine-Tuning:              LoRA Fine-Tuning:
───────────────────            ──────────────────────
Modify ALL model weights       Modify SMALL adapter weights
(billions of parameters)       (millions of parameters)

Requires massive GPU           Runs on consumer GPU
Expensive ($$$)                Affordable ($)
Slow (hours-days)              Fast (minutes-hours)
Can degrade general ability    Preserves general ability
```

### Using LoRA Models in .NET

```csharp
// Fine-tuned models integrate via the same IChatClient!
// No code changes — just a different model ID.

// OpenAI fine-tuned model:
IChatClient client = new OpenAIClient(apiKey)
    .AsChatClient("ft:gpt-4o-mini-2024-07-18:my-org::abc123");

// Local fine-tuned model via Ollama:
IChatClient localClient = new OllamaChatClient(
    new Uri("http://localhost:11434"),
    "my-finetuned-model:latest");

// Same business logic works with both!
var response = await client.GetResponseAsync("Analyze this code...");
```

---

## 💡 Fine-Tuning Best Practices

| Practice | Details |
|----------|---------|
| **Start with prompting** | Only fine-tune after exhausting prompt engineering |
| **Quality > Quantity** | 100 perfect examples > 10,000 mediocre ones |
| **Consistent format** | All examples should follow the same structure |
| **Evaluate before & after** | Measure improvement with test set |
| **Version your models** | Track which data produced which model |
| **Don't overtrain** | 2-4 epochs is usually sufficient |

---

## 🧪 Self-Assessment

1. What is the difference between fine-tuning and RAG?
2. What is LoRA and why is it preferred over full fine-tuning?
3. How many training examples are typically needed?
4. How do you use a fine-tuned model in .NET code?
5. When should you NOT fine-tune?

<details>
<summary>Answers</summary>

1. RAG adds external data at runtime; fine-tuning changes model weights permanently
2. LoRA modifies small adapter weights instead of all parameters — cheaper, faster, preserves general ability
3. 100-10,000 high-quality examples (quality matters more than quantity)
4. Same `IChatClient` — just change the model ID string
5. When prompt engineering or RAG can solve the problem (they're cheaper and faster)

</details>

---

## ➡️ Next

Continue to **[Day 4: Advanced Prompt Engineering](../Day-04-Advanced-Prompt-Engineering/README.md)**
