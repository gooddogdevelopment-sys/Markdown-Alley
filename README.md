# Markdown-Alley

A personal collection of development documentation, notes, and resources. Built with MkDocs Material and deployed to GitHub Pages.

## Table of Contents

- [APIs](docs/General/APIs.md) — Curated list of publicly accessible APIs organized by category (News, Geography, Space, Finance, Weather, and more)
- [Coding Articles](docs/General/CodingArticles.md) — Links to useful coding tutorials and articles
- [Web Resources](docs/General/WebResources.md) — Web-based development tools and resources

### AI

- [Ollama Docker Setup](docs/AI/Ollama/OllamaDockerCompose.md) — Docker Compose configuration for running Ollama with Open WebUI, including GPU support and private network options

### .NET

- [Access Modifiers](docs/Dotnet/Classes/AccessModifiers.md) — The six C# access modifiers (`public`, `private`, `protected`, `internal`, `protected internal`, `private protected`) with examples
- [Dependency Injection & Service Lifetimes](docs/Dotnet/DependencyInjection/ServiceLifetimes.md) — Registering services in ASP.NET Core and the transient, scoped, and singleton lifetimes, including captive dependencies
- [DbContext Setup](docs/Dotnet/Database/DotNetDbContext.md) — Setting up Entity Framework Core DbContext, dependency injection, and EF Core migrations
- [Overriding SaveChanges](docs/Dotnet/Database/OverridingSaveChangesDbContext.md) — Automatically setting `CreatedAt`/`UpdatedAt` timestamps by overriding `SaveChangesAsync`
- [Auto Apply Migrations](docs/Dotnet/Database/AutoApplyMigrations.md) — Automatically applying EF Core migrations on app startup using `MigrateAsync`
- [Global Error Handling](docs/Dotnet/Middleware/GlobalErrorHandling.md) — Implementing global exception handling middleware in ASP.NET Core using `IExceptionHandler`
- [GitHub Actions Build & Test](docs/Dotnet/CI-CD/GithubBuildAndTest.md) — Example GitHub Actions workflow to build, restore, and test a .NET project on push
- [.NET 10 Core API with PostgreSQL Template](docs/Dotnet/Templates/Core10WPostgres.md) — Production-ready .NET 10 Web API template pre-configured with PostgreSQL, Serilog, Swagger, and global exception handling
- [EditorConfig Example](docs/Dotnet/Coding-Standards/editorconfig.md) — Example `.editorconfig` for enforcing .NET coding standards

### TypeScript

- [NestJS Basic Setup](docs/Typescript/NestJS/BasicSetup.md) — Getting started with a new NestJS project using the NestJS CLI
- [NestJS Swagger Setup](docs/Typescript/NestJS/Controllers/SwaggerSetup.md) — Installing and configuring Swagger docs in a NestJS app, including JWT Bearer auth support
- [NestJS Swagger Controller Annotations](docs/Typescript/NestJS/Controllers/SwaggerAnnotations.md) — Annotating NestJS controllers with Swagger tags, auth, and response types

### DevOps

- [CI/CD Pipeline Fundamentals](docs/DevOps/CI-CD/PipelineFundamentals.md) — Stack-agnostic pipeline concepts: CI vs. CD, triggers, jobs/steps/stages, runners, artifacts, environments, and caching

