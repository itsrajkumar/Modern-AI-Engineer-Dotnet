# Day 4: Model Context Protocol (MCP)

> **Type:** 💻 Code | **Time:** ~3 hours | **Project:** MCPIntegrationDemo
>
> 🆕 *MCP is the open standard for connecting AI agents to external tools and data*

---

## 🎯 Learning Objectives

- Understand Model Context Protocol (MCP) and why it matters
- Connect a .NET AI agent to external tools via MCP
- Build an MCP server that exposes C# functionality
- Use MCP for tool discovery, invocation, and resource access
- Compare MCP with traditional function calling

---

## 📖 What is MCP?

**Model Context Protocol** is an open standard (started by Anthropic, adopted by Microsoft, OpenAI, Google) that provides a **universal interface** for AI models to discover and use external tools.

```
Without MCP:                          With MCP:
──────────────────                    ──────────────────
Each tool = custom code               Each tool = standard protocol
No discovery                          Auto-discovery of tools
Tight coupling                        Loose coupling
Rebuilj for each provider             Works with any AI framework

┌──────────┐  custom   ┌──────┐      ┌──────────┐  MCP    ┌──────────┐
│ Agent    │──code───►│Tool A │      │ Agent    │───────►│MCP Server│
│          │  custom   ├──────┤      │          │         │          │
│          │──code───►│Tool B │      │          │         │ • Tool A │
│          │  custom   ├──────┤      │          │         │ • Tool B │
│          │──code───►│Tool C │      │          │         │ • Tool C │
└──────────┘          └──────┘      └──────────┘         └──────────┘
  3 integrations                       1 integration (standardized!)
```

---

## 🏗️ MCP Architecture

```
┌─────────────────────────────────────────────────────┐
│                     MCP ECOSYSTEM                    │
│                                                      │
│  ┌─────────────────┐         ┌───────────────────┐  │
│  │   MCP CLIENT     │  JSON   │   MCP SERVER       │  │
│  │   (Your Agent)   │◄──────►│   (Tool Provider)  │  │
│  │                  │  RPC    │                    │  │
│  │  • Discovers     │         │  • Exposes tools   │  │
│  │    available     │         │  • Provides        │  │
│  │    tools         │         │    resources       │  │
│  │  • Calls tools   │         │  • Returns         │  │
│  │  • Gets results  │         │    results         │  │
│  └─────────────────┘         └───────────────────┘  │
│                                                      │
│  Transport: stdio (local) | HTTP/SSE (remote)        │
└─────────────────────────────────────────────────────┘
```

### MCP Capabilities

| Capability | Description | Example |
|-----------|-------------|---------|
| **Tools** | Functions the AI can call | `GetWeather(city)`, `SearchDB(query)` |
| **Resources** | Data sources the AI can read | Files, databases, APIs |
| **Prompts** | Reusable prompt templates | Pre-built task templates |

---

## 💻 Code Sample: Building an MCP Server in .NET

```csharp
using Microsoft.Extensions.Hosting;
using ModelContextProtocol.Server;
using System.ComponentModel;

// =====================================================
// MCP Server: Expose .NET tools to any AI agent
// =====================================================

var builder = Host.CreateApplicationBuilder(args);

builder.Services.AddMcpServer()
    .WithStdioServerTransport()
    .WithToolsFromAssembly();

var app = builder.Build();
await app.RunAsync();

// =====================================================
// Tool definitions — auto-discovered by MCP clients
// =====================================================

[McpServerToolType]
public static class DotNetTools
{
    [McpServerTool]
    [Description("Gets the latest NuGet package version for a given package name")]
    public static async Task<string> GetNuGetVersion(
        [Description("NuGet package name, e.g. 'Microsoft.Extensions.AI'")] string packageName)
    {
        using var http = new HttpClient();
        try
        {
            var url = $"https://api.nuget.org/v3-flatcontainer/{packageName.ToLower()}/index.json";
            var json = await http.GetStringAsync(url);
            // Parse the latest version from the response
            var versions = System.Text.Json.JsonSerializer.Deserialize<NuGetVersions>(json);
            var latest = versions?.Versions?.LastOrDefault() ?? "Not found";
            return $"Latest version of {packageName}: {latest}";
        }
        catch
        {
            return $"Package '{packageName}' not found on NuGet";
        }
    }

    [McpServerTool]
    [Description("Checks if a .NET port number is commonly used and what for")]
    public static string CheckPort(
        [Description("Port number to check")] int port)
    {
        return port switch
        {
            80 => "HTTP (web traffic)",
            443 => "HTTPS (secure web traffic)",
            5000 or 5001 => "ASP.NET Core default (HTTP/HTTPS)",
            5432 => "PostgreSQL database",
            27017 => "MongoDB database",
            6379 => "Redis cache",
            8080 => "Alternative HTTP / Docker apps",
            11434 => "Ollama AI model server",
            _ => $"Port {port}: No common .NET association found"
        };
    }

    [McpServerTool]
    [Description("Generates a GUID in the specified format")]
    public static string GenerateGuid(
        [Description("Format: 'D' (default), 'N' (no hyphens), 'B' (braces)")] string format = "D")
    {
        return Guid.NewGuid().ToString(format);
    }
}

record NuGetVersions(string[]? Versions);
```

---

## 💻 Code Sample: MCP Client (Consuming Tools)

```csharp
using Microsoft.Extensions.AI;
using ModelContextProtocol.Client;

// =====================================================
// MCP Client: Discover and use tools from MCP servers
// =====================================================

// Connect to an MCP server (stdio-based)
var mcpClient = await McpClientFactory.CreateAsync(
    new McpClientOptions
    {
        Id = "dotnet-tools",
        Name = "DotNet Tools",
        TransportType = TransportType.StdIo,
        TransportOptions = new()
        {
            ["command"] = "dotnet",
            ["arguments"] = "run --project ./McpServer"
        }
    });

// Discover available tools automatically
var tools = await mcpClient.ListToolsAsync();
Console.WriteLine($"Discovered {tools.Count} MCP tools:");
foreach (var tool in tools)
    Console.WriteLine($"  • {tool.Name}: {tool.Description}");

// Use discovered tools with IChatClient
IChatClient chatClient = new ChatClientBuilder(innerClient)
    .UseFunctionInvocation()
    .Build();

var options = new ChatOptions
{
    Tools = tools.Select(t => t.AsAITool()).ToList()
};

var response = await chatClient.GetResponseAsync(
    "What's the latest version of Microsoft.Extensions.AI on NuGet?",
    options);

Console.WriteLine($"\nAI: {response.Message.Text}");
```

---

## 📊 MCP vs Traditional Function Calling

| Feature | Traditional (MEAI/SK) | MCP |
|---------|----------------------|-----|
| **Tool definition** | In your code | In separate server |
| **Discovery** | Manual registration | Automatic via protocol |
| **Transport** | In-process | stdio / HTTP / SSE |
| **Cross-language** | .NET only | Any language |
| **Ecosystem** | Your tools only | Shared MCP servers |
| **Updates** | Redeploy app | Update server independently |
| **Security** | Same process trust | Sandboxed, separate process |

---

## 🔑 Key Takeaways

- MCP = **USB-C for AI tools** — one standard connector for everything
- MCP servers **expose tools** that any AI agent can discover and use
- `.NET` has first-class MCP support via `ModelContextProtocol` NuGet
- **stdio transport** for local tools, **HTTP/SSE** for remote services
- MCP tools integrate seamlessly with `IChatClient` via `AsAITool()`

---

## 📚 References

- [MCP Specification](https://modelcontextprotocol.io/)
- [.NET MCP SDK](https://github.com/modelcontextprotocol/csharp-sdk)
- [MCP in Visual Studio](https://learn.microsoft.com/visualstudio/mcp/)

---

## ➡️ Next

Continue to **[Day 5: Human-in-the-Loop & Agent Safety](../Day-05-Human-in-the-Loop/README.md)**
