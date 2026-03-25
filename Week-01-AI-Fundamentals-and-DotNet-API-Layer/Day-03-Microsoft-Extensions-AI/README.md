# Day 3: Introduction to Microsoft.Extensions.AI

> **Type:** 💻 Code | **Time:** ~3 hours | **Project:** ExtensionsAIDemo
>
> 🆕 *Updated for .NET 10 with content from [Lesson 1](https://github.com/microsoft/Generative-AI-for-beginners-dotnet/blob/main/01-IntroductionToGenerativeAI/readme.md) and [Lesson 2](https://github.com/microsoft/Generative-AI-for-beginners-dotnet/blob/main/02-GenerativeAITechniques/readme.md) of Generative AI for Beginners .NET v2*

---

## 🎯 Learning Objectives

- Understand Microsoft's unified AI abstraction for .NET
- Learn the `IChatClient` and `IEmbeddingGenerator` interfaces
- Set up a project using `Microsoft.Extensions.AI`
- Understand how the abstraction decouples your app from specific AI providers

---

## 📖 What is Microsoft.Extensions.AI?

`Microsoft.Extensions.AI` is Microsoft's **unified abstraction layer** for AI services in .NET. Think of it like `ILogger` for logging — it doesn't care whether you use Serilog, NLog, or Console logging underneath.

### The Problem It Solves

```
WITHOUT Microsoft.Extensions.AI:
┌─────────────────────────────────┐
│ Your .NET App                    │
│ ┌─────────────────────────────┐ │
│ │ OpenAI-specific code         │ │──► Locked to OpenAI
│ │ using Azure.AI.OpenAI;       │ │
│ │ var client = new OpenAI...() │ │
│ └─────────────────────────────┘ │
└─────────────────────────────────┘

WITH Microsoft.Extensions.AI:
┌─────────────────────────────────┐
│ Your .NET App                    │
│ ┌─────────────────────────────┐ │
│ │ IChatClient chatClient       │ │──► Provider agnostic!
│ │ (Injected via DI)            │ │
│ └──────────┬──────────────────┘ │
│            │ DI resolves to...   │
│  ┌─────────▼─────────────┐      │
│  │ OpenAI / Azure / Ollama│      │──► Swap providers with config
│  └───────────────────────┘      │
└─────────────────────────────────┘
```

### .NET Analogy
```csharp
// Just like ILogger<T> abstracts logging providers...
public class MyService(ILogger<MyService> logger) { }

// IChatClient abstracts AI providers!
public class MyAIService(IChatClient chatClient) { }
```

---

## 🏗️ Core Interfaces

### `IChatClient`
The main interface for sending chat messages and getting responses.

```csharp
public interface IChatClient : IDisposable
{
    Task<ChatCompletion> CompleteAsync(
        IList<ChatMessage> chatMessages,
        ChatOptions? options = null,
        CancellationToken cancellationToken = default);

    IAsyncEnumerable<StreamingChatCompletionUpdate> CompleteStreamingAsync(
        IList<ChatMessage> chatMessages,
        ChatOptions? options = null,
        CancellationToken cancellationToken = default);

    ChatClientMetadata Metadata { get; }
}
```

### `IEmbeddingGenerator<TInput, TEmbedding>`
For generating embeddings from text (used heavily in Weeks 3-4).

```csharp
public interface IEmbeddingGenerator<TInput, TEmbedding> : IDisposable
    where TEmbedding : Embedding
{
    Task<GeneratedEmbeddings<TEmbedding>> GenerateAsync(
        IEnumerable<TInput> values,
        EmbeddingGenerationOptions? options = null,
        CancellationToken cancellationToken = default);
}
```

---

## 💻 Code Project

### Project Setup

```powershell
# Navigate to the day's folder
cd "d:\Study\AI-Engineer\AI-Engineer-With-.Net\Week-01-AI-Fundamentals-and-DotNet-API-Layer\Day-03-Microsoft-Extensions-AI\src"

# Create the project
dotnet new console -n ExtensionsAIDemo
cd ExtensionsAIDemo

# Add NuGet packages
dotnet add package Microsoft.Extensions.AI --prerelease
dotnet add package Microsoft.Extensions.AI.OpenAI --prerelease
dotnet add package Microsoft.Extensions.Configuration.UserSecrets

# Initialize user secrets
dotnet user-secrets init
dotnet user-secrets set "OpenAI:ApiKey" "sk-your-key-here"
dotnet user-secrets set "OpenAI:ModelId" "gpt-4o-mini"
```

### Program.cs

```csharp
using Microsoft.Extensions.AI;
using Microsoft.Extensions.Configuration;
using OpenAI;

// =====================================================
// Day 3: Microsoft.Extensions.AI Introduction
// =====================================================
// This demonstrates the unified AI abstraction layer.
// The IChatClient interface works with ANY provider.
// =====================================================

// Load configuration from user secrets
var config = new ConfigurationBuilder()
    .AddUserSecrets<Program>()
    .Build();

var apiKey = config["OpenAI:ApiKey"] 
    ?? throw new InvalidOperationException("OpenAI:ApiKey not configured. Run: dotnet user-secrets set \"OpenAI:ApiKey\" \"sk-your-key\"");
var modelId = config["OpenAI:ModelId"] ?? "gpt-4o-mini";

// Create the AI client using Microsoft.Extensions.AI abstraction
// Note: We program against IChatClient, NOT OpenAIClient directly
IChatClient chatClient = new OpenAIClient(apiKey)
    .AsChatClient(modelId);

// =====================================================
// Example 1: Simple completion
// =====================================================
Console.WriteLine("=== Example 1: Simple Completion ===\n");

var response = await chatClient.CompleteAsync("What is .NET in one sentence?");
Console.WriteLine($"Response: {response.Message.Text}");
Console.WriteLine($"Model: {response.ModelId}");
Console.WriteLine($"Tokens Used - Input: {response.Usage?.InputTokenCount}, Output: {response.Usage?.OutputTokenCount}");

// =====================================================
// Example 2: Using ChatMessage list (structured)
// =====================================================
Console.WriteLine("\n=== Example 2: Structured Messages ===\n");

var messages = new List<ChatMessage>
{
    new(ChatRole.System, "You are a helpful .NET expert. Answer concisely."),
    new(ChatRole.User, "What is Dependency Injection?")
};

var structuredResponse = await chatClient.CompleteAsync(messages);
Console.WriteLine($"Response: {structuredResponse.Message.Text}");

// =====================================================
// Example 3: Streaming (real-time token output)
// =====================================================
Console.WriteLine("\n=== Example 3: Streaming Response ===\n");

Console.Write("Streaming: ");
await foreach (var update in chatClient.CompleteStreamingAsync("Explain SOLID principles in C# briefly."))
{
    Console.Write(update.Text);
}
Console.WriteLine();

// =====================================================
// Example 4: Using ChatOptions for configuration
// =====================================================
Console.WriteLine("\n=== Example 4: With Options ===\n");

var options = new ChatOptions
{
    Temperature = 0.0f,        // Deterministic
    MaxOutputTokens = 100,     // Limit response length
    TopP = 0.9f
};

var deterministicResponse = await chatClient.CompleteAsync(
    "Give me a one-line C# code example using LINQ.",
    options);
Console.WriteLine($"Deterministic: {deterministicResponse.Message.Text}");

Console.WriteLine("\n✅ Day 3 Complete! You've used Microsoft.Extensions.AI successfully.");
```

---

## 🔑 Key Takeaways

1. **`IChatClient`** is the core abstraction — always program against this interface
2. **Provider swap = one line change** — switch from OpenAI to Azure by changing the constructor
3. **Streaming** is built-in via `IAsyncEnumerable<T>` — a C# native pattern
4. **`ChatOptions`** control temperature, max tokens, and other parameters
5. **User Secrets** keep API keys out of source control
6. 🆕 **`.NET 10:`** Use `GetResponseAsync()` as the simpler API (replaces `CompleteAsync` in some scenarios)
7. 🆕 **`.NET 10:`** Use `ChatClientBuilder` middleware for caching, telemetry, and retries

### 🆕 .NET 10 Middleware Preview

In .NET 10, `Microsoft.Extensions.AI` supports composable middleware pipelines (covered in detail in [Week 7, Day 3](../../Week-07-Responsible-AI-and-Production/Day-03-Structured-Output-and-Middleware/README.md)):

```csharp
// .NET 10: Compose AI behaviors like ASP.NET Core middleware
builder.Services.AddChatClient(pipeline => pipeline
    .UseOpenTelemetry()           // Automatic distributed traces
    .UseDistributedCache()        // Cache identical prompts
    .UseFunctionInvocation()      // Auto function/tool calling
    .Use(new OpenAIClient(apiKey)
        .AsChatClient("gpt-5-mini")));
```

### 🆕 .NET 10: Structured Output

Get strongly-typed C# objects directly from AI (covered in [Week 7, Day 3](../../Week-07-Responsible-AI-and-Production/Day-03-Structured-Output-and-Middleware/README.md)):

```csharp
// Instead of parsing free-form text...
var result = await chatClient.GetResponseAsync<ProductInfo>(
    "Extract product details from this description...");

Console.WriteLine(result.Result?.Name);  // Strongly typed!
```

---

## 🔄 Switching Providers (Why This Matters)

```csharp
// OpenAI
IChatClient chatClient = new OpenAIClient(apiKey)
    .AsChatClient("gpt-4o-mini");

// Azure OpenAI (just change the constructor!)
IChatClient chatClient = new AzureOpenAIClient(
        new Uri(endpoint), new AzureKeyCredential(key))
    .AsChatClient("gpt-4o-deployment");

// Ollama (local, free!)
IChatClient chatClient = new OllamaChatClient(
    new Uri("http://localhost:11434"), "llama3.1");
```

**Your application code using `IChatClient` doesn't change at all!**

---

## 📚 References

- [Microsoft.Extensions.AI Overview](https://learn.microsoft.com/dotnet/ai/ai-extensions)
- [Microsoft.Extensions.AI GitHub](https://github.com/dotnet/extensions/tree/main/src/Libraries/Microsoft.Extensions.AI)
- [Unified AI Building blocks for .NET](https://devblogs.microsoft.com/dotnet/unified-ai-building-blocks-for-dotnet/)

---

## ➡️ Next

Continue to **[Day 4: Your First API Connection](../Day-04-First-API-Connection/README.md)**
