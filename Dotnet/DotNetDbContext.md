# Setting Up DbContext
The DbContext is the core class in Entity Framework Core (EF Core). It represents a session with the database and acts as a bridge between your domain entities and the database itself. It is responsible for querying, saving, and managing the entity objects during runtime.

## 1. Required Packages
To get started, you need the core EF Core package, the design tools, and the data provider package for your specific database engine (e.g., PostgreSQL, SQL Server, SQLite).

```sh
dotnet add package Microsoft.EntityFrameworkCore
dotnet add package Microsoft.EntityFrameworkCore.Design
# Example provider: dotnet add package Npgsql.EntityFrameworkCore.PostgreSQL
```
## 2. Defining the Entities
Before configuring the context, you need the data models (entities) that will be mapped to your database tables.

```C#
namespace Infrastructure.Data.Models;

public class ExampleEntity
{
    public int Id { get; set; }
    public string Name { get; set; } = string.Empty;
    public DateTime CreatedAt { get; set; }
}
```
## 3. Creating the AppDbContext Class
Your custom context class must inherit from EF Core's base class. It exposes ```DbSet<TEntity>``` properties for each entity type you want to manage.

```C#
using Microsoft.EntityFrameworkCore;
using Infrastructure.Data.Models;

namespace Infrastructure.Data;

public class AppDbContext(DbContextOptions<AppDbContext> options) : DbContext(options);
{
    // Constructor accepting DbContextOptions, passing configuration to the base class. Uncomment if not using primary constructor
    /*
    public AppDbContext(DbContextOptions<AppDbContext> options) : base(options)
    {
    }
    */
    // DbSets represent the tables in the database
    public DbSet<ExampleEntity> ExampleEntities => Set<ExampleEntity>();

    // Optional: Override OnModelCreating to configure the database schema via the Fluent API
    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        base.OnModelCreating(modelBuilder);

        // Example: Configuring a property constraint
        modelBuilder.Entity<ExampleEntity>()
            .Property(e => e.Name)
            .IsRequired()
            .HasMaxLength(100);
    }
}
```
## 4. Registering in Dependency Injection
In a modern .NET Core web application, you register the context in your Program.cs file using Dependency Injection (DI). This allows the context to be injected into your controllers, services, or repositories.

```C#
using Microsoft.EntityFrameworkCore;
using Infrastructure.Data;

var builder = WebApplication.CreateBuilder(args);

// Fetch the connection string from appsettings.json
var connectionString = builder.Configuration.GetConnectionString("DefaultConnection");

// Register the DbContext with your chosen database provider
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseNpgsql(connectionString)); // Replace UseNpgsql with UseSqlServer, UseSqlite, etc. as needed

var app = builder.Build();

// ... middleware configuration ...

app.Run();
```

## 5. Applying Migrations
Once the entities and the context are set up, use the EF Core tools via the .NET CLI to generate and apply the database schema.

#### Create the initial migration:
```sh
dotnet ef migrations add InitialCreate

dotnet ef migrations add InitialCreate -o Data/Migrations #With output directly specified
```

#### Apply the migration to the database:
```sh
dotnet ef database update
```
