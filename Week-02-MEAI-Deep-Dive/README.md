# Week 2: Microsoft.Extensions.AI — Deep Dive

> **Goal:** Master the unified AI abstraction layer that powers ALL .NET AI development.

---

## 🎯 Learning Objectives

By the end of this week, you will:
- Deeply understand `IChatClient` and `IEmbeddingGenerator` abstractions
- Build streaming AI applications using `IAsyncEnumerable<T>`
- Get strongly-typed C# objects from AI with structured output
- Implement function calling without Semantic Kernel (pure MEAI)
- Compose middleware pipelines with `ChatClientBuilder`

---

## 📅 Daily Breakdown

| Day | Topic | Type | Time |
|-----|-------|------|------|
| 1 | [IChatClient & Provider Abstraction](./Day-01-IChatClient-Provider-Abstraction/README.md) | 💻 Code | ~3 hrs |
| 2 | [Streaming & Async Patterns](./Day-02-Streaming-and-Async-Patterns/README.md) | 💻 Code | ~3 hrs |
| 3 | [Structured Output](./Day-03-Structured-Output/README.md) | 💻 Code | ~3 hrs |
| 4 | [Function Calling with MEAI](./Day-04-Function-Calling-With-MEAI/README.md) | 💻 Code | ~3 hrs |
| 5 | [Middleware Pipelines](./Day-05-Middleware-Pipelines/README.md) | 💻 Code | ~3 hrs |

---

## 🏗️ Why MEAI Before Semantic Kernel?

```
                    ┌─────────────────────────────────────────┐
                    │          YOUR .NET APPLICATION           │
                    │                                          │
                    │  ┌───────────────────────────────────┐  │
                    │  │     Semantic Kernel (Week 4)       │  │
                    │  │     High-level orchestrator        │  │
                    │  └───────────────┬───────────────────┘  │
                    │                  │ Uses                   │
                    │  ┌───────────────▼───────────────────┐  │
                    │  │ Microsoft.Extensions.AI (Week 2)  │  │  ◄── THIS WEEK
                    │  │ IChatClient, IEmbeddingGenerator  │  │
                    │  │ Middleware, Structured Output      │  │
                    │  └───────────────┬───────────────────┘  │
                    │                  │                        │
                    │  ┌───────┬───────┴───────┬────────────┐ │
                    │  │OpenAI │  Azure OpenAI │    Ollama  │ │
                    │  └───────┘  └────────────┘  └─────────┘ │
                    └──────────────────────────────────────────┘
```

MEAI is the **foundation** that everything else builds on. Semantic Kernel, MAF, and custom code all use `IChatClient` under the hood. You need to master MEAI first to understand how the higher-level frameworks work.

---

## 🔑 Key Abstractions This Week

| Interface | Purpose | Analogy |
|-----------|---------|---------|
| `IChatClient` | Send messages, get AI responses | Like `HttpClient` for AI |
| `IEmbeddingGenerator<T,E>` | Convert text → vectors | Like a hash function, but for meaning |
| `ChatClientBuilder` | Compose middleware pipeline | Like ASP.NET Core middleware |
| `AIFunction` | Expose C# methods to AI | Like API controller actions |
| `ChatOptions` | Configure AI behavior | Like `HttpRequestMessage` options |

---

## ➡️ Start Here

Begin with **[Day 1: IChatClient & Provider Abstraction](./Day-01-IChatClient-Provider-Abstraction/README.md)**
