# Access Modifiers

Access modifiers control the visibility and accessibility of types and their members (fields, properties, methods, etc.). They determine which parts of your codebase can see and interact with a given member, making them a key tool for encapsulation.

.NET provides six access modifiers: `public`, `private`, `protected`, `internal`, `protected internal`, and `private protected`.

For additional reference, see the [Microsoft documentation on access modifiers](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/access-modifiers).

---

## public

Accessible from anywhere — same class, same assembly, or any external assembly.

```csharp
public class Car
{
    public string Make { get; set; }
}

// Accessible from anywhere
var car = new Car();
car.Make = "Toyota";
```

---

## private

Accessible only within the class or struct it is declared in. This is the default for class members if no modifier is specified.

```csharp
public class BankAccount
{
    private decimal _balance;

    public void Deposit(decimal amount) => _balance += amount;
    public decimal GetBalance() => _balance;
}

// _balance is not accessible here
var account = new BankAccount();
account.Deposit(100);
```

---

## protected

Accessible within the class it is declared in and by any class that inherits from it, but not from outside those classes.

```csharp
public class Animal
{
    protected string Name { get; set; }
}

public class Dog : Animal
{
    public void SetName(string name) => Name = name; // OK — inherited
}

// Not accessible here
var dog = new Dog();
// dog.Name = "Rex"; // Error
```

---

## internal

Accessible from anywhere within the same assembly, but not from other assemblies. Useful for members that should be shared across your project but hidden from external consumers.

```csharp
// In MyApp.dll
internal class ConfigLoader
{
    internal string LoadConfig() => "config data";
}

// Accessible anywhere else inside MyApp.dll
var loader = new ConfigLoader();
loader.LoadConfig();

// Not accessible from a different assembly (e.g. a consumer NuGet package)
```

---

## protected internal

Accessible within the same assembly **or** from a derived class in any assembly. It is the union of `protected` and `internal`.

```csharp
public class Logger
{
    protected internal void Log(string message) =>
        Console.WriteLine(message);
}

// Accessible from the same assembly
var logger = new Logger();
logger.Log("Hello");

// Also accessible from a subclass in a different assembly
public class FileLogger : Logger
{
    public void LogToFile(string message) => Log(message); // OK
}
```

---

## private protected

Accessible only within the same class **or** a derived class, and only if both are in the same assembly. It is the intersection of `protected` and `internal`.

```csharp
public class Repository
{
    private protected void ExecuteQuery(string sql)
    {
        // Internal implementation
    }
}

// Only a subclass in the same assembly can access this
public class UserRepository : Repository
{
    public void GetUsers()
    {
        ExecuteQuery("SELECT * FROM Users"); // OK — same assembly + derived
    }
}

// A subclass in a different assembly cannot access ExecuteQuery
```
