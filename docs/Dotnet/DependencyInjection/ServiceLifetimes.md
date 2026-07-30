# Dependency Injection and Service Lifetimes

Dependency injection (DI) is the built-in pattern ASP.NET Core uses to supply a class with the services it depends on, rather than having the class construct them itself. Service lifetimes control how long each resolved instance lives and when a new one is created.

The framework configures a service container (an `IServiceCollection` at startup, resolved through an `IServiceProvider` at runtime) so that constructor parameters are filled in automatically. Registering a service means telling the container which concrete type to hand out for a requested type, and with which lifetime.

There are three lifetimes to choose from:

- **Transient** — a new instance is created every time the service is requested. Best for lightweight, stateless services.
- **Scoped** — one instance is shared within a single scope (per HTTP request in ASP.NET Core). Best for request-bound work such as an EF Core `DbContext`.
- **Singleton** — one instance is created and reused for the entire application lifetime. Best for stateless or thread-safe shared state.

For additional reference, see the [Microsoft documentation on dependency injection](https://learn.microsoft.com/en-us/dotnet/core/extensions/dependency-injection).

---

## Registering Services

Services are registered against the `IServiceCollection` exposed by `builder.Services` in `Program.cs`. The common pattern registers an interface mapped to its implementation so consumers depend on the abstraction.

```csharp
var builder = WebApplication.CreateBuilder(args);

// Interface -> implementation
builder.Services.AddScoped<IEmailSender, SmtpEmailSender>();

// Concrete type only
builder.Services.AddSingleton<Clock>();

// Factory when construction needs logic or other services
builder.Services.AddScoped<IOrderService>(sp =>
{
    var repo = sp.GetRequiredService<IOrderRepository>();
    return new OrderService(repo, DateTime.UtcNow);
});

var app = builder.Build();
```

---

## Constructor Injection

The primary way to consume a service is to declare it as a constructor parameter. The container resolves the parameter when it creates the class. This is the preferred approach because dependencies are explicit and the type cannot be constructed without them.

```csharp
public class OrderController : ControllerBase
{
    private readonly IOrderService _orders;

    public OrderController(IOrderService orders)
    {
        _orders = orders;
    }

    [HttpPost]
    public IActionResult Create(OrderRequest request) =>
        Ok(_orders.Place(request));
}
```

---

## Transient

`AddTransient` creates a new instance every time the service is requested. Use it for lightweight, stateless services. Because a fresh object is produced on each resolution, transient services never share state between callers.

```csharp
builder.Services.AddTransient<IGuidProvider, GuidProvider>();

public class GuidProvider : IGuidProvider
{
    public Guid NewId() => Guid.NewGuid();
}
```

Two constructor parameters of the same transient type resolve to two different instances.

---

## Scoped

`AddScoped` creates one instance per scope. In ASP.NET Core, a scope is created per HTTP request, so every service resolved during a single request shares the same instance, and the next request gets a new one. This is the right default for services tied to a unit of work, such as an Entity Framework Core `DbContext`.

```csharp
builder.Services.AddScoped<IOrderRepository, OrderRepository>();

// EF Core registers DbContext as scoped by default
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseNpgsql(builder.Configuration.GetConnectionString("Default")));
```

Another common use is a per-request context object, such as a service that exposes the current user resolved from the incoming request. Registering it as scoped guarantees every service handling the same request sees the same user, while each new request gets a fresh instance.

```csharp
builder.Services.AddHttpContextAccessor();
builder.Services.AddScoped<ICurrentUser, CurrentUser>();

public class CurrentUser : ICurrentUser
{
    public CurrentUser(IHttpContextAccessor accessor)
    {
        var user = accessor.HttpContext?.User;
        UserId = user?.FindFirst("sub")?.Value;
    }

    public string? UserId { get; }
}
```

Outside of a request (for example in a background task), you must create a scope explicitly with `IServiceScopeFactory` before resolving scoped services.

```csharp
using var scope = serviceScopeFactory.CreateScope();
var repo = scope.ServiceProvider.GetRequiredService<IOrderRepository>();
```

---

## Singleton

`AddSingleton` creates a single instance the first time it is requested (or at registration if you supply an instance) and reuses it for the lifetime of the application. Use it for stateless services or shared state that is safe for concurrent access. Singletons must be thread-safe because they are shared across all requests.

```csharp
builder.Services.AddSingleton<IClock, SystemClock>();

// Registering an existing instance
builder.Services.AddSingleton(new MetricsCollector());
```

A common use is an in-memory cache shared across every request. Because it is created once and accessed concurrently, its internal state must be thread-safe — here a `ConcurrentDictionary` provides that guarantee.

```csharp
builder.Services.AddSingleton<ILookupCache, LookupCache>();

public class LookupCache : ILookupCache
{
    private readonly ConcurrentDictionary<string, string> _cache = new();

    public string GetOrAdd(string key, Func<string, string> factory) =>
        _cache.GetOrAdd(key, factory);
}
```

---

## Captive Dependencies

A captive dependency occurs when a longer-lived service holds a reference to a shorter-lived one, keeping it alive past its intended lifetime. The most common mistake is injecting a scoped service (or a `DbContext`) into a singleton — the scoped instance is effectively promoted to a singleton and is shared across requests, which is usually a bug and not thread-safe.

```csharp
// Problematic: singleton captures a scoped DbContext
public class CacheWarmer // registered as singleton
{
    private readonly AppDbContext _db; // scoped — now captive
    public CacheWarmer(AppDbContext db) => _db = db;
}
```

Resolve a fresh scope when a singleton needs scoped work:

```csharp
public class CacheWarmer
{
    private readonly IServiceScopeFactory _scopeFactory;
    public CacheWarmer(IServiceScopeFactory scopeFactory) =>
        _scopeFactory = scopeFactory;

    public void Warm()
    {
        using var scope = _scopeFactory.CreateScope();
        var db = scope.ServiceProvider.GetRequiredService<AppDbContext>();
        // use db within this scope
    }
}
```

In the Development environment the framework validates scopes at startup and throws if a scoped service is resolved from the root provider, which helps catch these errors early.

---

## Choosing a Lifetime

Transient suits stateless helpers where a fresh instance is cheap and sharing is undesirable. Scoped suits anything representing a single request's unit of work, most notably a `DbContext` and the repositories built on it. Singleton suits stateless services or carefully synchronized shared state such as caches, clocks, and configuration holders. When in doubt, prefer scoped for request-bound work and singleton only for genuinely shared, thread-safe state.

---

## See Also

- [DbContext Setup](../Database/DotNetDbContext.md) — registering `AppDbContext` as a scoped service via `AddDbContext`.
- [Global Error Handling](../Middleware/GlobalErrorHandling.md) — `IExceptionHandler` registered through the same service container.
