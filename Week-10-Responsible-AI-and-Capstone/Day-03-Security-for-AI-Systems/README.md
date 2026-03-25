# Day 3: Security for AI Systems

> **Type:** 💻 Code | **Time:** ~3 hours | **Project:** AISecurityDemo

---

## 🎯 Learning Objectives

- Secure API keys using `AzureCliCredential` and user-secrets
- Defend against prompt injection, data exfiltration, and model abuse
- Implement rate limiting and cost controls for AI endpoints
- Build audit logging for all AI interactions
- Understand the OWASP Top 10 for LLM Applications

---

## 📖 OWASP Top 10 for LLM Applications

| # | Vulnerability | .NET Defense |
|---|--------------|--------------|
| 1 | **Prompt Injection** | Input validation middleware (Week 9, Day 3) |
| 2 | **Insecure Output** | Output filtering, never trust AI output as code |
| 3 | **Training Data Poisoning** | Use trusted model providers |
| 4 | **Model Denial of Service** | Rate limiting, token budgets |
| 5 | **Supply Chain** | Pin NuGet versions, verify packages |
| 6 | **Sensitive Info Disclosure** | PII redaction, no secrets in prompts |
| 7 | **Insecure Plugin Design** | Validate tool inputs, least privilege |
| 8 | **Excessive Agency** | Human-in-the-Loop for destructive actions |
| 9 | **Overreliance** | Disclaimers, human review for critical decisions |
| 10 | **Model Theft** | Don't expose model weights, use cloud APIs |

---

## 💻 Code Sample: Secure AI Configuration

```csharp
using Azure.Identity;
using Microsoft.Extensions.AI;

// =====================================================
// SECURE: Use AzureCliCredential (no API keys in code!)
// =====================================================
var credential = new AzureCliCredential();

IChatClient chatClient = new Azure.AI.OpenAI.AzureOpenAIClient(
    new Uri(config["AzureOpenAI:Endpoint"]!),
    credential)
    .AsChatClient("gpt-5");

// =====================================================
// SECURE: Rate limiting per user
// =====================================================
public class PerUserRateLimiter : DelegatingChatClient
{
    private readonly Dictionary<string, (int Count, DateTime Window)> _userCounts = new();
    private readonly int _maxRequestsPerMinute;

    public PerUserRateLimiter(IChatClient inner, int maxPerMinute = 10) : base(inner)
        => _maxRequestsPerMinute = maxPerMinute;

    public override async Task<ChatResponse> GetResponseAsync(
        IList<ChatMessage> chatMessages,
        ChatOptions? options = null,
        CancellationToken cancellationToken = default)
    {
        var userId = options?.AdditionalProperties?["userId"]?.ToString() ?? "anonymous";

        lock (_userCounts)
        {
            if (_userCounts.TryGetValue(userId, out var entry))
            {
                if (DateTime.UtcNow - entry.Window < TimeSpan.FromMinutes(1))
                {
                    if (entry.Count >= _maxRequestsPerMinute)
                        throw new InvalidOperationException(
                            $"Rate limit exceeded for user {userId}. " +
                            $"Max {_maxRequestsPerMinute} requests/minute.");

                    _userCounts[userId] = (entry.Count + 1, entry.Window);
                }
                else
                {
                    _userCounts[userId] = (1, DateTime.UtcNow);
                }
            }
            else
            {
                _userCounts[userId] = (1, DateTime.UtcNow);
            }
        }

        return await base.GetResponseAsync(chatMessages, options, cancellationToken);
    }
}

// =====================================================
// SECURE: Audit logging for all AI interactions
// =====================================================
public class AuditLoggingMiddleware : DelegatingChatClient
{
    private readonly ILogger<AuditLoggingMiddleware> _logger;

    public AuditLoggingMiddleware(IChatClient inner, ILogger<AuditLoggingMiddleware> logger)
        : base(inner) => _logger = logger;

    public override async Task<ChatResponse> GetResponseAsync(
        IList<ChatMessage> chatMessages,
        ChatOptions? options = null,
        CancellationToken cancellationToken = default)
    {
        var userId = options?.AdditionalProperties?["userId"]?.ToString() ?? "unknown";
        var userMessage = chatMessages.LastOrDefault(m => m.Role == ChatRole.User)?.Text ?? "";

        _logger.LogInformation(
            "AI Request | User: {UserId} | Input Length: {Length} | Timestamp: {Time}",
            userId, userMessage.Length, DateTime.UtcNow);

        var response = await base.GetResponseAsync(chatMessages, options, cancellationToken);

        _logger.LogInformation(
            "AI Response | User: {UserId} | Tokens: {In}/{Out} | Model: {Model}",
            userId,
            response.Usage?.InputTokenCount,
            response.Usage?.OutputTokenCount,
            response.ModelId);

        return response;
    }
}
```

---

## 🏗️ Security Architecture

```
┌─────────────────────────────────────────────────────┐
│                AI SECURITY LAYERS                    │
│                                                      │
│  ┌───────────────────────────────────────────────┐  │
│  │ Identity & Auth                               │  │
│  │  • AzureCliCredential (no API keys in code)   │  │
│  │  • User authentication via JWT / OAuth        │  │
│  │  • Role-based access control (RBAC)           │  │
│  └───────────────────────────────────────────────┘  │
│                                                      │
│  ┌───────────────────────────────────────────────┐  │
│  │ Input Protection                              │  │
│  │  • Prompt injection detection                 │  │
│  │  • Input sanitization & length limits         │  │
│  │  • PII redaction before sending to provider   │  │
│  └───────────────────────────────────────────────┘  │
│                                                      │
│  ┌───────────────────────────────────────────────┐  │
│  │ Cost & Abuse Protection                       │  │
│  │  • Per-user rate limiting                     │  │
│  │  • Token budget enforcement                   │  │
│  │  • Anomaly detection (sudden cost spikes)     │  │
│  └───────────────────────────────────────────────┘  │
│                                                      │
│  ┌───────────────────────────────────────────────┐  │
│  │ Audit & Compliance                            │  │
│  │  • All AI interactions logged                 │  │
│  │  • User, prompt hash, tokens, cost tracked    │  │
│  │  • Immutable audit trail for compliance       │  │
│  └───────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────┘
```

---

## 🔑 Key Takeaways

| Security Practice | Implementation |
|------------------|----------------|
| **No API keys in code** | `AzureCliCredential` or `user-secrets` |
| **Rate limiting** | Per-user request limits via middleware |
| **Audit logging** | Log every AI interaction with user ID |
| **Input validation** | Block prompt injection patterns |
| **Cost controls** | Token budget caps per user/day |
| **Data privacy** | PII redaction before sending to AI provider |

---

## ➡️ Next

Continue to **[Day 4: Capstone Architecture & Setup](../Day-04-Capstone-Architecture/README.md)**
