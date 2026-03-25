# Day 2: Local Models & Providers

> **Type:** 💻 Code | **Time:** ~3 hours | **Project:** LocalModelsDemo
>
> 🆕 *Based on [Lesson 5: Local Models](https://github.com/microsoft/Generative-AI-for-beginners-dotnet/blob/main/05-LocalModels/readme.md) from Generative AI for Beginners .NET v2*

---

## 🎯 Learning Objectives

- Install and run Ollama for local AI model inference
- Use Docker Model Runner for containerized model serving
- Connect local models to .NET via `IChatClient`
- Build a hybrid fallback strategy (local + cloud)
- Understand when local models are the right choice

---

## 📖 Why Local Models?

```
Cloud Models:                     Local Models:
──────────────                    ──────────────────
✅ Most capable (GPT-5)           ✅ FREE — no API costs
✅ Always latest version           ✅ 100% private — data never leaves
❌ API costs ($$$)                 ✅ No internet required
❌ Data leaves your machine        ✅ Low latency (no network)
❌ Rate limits                     ❌ Less capable than cloud models
❌ Internet required               ❌ Requires local GPU/RAM
```

### When to Use Local Models

| Scenario | Recommendation |
|----------|---------------|
| Development / prototyping | ✅ Local (save money!) |
| Sensitive data (PII, medical) | ✅ Local (data stays local) |
| Offline environments | ✅ Local (no internet needed) |
| Maximum quality needed | ☁️ Cloud (GPT-5, Claude) |
| Production at scale | ☁️ Cloud (managed infra) |
| Cost-sensitive production | 🔄 Hybrid (local + cloud fallback) |

---

## 💻 Setting Up Ollama

```powershell
# Install Ollama (Windows)
winget install Ollama.Ollama

# Start the server
ollama serve

# Pull popular models
ollama pull phi4-mini          # 3.8B params, fast, Microsoft
ollama pull llama3.2:3b        # 3B params, Meta
ollama pull phi4-mini-vision   # Vision-capable
ollama pull nomic-embed-text   # Embedding model (for Week 5)

# Test it
ollama run phi4-mini "Hello from .NET!"
```

---

## 💻 Code Sample: Local + Cloud Hybrid

```csharp
using Microsoft.Extensions.AI;
using Microsoft.Extensions.Configuration;
using OpenAI;

// =====================================================
// Day 2: Local Models & Hybrid Fallback
// =====================================================

var config = new ConfigurationBuilder().AddUserSecrets<Program>().Build();

// =====================================================
// Provider 1: Ollama (local, free)
// =====================================================
IChatClient localClient = new OllamaChatClient(
    new Uri("http://localhost:11434"), "phi4-mini");

// =====================================================
// Provider 2: OpenAI (cloud, paid)
// =====================================================
IChatClient cloudClient = new OpenAIClient(config["OpenAI:ApiKey"]!)
    .AsChatClient("gpt-4o-mini");

// =====================================================
// Hybrid Strategy: Try local first, fall back to cloud
// =====================================================
Console.WriteLine("╔══════════════════════════════════════════════╗");
Console.WriteLine("║  Local Models & Hybrid Fallback Demo         ║");
Console.WriteLine("╚══════════════════════════════════════════════╝\n");

async Task<string> HybridQueryAsync(string prompt)
{
    try
    {
        Console.Write("  🏠 Trying local model... ");
        var sw = System.Diagnostics.Stopwatch.StartNew();
        var response = await localClient.GetResponseAsync(prompt,
            new ChatOptions { MaxOutputTokens = 200 });
        sw.Stop();
        Console.WriteLine($"✅ ({sw.ElapsedMilliseconds}ms)");
        return response.Message.Text ?? "";
    }
    catch (Exception ex)
    {
        Console.WriteLine($"❌ ({ex.Message})");
        Console.Write("  ☁️ Falling back to cloud... ");
        var sw = System.Diagnostics.Stopwatch.StartNew();
        var response = await cloudClient.GetResponseAsync(prompt,
            new ChatOptions { MaxOutputTokens = 200 });
        sw.Stop();
        Console.WriteLine($"✅ ({sw.ElapsedMilliseconds}ms)");
        return response.Message.Text ?? "";
    }
}

// Test the hybrid strategy
var result = await HybridQueryAsync(
    "What is dependency injection in .NET? Answer in 2 sentences.");
Console.WriteLine($"\n  📝 {result}\n");

// =====================================================
// Comparison: Local vs Cloud
// =====================================================
Console.WriteLine("--- Speed Comparison ---\n");

var prompt = "What is the capital of France? One word answer.";

var localSw = System.Diagnostics.Stopwatch.StartNew();
try
{
    var localResult = await localClient.GetResponseAsync(prompt,
        new ChatOptions { Temperature = 0f });
    localSw.Stop();
    Console.WriteLine($"  🏠 Local:  {localResult.Message.Text?.Trim()} ({localSw.ElapsedMilliseconds}ms)");
}
catch
{
    localSw.Stop();
    Console.WriteLine($"  🏠 Local:  unavailable");
}

var cloudSw = System.Diagnostics.Stopwatch.StartNew();
var cloudResult = await cloudClient.GetResponseAsync(prompt,
    new ChatOptions { Temperature = 0f });
cloudSw.Stop();
Console.WriteLine($"  ☁️ Cloud:  {cloudResult.Message.Text?.Trim()} ({cloudSw.ElapsedMilliseconds}ms)");

Console.WriteLine("\n✅ Local models demo complete!");
```

---

## 📊 Popular Local Models for .NET

| Model | Params | Size | Best For | Speed |
|-------|--------|------|----------|-------|
| **phi4-mini** | 3.8B | ~2GB | General chat, coding | ⚡ Fast |
| **llama3.2:3b** | 3B | ~2GB | General purpose | ⚡ Fast |
| **mistral:7b** | 7B | ~4GB | High quality text | 🔷 Medium |
| **codellama:7b** | 7B | ~4GB | Code generation | 🔷 Medium |
| **phi4-mini-vision** | 3.8B | ~2GB | Image analysis | ⚡ Fast |
| **nomic-embed-text** | 137M | ~274MB | Embeddings | ⚡⚡ Very fast |

---

## 🔑 Key Takeaways

| Concept | Details |
|---------|---------|
| **Ollama** | Run models locally, free, same `IChatClient` |
| **Hybrid fallback** | Try local first, cloud if unavailable |
| **Same interface** | `OllamaChatClient` implements `IChatClient` |
| **No code changes** | Swap providers via config, not code |
| **Cost savings** | Dev/test locally, deploy to cloud for production |

---

## ➡️ Next

Continue to **[Day 3: Fine-Tuning Concepts](../Day-03-Fine-Tuning-Concepts/README.md)**
