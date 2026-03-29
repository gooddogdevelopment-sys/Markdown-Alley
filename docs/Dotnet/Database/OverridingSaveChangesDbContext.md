# Setting Values on Save in DbContext
In entity framework, you can set the save changes values on global entities

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
                    // Prevent CreatedAt from being overwritten
                    entry.Property(e => e.CreatedAt).IsModified = false;
                    break;
            }
        }

        return base.SaveChangesAsync(ct);
    }
```