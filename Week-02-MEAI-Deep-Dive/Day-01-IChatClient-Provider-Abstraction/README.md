# Day 1: IChatClient & Provider Abstraction

> **Type:** 💻 Code | **Time:** ~3 hours | **Project:** ProviderSwapDemo
>
> 🆕 *Based on [Lesson 1: Introduction to Generative AI](https://github.com/microsoft/Generative-AI-for-beginners-dotnet/blob/main/01-IntroductionToGenerativeAI/readme.md) from Generative AI for Beginners .NET v2*

---

## 🎯 Learning Objectives

- Understand `IChatClient` as the universal AI interface in .NET 10
- Swap between OpenAI, Azure OpenAI, and Ollama with zero code changes
- Use Dependency Injection to register AI providers
- Understand `ChatMessage`, `ChatRole`, and `ChatResponse`
- Master `GetResponseAsync()` — the .NET 10 primary API

---

## 📖 The Provider Abstraction Pattern

`IChatClient` is the **single interface** you program against. The provider is just a constructor argument.

```
                    ┌───────────────────────────┐
                    │     Your Application      │
                    │                           │
                    │   IChatClient chatClient  │
                    │   └── GetResponseAsync()  │
                    │   └── GetStreamingAsync()  │
                    └────────────┬──────────────┘
                                 │
                    ┌────────────▼──────────────┐
                    │     IChatClient            │
                    │     (abstraction)          │
                    └────────────┬──────────────┘
                                 │
            ┌────────────────────┼────────────────────┐
            ▼                    ▼                    ▼
    ┌──────────────┐    ┌──────────────┐    ┌──────────────┐
    │   OpenAI     │    │ Azure OpenAI │    │    Ollama    │
    │ .AsChatClient│    │ .AsChatClient│    │ .AsChatClient│
    │ ("gpt-5-mini")│    │ ("gpt-5")   │    │ ("phi4-mini")│
    └──────────────┘    └──────────────┘    └──────────────┘
```

### .NET Analogy

```csharp
// You already know this pattern from .NET!
// ILogger → ConsoleLogger, FileLogger, ApplicationInsightsLogger
// IDbConnection → SqlConnection, NpgsqlConnection, SqliteConnection
// IChatClient → OpenAIChatClient, AzureChatClient, OllamaChatClient

// Same pattern, different domain:
ILogger logger = new ConsoleLogger();           // Logging
IDbConnection db = new SqlConnection(connStr);  // Database
IChatClient ai = new OpenAIClient(key)          // AI ✨
    .AsChatClient("gpt-5-mini");
```

---

## 💻 Code Project

### Project Setup

```powershell
cd "d:\Study\AI-Engineer\AI-Engineer-With-.Net\Week-02-MEAI-Deep-Dive\Day-01-IChatClient-Provider-Abstraction\src"

dotnet new console -n ProviderSwapDemo
cd ProviderSwapDemo

dotnet add package Microsoft.Extensions.AI
dotnet add package Microsoft.Extensions.AI.OpenAI
dotnet add package Microsoft.Extensions.AI.Ollama
dotnet add package Microsoft.Extensions.Configuration.UserSecrets
dotnet add package Microsoft.Extensions.DependencyInjection
dotnet add package Microsoft.Extensions.Hosting

dotnet user-secrets init
dotnet user-secrets set "OpenAI:ApiKey" "sk-your-key-here"
```

### Program.cs

```csharp
using Microsoft.Extensions.AI;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Hosting;
using OpenAI;

// =====================================================
// Day 1: IChatClient & Provider Abstraction
// =====================================================
// Demonstrates swapping AI providers with ZERO code changes.
// The business logic only depends on IChatClient.
// =====================================================

var builder = Host.CreateApplicationBuilder(args);

// =====================================================
// PROVIDER CONFIGURATION — Change ONLY this section!
// =====================================================
var provider = builder.Configuration["AI:Provider"] ?? "openai";

switch (provider.ToLower())
{
    case "openai":
        var apiKey = builder.Configuration["OpenAI:ApiKey"]
            ?? throw new InvalidOperationException("Set OpenAI:ApiKey");
        builder.Services.AddChatClient(
            new OpenAIClient(apiKey).AsChatClient("gpt-4o-mini"));
        break;

    case "azure":
        // Uses AzureCliCredential — no API keys needed!
        var credential = new Azure.Identity.AzureCliCredential();
        var endpoint = builder.Configuration["AzureOpenAI:Endpoint"]!;
        builder.Services.AddChatClient(
            new Azure.AI.OpenAI.AzureOpenAIClient(
                new Uri(endpoint), credential)
            .AsChatClient("gpt-5"));
        break;

    case "ollama":
        // Free, local, no API key!
        builder.Services.AddChatClient(
            new OllamaChatClient(new Uri("http://localhost:11434"), "phi4-mini"));
        break;
}

var app = builder.Build();
var chatClient = app.Services.GetRequiredService<IChatClient>();

// =====================================================
// THE BUSINESS LOGIC — Same regardless of provider!
// =====================================================
Console.WriteLine("╔══════════════════════════════════════════════╗");
Console.WriteLine($"║  Provider: {provider.ToUpper(),-34}║");
Console.WriteLine("║  IChatClient Provider Abstraction Demo       ║");
Console.WriteLine("╚══════════════════════════════════════════════╝\n");

// Example 1: Simple GetResponseAsync (new in .NET 10)
Console.WriteLine("--- Example 1: GetResponseAsync ---\n");

var response = await chatClient.GetResponseAsync(
    "Explain dependency injection in C# in exactly 2 sentences.");

Console.WriteLine($"📝 {response.Message.Text}\n");
Console.WriteLine($"📊 Model: {response.ModelId}");
Console.WriteLine($"📊 Tokens: {response.Usage?.InputTokenCount} in / "
    + $"{response.Usage?.OutputTokenCount} out");

// Example 2: ChatOptions for control
Console.WriteLine("\n--- Example 2: With ChatOptions ---\n");

var options = new ChatOptions
{
    Temperature = 0.0f,
    MaxOutputTokens = 100,
    StopSequences = ["---"]
};

var preciseResponse = await chatClient.GetResponseAsync(
    "List exactly 3 C# design patterns, numbered 1-3.",
    options);

Console.WriteLine(preciseResponse.Message.Text);

// Example 3: Multi-turn conversation
Console.WriteLine("\n--- Example 3: Multi-turn Conversation ---\n");

var messages = new List<ChatMessage>
{
    new(ChatRole.System, "You are a .NET 10 expert. Be concise."),
    new(ChatRole.User, "What is the most important new feature in .NET 10?")
};

var turn1 = await chatClient.GetResponseAsync(messages, options);
Console.WriteLine($"AI: {turn1.Message.Text}\n");

// Add assistant response + follow-up
messages.Add(turn1.Message);
messages.Add(new ChatMessage(ChatRole.User, "Show me a code example of that."));

var turn2 = await chatClient.GetResponseAsync(messages);
Console.WriteLine($"AI: {turn2.Message.Text}");

Console.WriteLine("\n✅ Provider abstraction demo complete!");
```

---

## 📖 IChatClient API Reference

| Method | Purpose | When to Use |
|--------|---------|-------------|
| `GetResponseAsync(string)` | Simple text → response | Single prompts, quick queries |
| `GetResponseAsync(IList<ChatMessage>)` | Multi-turn conversation | Chatbots, contextual conversations |
| `GetStreamingResponseAsync(...)` | Token-by-token streaming | Interactive UIs (Day 2) |

### The ChatMessage Object

```csharp
// ChatMessage is the universal message format
var message = new ChatMessage(ChatRole.User, "Hello!");

// Roles available:
ChatRole.System    // Sets behavior (like middleware)
ChatRole.User      // Human's input
ChatRole.Assistant // AI's previous responses
ChatRole.Tool      // Function call results (Day 4)
```

---

## 🔑 Key Takeaways

| Concept | Details |
|---------|---------|
| **`IChatClient`** | The ONE interface for all AI providers |
| **Provider Swap** | Change constructor call only, all logic unchanged |
| **`GetResponseAsync`** | The primary .NET 10 API (simpler than `CompleteAsync`) |
| **`ChatOptions`** | Temperature, max tokens, stop sequences |
| **DI Registration** | `builder.Services.AddChatClient(...)` |
| **`AzureCliCredential`** | No API keys in code for Azure |

---

## 🧪 Self-Assessment

1. What interface do you program against for AI in .NET 10?
2. How many lines of code change to swap from OpenAI → Ollama?
3. What is `ChatRole.System` used for?
4. What method replaces `CompleteAsync` in .NET 10?
5. What authentication approach does Azure recommend?

<details>
<summary>Answers</summary>

1. `IChatClient` — the universal abstraction from Microsoft.Extensions.AI
2. ONE line — only the constructor/registration changes
3. Sets the AI's persona, rules, and behavior (like middleware)
4. `GetResponseAsync()` — simpler, returns `ChatResponse` directly
5. `AzureCliCredential` — token-based, no API keys in code

</details>

---

## 📚 References

- [Microsoft.Extensions.AI Documentation](https://learn.microsoft.com/dotnet/ai/ai-extensions)
- [IChatClient API Reference](https://learn.microsoft.com/dotnet/api/microsoft.extensions.ai.ichatclient)
- [v2 Course: Lesson 1](https://github.com/microsoft/Generative-AI-for-beginners-dotnet/blob/main/01-IntroductionToGenerativeAI/readme.md)

---

## ➡️ Next

Continue to **[Day 2: Streaming & Async Patterns](../Day-02-Streaming-and-Async-Patterns/README.md)**
