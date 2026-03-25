# Day 3: Multi-Agent Workflows

> **Type:** 💻 Code | **Time:** ~3 hours | **Project:** MultiAgentDemo
>
> 🆕 *Based on [Lesson 4: AI Agents with MAF](https://github.com/microsoft/Generative-AI-for-beginners-dotnet/blob/main/04-AgentsWithMAF/readme.md)*

---

## 🎯 Learning Objectives

- Build multi-agent systems where specialized agents collaborate
- Implement `AgentPipeline` for sequential agent workflows
- Implement `GroupChat` for agent debate/collaboration
- Use handoff patterns for escalation between agents
- Understand when to use which orchestration pattern

---

## 📖 Multi-Agent Patterns

```
Pattern 1: SEQUENTIAL PIPELINE
───────────────────────────────
User → [Research Agent] → [Writer Agent] → [Editor Agent] → Output
       Gets facts          Writes draft     Polishes text

Pattern 2: CONCURRENT (Parallel)
───────────────────────────────
             ┌── [Safety Agent]  ──┐
User ────────┤                     ├──► Merged Output
             └── [Quality Agent] ──┘

Pattern 3: GROUP CHAT
───────────────────────────────
User Question → [Agent A] → [Agent B] → [Agent A] → ... → Consensus
                 Proposes    Critiques    Refines

Pattern 4: HANDOFF
───────────────────────────────
User → [General Agent] ───X──► [Specialist Agent] → Output
       "I can't handle this,     Takes over for
        passing to specialist"    complex queries
```

---

## 💻 Code Sample: Sequential Pipeline

```csharp
using Microsoft.Extensions.AI;

// =====================================================
// Multi-Agent Pipeline: Research → Write → Edit
// =====================================================

IChatClient chatClient = /* your provider */;

// Agent 1: Researcher — gathers facts
var researchAgent = new ChatMessage(ChatRole.System, """
    You are a Research Agent. Your ONLY job is to:
    1. List 5 key facts about the given topic
    2. Include statistics and dates where possible
    3. Format as numbered bullet points
    Do NOT write prose. Just facts.
    """);

// Agent 2: Writer — creates content from facts
var writerAgent = new ChatMessage(ChatRole.System, """
    You are a Writer Agent. You receive research notes and:
    1. Transform them into a well-structured blog post
    2. Add an engaging introduction and conclusion
    3. Keep it under 300 words
    4. Use a professional but approachable tone
    """);

// Agent 3: Editor — polishes the final output
var editorAgent = new ChatMessage(ChatRole.System, """
    You are an Editor Agent. You receive a blog post draft and:
    1. Fix any grammar or style issues
    2. Improve clarity and flow
    3. Add a compelling title
    4. Ensure the tone is consistent
    Return ONLY the final, polished version.
    """);

// =====================================================
// Run the pipeline
// =====================================================
var topic = "The impact of AI on .NET development in 2026";

Console.WriteLine($"📝 Topic: {topic}\n");

// Step 1: Research
Console.WriteLine("🔍 Step 1: Research Agent working...");
var researchMessages = new List<ChatMessage>
{
    researchAgent,
    new(ChatRole.User, $"Research this topic: {topic}")
};
var researchResult = await chatClient.GetResponseAsync(researchMessages);
Console.WriteLine($"📋 Research:\n{researchResult.Message.Text}\n");

// Step 2: Write (using research output)
Console.WriteLine("✍️ Step 2: Writer Agent working...");
var writerMessages = new List<ChatMessage>
{
    writerAgent,
    new(ChatRole.User, $"Write a blog post based on these research notes:\n\n{researchResult.Message.Text}")
};
var writeResult = await chatClient.GetResponseAsync(writerMessages);
Console.WriteLine($"📄 Draft:\n{writeResult.Message.Text}\n");

// Step 3: Edit (using writer output)
Console.WriteLine("📝 Step 3: Editor Agent working...");
var editorMessages = new List<ChatMessage>
{
    editorAgent,
    new(ChatRole.User, $"Edit and polish this blog post:\n\n{writeResult.Message.Text}")
};
var editResult = await chatClient.GetResponseAsync(editorMessages);
Console.WriteLine($"✅ Final:\n{editResult.Message.Text}");
```

---

## 💻 Code Sample: Group Chat (Debate Pattern)

```csharp
// =====================================================
// Group Chat: Two experts debate a topic
// =====================================================

var optimist = new ChatMessage(ChatRole.System, """
    You are an AI Optimist. You believe AI will massively improve
    software development. Give ONE short argument (2-3 sentences).
    Respond directly to the previous speaker's point.
    """);

var skeptic = new ChatMessage(ChatRole.System, """
    You are an AI Skeptic. You believe AI introduces more risks than
    benefits to software development. Give ONE short counter-argument
    (2-3 sentences). Respond directly to the previous speaker's point.
    """);

var moderator = new ChatMessage(ChatRole.System, """
    You are a Moderator. After hearing both sides, provide a balanced
    summary in 3 bullet points: areas of agreement, key disagreements,
    and your recommendation.
    """);

var debate = new List<string>();
var topic = "Should AI write production code?";

Console.WriteLine($"🎙️ Debate: {topic}\n");

// Round 1: Optimist opens
var r1 = await chatClient.GetResponseAsync(new List<ChatMessage>
{
    optimist,
    new(ChatRole.User, $"Opening argument for: {topic}")
});
debate.Add($"Optimist: {r1.Message.Text}");
Console.WriteLine($"😊 Optimist: {r1.Message.Text}\n");

// Round 1: Skeptic responds
var r2 = await chatClient.GetResponseAsync(new List<ChatMessage>
{
    skeptic,
    new(ChatRole.User, $"Respond to: {r1.Message.Text}")
});
debate.Add($"Skeptic: {r2.Message.Text}");
Console.WriteLine($"🤔 Skeptic: {r2.Message.Text}\n");

// Round 2: Optimist rebuts
var r3 = await chatClient.GetResponseAsync(new List<ChatMessage>
{
    optimist,
    new(ChatRole.User, $"Rebuttal to: {r2.Message.Text}")
});
debate.Add($"Optimist: {r3.Message.Text}");
Console.WriteLine($"😊 Optimist: {r3.Message.Text}\n");

// Moderator summarizes
var summary = await chatClient.GetResponseAsync(new List<ChatMessage>
{
    moderator,
    new(ChatRole.User, $"Summarize this debate:\n{string.Join("\n", debate)}")
});
Console.WriteLine($"⚖️ Moderator: {summary.Message.Text}");
```

---

## 📊 Orchestration Pattern Comparison

| Pattern | Agents | Communication | Best For |
|---------|--------|---------------|----------|
| **Pipeline** | 2-5 | Sequential (A→B→C) | Content creation, data processing |
| **Concurrent** | 2-4 | Parallel (A∥B) | Independent analysis, speed |
| **Group Chat** | 2-6 | Round-robin | Debate, peer review, brainstorming |
| **Handoff** | 2 | Conditional transfer | Escalation, specialization |

---

## 🔑 Key Takeaways

- Multi-agent = **specialized agents** collaborating on a task
- **Pipeline** for ordered workflows (research → write → edit)
- **Group Chat** for collaborative debate and consensus
- Each agent has its own system prompt (persona)
- The output of one agent becomes the input of the next

---

## ➡️ Next

Continue to **[Day 4: Model Context Protocol (MCP)](../Day-04-Model-Context-Protocol/README.md)**
