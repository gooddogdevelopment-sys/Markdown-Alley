# Auto Apply Migrations

Apply any pending EF Core migrations automatically at application startup.

Add the following code in `Program.cs`, after `var app = builder.Build();` and before `app.Run();`:

```csharp
using (var scope = app.Services.CreateScope())
{
    var db = scope.ServiceProvider.GetRequiredService<AppDbContext>();
    await db.Database.MigrateAsync();
}
```

## See Also

- [DbContext Setup](DotNetDbContext.md) — setting up `AppDbContext`, dependency injection, and EF Core migrations.
- [Overriding SaveChanges](OverridingSaveChangesDbContext.md) — automatically setting `CreatedAt`/`UpdatedAt` timestamps.
