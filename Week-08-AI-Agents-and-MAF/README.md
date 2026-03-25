# Week 8: AI Agents & Microsoft Agent Framework

> **Goal:** Build autonomous, multi-agent systems using MAF, MCP, and production safety patterns.
>
> 🆕 *Updated for Microsoft Agent Framework (MAF) — the successor to Semantic Kernel Planners*

---

## 🎯 Learning Objectives

By the end of this week, you will:
- Understand agentic architecture patterns (ReAct, tool-use, planning)
- Build agents using both Semantic Kernel and the Microsoft Agent Framework
- Create multi-agent workflows (sequential, concurrent, group chat)
- Integrate external tools via Model Context Protocol (MCP)
- Implement Human-in-the-Loop safety patterns for agent actions

---

## 📅 Daily Breakdown

| Day | Topic | Type | Time |
|-----|-------|------|------|
| 1 | [Agentic Architecture & Patterns](./Day-01-Agentic-Architecture/README.md) | 📖 Theory | ~3 hrs |
| 2 | [SK Planners → MAF Agents](./Day-02-SK-Planners-to-MAF/README.md) | 💻 Code | ~3 hrs |
| 3 | [Multi-Agent Workflows](./Day-03-Multi-Agent-Workflows/README.md) | 💻 Code | ~3 hrs |
| 4 | [Model Context Protocol (MCP)](./Day-04-Model-Context-Protocol/README.md) | 💻 Code | ~3 hrs |
| 5 | [Human-in-the-Loop & Agent Safety](./Day-05-Human-in-the-Loop/README.md) | 💻 Code | ~3 hrs |

---

## 🏗️ Agent Evolution in .NET

```
2024 (SK v1.0)                2025 (SK v1.x)              2026 (MAF + SK)
─────────────────            ─────────────────           ─────────────────
StepwisePlanner              FunctionChoiceBehavior       Microsoft Agent
(deprecated)                 .Auto()                      Framework (MAF)
                             ↕                            ↕
Explicit plan                LLM natively decides         Dedicated agent
generation                   which tools to call          building SDK
                                                          ↕
                                                          Multi-agent
                                                          orchestration +
                                                          MCP integration
```

---

## 📊 SK vs MAF Decision Guide

```
What are you building?
│
├── Simple tool calling? ─────────► Semantic Kernel (Week 4)
│                                   FunctionChoiceBehavior.Auto()
│
├── Single autonomous agent? ─────► MAF ChatCompletionAgent (Day 2)
│                                   Dedicated, purpose-built
│
├── Multiple agents working ──────► MAF AgentPipeline (Day 3)
│   together?                       Sequential or concurrent
│
├── Agents that debate / ─────────► MAF GroupChat (Day 3)
│   collaborate?                    Round-robin conversations
│
└── External tool ────────────────► MCP Protocol (Day 4)
    integration?                    Standardized tool discovery
```

---

## ➡️ Start Here

Begin with **[Day 1: Agentic Architecture & Patterns](./Day-01-Agentic-Architecture/README.md)**
