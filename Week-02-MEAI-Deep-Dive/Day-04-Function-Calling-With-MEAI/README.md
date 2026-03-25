# Day 4: Function Calling with MEAI

> **Type:** 💻 Code | **Time:** ~3 hours | **Project:** MEAIToolCalling

---

## 🎯 Learning Objectives

- Implement function/tool calling using pure `Microsoft.Extensions.AI` (no Semantic Kernel)
- Use `AIFunctionFactory.Create()` to expose C# methods to AI
- Use `UseFunctionInvocation()` middleware for automatic tool execution
- Understand the difference between MEAI and SK function calling

---

## 📖 MEAI Function Calling vs Semantic Kernel

```
MEAI Function Calling:                 SK Function Calling:
────────────────────────               ─────────────────────
AIFunctionFactory.Create()             [KernelFunction] attribute
ChatOptions.Tools                      kernel.Plugins.AddFromType<>()
UseFunctionInvocation()                FunctionChoiceBehavior.Auto()
Pure IChatClient                       Requires Kernel object
Lightweight, no framework              Full orchestration framework
```

**Use MEAI** when you want lightweight tool calling.
**Use SK** when you need plugins, planners, and orchestration.

---

## 💻 Code Sample

```csharp
using Microsoft.Extensions.AI;
using Microsoft.Extensions.Configuration;
using OpenAI;

// =====================================================
// Day 4: Function Calling with MEAI
// =====================================================
// Expose C# methods to the AI WITHOUT Semantic Kernel.
// The AI decides when to call your code!
// =====================================================

var config = new ConfigurationBuilder().AddUserSecrets<Program>().Build();
var apiKey = config["OpenAI:ApiKey"]!;

// =====================================================
// Step 1: Define tools using AIFunctionFactory
// =====================================================
var tools = new List<AITool>
{
    AIFunctionFactory.Create(
        (string city) =>
        {
            return city.ToLower() switch
            {
                "london" => "15°C, Cloudy",
                "tokyo" => "22°C, Sunny",
                "new york" => "18°C, Partly cloudy",
                "mumbai" => "32°C, Humid",
                _ => "Weather data not available"
            };
        },
        name: "GetWeather",
        description: "Gets the current weather for a city"),

    AIFunctionFactory.Create(
        (double amount, string from, string to) =>
        {
            var rates = new Dictionary<string, double>
            {
                ["usd_eur"] = 0.92, ["usd_gbp"] = 0.79,
                ["usd_jpy"] = 149.5, ["usd_inr"] = 83.2,
                ["eur_usd"] = 1.09, ["gbp_usd"] = 1.27
            };
            var key = $"{from.ToLower()}_{to.ToLower()}";
            return rates.TryGetValue(key, out var rate)
                ? $"{amount} {from.ToUpper()} = {amount * rate:F2} {to.ToUpper()}"
                : $"Conversion rate not available for {from} → {to}";
        },
        name: "ConvertCurrency",
        description: "Converts an amount from one currency to another"),

    AIFunctionFactory.Create(
        () => DateTime.Now.ToString("dddd, MMMM dd, yyyy HH:mm:ss"),
        name: "GetCurrentDateTime",
        description: "Gets the current date and time")
};

// =====================================================
// Step 2: Configure IChatClient with function calling
// =====================================================
IChatClient chatClient = new ChatClientBuilder(
    new OpenAIClient(apiKey).AsChatClient("gpt-4o-mini"))
    .UseFunctionInvocation()  // Automatically executes tools!
    .Build();

// =====================================================
// Step 3: Let the AI use your tools
// =====================================================
Console.WriteLine("╔══════════════════════════════════════════════╗");
Console.WriteLine("║  MEAI Function Calling Demo                  ║");
Console.WriteLine("║  AI can call: GetWeather, ConvertCurrency,   ║");
Console.WriteLine("║  GetCurrentDateTime                          ║");
Console.WriteLine("╚══════════════════════════════════════════════╝\n");

var options = new ChatOptions { Tools = tools };

// Test 1: Simple tool call
Console.WriteLine("--- Test 1: Weather lookup ---");
var response = await chatClient.GetResponseAsync(
    "What's the weather in Tokyo right now?", options);
Console.WriteLine($"AI: {response.Message.Text}\n");

// Test 2: Multi-step tool calling
Console.WriteLine("--- Test 2: Multi-step (weather + currency) ---");
var response2 = await chatClient.GetResponseAsync(
    "I'm traveling from New York to London. What's the weather in both cities, " +
    "and how much is 500 USD in GBP? Also, what's today's date?", options);
Console.WriteLine($"AI: {response2.Message.Text}\n");

// Test 3: Interactive chat with tools
Console.WriteLine("--- Test 3: Interactive Chat with Tools ---");
Console.WriteLine("Ask about weather, currency, or time (type 'exit' to quit)\n");

while (true)
{
    Console.ForegroundColor = ConsoleColor.Cyan;
    Console.Write("You: ");
    Console.ResetColor();

    var input = Console.ReadLine();
    if (string.IsNullOrWhiteSpace(input) || input == "exit") break;

    var result = await chatClient.GetResponseAsync(input, options);
    Console.ForegroundColor = ConsoleColor.Green;
    Console.WriteLine($"AI: {result.Message.Text}\n");
    Console.ResetColor();
}

Console.WriteLine("\n✅ MEAI Function Calling demo complete!");
```

---

## 📖 How It Works Internally

```
User: "What's the weather in Tokyo and convert 100 USD to JPY?"
         │
         ▼
    ┌─────────────────────────────────────────────┐
    │  LLM sees available tools:                  │
    │  • GetWeather(city)                         │
    │  • ConvertCurrency(amount, from, to)        │
    │  • GetCurrentDateTime()                     │
    │                                             │
    │  LLM decides to call TWO tools:             │
    │  1. GetWeather("Tokyo") → "22°C, Sunny"     │
    │  2. ConvertCurrency(100, "usd", "jpy")      │
    │     → "100 USD = 14,950.00 JPY"             │
    │                                             │
    │  UseFunctionInvocation() middleware          │
    │  automatically executes both and sends      │
    │  results back to the LLM                    │
    │                                             │
    │  LLM composes final answer:                 │
    │  "Tokyo is 22°C and sunny. 100 USD =        │
    │   14,950 JPY."                              │
    └─────────────────────────────────────────────┘
```

---

## 🔑 MEAI vs SK Function Calling Comparison

| Feature | MEAI | Semantic Kernel |
|---------|------|-----------------|
| **Define tools** | `AIFunctionFactory.Create()` | `[KernelFunction]` attribute |
| **Register tools** | `ChatOptions.Tools` | `kernel.Plugins.AddFromType<>()` |
| **Auto-execute** | `UseFunctionInvocation()` middleware | `FunctionChoiceBehavior.Auto()` |
| **Dependencies** | Just `Microsoft.Extensions.AI` | Full `SemanticKernel` package |
| **Best for** | Simple tool calling | Complex orchestration |
| **Learning curve** | Low | Higher |

---

## 📚 References

- [AIFunctionFactory Documentation](https://learn.microsoft.com/dotnet/api/microsoft.extensions.ai.aifunctionfactory)
- [Function Calling in MEAI](https://learn.microsoft.com/dotnet/ai/ai-extensions)
- [v2 Course: Tool Calling](https://github.com/microsoft/Generative-AI-for-beginners-dotnet/blob/main/02-GenerativeAITechniques/readme.md)

---

## ➡️ Next

Continue to **[Day 5: Middleware Pipelines](../Day-05-Middleware-Pipelines/README.md)**
