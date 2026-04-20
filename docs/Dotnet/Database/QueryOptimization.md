# Query Optimization Using Entity Framework

### Only select the fields you need using `.Select()`
```csharp
// AVOID: Pulls all columns
var user = await context.Users.FirstAsync(u => u.Id == id);

// Pulls only what is needed
var user = await context.Users
    .Where(u => u.Id == id)
    .Select(u => new { u.Username, u.Email })
    .FirstAsync();
```

### Avoid Queries in Loops
```csharp
// Avoid
var blogs = context.Blogs.ToList();
foreach (var blog in blogs) {
    var count = blog.Posts.Count(); 
}

// Uses include instead of looping through the blogs
var blogs = context.Blogs.Include(b => b.Posts).ToList();
```

### Use NoTracking for readonly queries. Optimizes memory and CPU usage
```csharp
var report = await context.Products
    .AsNoTracking()
    .Where(p => p.InStock)
    .ToListAsync();
```

### Use Split query when joining multiple tables
```csharp
var store = await context.Stores
    .Include(s => s.Products)
    .Include(s => s.Employees)
    .AsSplitQuery() 
    .FirstOrDefaultAsync(s => s.Id == id);
```

