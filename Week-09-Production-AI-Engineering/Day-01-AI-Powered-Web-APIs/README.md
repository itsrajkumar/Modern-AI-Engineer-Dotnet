# Day 1: AI-Powered ASP.NET Core APIs

> **Type:** 💻 Code | **Time:** ~3 hours | **Project:** AIChatApi

---

## 🎯 Learning Objectives

- Build a production-ready AI chat API with ASP.NET Core Minimal APIs
- Implement streaming chat endpoints with Server-Sent Events (SSE)
- Structure AI interactions as proper API controllers
- Handle concurrent requests, errors, and timeouts
- Integrate DI-registered `IChatClient` into Web API context

---

## 💻 Code Sample: Complete AI Chat API

```csharp
using Microsoft.Extensions.AI;
using OpenAI;
using System.Text.Json;

// =====================================================
// Production AI Chat API — ASP.NET Core Minimal APIs
// =====================================================

var builder = WebApplication.CreateBuilder(args);

// Register IChatClient with middleware
builder.Services.AddChatClient(pipeline => pipeline
    .UseOpenTelemetry()
    .UseDistributedCache()
    .UseFunctionInvocation()
    .Use(new OpenAIClient(builder.Configuration["OpenAI:ApiKey"]!)
        .AsChatClient("gpt-4o-mini")));

builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

var app = builder.Build();
app.UseSwagger();
app.UseSwaggerUI();

// =====================================================
// Endpoint 1: Simple Chat (non-streaming)
// =====================================================
app.MapPost("/api/chat", async (
    ChatRequest request,
    IChatClient chatClient,
    CancellationToken ct) =>
{
    var messages = new List<ChatMessage>
    {
        new(ChatRole.System, request.SystemPrompt ?? "You are a helpful assistant."),
        new(ChatRole.User, request.Message)
    };

    var options = new ChatOptions
    {
        Temperature = request.Temperature ?? 0.7f,
        MaxOutputTokens = request.MaxTokens ?? 500
    };

    var response = await chatClient.GetResponseAsync(messages, options, ct);

    return Results.Ok(new ChatResponse(
        Message: response.Message.Text ?? "",
        Model: response.ModelId ?? "",
        InputTokens: response.Usage?.InputTokenCount ?? 0,
        OutputTokens: response.Usage?.OutputTokenCount ?? 0
    ));
})
.WithName("Chat")
.WithOpenApi();

// =====================================================
// Endpoint 2: Streaming Chat (Server-Sent Events)
// =====================================================
app.MapPost("/api/chat/stream", async (
    ChatRequest request,
    IChatClient chatClient,
    HttpContext httpContext,
    CancellationToken ct) =>
{
    httpContext.Response.ContentType = "text/event-stream";
    httpContext.Response.Headers.CacheControl = "no-cache";
    httpContext.Response.Headers.Connection = "keep-alive";

    var messages = new List<ChatMessage>
    {
        new(ChatRole.System, request.SystemPrompt ?? "You are a helpful assistant."),
        new(ChatRole.User, request.Message)
    };

    await foreach (var update in chatClient.GetStreamingResponseAsync(
        messages, cancellationToken: ct))
    {
        if (!string.IsNullOrEmpty(update.Text))
        {
            var data = JsonSerializer.Serialize(new { text = update.Text });
            await httpContext.Response.WriteAsync($"data: {data}\n\n", ct);
            await httpContext.Response.Body.FlushAsync(ct);
        }
    }

    await httpContext.Response.WriteAsync("data: [DONE]\n\n", ct);
})
.WithName("ChatStream")
.WithOpenApi();

// =====================================================
// Endpoint 3: Structured Output (typed response)
// =====================================================
app.MapPost("/api/analyze", async (
    AnalyzeRequest request,
    IChatClient chatClient,
    CancellationToken ct) =>
{
    var result = await chatClient.GetResponseAsync<ContentAnalysis>(
        $"Analyze the following text for sentiment, topics, and language:\n\n{request.Text}",
        cancellationToken: ct);

    return Results.Ok(result.Result);
})
.WithName("Analyze")
.WithOpenApi();

app.Run();

// =====================================================
// DTOs
// =====================================================
record ChatRequest(
    string Message,
    string? SystemPrompt = null,
    float? Temperature = null,
    int? MaxTokens = null);

record ChatResponse(
    string Message,
    string Model,
    int InputTokens,
    int OutputTokens);

record AnalyzeRequest(string Text);

record ContentAnalysis(
    string Sentiment,
    string[] Topics,
    string Language,
    float ConfidenceScore);
```

---

## 📖 API Design Patterns

```
┌─────────────────────────────────────────────────────┐
│                  AI API PATTERNS                     │
│                                                      │
│  POST /api/chat          → Full response (JSON)     │
│  POST /api/chat/stream   → SSE streaming             │
│  POST /api/analyze       → Structured output (JSON)  │
│  POST /api/embed         → Embedding generation       │
│  POST /api/agent/run     → Agent execution            │
│                                                      │
│  Standard HTTP patterns:                              │
│  • CancellationToken for timeouts                    │
│  • 429 for rate limiting                              │
│  • 503 for provider outages                           │
│  • Swagger/OpenAPI documentation                      │
└─────────────────────────────────────────────────────┘
```

---

## 🔑 Key Takeaways

| Pattern | Implementation |
|---------|---------------|
| **DI Registration** | `builder.Services.AddChatClient(pipeline => ...)` |
| **Non-streaming** | `POST /api/chat` → JSON response |
| **Streaming** | `POST /api/chat/stream` → SSE (`text/event-stream`) |
| **Structured** | `GetResponseAsync<T>()` → typed C# objects |
| **Cancellation** | `CancellationToken` propagated from HTTP request |

---

## ➡️ Next

Continue to **[Day 2: Observability & Monitoring](../Day-02-Observability-and-Monitoring/README.md)**
