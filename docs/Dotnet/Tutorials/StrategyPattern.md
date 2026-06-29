# Strategy Pattern — AI Content Generator

The **Strategy Pattern** defines a family of algorithms, encapsulates each one, and makes them interchangeable — letting the caller swap behaviour at runtime without changing the code that uses it. This tutorial builds a .NET 10 Web API that uses the pattern to switch between **OpenAI** and **Google Gemini** as AI content generation backends. The calling code never knows (or cares) which provider it's talking to.

**What you'll build:** A `/api/AiContentGen/generate` endpoint that accepts a prompt and a provider name, routes to the correct LLM, and returns the generated text.

---

## Prerequisites

- [.NET 10 SDK](https://dotnet.microsoft.com/download)
- [Docker Desktop](https://www.docker.com/products/docker-desktop/)
- An API key for at least one provider:
    - **OpenAI** → https://platform.openai.com/api-keys
    - **Gemini** → https://aistudio.google.com/apikey

---

## Project Structure

```
strategy-patterns/
├── strategy-patterns/
│   ├── Controllers/
│   │   └── AiContentGenController.cs
│   ├── Middleware/
│   │   └── GlobalExceptionHandler.cs
│   ├── Models/
│   │   └── GenerateContentRequest.cs
│   ├── Services/
│   │   ├── IAiContentGenerator.cs   ← the strategy interface
│   │   ├── OpenAiGenerator.cs       ← concrete strategy
│   │   ├── GeminiGenerator.cs       ← concrete strategy
│   │   └── ContentService.cs        ← resolves and calls the active strategy
│   ├── appsettings.json
│   ├── appsettings.Development.json
│   ├── Dockerfile
│   └── Program.cs
├── compose.yaml
└── .env.example
```

---

## NuGet Packages

```bash
dotnet add package Google.GenAI
dotnet add package Microsoft.AspNetCore.OpenApi
dotnet add package Microsoft.Extensions.AI
dotnet add package Microsoft.Extensions.AI.OpenAI
```

---

## Step 1 — Define the Strategy Interface

The interface is the contract that every strategy must fulfil. `ContentService` and the controller only ever depend on this — never on a concrete provider.

```csharp
// Services/IAiContentGenerator.cs
namespace strategy_patterns.Services;

public interface IAiContentGenerator
{
    Task<string> GenerateTextAsync(string prompt, string modelId);
}
```

---

## Step 2 — Implement the Concrete Strategies

Each class calls its own SDK directly. This is the point of the pattern: two completely different API shapes hidden behind the same interface.

### OpenAI

```csharp
// Services/OpenAiGenerator.cs
using OpenAI.Chat;

namespace strategy_patterns.Services;

public class OpenAiGenerator(string apiKey) : IAiContentGenerator
{
    public async Task<string> GenerateTextAsync(string prompt, string modelId)
    {
        var client = new ChatClient(modelId, apiKey);
        var response = await client.CompleteChatAsync(prompt);
        return response.Value.Content[0].Text;
    }
}
```

### Gemini

```csharp
// Services/GeminiGenerator.cs
using Google.GenAI;

namespace strategy_patterns.Services;

public class GeminiGenerator(string apiKey) : IAiContentGenerator
{
    public async Task<string> GenerateTextAsync(string prompt, string modelId)
    {
        var client = new Client(apiKey: apiKey);
        var response = await client.Models.GenerateContentAsync(modelId, prompt);
        return response.Text!;
    }
}
```

---

## Step 3 — Create the Context (ContentService)

`ContentService` is the *context* in strategy pattern terms. It holds no provider logic — it just resolves the right strategy by key and delegates to it.

Keyed DI (`GetKeyedService`) lets multiple implementations of the same interface coexist in the container, distinguished by a string key.

```csharp
// Services/ContentService.cs
namespace strategy_patterns.Services;

public interface IContentService
{
    Task<string> CreatePostAsync(string prompt, string providerName, string modelId);
}

public class ContentService(IServiceProvider serviceProvider, ILogger<ContentService> logger) : IContentService
{
    public async Task<string> CreatePostAsync(string prompt, string providerName, string modelId)
    {
        var generator = serviceProvider.GetKeyedService<IAiContentGenerator>(providerName);

        if (generator != null) return await generator.GenerateTextAsync(prompt, modelId);

        logger.LogError("The AI provider is not supported: '{providerName}'", providerName);
        throw new ArgumentException($"AI Provider '{providerName}' is not supported.");
    }
}
```

---

## Step 4 — Register Services in Program.cs

Each strategy is registered as a **keyed scoped service**. The factory lambda reads the API key from configuration and constructs the generator — failing fast with `ArgumentException.ThrowIfNullOrWhiteSpace` if the key is missing.

```csharp
using strategy_patterns.Services;

var builder = WebApplication.CreateBuilder(args);

builder.Services.AddExceptionHandler<GlobalExceptionHandler>();
builder.Services.AddProblemDetails();
builder.Services.AddControllers();
builder.Services.AddOpenApi();

builder.Services.AddKeyedScoped<IAiContentGenerator, OpenAiGenerator>("OpenAI", (provider, _) =>
{
    var config = provider.GetRequiredService<IConfiguration>();
    var apiKey = config["OpenAI:ApiKey"];
    ArgumentException.ThrowIfNullOrWhiteSpace(apiKey, "OpenAI:ApiKey");
    return new OpenAiGenerator(apiKey);
});

builder.Services.AddKeyedScoped<IAiContentGenerator, GeminiGenerator>("Gemini", (provider, _) =>
{
    var config = provider.GetRequiredService<IConfiguration>();
    var apiKey = config["Gemini:ApiKey"];
    ArgumentException.ThrowIfNullOrWhiteSpace(apiKey, "Gemini:ApiKey");
    return new GeminiGenerator(apiKey);
});

builder.Services.AddScoped<IContentService, ContentService>();

var app = builder.Build();

if (app.Environment.IsDevelopment())
    app.MapOpenApi();

app.UseAuthorization();
app.MapControllers();
app.Run();
```

---

## Step 5 — Add the Request Model

```csharp
// Models/GenerateContentRequest.cs
namespace strategy_patterns.Models;

public record GenerateContentRequest(
    string Prompt,
    string ProviderName,  // "OpenAI" or "Gemini"
    string ModelId
);
```

---

## Step 6 — Add the Controller

The controller passes `ProviderName` directly to `ContentService`, which resolves the correct strategy. The controller itself has no awareness of which provider is being used.

```csharp
// Controllers/AiContentGenController.cs
using Microsoft.AspNetCore.Mvc;
using strategy_patterns.Models;
using strategy_patterns.Services;

namespace strategy_patterns.Controllers;

[ApiController]
[Route("api/[controller]")]
public class AiContentGenController(IContentService contentService) : ControllerBase
{
    [HttpPost("generate")]
    public async Task<IActionResult> Generate([FromBody] GenerateContentRequest request)
    {
        var result = await contentService.CreatePostAsync(request.Prompt, request.ProviderName, request.ModelId);
        return Ok(new { Content = result });
    }
}
```

---

## Step 7 — Configuration

### Local Development

Add keys to `appsettings.Development.json` (do not commit this file if it contains real keys):

```json
{
  "OpenAI": {
    "ApiKey": "your-openai-key"
  },
  "Gemini": {
    "ApiKey": "your-gemini-key"
  }
}
```

### Docker

Create a `.env` file from the provided example:

```bash
cp .env.example .env
```

```env
OPENAI_API_KEY=your-openai-key
GEMINI_API_KEY=your-gemini-key
```

The `compose.yaml` maps these into the container as ASP.NET Core configuration environment variables (double underscore `__` maps to `:` in config keys):

```yaml
services:
  strategy-patterns:
    image: strategy-patterns
    build:
      context: .
      dockerfile: strategy-patterns/Dockerfile
    ports:
      - "8080:8080"
    environment:
      - ASPNETCORE_ENVIRONMENT=Production
      - OpenAI__ApiKey=${OPENAI_API_KEY}
      - Gemini__ApiKey=${GEMINI_API_KEY}
```

---

## Running the Project

### Local

```bash
cd strategy-patterns
dotnet run
```

API available at `http://localhost:5244`.

### Docker

```bash
docker compose up --build
```

API available at `http://localhost:8080`.

---

## Usage

`POST /api/AiContentGen/generate`

```json
{
  "prompt": "Write a short blog post about the strategy pattern in C#",
  "providerName": "OpenAI",
  "modelId": "gpt-4o"
}
```

Switch providers by changing `providerName` to `"Gemini"` and `modelId` to a Gemini model such as `"gemini-2.0-flash"`. No other change is needed.

```bash
curl -X POST http://localhost:8080/api/AiContentGen/generate \
  -H "Content-Type: application/json" \
  -d '{"prompt": "Explain the strategy pattern", "providerName": "Gemini", "modelId": "gemini-2.0-flash"}'
```
