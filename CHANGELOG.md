# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

## [3.0.0] - 2026-03-25

### Added — 🆕 Major Expansion: 7 Weeks → 10 Weeks (50 Days)

#### New Week 2: Microsoft.Extensions.AI — Deep Dive
- Day 1: IChatClient & Provider Abstraction — swap providers with zero code changes
- Day 2: Streaming & Async Patterns — `IAsyncEnumerable`, cancellation, interactive chat
- Day 3: Structured Output — `GetResponseAsync<T>()`, typed C# objects from AI
- Day 4: Function Calling with MEAI — `AIFunctionFactory`, `UseFunctionInvocation()`
- Day 5: Middleware Pipelines — `ChatClientBuilder`, custom `DelegatingChatClient`

#### New Week 3: Advanced AI Techniques
- Day 1: Multimodal AI — Vision (image analysis), `ImageContent`, local vision models
- Day 2: Local Models & Providers — Ollama setup, hybrid fallback strategy
- Day 3: Fine-Tuning Concepts — LoRA, decision matrix (prompt vs RAG vs fine-tune)
- Day 4: Advanced Prompt Engineering — ReAct, Tree-of-Thought, Self-Consistency
- Day 5: AI Evaluation Basics — LLM-as-Judge, evaluation harness, quality metrics

#### New Week 8: AI Agents & Microsoft Agent Framework (expanded)
- Day 3: Multi-Agent Workflows — sequential pipelines, group chat, debate patterns
- Day 4: Model Context Protocol (MCP) — MCP server in .NET, tool discovery, integration

#### New Week 9: Production AI Engineering
- Day 1: AI-Powered ASP.NET Core APIs — streaming SSE, structured output endpoints
- Day 2: Observability & Monitoring — OpenTelemetry, cost tracking middleware
- Day 3: Content Safety & Guardrails — prompt injection defense, PII redaction pipeline
- Day 4: AI Testing & Evaluation — mock IChatClient, testing pyramid, eval suites
- Day 5: .NET 10 Migration Guide — diff-style before/after migration patterns

#### New Week 10: Responsible AI & Capstone
- Day 1: Responsible AI Principles — Microsoft's 6 principles, production checklist
- Day 2: Bias Detection & Fairness — automated demographic testing, fairness metrics
- Day 3: Security for AI Systems — OWASP LLM Top 10, per-user rate limiting, audit logging
- Day 4: Capstone Architecture — Clean Architecture, Docker, full middleware pipeline
- Day 5: Capstone Build & Deploy — RAG agent, streaming API, deployment checklist

### Changed — Restructured Curriculum
- **Semantic Kernel** moved from Week 2 → Week 4 (MEAI must come first)
- **Embeddings** moved from Week 3 → Week 5
- **Vector Storage** moved from Week 4 → Week 6
- **RAG Pipeline** moved from Week 5 → Week 7
- **Agents** moved from Week 6 → Week 8 (expanded with MAF/MCP)
- **Week 7 content** split across Weeks 2, 3, 8, 9, 10
- Main README.md completely rewritten with new 10-week Mermaid diagram
- Progress tracker expanded from 35 → 50 items
- Tech stack table updated with MCP, OpenTelemetry, LLM-as-Judge
- References section expanded with MCP and .NET MCP SDK links

### Statistics
- **Weeks:** 7 → 10
- **Days:** 35 → 50
- **New days created:** 15
- **Days moved/refactored:** 8
- **Topic gaps filled:** 12

---

## [2.0.0] - 2026-03-25

### Added — 🆕 Based on [Generative AI for Beginners .NET v2](https://github.com/microsoft/Generative-AI-for-beginners-dotnet) (March 2026)
- **Week 7: Responsible AI & Production Patterns** (5 new days)
  - Day 1: Responsible AI Principles — Microsoft's 6 principles, bias types, fairness testing code
  - Day 2: Content Safety & Guardrails — Prompt injection defense, PII redaction, HITL for agents
  - Day 3: Structured Output & Middleware — `ChatClientBuilder` pipelines, typed AI responses
  - Day 4: Local Models & Deployment — Ollama, Docker Model Runner, AI Toolkit, Foundry Local
  - Day 5: .NET 10 Migration Guide — MAF agents, `AzureCliCredential`, modern patterns
- Generated architecture diagrams: .NET 10 AI stack, MAF architecture, SK→MAF evolution, Responsible AI principles, Middleware pipeline

### Changed — .NET 10 Updates
- Updated tech stack: .NET 8 → .NET 10 (LTS), C# 12 → C# 14
- Updated LLM Provider: Added Ollama local inference alongside Azure OpenAI
- Added Microsoft Agent Framework (MAF) as new agent SDK
- Added `AzureCliCredential` as standard auth pattern
- Updated model references: GPT-4o → GPT-5-mini, Phi-3 → Phi-4 Mini
- Updated roadmap diagram and tech stack table in README.md and roadmap.md
- Expanded progress tracker with Week 7 entries
- Added v2 course cross-references to existing Week 1 (Day 1, Day 3) and Week 6 (Day 1) materials
- Updated references section with v2 course, MAF, Responsible AI, .NET 10, and Ollama links

## [Unreleased]

### Changed
- Rebranded repository to **Dotnet AI Engineer Roadmap**.
- Added professional header banner and enhanced README.md structure.
- Modernized roadmap visualization with Mermaid diagrams.
- Standardized documentation titles across the repository.

## [1.0.0] - 2026-03-16

### Added
- Complete 6-week AI Engineer roadmap with .NET/C# ecosystem
- Week 1: AI Fundamentals & The .NET API Layer (5 days)
- Week 2: Microsoft Semantic Kernel — The Orchestrator (5 days)
- Week 3: Embeddings & Data Processing (5 days)
- Week 4: Vector Storage & Semantic Search (5 days)
- Week 5: Building the RAG Pipeline (5 days)
- Week 6: Autonomous AI Agents (5 days)
- Prerequisites and environment setup guides
- Code samples for each day with working .NET 8 projects
- Capstone project architecture and planning docs
- Repository foundation (LICENSE, CONTRIBUTING, .gitignore)
