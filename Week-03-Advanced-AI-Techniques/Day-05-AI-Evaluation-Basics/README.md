# Day 5: AI Evaluation Basics

> **Type:** 💻 Code | **Time:** ~3 hours | **Project:** AIEval

---

## 🎯 Learning Objectives

- Understand why AI evaluation is different from traditional testing
- Implement automated metrics: relevance, groundedness, coherence
- Build an evaluation harness for comparing prompts and models
- Use LLM-as-a-Judge for qualitative evaluation
- Create regression test suites for AI outputs

---

## 📖 Why Evaluate AI?

```
Traditional Software:           AI Software:
──────────────────────          ────────────────────────
Bug = code logic error          Bug = model generates wrong answer
Fix = change code               Fix = change prompt? model? data?
Test = assert exact output      Test = assert quality criteria
Deploy = deterministic           Deploy = probabilistic
Regression = code change broke   Regression = model update broke
             something                       output quality
```

---

## 📊 AI Evaluation Metrics

| Metric | Measures | Scale | How |
|--------|----------|-------|-----|
| **Relevance** | Does it answer the question? | 1-5 | LLM judge |
| **Groundedness** | Is it based on provided data? | 1-5 | LLM judge |
| **Coherence** | Is the response well-structured? | 1-5 | LLM judge |
| **Fluency** | Is it grammatically correct? | 1-5 | LLM judge |
| **Similarity** | How close to reference answer? | 0-1 | Cosine similarity |
| **Toxicity** | Is it safe and appropriate? | 0-1 | Classifier |
| **Latency** | How fast is the response? | ms | Timer |
| **Cost** | How many tokens consumed? | $ | Token count |

---

## 💻 Code Sample: Evaluation Harness

```csharp
using Microsoft.Extensions.AI;

/// <summary>
/// Automated AI evaluation harness.
/// Compares prompts, models, or configurations.
/// </summary>
public class AIEvaluator
{
    private readonly IChatClient _chatClient;
    private readonly IChatClient _judgeClient;

    public AIEvaluator(IChatClient chatClient, IChatClient judgeClient)
    {
        _chatClient = chatClient;
        _judgeClient = judgeClient;
    }

    /// <summary>
    /// Evaluates a single prompt against quality criteria.
    /// </summary>
    public async Task<EvalResult> EvaluateAsync(EvalCase testCase)
    {
        var sw = System.Diagnostics.Stopwatch.StartNew();

        // Get the AI response
        var response = await _chatClient.GetResponseAsync(
            testCase.Prompt,
            new ChatOptions { Temperature = 0f });

        sw.Stop();

        var output = response.Message.Text ?? "";

        // Automated metrics
        var wordCount = output.Split(' ').Length;
        var inputTokens = response.Usage?.InputTokenCount ?? 0;
        var outputTokens = response.Usage?.OutputTokenCount ?? 0;

        // LLM-as-Judge evaluation
        var judgeScores = await JudgeResponseAsync(
            testCase.Prompt, output, testCase.ReferenceAnswer);

        return new EvalResult
        {
            TestCase = testCase,
            Output = output,
            LatencyMs = sw.Elapsed.TotalMilliseconds,
            WordCount = wordCount,
            InputTokens = inputTokens,
            OutputTokens = outputTokens,
            Relevance = judgeScores.Relevance,
            Coherence = judgeScores.Coherence,
            Groundedness = judgeScores.Groundedness
        };
    }

    /// <summary>
    /// Uses a separate LLM to judge the quality of a response.
    /// This is the "LLM-as-a-Judge" pattern.
    /// </summary>
    private async Task<JudgeScores> JudgeResponseAsync(
        string question, string answer, string? referenceAnswer)
    {
        var judgePrompt = $"""
            You are an AI output quality judge. Rate the following response.
            
            Question: {question}
            Response: {answer}
            {(referenceAnswer != null ? $"Reference Answer: {referenceAnswer}" : "")}
            
            Rate each on a scale of 1-5:
            - Relevance: Does the response answer the question?
            - Coherence: Is the response well-structured and logical?
            - Groundedness: Is the response factually accurate?
            
            Respond ONLY in this JSON format:
            {{"relevance": X, "coherence": X, "groundedness": X}}
            """;

        var result = await _judgeClient.GetResponseAsync<JudgeScores>(
            judgePrompt,
            new ChatOptions { Temperature = 0f });

        return result.Result ?? new JudgeScores(3, 3, 3);
    }

    /// <summary>
    /// Runs a full evaluation suite and prints a report.
    /// </summary>
    public async Task RunSuiteAsync(IEnumerable<EvalCase> testCases)
    {
        Console.WriteLine("╔══════════════════════════════════════════════╗");
        Console.WriteLine("║        AI EVALUATION REPORT                  ║");
        Console.WriteLine("╚══════════════════════════════════════════════╝\n");

        var results = new List<EvalResult>();

        foreach (var testCase in testCases)
        {
            Console.Write($"  Evaluating: {testCase.Name}... ");
            var result = await EvaluateAsync(testCase);
            results.Add(result);
            Console.WriteLine($"✅ R:{result.Relevance}/5 C:{result.Coherence}/5 G:{result.Groundedness}/5");
        }

        // Summary
        Console.WriteLine($"\n📊 Summary ({results.Count} test cases):");
        Console.WriteLine($"  Avg Relevance:    {results.Average(r => r.Relevance):F1}/5");
        Console.WriteLine($"  Avg Coherence:    {results.Average(r => r.Coherence):F1}/5");
        Console.WriteLine($"  Avg Groundedness: {results.Average(r => r.Groundedness):F1}/5");
        Console.WriteLine($"  Avg Latency:      {results.Average(r => r.LatencyMs):F0}ms");
        Console.WriteLine($"  Total Tokens:     {results.Sum(r => r.InputTokens + r.OutputTokens)}");
    }
}

// =====================================================
// Data Models
// =====================================================
public record EvalCase(
    string Name,
    string Prompt,
    string? ReferenceAnswer = null);

public record EvalResult
{
    public EvalCase TestCase { get; init; } = null!;
    public string Output { get; init; } = "";
    public double LatencyMs { get; init; }
    public int WordCount { get; init; }
    public int InputTokens { get; init; }
    public int OutputTokens { get; init; }
    public int Relevance { get; init; }
    public int Coherence { get; init; }
    public int Groundedness { get; init; }
}

public record JudgeScores(int Relevance, int Coherence, int Groundedness);
```

### Running the Evaluation

```csharp
var evaluator = new AIEvaluator(chatClient, judgeClient);

var testCases = new[]
{
    new EvalCase("DI Explanation",
        "What is Dependency Injection in .NET?",
        "DI is a design pattern where dependencies are provided from outside a class."),

    new EvalCase("Async Explanation",
        "Explain async/await in C# in 2 sentences.",
        "async/await allows non-blocking asynchronous operations."),

    new EvalCase("Code Generation",
        "Write a C# method that reverses a string.",
        "public string Reverse(string s) => new string(s.Reverse().ToArray());")
};

await evaluator.RunSuiteAsync(testCases);
```

---

## 🔑 Key Takeaways

- AI evaluation = **quality metrics** not exact matching
- **LLM-as-a-Judge** uses a separate model to rate outputs
- Track **relevance, coherence, groundedness** as core metrics
- Build **evaluation suites** that run on every prompt/model change
- Use **Temperature = 0** for reproducible evaluations

---

## 📚 References

- [Azure AI Evaluation SDK](https://learn.microsoft.com/azure/ai-studio/how-to/evaluate-generative-ai-app)
- [LLM-as-Judge pattern](https://arxiv.org/abs/2306.05685)

---

## ➡️ Week Complete!

Continue to **[Week 4: Microsoft Semantic Kernel](../../Week-04-Semantic-Kernel-Orchestrator/README.md)** 🎉
