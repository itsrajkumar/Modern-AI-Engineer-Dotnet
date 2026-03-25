# Day 5: Capstone — Build & Deploy

> **Type:** 🏗️ Project | **Time:** ~4 hours
>
> Complete the AI Retail Assistant with RAG, plugins, testing, and deployment.

---

## 🎯 Objectives

- Implement the RAG pipeline for store policy Q&A
- Build Order and Refund plugins with Human-in-the-Loop
- Add streaming chat endpoint with SSE
- Write unit and integration tests
- Deploy the full stack with Docker Compose

---

## 💻 Implementation: RAG Service

```csharp
public class RagService
{
    private readonly IVectorRepository _vectorRepo;
    private readonly IEmbeddingGenerator<string, Embedding<float>> _embedder;

    public RagService(
        IVectorRepository vectorRepo,
        IEmbeddingGenerator<string, Embedding<float>> embedder)
    {
        _vectorRepo = vectorRepo;
        _embedder = embedder;
    }

    public async Task<string> RetrieveContextAsync(string question, int topK = 3)
    {
        var embedding = await _embedder.GenerateEmbeddingAsync(question);
        var results = await _vectorRepo.SearchAsync(
            embedding.Vector.ToArray(), limit: topK);

        if (!results.Any())
            return "No relevant policy documents found.";

        return string.Join("\n\n---\n\n",
            results.Select((r, i) =>
                $"[Source {i + 1}: {r.Source}]\n{r.Text}"));
    }
}
```

---

## 💻 Implementation: Retail Agent

```csharp
public class RetailAgent
{
    private readonly IChatClient _chatClient;
    private readonly RagService _ragService;

    public RetailAgent(IChatClient chatClient, RagService ragService)
    {
        _chatClient = chatClient;
        _ragService = ragService;
    }

    public async IAsyncEnumerable<string> ChatStreamAsync(
        string userMessage,
        List<ChatMessage> history,
        [EnumeratorCancellation] CancellationToken ct = default)
    {
        // Retrieve relevant context via RAG
        var context = await _ragService.RetrieveContextAsync(userMessage);

        var tools = new List<AITool>
        {
            AIFunctionFactory.Create(LookupOrder, "LookupOrder",
                "Looks up an order by order ID"),
            AIFunctionFactory.Create(ProcessRefund, "ProcessRefund",
                "Processes a refund for an order (requires approval)")
        };

        var messages = new List<ChatMessage>
        {
            new(ChatRole.System, $"""
                You are a helpful retail assistant for TechStore.
                Use the following store policy context to answer questions:

                {context}

                Rules:
                - Always cite your sources when answering policy questions
                - For order lookups, use the LookupOrder tool
                - For refunds, use the ProcessRefund tool
                - Be concise and helpful
                """)
        };
        messages.AddRange(history);
        messages.Add(new(ChatRole.User, userMessage));

        var options = new ChatOptions { Tools = tools };

        await foreach (var update in _chatClient.GetStreamingResponseAsync(
            messages, options, ct))
        {
            if (!string.IsNullOrEmpty(update.Text))
                yield return update.Text;
        }
    }

    private static string LookupOrder(string orderId) =>
        orderId.ToUpper() switch
        {
            "ORD-001" => "Order ORD-001: Laptop Pro 15 × 1, shipped 2026-03-20, delivered",
            "ORD-002" => "Order ORD-002: Wireless Mouse × 2, processing, ETA 2026-03-28",
            _ => $"Order {orderId} not found"
        };

    private static string ProcessRefund(string orderId, string reason) =>
        $"⚠️ Refund request for {orderId} queued for human approval. Reason: {reason}";
}
```

---

## 💻 Implementation: Streaming Endpoint

```csharp
public static class RetailEndpoints
{
    public static void MapRetailEndpoints(this WebApplication app)
    {
        app.MapPost("/api/chat/stream", async (
            ChatRequest request,
            RetailAgent agent,
            HttpContext httpContext,
            CancellationToken ct) =>
        {
            httpContext.Response.ContentType = "text/event-stream";
            httpContext.Response.Headers.CacheControl = "no-cache";

            await foreach (var chunk in agent.ChatStreamAsync(
                request.Message, request.History ?? [], ct))
            {
                var data = System.Text.Json.JsonSerializer.Serialize(
                    new { text = chunk });
                await httpContext.Response.WriteAsync($"data: {data}\n\n", ct);
                await httpContext.Response.Body.FlushAsync(ct);
            }

            await httpContext.Response.WriteAsync("data: [DONE]\n\n", ct);
        })
        .WithName("ChatStream")
        .WithOpenApi();
    }
}

record ChatRequest(string Message, List<ChatMessage>? History = null);
```

---

## ✅ Final Checklist

### Implementation
- [ ] RAG pipeline for store policies
- [ ] Order lookup plugin
- [ ] Refund plugin with Human-in-the-Loop
- [ ] Streaming chat endpoint (SSE)
- [ ] Structured output for analytics

### Safety & Quality
- [ ] Prompt injection guard middleware
- [ ] PII redaction middleware
- [ ] OpenTelemetry traces and metrics
- [ ] Cost tracking middleware

### Testing
- [ ] Unit tests with mocked IChatClient
- [ ] Integration tests with real model
- [ ] AI evaluation suite (relevance, coherence)

### Deployment
- [ ] Docker Compose (API + PostgreSQL + Ollama)
- [ ] Swagger documentation
- [ ] README with setup instructions

---

## 🏆 Congratulations!

You've completed the **10-week AI Engineering with .NET** roadmap!

### Skills You've Gained:

| Week | Skill |
|------|-------|
| 1 | AI fundamentals, prompt engineering, MEAI basics |
| 2 | IChatClient, streaming, structured output, middleware |
| 3 | Multimodal AI, local models, fine-tuning, evaluation |
| 4 | Semantic Kernel orchestration, plugins |
| 5 | Embeddings, chunking, batch processing |
| 6 | Vector databases, hybrid search |
| 7 | RAG pipeline, retrieval, augmentation |
| 8 | AI agents, MAF, MCP, multi-agent workflows |
| 9 | Production APIs, observability, testing, safety |
| 10 | Responsible AI, security, capstone deployment |

**You are now a .NET AI Engineer!** 🎉
