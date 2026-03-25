# Week 9: Production AI Engineering

> **Goal:** Ship AI features to production with observability, safety, testing, and modern .NET 10 patterns.

---

## 🎯 Learning Objectives

By the end of this week, you will:
- Build AI-powered ASP.NET Core Web APIs with streaming endpoints
- Implement OpenTelemetry for AI observability and cost tracking
- Deploy content safety guardrails for production AI systems
- Write unit and integration tests for AI-powered features
- Migrate existing code to .NET 10 patterns and best practices

---

## 📅 Daily Breakdown

| Day | Topic | Type | Time |
|-----|-------|------|------|
| 1 | [AI-Powered ASP.NET Core APIs](./Day-01-AI-Powered-Web-APIs/README.md) | 💻 Code | ~3 hrs |
| 2 | [Observability & Monitoring](./Day-02-Observability-and-Monitoring/README.md) | 💻 Code | ~3 hrs |
| 3 | [Content Safety & Guardrails](./Day-03-Content-Safety-and-Guardrails/README.md) | 💻 Code | ~3 hrs |
| 4 | [AI Testing & Evaluation](./Day-04-AI-Testing-and-Evaluation/README.md) | 💻 Code | ~3 hrs |
| 5 | [.NET 10 Migration Guide](./Day-05-DotNet10-Migration-Guide/README.md) | 📖 Theory + Code | ~3 hrs |

---

## 🏗️ Production AI Architecture

```
                    ┌─────────────────────────────────────────────┐
                    │          PRODUCTION AI SYSTEM                │
                    │                                              │
 Client ──────────►│ ┌─────────────────────────────────────────┐ │
 (Browser/App)      │ │  ASP.NET Core API (Day 1)               │ │
                    │ │  POST /api/chat       (streaming)       │ │
                    │ │  POST /api/analyze    (structured)      │ │
                    │ └──────────────┬────────────────────────┘ │
                    │                │                            │
                    │ ┌──────────────▼────────────────────────┐ │
                    │ │  Middleware Pipeline (Week 2, Day 5)   │ │
                    │ │  ┌──────────────────────────────────┐  │ │
                    │ │  │ OpenTelemetry (Day 2)            │  │ │
                    │ │  │ → traces, metrics, logs          │  │ │
                    │ │  ├──────────────────────────────────┤  │ │
                    │ │  │ Content Safety (Day 3)           │  │ │
                    │ │  │ → PII redaction, prompt guards   │  │ │
                    │ │  ├──────────────────────────────────┤  │ │
                    │ │  │ Distributed Cache                │  │ │
                    │ │  │ → avoid duplicate API calls      │  │ │
                    │ │  ├──────────────────────────────────┤  │ │
                    │ │  │ Rate Limiting                    │  │ │
                    │ │  │ → protect API budget             │  │ │
                    │ │  └──────────────────────────────────┘  │ │
                    │ └──────────────┬────────────────────────┘ │
                    │                │                            │
                    │ ┌──────────────▼────────────────────────┐ │
                    │ │  IChatClient (Provider)               │ │
                    │ │  OpenAI / Azure / Ollama              │ │
                    │ └──────────────────────────────────────┘ │
                    └─────────────────────────────────────────────┘
```

---

## 📊 Production Checklist

| Area | Must Have | Details |
|------|-----------|---------|
| **Observability** | ✅ | Distributed traces, token counting, cost tracking |
| **Safety** | ✅ | Prompt injection defense, PII redaction, output filtering |
| **Testing** | ✅ | Unit tests for prompts, integration tests for pipelines |
| **Error Handling** | ✅ | Retry policies, fallback providers, graceful degradation |
| **Cost Control** | ✅ | Rate limiting, caching, token budgets per request |
| **Security** | ✅ | `AzureCliCredential`, no API keys in code, audit logging |
| **Performance** | ✅ | Streaming for UX, batch operations for throughput |

---

## ➡️ Start Here

Begin with **[Day 1: AI-Powered ASP.NET Core APIs](./Day-01-AI-Powered-Web-APIs/README.md)**
