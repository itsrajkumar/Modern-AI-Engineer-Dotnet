# Day 1: Multimodal AI (Vision & Images)

> **Type:** 💻 Code | **Time:** ~3 hours | **Project:** MultimodalDemo
>
> 🆕 *Covers [Lesson 3: Working with Images and Multimedia](https://github.com/microsoft/Generative-AI-for-beginners-dotnet/blob/main/03-CoreGenerativeAITechniques/readme.md) from Generative AI for Beginners .NET v2*

---

## 🎯 Learning Objectives

- Send images to AI models for analysis (Vision)
- Generate images from text prompts (DALL-E / Image Generation)
- Handle multimodal `ChatMessage` content (text + images)
- Build a practical image analysis pipeline in C#
- Understand token costs for vision requests

---

## 📖 What is Multimodal AI?

**Multimodal AI** can process and generate multiple types of content:

```
Traditional LLM:           Multimodal LLM:
──────────────────         ──────────────────────────
Text → Text                Text    → Text
                           Image   → Text (describe it!)
                           Text    → Image (generate it!)
                           Audio   → Text (transcribe it!)
                           Text+Image → Text (analyze together!)
```

### Models with Vision Support

| Model | Vision | Image Gen | Audio | Provider |
|-------|--------|-----------|-------|----------|
| GPT-5 | ✅ | ✅ (DALL-E) | ✅ | OpenAI |
| GPT-4o | ✅ | ❌ | ✅ | OpenAI / Azure |
| Gemini 2.0 | ✅ | ✅ | ✅ | Google |
| Phi-4-vision | ✅ | ❌ | ❌ | Ollama (local) |
| LLaVA | ✅ | ❌ | ❌ | Ollama (local) |

---

## 💻 Code Sample: Vision (Image Analysis)

```csharp
using Microsoft.Extensions.AI;
using Microsoft.Extensions.Configuration;
using OpenAI;

// =====================================================
// Day 1: Multimodal AI — Vision
// =====================================================

var config = new ConfigurationBuilder().AddUserSecrets<Program>().Build();
var apiKey = config["OpenAI:ApiKey"]!;

IChatClient chatClient = new OpenAIClient(apiKey)
    .AsChatClient("gpt-4o");

// =====================================================
// Example 1: Analyze an image from URL
// =====================================================
Console.WriteLine("=== Image Analysis from URL ===\n");

var imageUrl = "https://upload.wikimedia.org/wikipedia/commons/thumb/e/ee/Logo_of_.NET.svg/512px-Logo_of_.NET.svg.png";

var messages = new List<ChatMessage>
{
    new(ChatRole.User, [
        new TextContent("What is shown in this image? Describe it in detail."),
        new ImageContent(new Uri(imageUrl))
    ])
};

var response = await chatClient.GetResponseAsync(messages);
Console.WriteLine($"AI sees: {response.Message.Text}\n");

// =====================================================
// Example 2: Analyze a local image file
// =====================================================
Console.WriteLine("=== Local Image Analysis ===\n");

var imagePath = "screenshot.png"; // Put any image here
if (File.Exists(imagePath))
{
    var imageBytes = await File.ReadAllBytesAsync(imagePath);
    var base64 = Convert.ToBase64String(imageBytes);

    var localMessages = new List<ChatMessage>
    {
        new(ChatRole.User, [
            new TextContent("Analyze this screenshot. What application is shown? List all UI elements you can see."),
            new ImageContent(imageBytes, "image/png")
        ])
    };

    var localResponse = await chatClient.GetResponseAsync(localMessages);
    Console.WriteLine($"Analysis: {localResponse.Message.Text}\n");
}

// =====================================================
// Example 3: Code Review from Screenshot
// =====================================================
Console.WriteLine("=== Practical: AI Code Review from Screenshot ===\n");

var codeReviewMessages = new List<ChatMessage>
{
    new(ChatRole.System, """
        You are a senior C# code reviewer. When shown code screenshots:
        1. Identify the programming language
        2. List any bugs or issues (numbered)
        3. Suggest improvements
        4. Rate the code quality 1-10
        Format your response with clear headers.
        """),
    new(ChatRole.User, [
        new TextContent("Review this code for bugs and suggest improvements:"),
        new ImageContent(new Uri("https://raw.githubusercontent.com/microsoft/Generative-AI-for-beginners-dotnet/main/03-CoreGenerativeAITechniques/images/code-sample.png"))
    ])
};

var reviewResponse = await chatClient.GetResponseAsync(codeReviewMessages);
Console.WriteLine(reviewResponse.Message.Text);

Console.WriteLine("\n✅ Multimodal AI demo complete!");
```

---

## 📖 Token Costs for Vision

Vision requests use more tokens than text:

| Image Size | Approximate Tokens | Cost (GPT-4o) |
|-----------|-------------------|-------|
| 512×512 (low detail) | ~85 tokens | ~$0.0004 |
| 1024×1024 (high detail) | ~765 tokens | ~$0.004 |
| 2048×2048 (high detail) | ~1,600 tokens | ~$0.008 |

> **Tip:** Use `ImageContent` with `detail: "low"` for cheaper analysis when high resolution isn't needed.

---

## 🏗️ Multimodal Content Types

```csharp
// Microsoft.Extensions.AI provides these content types:
new TextContent("Describe this image")      // Text
new ImageContent(uri)                        // Image from URL
new ImageContent(bytes, "image/png")         // Image from bytes
new AudioContent(bytes, "audio/wav")         // Audio (where supported)

// Combine them in a single ChatMessage:
var message = new ChatMessage(ChatRole.User, [
    new TextContent("What's in these images?"),
    new ImageContent(image1Uri),
    new ImageContent(image2Uri)
]);
```

---

## 🔑 Key Takeaways

| Concept | Details |
|---------|---------|
| **Vision** | Send images + text to AI for analysis |
| **ImageContent** | Accepts URLs or byte arrays |
| **Multi-image** | Send multiple images in one message |
| **Cost** | Vision tokens are more expensive than text |
| **Local models** | Phi-4-vision and LLaVA support vision locally |
| **Practical uses** | Code review, document analysis, accessibility |

---

## 🧪 Self-Assessment

1. What C# class represents image content in a ChatMessage?
2. Can you send multiple images in a single request?
3. How many extra tokens does a 1024×1024 image cost?
4. Name two local models that support vision.
5. What's a practical use case for vision AI in .NET development?

<details>
<summary>Answers</summary>

1. `ImageContent` — accepts URI or byte array
2. Yes — add multiple `ImageContent` objects to the content list
3. ~765 tokens
4. Phi-4-vision and LLaVA (via Ollama)
5. Automated code review from screenshots, document OCR, UI testing validation

</details>

---

## ➡️ Next

Continue to **[Day 2: Local Models & Providers](../Day-02-Local-Models-and-Providers/README.md)**
