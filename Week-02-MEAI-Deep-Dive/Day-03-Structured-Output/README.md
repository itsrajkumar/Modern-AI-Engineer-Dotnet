# Day 3: Structured Output

> **Type:** 💻 Code | **Time:** ~3 hours | **Project:** StructuredOutputDemo
>
> 🆕 *Based on [Lesson 2: Generative AI Techniques — Structured Output](https://github.com/microsoft/Generative-AI-for-beginners-dotnet/blob/main/02-GenerativeAITechniques/readme.md)*

---

## 🎯 Learning Objectives

- Get strongly-typed C# objects from AI responses
- Use `GetResponseAsync<T>()` for automatic JSON deserialization
- Define response schemas using C# records
- Handle validation and error cases with structured output
- Compare JSON mode vs structured output

---

## 📖 The Problem: Parsing AI Text

```
WITHOUT Structured Output:          WITH Structured Output:
────────────────────────────       ────────────────────────────
AI returns: "The sentiment is      AI returns: C# object directly!
positive and the score is 0.85     
and the language is English"       SentimentResult {
                                     Sentiment: "Positive",
How do you parse this? Regex?        Score: 0.85,
String.Split? What if format         Language: "English"
changes?                           }

Fragile! ❌                         Type-safe! ✅
```

---

## 💻 Code Sample

```csharp
using Microsoft.Extensions.AI;
using OpenAI;

var config = new ConfigurationBuilder().AddUserSecrets<Program>().Build();
IChatClient chatClient = new OpenAIClient(config["OpenAI:ApiKey"]!)
    .AsChatClient("gpt-4o-mini");

// =====================================================
// Example 1: Simple Structured Output
// =====================================================
Console.WriteLine("=== Example 1: Sentiment Analysis ===\n");

var sentimentResult = await chatClient.GetResponseAsync<SentimentResult>(
    "Analyze the sentiment of: 'I absolutely love programming in C# with .NET 10!'");

Console.WriteLine($"Sentiment: {sentimentResult.Result?.Sentiment}");
Console.WriteLine($"Score:     {sentimentResult.Result?.Score}");
Console.WriteLine($"Language:  {sentimentResult.Result?.Language}");

// =====================================================
// Example 2: Complex Nested Object
// =====================================================
Console.WriteLine("\n=== Example 2: Code Review ===\n");

var reviewResult = await chatClient.GetResponseAsync<CodeReview>(
    """
    Review this code:
    public void Process(string data) {
        var result = data.Split(',');
        Console.WriteLine(result[0]);
    }
    """);

var review = reviewResult.Result;
Console.WriteLine($"Quality: {review?.QualityScore}/10");
Console.WriteLine($"Issues:");
foreach (var issue in review?.Issues ?? [])
    Console.WriteLine($"  ⚠️ [{issue.Severity}] {issue.Description}");

// =====================================================
// Example 3: List/Array Output
// =====================================================
Console.WriteLine("\n=== Example 3: Topic Extraction ===\n");

var topicsResult = await chatClient.GetResponseAsync<TopicExtraction>(
    "Extract topics from: '.NET 10 brings improved AI integration with MEAI, " +
    "better performance through native AOT, and enhanced container support.'");

Console.WriteLine($"Topics: {string.Join(", ", topicsResult.Result?.Topics ?? [])}");
Console.WriteLine($"Category: {topicsResult.Result?.PrimaryCategory}");

// =====================================================
// Response Models (C# Records)
// =====================================================
record SentimentResult(
    string Sentiment,     // "Positive", "Negative", "Neutral"
    double Score,         // 0.0 to 1.0
    string Language);

record CodeReview(
    int QualityScore,          // 1-10
    CodeIssue[] Issues,
    string[] Suggestions,
    string Summary);

record CodeIssue(
    string Severity,           // "Critical", "Warning", "Info"
    string Description,
    int? LineNumber);

record TopicExtraction(
    string[] Topics,
    string PrimaryCategory,
    double ConfidenceScore);
```

---

## 📖 How It Works

```
Your Code                          Under the Hood
──────────────────                 ──────────────────────────
GetResponseAsync<T>(prompt)   ──►  1. Generates JSON schema from T
                                   2. Adds "response_format" to request
                                   3. Sends to AI model
                              ◄──  4. AI returns valid JSON
                                   5. Deserializes to T
Returns: ChatResponse<T>           6. Returns typed result
  .Result = your C# object!
```

---

## 🔑 Key Takeaways

| Concept | Details |
|---------|---------|
| **`GetResponseAsync<T>()`** | Returns typed C# objects from AI |
| **C# Records** | Best way to define response schemas |
| **Nested objects** | Full support for complex hierarchies |
| **Arrays** | AI can return lists and arrays |
| **Validation** | Type system catches format errors at compile time |

---

## ➡️ Next

Continue to **[Day 4: Function Calling with MEAI](../Day-04-Function-Calling-With-MEAI/README.md)**
