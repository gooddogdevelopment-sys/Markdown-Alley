# .NET 10 Core API with PostgreSQL Template

## Overview

This is a production-ready .NET 10 Web API template pre-configured with PostgreSQL via Entity Framework Core. It is designed to serve as a clean, opinionated starting point for new API projects, eliminating boilerplate setup so you can focus on building features immediately.

**Target Framework:** .NET 10
**Database:** PostgreSQL (via Npgsql EF Core provider)
**Repository:** `dotnet-core-api-w-postgres`

The template bundles the following capabilities out of the box:

- Structured logging with **Serilog**
- PostgreSQL database access via **Entity Framework Core 10**
- API documentation via **Swagger (Swashbuckle)** and the native **OpenAPI** endpoint
- Global exception handling following **RFC 9457 Problem Details**
- A `BaseEntity` model with automatic `CreatedAt`/`UpdatedAt` timestamp management
- Docker and Docker Compose support
- A GitHub Actions CI pipeline

---

## Project Structure

```
dotnet-core-api-w-postgres/
├── .github/
│   └── workflows/
│       └── build.yml              # CI pipeline
├── dotnet-core-api-w-postgres/
│   ├── Controllers/
│   │   └── PingController.cs      # Health check endpoint
│   ├── Data/
│   │   └── AppDbContext.cs        # EF Core DbContext
│   ├── Middleware/
│   │   └── GlobalExceptionHandler.cs
│   ├── Models/
│   │   └── BaseEntity.cs          # Base model with audit fields
│   ├── Properties/
│   │   └── launchSettings.json
│   ├── appsettings.json
│   ├── appsettings.Development.json
│   ├── Dockerfile
│   └── Program.cs                 # Application entry point & service registration
├── compose.yaml                   # Docker Compose for local development
└── dotnet-core-api-w-postgres.sln
```

---

## NuGet Dependencies

| Package | Version | Purpose |
|---|---|---|
| `Microsoft.AspNetCore.OpenApi` | 10.0.2 | Native OpenAPI endpoint (`/openapi/v1.json`) |
| `Microsoft.EntityFrameworkCore.Design` | 10.0.5 | EF Core tooling (migrations, scaffolding) |
| `Npgsql.EntityFrameworkCore.PostgreSQL` | 10.0.1 | PostgreSQL EF Core provider |
| `Serilog.AspNetCore` | 10.0.0 | Structured logging + HTTP request logging |
| `Swashbuckle.AspNetCore` | 10.1.5 | Swagger UI and Swagger JSON generation |

---

## Logging

Logging is handled by **Serilog**, configured directly in `Program.cs` before the application builder is built. This ensures that any startup errors are captured by the logger.

```csharp
Log.Logger = new LoggerConfiguration()
    .MinimumLevel.Information()
    .WriteTo.Console()
    .Enrich.FromLogContext()
    .CreateLogger();

builder.Host.UseSerilog();
```

The logger is configured with:

- **Minimum level:** `Information` — debug-level noise is suppressed in all environments by default.
- **Console sink:** All log output goes to stdout, making it compatible with Docker and cloud log aggregators.
- **Log context enrichment:** Properties pushed onto the `LogContext` (e.g., correlation IDs, user identifiers) are automatically included in every log event within that context scope.

### HTTP Request Logging

After the app is built, Serilog's request logging middleware is registered:

```csharp
app.UseSerilogRequestLogging();
```

This replaces ASP.NET Core's default per-request log output with a single, structured Serilog log event per request, including method, path, status code, and elapsed time. This middleware is optional and can be removed if the default ASP.NET Core request logging is preferred.

### Extending Logging

To add additional sinks (e.g., file, Seq, Application Insights), extend the `LoggerConfiguration` chain in `Program.cs`:

```csharp
Log.Logger = new LoggerConfiguration()
    .MinimumLevel.Information()
    .WriteTo.Console()
    .WriteTo.File("logs/log-.txt", rollingInterval: RollingInterval.Day) // example
    .Enrich.FromLogContext()
    .CreateLogger();
```

---

## DbContext Configuration

### Registration

The `AppDbContext` is registered in `Program.cs` using the `Default` connection string from configuration:

```csharp
builder.Services.AddDbContext<AppDbContext>(opts =>
    opts.UseNpgsql(builder.Configuration.GetConnectionString("Default")));
```

The connection string should be added to `appsettings.json` (or via an environment variable / secrets manager in production):

```json
{
  "ConnectionStrings": {
    "Default": "Host=localhost;Database=dotnet_api_db;Username=postgres;Password=postgres"
  }
}
```

When running via Docker Compose the connection string is injected through the environment variable `ConnectionStrings__Default` (note the double underscore — ASP.NET Core's configuration system maps this to the `ConnectionStrings:Default` key, matching the `"Default"` key used in `GetConnectionString("Default")`).

> **Note:** The `compose.yaml` in this template uses `ConnectionStrings__DefaultConnection`, which would map to the key `"DefaultConnection"` — a mismatch with the `"Default"` key expected by `Program.cs`. If you use Docker Compose and find the API cannot connect to the database, update the environment variable in `compose.yaml` from `ConnectionStrings__DefaultConnection` to `ConnectionStrings__Default`.

### AppDbContext

`AppDbContext` extends `DbContext` and uses the primary constructor pattern introduced in C# 12:

```csharp
public class AppDbContext(DbContextOptions<AppDbContext> options) : DbContext(options)
```

**Entity configurations** are loaded automatically via `OnModelCreating`:

```csharp
modelBuilder.ApplyConfigurationsFromAssembly(typeof(AppDbContext).Assembly);
```

Any class implementing `IEntityTypeConfiguration<T>` in the assembly will be discovered and applied automatically. This keeps entity configuration out of `AppDbContext` and in dedicated, per-entity configuration classes.

**Adding new entities** — uncomment or add a `DbSet` property following the pattern shown in the placeholder comment:

```csharp
public DbSet<MyEntity> MyEntities => Set<MyEntity>();
```

### Automatic Timestamp Management

`SaveChangesAsync` is overridden to automatically manage `CreatedAt` and `UpdatedAt` on any entity that inherits from `BaseEntity`:

```csharp
public override Task<int> SaveChangesAsync(CancellationToken ct = default)
{
    var now = DateTime.UtcNow;

    foreach (var entry in ChangeTracker.Entries<BaseEntity>())
    {
        switch (entry.State)
        {
            case EntityState.Added:
                entry.Entity.CreatedAt = now;
                entry.Entity.UpdatedAt = now;
                break;
            case EntityState.Modified:
                entry.Entity.UpdatedAt = now;
                entry.Property(e => e.CreatedAt).IsModified = false; // prevents overwrite
                break;
        }
    }

    return base.SaveChangesAsync(ct);
}
```

All timestamps are stored in **UTC**. On `Added`, both fields are set. On `Modified`, only `UpdatedAt` is set and `CreatedAt` is explicitly marked as unmodified to prevent accidental overwrites.

### BaseEntity

All domain entities should inherit from `BaseEntity` to get automatic audit field management:

```csharp
public abstract class BaseEntity
{
    [Key]
    public int Id { get; set; }
    public DateTime CreatedAt { get; set; }
    public DateTime UpdatedAt { get; set; }
}
```

---

## Swagger / OpenAPI

The template registers both **Swashbuckle** (for the Swagger UI) and the native **ASP.NET Core OpenAPI** endpoint.

### Registration

```csharp
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen(c =>
{
    c.SwaggerDoc("v1", new OpenApiInfo { Title = "API Template", Version = "v1" });
    c.AddSecurityDefinition("Bearer", new OpenApiSecurityScheme
    {
        Name = "Authorization",
        Type = SecuritySchemeType.ApiKey,
        In = ParameterLocation.Header,
        Description = "JWT Authorization header using the Bearer scheme. Example: \"Bearer {token}\""
    });
});
builder.Services.AddOpenApi();
```

A Bearer token security definition is pre-registered in the Swagger UI, making it straightforward to test authenticated endpoints once authentication middleware is wired up.

### Middleware

```csharp
app.MapOpenApi();     // Native OpenAPI JSON: /openapi/v1.json
app.UseSwagger();     // Swashbuckle JSON: /swagger/v1/swagger.json
app.UseSwaggerUI();   // Swagger UI: /swagger/index.html
```

By default, the Swagger UI is available in **all environments**. If you want to restrict it to the development environment only, replace the above block with the commented-out version already present in `Program.cs`:

```csharp
if (app.Environment.IsDevelopment())
{
    app.MapOpenApi();
    app.UseSwagger();
    app.UseSwaggerUI();
}
```

### Accessing the Docs

| URL | Description |
|---|---|
| `/swagger/index.html` | Swagger UI |
| `/swagger/v1/swagger.json` | Swashbuckle-generated OpenAPI JSON |
| `/openapi/v1.json` | Native ASP.NET Core OpenAPI JSON |

---

## Global Exception Handling

Unhandled exceptions are caught by `GlobalExceptionHandler`, which implements `IExceptionHandler` — the standard ASP.NET Core exception handling interface introduced in .NET 8.

### Registration

```csharp
builder.Services.AddExceptionHandler<GlobalExceptionHandler>();
builder.Services.AddProblemDetails();
```

### Behavior

The handler logs every unhandled exception at the `Error` level and returns a structured **RFC 9457 Problem Details** JSON response. Specific exception types are mapped to appropriate HTTP status codes:

| Exception Type | HTTP Status |
|---|---|
| `ArgumentException` | 400 Bad Request |
| `KeyNotFoundException` | 404 Not Found |
| `UnauthorizedAccessException` | 401 Unauthorized |
| All other exceptions | 500 Internal Server Error |

Every response includes the ASP.NET Core `TraceIdentifier` as an extension field (`traceId`), enabling correlation between client-facing error responses and server-side log entries.

**Example response:**

```json
{
  "status": 404,
  "title": "Not Found",
  "detail": "The requested resource could not be found.",
  "traceId": "00-abc123-def456-00"
}
```

To add new exception mappings, extend the `switch` expression in `GlobalExceptionHandler.TryHandleAsync`.

---

## Ping / Health Check Endpoint

A `PingController` is included as a minimal health check:

```
GET /ping  →  200 OK  "Pong"
```

This is also the endpoint used by the Docker Compose health check for the API container.

---

## Docker

### Dockerfile

The `Dockerfile` uses a multi-stage build:

1. **base** — runtime image (`mcr.microsoft.com/dotnet/aspnet:10.0`), exposes ports `8080` and `8081`.
2. **build** — SDK image (`mcr.microsoft.com/dotnet/sdk:10.0`), restores and builds the project in Release configuration.
3. **publish** — publishes the build output.
4. **final** — copies the published output into the runtime image.

The default target OS for Docker is **Linux** (set via `<DockerDefaultTargetOS>Linux</DockerDefaultTargetOS>` in the `.csproj`).

### Docker Compose

`compose.yaml` defines a local development stack with two services:

**postgres** — PostgreSQL 16 with a persistent named volume (`postgres_data`) and a health check that polls `pg_isready`.

**dotnet-core-api-w-postgres** — The API container, which:
- Builds from the local `Dockerfile`
- Exposes ports `8080` (HTTP) and `8081` (HTTPS)
- Injects the database connection string via environment variable
- Depends on the `postgres` service being healthy before starting
- Has its own health check that polls `GET /ping`

Both services run on a shared bridge network (`dotnet_network`).

**To start the full stack locally:**

```bash
docker compose up --build
```

---

## GitHub Actions CI Pipeline

The workflow defined in `.github/workflows/build.yml` runs on every push and pull request to `main`.

**Steps:**

1. Checkout the repository
2. Set up .NET 10
3. Restore NuGet dependencies
4. Build the solution in Release configuration
5. Run all tests and output results in TRX format
6. Upload test results as a build artifact (always, even on failure)

---

## Getting Started

### Local Development (without Docker)

1. Ensure you have the [.NET 10 SDK](https://dotnet.microsoft.com/download) installed.
2. Start a local PostgreSQL instance (or use the Docker Compose `postgres` service: `docker compose up postgres -d`).
3. Add your connection string to `appsettings.Development.json`:

```json
{
  "ConnectionStrings": {
    "Default": "Host=localhost;Database=dotnet_api_db;Username=postgres;Password=postgres"
  }
}
```

4. Run the application:

```bash
dotnet run --project dotnet-core-api-w-postgres
```

5. Open the Swagger UI at `http://localhost:5109/swagger/index.html`.

### Running Migrations

```bash
# Add a new migration
dotnet ef migrations add <MigrationName> --project dotnet-core-api-w-postgres

# Apply migrations to the database
dotnet ef database update --project dotnet-core-api-w-postgres
```

### Using as a Template

To use this as the base for a new project, clone the repository and do a global find-and-replace of `dotnet-core-api-w-postgres` and `dotnet_core_api_w_postgres` with your new project name. Update the `SwaggerDoc` title in `Program.cs` to reflect the new API name.
