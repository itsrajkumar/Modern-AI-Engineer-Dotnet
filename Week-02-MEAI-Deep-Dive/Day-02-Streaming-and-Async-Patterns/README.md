# Day 2: Streaming & Async Patterns

> **Type:** 💻 Code | **Time:** ~3 hours | **Project:** StreamingDemo

---

## 🎯 Learning Objectives

- Implement real-time token-by-token streaming with `GetStreamingResponseAsync`
- Understand `IAsyncEnumerable<T>` — the C# pattern that powers AI streaming
- Build a responsive console chat with live typing effect
- Handle streaming errors and cancellation gracefully
- Compare streaming vs non-streaming performance

---

## 📖 Why Streaming Matters

```
Without Streaming:                    With Streaming:
                                      
User asks question                    User asks question
  ↓                                     ↓
  │  ⏳ 3-5 seconds of nothing         │  Tokens arrive immediately!
  │  (waiting for FULL response)        │  "The"
  │                                     │  "The answer"
  │                                     │  "The answer is"
  ↓                                     │  "The answer is that"
Full response appears at once           │  ...
                                        ↓
                                      Complete (feels instant!)
```

**Streaming creates a "typing" effect** — essential for any interactive AI application.

---

## 💻 Code Sample

### Program.cs

```csharp
using Microsoft.Extensions.AI;
using Microsoft.Extensions.Configuration;
using OpenAI;

// =====================================================
// Day 2: Streaming & Async Patterns
// =====================================================

var config = new ConfigurationBuilder().AddUserSecrets<Program>().Build();
var apiKey = config["OpenAI:ApiKey"]!;

IChatClient chatClient = new OpenAIClient(apiKey)
    .AsChatClient("gpt-4o-mini");

// =====================================================
// Example 1: Basic Streaming
// =====================================================
Console.WriteLine("╔══════════════════════════════════════════════╗");
Console.WriteLine("║  Day 2: Streaming & Async Patterns           ║");
Console.WriteLine("╚══════════════════════════════════════════════╝\n");

Console.WriteLine("--- Example 1: Token-by-Token Streaming ---\n");
Console.Write("AI: ");

await foreach (var update in chatClient.GetStreamingResponseAsync(
    "Explain async/await in C# in 3 short sentences."))
{
    Console.Write(update.Text);
    // Each 'update' contains 1-3 tokens (words/word parts)
}
Console.WriteLine("\n");

// =====================================================
// Example 2: Streaming with Metadata
// =====================================================
Console.WriteLine("--- Example 2: Streaming with Usage Tracking ---\n");

var messages = new List<ChatMessage>
{
    new(ChatRole.System, "You are a concise .NET expert."),
    new(ChatRole.User, "What is the difference between Task and ValueTask?")
};

int chunkCount = 0;
Console.Write("AI: ");

await foreach (var update in chatClient.GetStreamingResponseAsync(messages))
{
    Console.Write(update.Text);
    chunkCount++;
}

Console.WriteLine($"\n\n📊 Received {chunkCount} streaming chunks");

// =====================================================
// Example 3: Cancellation Token Support
// =====================================================
Console.WriteLine("\n--- Example 3: Cancellation ---\n");

using var cts = new CancellationTokenSource(TimeSpan.FromSeconds(3));

try
{
    Console.Write("AI (will cancel after 3s): ");
    await foreach (var update in chatClient.GetStreamingResponseAsync(
        "Write a very long essay about the history of .NET...",
        cancellationToken: cts.Token))
    {
        Console.Write(update.Text);
    }
}
catch (OperationCanceledException)
{
    Console.ForegroundColor = ConsoleColor.Yellow;
    Console.WriteLine("\n⚠️ Stream cancelled after 3 seconds (as planned)");
    Console.ResetColor();
}

// =====================================================
// Example 4: Interactive Streaming Chat
// =====================================================
Console.WriteLine("\n--- Example 4: Interactive Streaming Chat ---");
Console.WriteLine("Type messages (type 'exit' to quit)\n");

var chatHistory = new List<ChatMessage>
{
    new(ChatRole.System, """
        You are a helpful .NET assistant. Keep responses under 100 words.
        Use code examples when relevant.
        """)
};

while (true)
{
    Console.ForegroundColor = ConsoleColor.Cyan;
    Console.Write("You: ");
    Console.ResetColor();

    var input = Console.ReadLine();
    if (string.IsNullOrWhiteSpace(input) || input.Equals("exit", StringComparison.OrdinalIgnoreCase))
        break;

    chatHistory.Add(new ChatMessage(ChatRole.User, input));

    Console.ForegroundColor = ConsoleColor.Green;
    Console.Write("AI: ");
    Console.ResetColor();

    var fullResponse = new System.Text.StringBuilder();

    await foreach (var update in chatClient.GetStreamingResponseAsync(chatHistory))
    {
        Console.Write(update.Text);
        fullResponse.Append(update.Text);
    }

    // Add the complete response to history
    chatHistory.Add(new ChatMessage(ChatRole.Assistant, fullResponse.ToString()));
    Console.WriteLine("\n");
}

Console.WriteLine("\n✅ Streaming demo complete!");
```

---

## 📖 Understanding IAsyncEnumerable

`IAsyncEnumerable<T>` is a C# language feature (since C# 8.0) that allows **asynchronous iteration**:

```csharp
// Synchronous iteration (familiar):
foreach (var item in collection) { ... }

// Asynchronous iteration (for streaming):
await foreach (var item in asyncStream) { ... }

// Under the hood, GetStreamingResponseAsync returns:
// IAsyncEnumerable<ChatResponseUpdate>
```

### How It Maps to AI Streaming

```
Server sends:  "The" → " answer" → " is" → " 42" → [DONE]
                 ↓         ↓          ↓        ↓       ↓
C# receives: update1  update2    update3  update4   (loop ends)
```

---

## 🔑 Streaming vs Non-Streaming

| Aspect | `GetResponseAsync` | `GetStreamingResponseAsync` |
|--------|--------|-----|
| **Response** | Full text at once | Token-by-token |
| **Latency** | High (wait for all) | Low (first token fast) |
| **UX** | Good for background | Essential for interactive |
| **Memory** | All in memory | Constant memory |
| **Cancellation** | Wait or cancel | Cancel mid-stream |
| **Use Case** | Batch processing, APIs | Chat UIs, real-time |

---

## 📚 References

- [IAsyncEnumerable in C#](https://learn.microsoft.com/en-us/dotnet/csharp/asynchronous-programming/generate-consume-asynchronous-stream)
- [Streaming in Microsoft.Extensions.AI](https://learn.microsoft.com/dotnet/ai/ai-extensions)

---

## ➡️ Next

Continue to **[Day 3: Structured Output](../Day-03-Structured-Output/README.md)**
