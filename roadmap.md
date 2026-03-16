# 🗺️ Dotnet AI Engineer — Visual Roadmap

> **A strategic overview of the 6-week journey to mastering AI engineering within the .NET ecosystem.**

---

## The Complete Journey

```mermaid
graph TD
    START([🎯 START: Prerequisites]) --> W1
    
    subgraph W1[Week 1: AI Fundamentals]
        W1D1[Day 1: AI Theory<br/>LLMs, Tokens, Temperature]
        W1D2[Day 2: Prompt Engineering<br/>Zero-shot, Few-shot, CoT]
        W1D3[Day 3: Microsoft.Extensions.AI<br/>IChatClient Abstraction]
        W1D4[Day 4: First API Connection<br/>Console App + OpenAI]
        W1D5[Day 5: System Prompts<br/>Chat History & Roles]
        W1D1 --> W1D2 --> W1D3 --> W1D4 --> W1D5
    end
    
    W1 --> W2
    
    subgraph W2[Week 2: Semantic Kernel]
        W2D1[Day 1: Kernel Architecture<br/>DI Container for AI]
        W2D2[Day 2: Semantic Functions<br/>Prompt Templates]
        W2D3[Day 3: Native C# Plugins<br/>KernelFunction Attribute]
        W2D4[Day 4: Tool Calling<br/>AI Invokes Your C# Code]
        W2D5[Day 5: State Management<br/>Sliding Window & Summarization]
        W2D1 --> W2D2 --> W2D3 --> W2D4 --> W2D5
    end
    
    W2 --> W3
    
    subgraph W3[Week 3: Embeddings]
        W3D1[Day 1: Embedding Theory<br/>Text → float Vector]
        W3D2[Day 2: Generating Embeddings<br/>IEmbeddingGenerator in C#]
        W3D3[Day 3: Document Chunking<br/>Split Docs for RAG]
        W3D4[Day 4: Cosine Similarity<br/>Semantic Search Math]
        W3D5[Day 5: Batch Pipeline<br/>Process Doc Directories]
        W3D1 --> W3D2 --> W3D3 --> W3D4 --> W3D5
    end
    
    W3 --> W4
    
    subgraph W4[Week 4: Vector Storage]
        W4D1[Day 1: Vector DB Theory<br/>ANN Algorithms, HNSW]
        W4D2[Day 2: MongoDB Atlas<br/>$vectorSearch Pipeline]
        W4D3[Day 3: PostgreSQL pgvector<br/>SQL + Vector Search]
        W4D4[Day 4: Hybrid Search<br/>Keyword + Semantic]
        W4D5[Day 5: Repository Pattern<br/>IVectorRepository]
        W4D1 --> W4D2 --> W4D3 --> W4D4 --> W4D5
    end
    
    W4 --> W5
    
    subgraph W5[Week 5: RAG Pipeline]
        W5D1[Day 1: RAG Architecture<br/>Ingestion + Query Pipelines]
        W5D2[Day 2: Retrieval Step<br/>Question → Vector → Search]
        W5D3[Day 3: Augmentation Step<br/>Context Injection Prompt]
        W5D4[Day 4: End-to-End RAG<br/>Complete Implementation]
        W5D5[Day 5: Edge Cases<br/>Fallbacks, Security, Logging]
        W5D1 --> W5D2 --> W5D3 --> W5D4 --> W5D5
    end
    
    W5 --> W6
    
    subgraph W6[Week 6: AI Agents]
        W6D1[Day 1: Agentic Architecture<br/>ReAct Pattern]
        W6D2[Day 2: SK Planners<br/>Auto Function Calling]
        W6D3[Day 3: Multi-Plugin Agents<br/>Inventory + Tax + Customer]
        W6D4[Day 4: Human-in-the-Loop<br/>Approval Gates]
        W6D5[Day 5: Capstone Planning<br/>Portfolio Project]
        W6D1 --> W6D2 --> W6D3 --> W6D4 --> W6D5
    end
    
    W6 --> CAPSTONE([🏆 CAPSTONE: AI Retail Assistant])
```

---

## Technology Stack Map

```
┌──────────────────────────────────────────────────────────────────┐
│                    YOUR .NET AI APPLICATION                       │
│                                                                   │
│  ┌─────────────────────────────────────────────────────────────┐ │
│  │                    WEEK 6: AGENTS                            │ │
│  │  Semantic Kernel Planners, ReAct, Human-in-the-Loop          │ │
│  ├─────────────────────────────────────────────────────────────┤ │
│  │                    WEEK 5: RAG PIPELINE                      │ │
│  │  Retrieval → Augmentation → Generation → Validation          │ │
│  ├─────────────────────────────────────────────────────────────┤ │
│  │         WEEK 4: VECTOR STORAGE          │  WEEK 3: EMBEDDINGS│ │
│  │  MongoDB Atlas, pgvector, Hybrid Search │  Chunk, Embed,     │ │
│  │  Repository Pattern                     │  Cosine Similarity  │ │
│  ├─────────────────────────────────────────────────────────────┤ │
│  │                    WEEK 2: SEMANTIC KERNEL                   │ │
│  │  Kernel, Plugins, Semantic Functions, Tool Calling           │ │
│  ├─────────────────────────────────────────────────────────────┤ │
│  │                    WEEK 1: FUNDAMENTALS                      │ │
│  │  LLM Theory, Prompts, Microsoft.Extensions.AI, Chat Roles   │ │
│  └─────────────────────────────────────────────────────────────┘ │
│                                                                   │
│  ┌─────────────────────────────────────────────────────────────┐ │
│  │  FOUNDATION: .NET 8, C# 12, Azure/OpenAI, Docker            │ │
│  └─────────────────────────────────────────────────────────────┘ │
└──────────────────────────────────────────────────────────────────┘
```

---

## Key Skills Acquired

| Skill | Where Learned | Industry Relevance |
|-------|--------------|-------------------|
| LLM API integration | Week 1 | Every AI application |
| Prompt engineering | Week 1-2 | Core AI Engineering skill |
| AI orchestration (SK) | Week 2 | Enterprise AI apps |
| Plugin development | Week 2-3 | Extending AI capabilities |
| Embeddings | Week 3 | Search, recommendations |
| Vector databases | Week 4 | RAG, similarity search |
| RAG pipelines | Week 5 | #1 enterprise AI pattern |
| Agent development | Week 6 | Next-gen AI applications |
| Clean Architecture | Capstone | Production .NET |
| Security (HITL) | Week 6 | Enterprise safety |
