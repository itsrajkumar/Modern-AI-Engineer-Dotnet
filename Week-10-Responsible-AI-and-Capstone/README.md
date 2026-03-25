# Week 10: Responsible AI & Capstone Project

> **Goal:** Apply responsible AI principles and build a portfolio-worthy capstone that demonstrates all 10 weeks of learning.

---

## 🎯 Learning Objectives

By the end of this week, you will:
- Understand and implement Microsoft's 6 Responsible AI principles
- Build bias detection and fairness testing for AI systems
- Secure AI applications against adversarial attacks
- Design and build a complete AI-powered retail assistant
- Create a portfolio-ready project combining RAG, agents, safety, and production patterns

---

## 📅 Daily Breakdown

| Day | Topic | Type | Time |
|-----|-------|------|------|
| 1 | [Responsible AI Principles](./Day-01-Responsible-AI-Principles/README.md) | 📖 Theory | ~3 hrs |
| 2 | [Bias Detection & Fairness](./Day-02-Bias-Detection-and-Fairness/README.md) | 💻 Code | ~3 hrs |
| 3 | [Security for AI Systems](./Day-03-Security-for-AI-Systems/README.md) | 💻 Code | ~3 hrs |
| 4 | [Capstone: Architecture & Setup](./Day-04-Capstone-Architecture/README.md) | 🏗️ Project | ~4 hrs |
| 5 | [Capstone: Build & Deploy](./Day-05-Capstone-Build-and-Deploy/README.md) | 🏗️ Project | ~4 hrs |

---

## 🏗️ Capstone: AI-Powered Retail Assistant

The capstone project combines **every skill** from the 10-week roadmap:

```
┌─────────────────────────────────────────────────────────┐
│           AI-POWERED RETAIL ASSISTANT (.NET 10)         │
│                                                          │
│  Week 1: AI fundamentals, prompt engineering             │
│  Week 2: MEAI middleware, structured output               │
│  Week 3: Multimodal (product image analysis)             │
│  Week 4: Semantic Kernel plugins                          │
│  Week 5: Product catalog embeddings                       │
│  Week 6: Vector storage (pgvector)                        │
│  Week 7: RAG for store policy Q&A                         │
│  Week 8: MAF agent for order processing                   │
│  Week 9: Production API, observability, testing           │
│  Week 10: Responsible AI guardrails, deployment           │
└─────────────────────────────────────────────────────────┘
```

### Updated Tech Stack (.NET 10)

| Layer | Technology |
|-------|-----------|
| API | ASP.NET Core 10 Minimal APIs |
| AI Abstraction | Microsoft.Extensions.AI (`IChatClient`) |
| Orchestration | Semantic Kernel + MAF |
| AI Provider | Azure OpenAI + Ollama (hybrid) |
| Vector Database | PostgreSQL + pgvector |
| Auth | `AzureCliCredential` |
| Middleware | OpenTelemetry, Content Safety, Caching |
| Testing | xUnit + NSubstitute + AI Eval |
| Deployment | Docker Compose |

---

## ➡️ Start Here

Begin with **[Day 1: Responsible AI Principles](./Day-01-Responsible-AI-Principles/README.md)**
