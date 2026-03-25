# Day 4: Capstone — Architecture & Setup

> **Type:** 🏗️ Project | **Time:** ~4 hours
>
> Build a portfolio-worthy AI-Powered Retail Assistant using everything from Weeks 1-10.

---

## 🎯 Objectives

- Design the capstone architecture using .NET 10 + MEAI + MAF
- Set up the solution structure with Clean Architecture
- Configure databases (PostgreSQL + pgvector) via Docker
- Register all AI services with middleware pipeline
- Build the initial chat endpoint

---

## 🏗️ Architecture

```
                    ┌─────────────────────────────────────────┐
                    │         AI Retail Assistant              │
                    │         ASP.NET Core 10 API              │
                    │                                          │
  Client ──────►  │  POST /api/chat/stream (SSE)             │
  (Swagger/UI)    │  POST /api/chat (JSON)                    │
                    │  POST /api/analyze (Structured Output)   │
                    │                                          │
                    │  ┌────────────────────────────────────┐  │
                    │  │  Middleware Pipeline (Week 2)       │  │
                    │  │  OpenTelemetry → Cache → Safety     │  │
                    │  │  → PII Redaction → Rate Limiting    │  │
                    │  └──────────────┬──────────────────┘  │
                    │                 │                       │
                    │  ┌──────────────▼──────────────────┐  │
                    │  │  Retail Agent (MAF)               │  │
                    │  │                                   │  │
                    │  │  Decides which tool to use:        │  │
                    │  │  📋 Policy Q? → RAG Pipeline       │  │
                    │  │  📦 Order?    → OrderPlugin        │  │
                    │  │  💰 Refund?   → RefundPlugin + HIL │  │
                    │  │  🖼️ Product?  → Vision Analysis    │  │
                    │  └──────────────┬──────────────────┘  │
                    │                 │                       │
                    │  ┌──────────────▼──────────────────┐  │
                    │  │  Data Layer                       │  │
                    │  │  PostgreSQL + pgvector (embeddings)│  │
                    │  │  PostgreSQL + EF Core (orders)     │  │
                    │  └──────────────────────────────────┘  │
                    └─────────────────────────────────────────┘
```

---

## 💻 Solution Structure

```powershell
# Create solution
dotnet new sln -n RetailAssistant
mkdir src tests

# API project
dotnet new webapi -n RetailAssistant.API -o src/RetailAssistant.API
# Core (business logic)
dotnet new classlib -n RetailAssistant.Core -o src/RetailAssistant.Core
# Infrastructure (data access)
dotnet new classlib -n RetailAssistant.Infrastructure -o src/RetailAssistant.Infrastructure
# Tests
dotnet new xunit -n RetailAssistant.Tests -o tests/RetailAssistant.Tests

# Add to solution
dotnet sln add src/RetailAssistant.API
dotnet sln add src/RetailAssistant.Core
dotnet sln add src/RetailAssistant.Infrastructure
dotnet sln add tests/RetailAssistant.Tests

# Add NuGet packages to API
cd src/RetailAssistant.API
dotnet add package Microsoft.Extensions.AI
dotnet add package Microsoft.Extensions.AI.OpenAI
dotnet add package Microsoft.Extensions.AI.Ollama
dotnet add package Microsoft.SemanticKernel
dotnet add package Azure.Identity
dotnet add package OpenTelemetry.Extensions.Hosting
dotnet add package OpenTelemetry.Exporter.OpenTelemetryProtocol
dotnet add package Npgsql.EntityFrameworkCore.PostgreSQL
dotnet add package Pgvector.EntityFrameworkCore
```

---

## 💻 Docker Compose

```yaml
# docker-compose.yml
services:
  postgres:
    image: pgvector/pgvector:pg16
    environment:
      POSTGRES_DB: retailassistant
      POSTGRES_USER: admin
      POSTGRES_PASSWORD: dev_password_123
    ports:
      - "5432:5432"
    volumes:
      - pgdata:/var/lib/postgresql/data

  ollama:
    image: ollama/ollama
    ports:
      - "11434:11434"
    volumes:
      - ollama_data:/root/.ollama

volumes:
  pgdata:
  ollama_data:
```

---

## 💻 Program.cs (API Registration)

```csharp
using Microsoft.Extensions.AI;
using Azure.Identity;
using RetailAssistant.Core;
using RetailAssistant.Infrastructure;

var builder = WebApplication.CreateBuilder(args);

// =====================================================
// AI Services — Full middleware pipeline
// =====================================================
builder.Services.AddChatClient(pipeline => pipeline
    .UseOpenTelemetry()
    .UseDistributedCache()
    .Use<PromptInjectionGuard>()
    .Use<PiiRedactionMiddleware>()
    .Use<CostTrackingMiddleware>()
    .UseFunctionInvocation()
    .Use(new OpenAIClient(builder.Configuration["OpenAI:ApiKey"]!)
        .AsChatClient("gpt-4o-mini")));

// =====================================================
// Application Services
// =====================================================
builder.Services.AddScoped<RetailAgent>();
builder.Services.AddScoped<RagService>();
builder.Services.AddScoped<OrderPlugin>();
builder.Services.AddScoped<RefundPlugin>();

// =====================================================
// Database
// =====================================================
builder.Services.AddDbContext<RetailDbContext>(options =>
    options.UseNpgsql(builder.Configuration.GetConnectionString("Postgres")));

builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

var app = builder.Build();
app.UseSwagger();
app.UseSwaggerUI();

// Map endpoints
app.MapRetailEndpoints();

app.Run();
```

---

## ✅ Day 4 Checklist

- [ ] Create solution structure
- [ ] Set up Docker Compose (PostgreSQL + Ollama)
- [ ] Configure AI middleware pipeline
- [ ] Register all services in DI
- [ ] Create initial `/api/chat` endpoint
- [ ] Add Swagger documentation
- [ ] Verify basic chat works

---

## ➡️ Next

Continue to **[Day 5: Capstone Build & Deploy](../Day-05-Capstone-Build-and-Deploy/README.md)**
