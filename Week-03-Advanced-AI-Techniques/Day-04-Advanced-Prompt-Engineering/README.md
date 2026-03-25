# Day 4: Advanced Prompt Engineering

> **Type:** 📖 Theory + Code | **Time:** ~3 hours | **Project:** AdvancedPrompts

---

## 🎯 Learning Objectives

- Master advanced prompting patterns beyond Zero-Shot and Few-Shot
- Implement ReAct (Reasoning + Acting) prompting
- Use Tree-of-Thought for complex problem solving
- Apply Self-Consistency for reliability
- Build prompt chains for multi-step workflows

---

## 📖 Advanced Prompting Patterns

### Pattern Evolution

```
Basic (Week 1):                  Advanced (This lesson):
────────────────────            ───────────────────────────
Zero-Shot                       ReAct (Reason + Act)
Few-Shot                        Tree-of-Thought
Chain-of-Thought                Self-Consistency
Role Prompting                  Prompt Chaining
Output Formatting               Meta-Prompting
```

---

## 🧠 Pattern 1: ReAct (Reasoning + Acting)

**Interleave thinking and tool-use** in a structured loop.

```
Thought: I need to find the current weather in Tokyo
Action: GetWeather("Tokyo")
Observation: 22°C, Sunny
Thought: Now I need to convert 100 USD to JPY for the trip
Action: ConvertCurrency(100, "USD", "JPY")
Observation: 14,950 JPY
Thought: I now have all the information to answer the user
Answer: Tokyo is 22°C and sunny. 100 USD = 14,950 JPY.
```

### C# Implementation

```csharp
var systemPrompt = """
    You are an assistant that uses the ReAct pattern.
    For each step, output:
    
    Thought: [your reasoning about what to do next]
    Action: [tool name and parameters]
    Observation: [result of the action]
    
    Repeat until you have enough information, then output:
    Answer: [your final response to the user]
    
    Available actions:
    - SearchDatabase(query) - searches the knowledge base
    - Calculate(expression) - evaluates a math expression
    - GetDateTime() - gets current date/time
    """;
```

---

## 🌳 Pattern 2: Tree-of-Thought

**Explore multiple reasoning paths** and pick the best one. Great for complex problems.

```
Problem: "Design a .NET microservices architecture for e-commerce"

Branch A: Event-driven with RabbitMQ
  ├── Pro: Loose coupling, scalable
  ├── Con: Complex debugging
  └── Score: 7/10

Branch B: gRPC direct communication
  ├── Pro: Type-safe, fast
  ├── Con: Tight coupling
  └── Score: 5/10

Branch C: Hybrid (API Gateway + Events)
  ├── Pro: Best of both worlds
  ├── Con: More infrastructure
  └── Score: 9/10 ← SELECTED

Final Answer: "I recommend Option C because..."
```

### C# Implementation

```csharp
var treeOfThoughtPrompt = """
    For the following problem, explore THREE different approaches.
    For each approach:
    1. Describe the solution briefly
    2. List 2 pros and 2 cons
    3. Rate it 1-10 for the given context
    
    Then select the BEST approach and explain why.
    
    Problem: {{$problem}}
    Context: {{$context}}
    
    Format:
    ## Approach 1: [Name]
    Description: ...
    Pros: ...
    Cons: ...
    Score: X/10
    
    ## Approach 2: [Name]
    ...
    
    ## Approach 3: [Name]
    ...
    
    ## Selected: [Name]
    Reasoning: ...
    """;

var response = await chatClient.GetResponseAsync(new List<ChatMessage>
{
    new(ChatRole.System, treeOfThoughtPrompt
        .Replace("{{$problem}}", "Design a caching strategy for our AI API")
        .Replace("{{$context}}", ".NET 10, high traffic, cost-sensitive"))
});
```

---

## 🔄 Pattern 3: Self-Consistency

**Ask the same question multiple times** and take the majority answer. Reduces hallucination.

```csharp
/// <summary>
/// Asks the AI the same question N times and returns the most common answer.
/// Dramatically reduces hallucination for factual queries.
/// </summary>
public static async Task<string> SelfConsistentQuery(
    IChatClient chatClient,
    string question,
    int samples = 5)
{
    var answers = new List<string>();

    for (int i = 0; i < samples; i++)
    {
        var response = await chatClient.GetResponseAsync(
            question,
            new ChatOptions { Temperature = 0.7f }); // Some variation

        answers.Add(response.Message.Text?.Trim() ?? "");
    }

    // Return the most frequent answer
    var mostCommon = answers
        .GroupBy(a => a)
        .OrderByDescending(g => g.Count())
        .First();

    Console.WriteLine($"Consistency: {mostCommon.Count()}/{samples} "
        + $"({mostCommon.Count() * 100 / samples}%)");

    return mostCommon.Key;
}

// Usage:
var answer = await SelfConsistentQuery(chatClient,
    "Is C# a compiled or interpreted language? Answer in one word.");
// Result: "Compiled" (5/5 = 100% consistency)
```

---

## 🔗 Pattern 4: Prompt Chaining

**Break complex tasks into sequential prompts**, where each step builds on the previous.

```csharp
// Chain: Extract → Validate → Transform → Format
async Task<string> ProcessCustomerEmail(IChatClient client, string email)
{
    // Step 1: Extract structured data
    var extracted = await client.GetResponseAsync<EmailData>(
        $"Extract customer name, issue type, and urgency from:\n{email}");

    // Step 2: Validate the extraction
    var validated = await client.GetResponseAsync(
        $"Is this extraction accurate? Data: {extracted.Result}\n"
        + $"Original email: {email}\nAnswer YES or NO with corrections.");

    // Step 3: Generate response
    var response = await client.GetResponseAsync(
        $"Write a professional customer service response for:\n"
        + $"Customer: {extracted.Result?.Name}\n"
        + $"Issue: {extracted.Result?.IssueType}\n"
        + $"Urgency: {extracted.Result?.Urgency}");

    return response.Message.Text ?? "";
}

record EmailData(string Name, string IssueType, string Urgency);
```

---

## 📊 Pattern Comparison

| Pattern | Best For | Complexity | Token Cost |
|---------|----------|------------|------------|
| **Zero-Shot** | Simple, well-defined tasks | Low | Low |
| **Few-Shot** | Specific output format | Low | Medium |
| **Chain-of-Thought** | Math, logic problems | Low | Medium |
| **ReAct** | Tool-using agents | High | High |
| **Tree-of-Thought** | Complex decisions | Medium | High |
| **Self-Consistency** | Factual accuracy | Low | High (N calls) |
| **Prompt Chaining** | Multi-step workflows | Medium | High (N calls) |

---

## 🔑 Key Takeaways

- **ReAct** = structured reasoning + tool use (ideal for agents)
- **Tree-of-Thought** = explore multiple paths before deciding
- **Self-Consistency** = vote across multiple answers for reliability
- **Prompt Chaining** = decompose complex tasks into simple steps
- **Temperature matters**: 0 for factual, 0.7+ for creative exploration

---

## ➡️ Next

Continue to **[Day 5: AI Evaluation Basics](../Day-05-AI-Evaluation-Basics/README.md)**
