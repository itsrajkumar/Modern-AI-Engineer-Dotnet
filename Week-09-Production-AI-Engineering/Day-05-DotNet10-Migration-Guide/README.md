# Day 5: .NET 10 Migration Guide

> **Type:** 📖 Theory + Code | **Time:** ~3 hours

---

## 🎯 Learning Objectives

- Migrate existing .NET 8/9 AI code to .NET 10 patterns
- Replace deprecated APIs with modern equivalents
- Update authentication from API keys to `AzureCliCredential`
- Adopt the new `GetResponseAsync` API (replacing `CompleteAsync`)
- Understand breaking changes and migration paths

---

## 📖 What Changed in .NET 10 for AI?

```
.NET 8/9 (Legacy)                  .NET 10 (Current)
──────────────────────             ──────────────────────
CompleteAsync()                    GetResponseAsync() ✅
ChatCompletion                     ChatResponse ✅
API key in code                    AzureCliCredential ✅
Manual JSON parsing                GetResponseAsync<T>() ✅
No middleware                      ChatClientBuilder pipeline ✅
Semantic Kernel only               MEAI + SK + MAF ✅
```

---

## 💻 Migration Examples

### 1. API Method Rename

```diff
// BEFORE (.NET 8/9)
- var result = await chatClient.CompleteAsync(messages);
- Console.WriteLine(result.Content);

// AFTER (.NET 10)
+ var result = await chatClient.GetResponseAsync(messages);
+ Console.WriteLine(result.Message.Text);
```

### 2. Authentication

```diff
// BEFORE: API key hardcoded or in config
- var client = new OpenAIClient("sk-hardcoded-key-BAD");

// AFTER: Token-based authentication (no keys in code!)
+ var credential = new AzureCliCredential();
+ var client = new AzureOpenAIClient(
+     new Uri(endpoint), credential);
```

### 3. Structured Output

```diff
// BEFORE: Parse JSON manually
- var response = await chatClient.GetResponseAsync("Return JSON...");
- var json = response.Message.Text;
- var result = JsonSerializer.Deserialize<MyType>(json); // fragile!

// AFTER: Type-safe structured output
+ var response = await chatClient.GetResponseAsync<MyType>("Analyze this...");
+ var result = response.Result; // strongly typed!
```

### 4. Function Calling

```diff
// BEFORE: SK-only function calling
- kernel.Plugins.AddFromType<MyPlugin>();
- var settings = new OpenAIPromptExecutionSettings
- {
-     FunctionChoiceBehavior = FunctionChoiceBehavior.Auto()
- };

// AFTER: MEAI function calling (no SK required)
+ var tools = new[] { AIFunctionFactory.Create(MyMethod, "name", "desc") };
+ IChatClient client = new ChatClientBuilder(innerClient)
+     .UseFunctionInvocation()
+     .Build();
+ var result = await client.GetResponseAsync(prompt,
+     new ChatOptions { Tools = tools });
```

### 5. Middleware Pipeline

```diff
// BEFORE: Manual logging, caching, etc.
- var response = await chatClient.GetResponseAsync(messages);
- _logger.LogInformation("AI call took {ms}ms", sw.ElapsedMilliseconds);
- await _cache.SetAsync(key, response);

// AFTER: Composable middleware
+ IChatClient client = new ChatClientBuilder(innerClient)
+     .UseOpenTelemetry()
+     .UseDistributedCache()
+     .UseFunctionInvocation()
+     .Build();
+ // Logging, caching, telemetry all handled automatically!
+ var response = await client.GetResponseAsync(messages);
```

### 6. Streaming

```diff
// BEFORE: Manual streaming with older APIs
- await foreach (var chunk in chatClient.CompleteStreamingAsync(messages))
- {
-     Console.Write(chunk.Content);
- }

// AFTER: Cleaner streaming API
+ await foreach (var update in chatClient.GetStreamingResponseAsync(messages))
+ {
+     Console.Write(update.Text);
+ }
```

---

## 📊 Migration Checklist

| # | Task | Priority |
|---|------|----------|
| 1 | Update `.csproj` to `net10.0` | 🔴 Critical |
| 2 | Update NuGet packages to latest | 🔴 Critical |
| 3 | Replace `CompleteAsync` → `GetResponseAsync` | 🔴 Critical |
| 4 | Replace `CompleteStreamingAsync` → `GetStreamingResponseAsync` | 🔴 Critical |
| 5 | Replace API keys with `AzureCliCredential` | 🟡 High |
| 6 | Add `ChatClientBuilder` middleware pipeline | 🟡 High |
| 7 | Replace manual JSON parsing with `GetResponseAsync<T>` | 🟢 Medium |
| 8 | Replace SK-only function calling with MEAI where appropriate | 🟢 Medium |
| 9 | Add OpenTelemetry instrumentation | 🟢 Medium |
| 10 | Update unit tests to mock `IChatClient` | 🟢 Medium |

---

## 📖 Package Version Reference

| Package | .NET 8/9 | .NET 10 |
|---------|----------|---------|
| `Microsoft.Extensions.AI` | Preview | **GA** (stable) |
| `Microsoft.Extensions.AI.OpenAI` | Preview | **GA** |
| `Microsoft.Extensions.AI.Ollama` | Preview | **GA** |
| `Microsoft.SemanticKernel` | 1.x | 1.x (stable) |
| `Azure.Identity` | 1.10+ | 1.13+ |
| `OpenAI` | 2.0+ | 2.1+ |

---

## 🔑 Key Takeaways

| Change | Why |
|--------|-----|
| `GetResponseAsync` | Simpler API, returns `ChatResponse` directly |
| `AzureCliCredential` | No API keys in code, better security |
| `ChatClientBuilder` | Composable middleware (like ASP.NET Core) |
| `GetResponseAsync<T>` | Type-safe structured output |
| `UseFunctionInvocation()` | Middleware-based tool calling |
| `UseOpenTelemetry()` | Built-in observability |

---

## 🏆 Week 9 Complete!

Continue to **[Week 10: Responsible AI & Capstone](../../Week-10-Responsible-AI-and-Capstone/README.md)** 🎉
