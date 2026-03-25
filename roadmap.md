# 🗺️ Dotnet AI Engineer — Visual Roadmap

> **A strategic overview of the 10-week journey to mastering AI engineering within the .NET 10 ecosystem.**
>
> 🆕 *v3.0 — Expanded to 10 Weeks (50 Days) — Includes MEAI Deep Dive, Multimodal AI, MCP, AI Testing, Production Engineering, and MAF*

---

## The Complete Journey

```mermaid
graph TD
    START([🎯 START: Prerequisites]) --> W1
    
    subgraph W1["Week 1: AI Fundamentals"]
        W1D1["Day 1: AI Theory<br/>LLMs, Tokens, Temperature"]
        W1D2["Day 2: Prompt Engineering<br/>Zero-shot, Few-shot, CoT"]
        W1D3["Day 3: Microsoft.Extensions.AI<br/>IChatClient Abstraction"]
        W1D4["Day 4: First API Connection<br/>Console App + OpenAI"]
        W1D5["Day 5: System Prompts<br/>Chat History & Roles"]
        W1D1 --> W1D2 --> W1D3 --> W1D4 --> W1D5
    end
    
    W1 --> W2
    
    subgraph W2["Week 2: MEAI Deep Dive 🆕"]
        W2D1["Day 1: IChatClient<br/>Provider Abstraction"]
        W2D2["Day 2: Streaming<br/>IAsyncEnumerable Patterns"]
        W2D3["Day 3: Structured Output<br/>GetResponseAsync&lt;T&gt;"]
        W2D4["Day 4: Function Calling<br/>MEAI-Native Tool Use"]
        W2D5["Day 5: Middleware Pipelines<br/>ChatClientBuilder"]
        W2D1 --> W2D2 --> W2D3 --> W2D4 --> W2D5
    end
    
    W2 --> W3
    
    subgraph W3["Week 3: Advanced Techniques 🆕"]
        W3D1["Day 1: Multimodal AI<br/>Vision & Image Analysis"]
        W3D2["Day 2: Local Models<br/>Ollama & Hybrid Fallback"]
        W3D3["Day 3: Fine-Tuning<br/>LoRA & Decision Matrix"]
        W3D4["Day 4: Advanced Prompting<br/>ReAct, Tree-of-Thought"]
        W3D5["Day 5: AI Evaluation<br/>LLM-as-Judge"]
        W3D1 --> W3D2 --> W3D3 --> W3D4 --> W3D5
    end
    
    W3 --> W4
    
    subgraph W4["Week 4: Semantic Kernel"]
        W4D1["Day 1: Kernel Architecture<br/>DI Container for AI"]
        W4D2["Day 2: Semantic Functions<br/>Prompt Templates"]
        W4D3["Day 3: Native C# Plugins<br/>KernelFunction Attribute"]
        W4D4["Day 4: Tool Calling<br/>AI Invokes Your C# Code"]
        W4D5["Day 5: State Management<br/>Sliding Window & Summarization"]
        W4D1 --> W4D2 --> W4D3 --> W4D4 --> W4D5
    end
    
    W4 --> W5
    
    subgraph W5["Week 5: Embeddings"]
        W5D1["Day 1: Embedding Theory<br/>Text → float Vector"]
        W5D2["Day 2: Generating Embeddings<br/>IEmbeddingGenerator in C#"]
        W5D3["Day 3: Document Chunking<br/>Split Docs for RAG"]
        W5D4["Day 4: Cosine Similarity<br/>Semantic Search Math"]
        W5D5["Day 5: Batch Pipeline<br/>Process Doc Directories"]
        W5D1 --> W5D2 --> W5D3 --> W5D4 --> W5D5
    end
    
    W5 --> W6
    
    subgraph W6["Week 6: Vector Storage"]
        W6D1["Day 1: Vector DB Theory<br/>ANN Algorithms, HNSW"]
        W6D2["Day 2: MongoDB Atlas<br/>$vectorSearch Pipeline"]
        W6D3["Day 3: PostgreSQL pgvector<br/>SQL + Vector Search"]
        W6D4["Day 4: Hybrid Search<br/>Keyword + Semantic"]
        W6D5["Day 5: Repository Pattern<br/>IVectorRepository"]
        W6D1 --> W6D2 --> W6D3 --> W6D4 --> W6D5
    end
    
    W6 --> W7
    
    subgraph W7["Week 7: RAG Pipeline"]
        W7D1["Day 1: RAG Architecture<br/>Ingestion + Query Pipelines"]
        W7D2["Day 2: Retrieval Step<br/>Question → Vector → Search"]
        W7D3["Day 3: Augmentation Step<br/>Context Injection Prompt"]
        W7D4["Day 4: End-to-End RAG<br/>Complete Implementation"]
        W7D5["Day 5: Edge Cases<br/>Fallbacks, Security, Logging"]
        W7D1 --> W7D2 --> W7D3 --> W7D4 --> W7D5
    end
    
    W7 --> W8
    
    subgraph W8["Week 8: AI Agents & MAF 🆕"]
        W8D1["Day 1: Agentic Architecture<br/>ReAct, Tool-Use, Planning"]
        W8D2["Day 2: SK → MAF Agents<br/>Agent Framework Migration"]
        W8D3["Day 3: Multi-Agent Workflows<br/>Pipeline & Group Chat"]
        W8D4["Day 4: MCP Protocol<br/>Universal Tool Connectivity"]
        W8D5["Day 5: Human-in-the-Loop<br/>Approval Gates & Safety"]
        W8D1 --> W8D2 --> W8D3 --> W8D4 --> W8D5
    end
    
    W8 --> W9
    
    subgraph W9["Week 9: Production Engineering 🆕"]
        W9D1["Day 1: AI Web APIs<br/>Streaming SSE Endpoints"]
        W9D2["Day 2: Observability<br/>OpenTelemetry & Cost Tracking"]
        W9D3["Day 3: Content Safety<br/>PII Redaction & Guards"]
        W9D4["Day 4: AI Testing<br/>Mock IChatClient & Eval"]
        W9D5["Day 5: .NET 10 Migration<br/>Before/After Patterns"]
        W9D1 --> W9D2 --> W9D3 --> W9D4 --> W9D5
    end
    
    W9 --> W10
    
    subgraph W10["Week 10: Responsible AI & Capstone 🆕"]
        W10D1["Day 1: Responsible AI<br/>6 Principles & Checklist"]
        W10D2["Day 2: Bias Detection<br/>Fairness Metrics & Testing"]
        W10D3["Day 3: AI Security<br/>OWASP LLM Top 10"]
        W10D4["Day 4: Capstone Setup<br/>Architecture & Docker"]
        W10D5["Day 5: Capstone Build<br/>Deploy & Portfolio"]
        W10D1 --> W10D2 --> W10D3 --> W10D4 --> W10D5
    end
    
    W10 --> CAPSTONE([🏆 CAPSTONE: AI Retail Assistant])
```

---

## Technology Stack Map

```
┌──────────────────────────────────────────────────────────────────┐
│                    YOUR .NET AI APPLICATION                       │
│                                                                   │
│  ┌─────────────────────────────────────────────────────────────┐ │
│  │              🆕 WEEK 10: RESPONSIBLE AI & CAPSTONE          │ │
│  │  6 Principles, Bias Detection, OWASP LLM Top 10, Capstone  │ │
│  ├─────────────────────────────────────────────────────────────┤ │
│  │              🆕 WEEK 9: PRODUCTION ENGINEERING              │ │
│  │  ASP.NET APIs, OpenTelemetry, Content Safety, AI Testing    │ │
│  ├─────────────────────────────────────────────────────────────┤ │
│  │              🆕 WEEK 8: AI AGENTS & MAF                     │ │
│  │  Agentic Patterns, MAF, Multi-Agent, MCP, Human-in-Loop    │ │
│  ├─────────────────────────────────────────────────────────────┤ │
│  │                    WEEK 7: RAG PIPELINE                      │ │
│  │  Retrieval → Augmentation → Generation → Validation          │ │
│  ├─────────────────────────────────────────────────────────────┤ │
│  │         WEEK 6: VECTOR STORAGE          │  WEEK 5: EMBEDDINGS│ │
│  │  MongoDB Atlas, pgvector, Hybrid Search │  Chunk, Embed,     │ │
│  │  Repository Pattern                     │  Cosine Similarity  │ │
│  ├─────────────────────────────────────────────────────────────┤ │
│  │                    WEEK 4: SEMANTIC KERNEL                   │ │
│  │  Kernel, Plugins, Semantic Functions, Tool Calling           │ │
│  ├─────────────────────────────────────────────────────────────┤ │
│  │              🆕 WEEK 3: ADVANCED AI TECHNIQUES              │ │
│  │  Multimodal, Local Models, Fine-Tuning, ReAct, Evaluation   │ │
│  ├─────────────────────────────────────────────────────────────┤ │
│  │              🆕 WEEK 2: MEAI DEEP DIVE                      │ │
│  │  IChatClient, Streaming, Structured Output, Middleware       │ │
│  ├─────────────────────────────────────────────────────────────┤ │
│  │                    WEEK 1: FUNDAMENTALS                      │ │
│  │  LLM Theory, Prompts, Microsoft.Extensions.AI, Chat Roles   │ │
│  └─────────────────────────────────────────────────────────────┘ │
│                                                                   │
│  ┌─────────────────────────────────────────────────────────────┐ │
│  │  FOUNDATION: .NET 10, C# 14, Azure/OpenAI/Ollama, MCP      │ │
│  └─────────────────────────────────────────────────────────────┘ │
└──────────────────────────────────────────────────────────────────┘
```

---

## Key Skills Acquired

| Skill | Where Learned | Industry Relevance |
|-------|--------------|-------------------|
| LLM API integration | Week 1 | Every AI application |
| Prompt engineering | Week 1 | Core AI Engineering skill |
| MEAI abstraction layer | Week 2 🆕 | Provider-agnostic AI code |
| Streaming & structured output | Week 2 🆕 | Real-time UX, data extraction |
| Middleware composition | Week 2 🆕 | Production AI pipelines |
| Multimodal AI (Vision) | Week 3 🆕 | Image analysis, accessibility |
| Local model deployment | Week 3 🆕 | Privacy, cost savings |
| Advanced prompting (ReAct) | Week 3 🆕 | Agentic AI reasoning |
| AI evaluation | Week 3 🆕 | Quality assurance for AI |
| AI orchestration (SK) | Week 4 | Enterprise AI apps |
| Plugin development | Week 4 | Extending AI capabilities |
| Embeddings | Week 5 | Search, recommendations |
| Vector databases | Week 6 | RAG, similarity search |
| RAG pipelines | Week 7 | #1 enterprise AI pattern |
| Agent development (MAF) | Week 8 🆕 | Next-gen AI applications |
| MCP integration | Week 8 🆕 | Universal tool connectivity |
| Multi-agent workflows | Week 8 🆕 | Complex orchestration |
| Production AI APIs | Week 9 🆕 | Shipping AI to production |
| AI observability | Week 9 🆕 | Cost tracking, monitoring |
| AI testing & evaluation | Week 9 🆕 | Reliable AI deployments |
| Content safety | Week 9 🆕 | Enterprise compliance |
| Responsible AI | Week 10 🆕 | Required for production |
| Bias detection | Week 10 🆕 | Fairness & ethics |
| AI security (OWASP) | Week 10 🆕 | Enterprise safety |
| Clean Architecture | Capstone | Production .NET |
