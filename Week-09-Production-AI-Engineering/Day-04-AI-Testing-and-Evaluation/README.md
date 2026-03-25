# Day 4: AI Testing & Evaluation

> **Type:** 💻 Code | **Time:** ~3 hours | **Project:** AITestingSuite

---

## 🎯 Learning Objectives

- Write unit tests for AI-powered features using xUnit
- Mock `IChatClient` for deterministic test results
- Build evaluation harnesses for prompt quality
- Implement regression testing for AI outputs
- Measure AI quality: accuracy, relevance, safety

---

## 📖 The AI Testing Challenge

```
Traditional Code:                  AI-Powered Code:
─────────────────                  ─────────────────────
Input → Deterministic Output       Input → Non-deterministic Output
Assert.Equal(4, Add(2,2))          Assert.??? (chatClient.GetResponse("..."))

Easy to test                       How do you test something
                                   that gives different answers
                                   every time?
```

### Solution: Test the CONTRACT, not the CONTENT

```
DON'T test:  "Response must be exactly 'DI is a design pattern...'"
DO test:     "Response must contain 'dependency injection'"
             "Response must be < 200 words"
             "Response must be valid JSON"
             "Response must not contain PII"
```

---

## 💻 Code Sample: Unit Testing with Mocked IChatClient

```csharp
using Microsoft.Extensions.AI;
using NSubstitute;
using Xunit;

/// <summary>
/// Unit tests for AI-powered services.
/// IChatClient is mocked — no API calls needed!
/// </summary>
public class AIChatServiceTests
{
    private readonly IChatClient _mockChatClient;
    private readonly AIChatService _sut;

    public AIChatServiceTests()
    {
        _mockChatClient = Substitute.For<IChatClient>();
        _sut = new AIChatService(_mockChatClient);
    }

    [Fact]
    public async Task SummarizeAsync_ReturnsTextWithinLimit()
    {
        // Arrange: Mock the AI response
        var mockResponse = new ChatResponse(
            new ChatMessage(ChatRole.Assistant, "This is a brief summary of the document."));

        _mockChatClient
            .GetResponseAsync(
                Arg.Any<IList<ChatMessage>>(),
                Arg.Any<ChatOptions>(),
                Arg.Any<CancellationToken>())
            .Returns(mockResponse);

        // Act
        var result = await _sut.SummarizeAsync("Long document text here...");

        // Assert
        Assert.NotNull(result);
        Assert.True(result.Length < 500, "Summary should be under 500 chars");
    }

    [Fact]
    public async Task ClassifyAsync_ReturnsValidCategory()
    {
        // Arrange
        var mockResponse = new ChatResponse(
            new ChatMessage(ChatRole.Assistant, "Technical"));

        _mockChatClient
            .GetResponseAsync(
                Arg.Any<IList<ChatMessage>>(),
                Arg.Any<ChatOptions>(),
                Arg.Any<CancellationToken>())
            .Returns(mockResponse);

        // Act
        var category = await _sut.ClassifyAsync("My API throws a 500 error");

        // Assert
        var validCategories = new[] { "Technical", "Billing", "General" };
        Assert.Contains(category, validCategories);
    }

    [Fact]
    public async Task SummarizeAsync_IncludesSystemPrompt()
    {
        // Arrange
        var mockResponse = new ChatResponse(
            new ChatMessage(ChatRole.Assistant, "Summary"));

        _mockChatClient
            .GetResponseAsync(
                Arg.Any<IList<ChatMessage>>(),
                Arg.Any<ChatOptions>(),
                Arg.Any<CancellationToken>())
            .Returns(mockResponse);

        // Act
        await _sut.SummarizeAsync("Test text");

        // Assert: Verify the system prompt was included
        await _mockChatClient.Received(1).GetResponseAsync(
            Arg.Is<IList<ChatMessage>>(msgs =>
                msgs.Any(m => m.Role == ChatRole.System)),
            Arg.Any<ChatOptions>(),
            Arg.Any<CancellationToken>());
    }
}

/// <summary>
/// The service under test.
/// </summary>
public class AIChatService
{
    private readonly IChatClient _chatClient;

    public AIChatService(IChatClient chatClient) => _chatClient = chatClient;

    public async Task<string> SummarizeAsync(string text)
    {
        var messages = new List<ChatMessage>
        {
            new(ChatRole.System, "Summarize in under 100 words."),
            new(ChatRole.User, text)
        };
        var response = await _chatClient.GetResponseAsync(messages);
        return response.Message.Text ?? "";
    }

    public async Task<string> ClassifyAsync(string ticket)
    {
        var messages = new List<ChatMessage>
        {
            new(ChatRole.System, "Classify into: Technical, Billing, or General. Reply with ONLY the category."),
            new(ChatRole.User, ticket)
        };
        var response = await _chatClient.GetResponseAsync(messages,
            new ChatOptions { Temperature = 0f });
        return response.Message.Text?.Trim() ?? "General";
    }
}
```

---

## 💻 Code Sample: AI Output Evaluation

```csharp
/// <summary>
/// Evaluates AI outputs against quality criteria.
/// Run these as integration tests against a real model.
/// </summary>
public class AIEvaluationTests
{
    private readonly IChatClient _chatClient; // Real client for eval

    public AIEvaluationTests()
    {
        // Use a real client for evaluation tests
        _chatClient = new OpenAIClient(Environment.GetEnvironmentVariable("OPENAI_API_KEY")!)
            .AsChatClient("gpt-4o-mini");
    }

    [Theory]
    [InlineData("What is C#?", "programming language")]
    [InlineData("What is Entity Framework?", "ORM")]
    [InlineData("What is dependency injection?", "design pattern")]
    public async Task ResponseContainsExpectedKeyword(string question, string expectedKeyword)
    {
        var response = await _chatClient.GetResponseAsync(question,
            new ChatOptions { Temperature = 0f });

        Assert.Contains(expectedKeyword, response.Message.Text,
            StringComparison.OrdinalIgnoreCase);
    }

    [Fact]
    public async Task ResponseRespectsMaxLength()
    {
        var response = await _chatClient.GetResponseAsync(
            "Explain .NET in exactly 2 sentences.",
            new ChatOptions { Temperature = 0f, MaxOutputTokens = 100 });

        var sentenceCount = response.Message.Text!
            .Split('.', StringSplitOptions.RemoveEmptyEntries).Length;

        Assert.InRange(sentenceCount, 1, 4); // Allow some flexibility
    }

    [Fact]
    public async Task JsonOutputIsValid()
    {
        var prompt = """
            Return a JSON object with fields: name (string), version (int).
            Describe the .NET framework. Return ONLY JSON.
            """;

        var response = await _chatClient.GetResponseAsync(prompt,
            new ChatOptions { Temperature = 0f });

        // Must parse as valid JSON
        var isValid = false;
        try
        {
            System.Text.Json.JsonDocument.Parse(response.Message.Text!);
            isValid = true;
        }
        catch { }

        Assert.True(isValid, $"Response was not valid JSON: {response.Message.Text}");
    }

    [Fact]
    public async Task ResponseDoesNotContainPII()
    {
        var response = await _chatClient.GetResponseAsync(
            "Generate a sample customer profile for testing.");

        var text = response.Message.Text!;

        // Should not contain real-looking SSNs, credit cards, etc.
        Assert.DoesNotMatch(@"\d{3}-\d{2}-\d{4}", text);   // SSN pattern
        Assert.DoesNotMatch(@"\d{4}-\d{4}-\d{4}-\d{4}", text); // CC pattern
    }
}
```

---

## 📊 AI Testing Pyramid

```
                    ┌──────────────┐
                    │   E2E Tests  │  Full pipeline with real model
                    │   (few)      │  Expensive, slow, flaky
                    ├──────────────┤
                    │  Integration │  Real model, specific prompts
                    │  Tests       │  Keyword checks, format validation
                    │  (moderate)  │
                    ├──────────────┤
                    │  Unit Tests  │  Mocked IChatClient
                    │  (many)      │  Fast, deterministic, free
                    └──────────────┘

    Focus: 70% unit | 20% integration | 10% E2E
```

---

## 🔑 Key Takeaways

| Strategy | When | Cost |
|----------|------|------|
| **Mock IChatClient** | Unit tests for business logic | Free, instant |
| **Keyword assertions** | Integration tests for relevance | API cost per test |
| **JSON validation** | Structured output testing | API cost per test |
| **PII checks** | Safety regression testing | API cost per test |
| **Temperature = 0** | Reduce flakiness in eval tests | Same cost |

---

## ➡️ Next

Continue to **[Day 5: .NET 10 Migration Guide](../Day-05-DotNet10-Migration-Guide/README.md)**
