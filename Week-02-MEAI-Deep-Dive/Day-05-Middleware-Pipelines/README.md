# Day 5: Middleware Pipelines

> **Type:** 💻 Code | **Time:** ~3 hours | **Project:** MiddlewareDemo
>
> 🆕 *Based on concepts from [v2 Course: Composing Middleware](https://github.com/microsoft/Generative-AI-for-beginners-dotnet)*

---

## 🎯 Learning Objectives

- Build `ChatClientBuilder` middleware pipelines
- Implement caching, logging, telemetry, and rate limiting middleware
- Write custom `DelegatingChatClient` middleware
- Understand middleware execution order
- Compare with ASP.NET Core middleware pipeline

---

## 📖 The Middleware Pattern

```
ASP.NET Core Middleware:           MEAI Middleware:
──────────────────────────        ──────────────────────────
app.UseAuthentication()            pipeline.UseOpenTelemetry()
app.UseRateLimiting()              pipeline.UseDistributedCache()
app.UseResponseCaching()           pipeline.UseFunctionInvocation()
app.UseRouting()                   pipeline.Use(provider)

HTTP Request → Middleware → Response    AI Request → Middleware → Response
```

---

## 💻 Code Sample

```csharp
using Microsoft.Extensions.AI;
using Microsoft.Extensions.Caching.Distributed;
using Microsoft.Extensions.Caching.Memory;
using Microsoft.Extensions.Options;
using OpenAI;

// =====================================================
// Day 5: Composing Middleware Pipelines
// =====================================================

var config = new ConfigurationBuilder().AddUserSecrets<Program>().Build();

// Create a fully-composed middleware pipeline
IChatClient chatClient = new ChatClientBuilder(
    new OpenAIClient(config["OpenAI:ApiKey"]!)
        .AsChatClient("gpt-4o-mini"))
    .UseOpenTelemetry()                       // 1. Traces & metrics
    .UseDistributedCache(new MemoryDistributedCache(   // 2. Caching
        Options.Create(new MemoryDistributedCacheOptions())))
    .UseFunctionInvocation()                  // 3. Auto tool execution
    .Use<LoggingMiddleware>()                 // 4. Custom logging
    .Use<RateLimitingMiddleware>()            // 5. Custom rate limiting
    .Build();

// =====================================================
// Test the pipeline
// =====================================================

// First call — hits the AI provider
Console.WriteLine("--- Call 1 (fresh) ---");
var r1 = await chatClient.GetResponseAsync("What is .NET 10?");
Console.WriteLine($"Response: {r1.Message.Text?[..80]}...\n");

// Second call — served from cache!
Console.WriteLine("--- Call 2 (cached) ---");
var r2 = await chatClient.GetResponseAsync("What is .NET 10?");
Console.WriteLine($"Response: {r2.Message.Text?[..80]}...\n");

// =====================================================
// Custom Middleware: Logging
// =====================================================
public class LoggingMiddleware : DelegatingChatClient
{
    public LoggingMiddleware(IChatClient inner) : base(inner) { }

    public override async Task<ChatResponse> GetResponseAsync(
        IList<ChatMessage> chatMessages,
        ChatOptions? options = null,
        CancellationToken cancellationToken = default)
    {
        Console.ForegroundColor = ConsoleColor.DarkGray;
        Console.WriteLine($"  📤 Sending {chatMessages.Count} messages to AI...");
        Console.ResetColor();

        var sw = System.Diagnostics.Stopwatch.StartNew();
        var response = await base.GetResponseAsync(
            chatMessages, options, cancellationToken);
        sw.Stop();

        Console.ForegroundColor = ConsoleColor.DarkGray;
        Console.WriteLine($"  📥 Received response in {sw.ElapsedMilliseconds}ms");
        Console.WriteLine($"  📊 Tokens: {response.Usage?.InputTokenCount} in / "
            + $"{response.Usage?.OutputTokenCount} out");
        Console.ResetColor();

        return response;
    }
}

// =====================================================
// Custom Middleware: Rate Limiting
// =====================================================
public class RateLimitingMiddleware : DelegatingChatClient
{
    private readonly SemaphoreSlim _semaphore = new(maxCount: 5);
    private int _requestCount = 0;

    public RateLimitingMiddleware(IChatClient inner) : base(inner) { }

    public override async Task<ChatResponse> GetResponseAsync(
        IList<ChatMessage> chatMessages,
        ChatOptions? options = null,
        CancellationToken cancellationToken = default)
    {
        if (!await _semaphore.WaitAsync(TimeSpan.FromSeconds(10), cancellationToken))
            throw new InvalidOperationException("Rate limit exceeded. Try again later.");

        try
        {
            Interlocked.Increment(ref _requestCount);
            Console.ForegroundColor = ConsoleColor.DarkYellow;
            Console.WriteLine($"  🚦 Rate limiter: request #{_requestCount} (5 concurrent max)");
            Console.ResetColor();

            return await base.GetResponseAsync(
                chatMessages, options, cancellationToken);
        }
        finally
        {
            _semaphore.Release();
        }
    }
}
```

---

## 📖 Middleware Execution Order

```
Request Flow (outer → inner):

User Request
  ↓
┌──────────────────────────┐
│ 1. OpenTelemetry         │  ← Starts trace span
│    ↓                     │
│ 2. Distributed Cache     │  ← Cache hit? Return early!
│    ↓                     │
│ 3. Function Invocation   │  ← Auto-execute AI tool calls
│    ↓                     │
│ 4. Logging Middleware    │  ← Log request details
│    ↓                     │
│ 5. Rate Limiter          │  ← Enforce concurrency limits
│    ↓                     │
│ 6. OpenAI Provider       │  ← Actual API call
└──────────────────────────┘
  ↓
Response (inner → outer)
```

---

## 🔑 Key Takeaways

| Concept | Details |
|---------|---------|
| **`ChatClientBuilder`** | Fluent API for composing middleware |
| **`DelegatingChatClient`** | Base class for custom middleware |
| **Built-in middleware** | `.UseOpenTelemetry()`, `.UseDistributedCache()`, `.UseFunctionInvocation()` |
| **Order matters** | Outer middleware runs first, inner runs last |
| **Caching** | Identical prompts return cached responses |

---

## ➡️ Week Complete!

Continue to **[Week 3: Advanced AI Techniques](../../Week-03-Advanced-AI-Techniques/README.md)** 🎉
