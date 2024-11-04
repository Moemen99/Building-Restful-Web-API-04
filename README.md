# Dependency Principles in .NET Core

## Dependency Inversion Principle (DIP)

### Core Concept
> "High-level modules should not depend on low-level modules. Both should depend on abstractions."

```mermaid
graph TD
    A[High Level Module] --> |Wrong| B[Low Level Module]
    C[High Level Module] --> |Correct| D[Abstraction]
    E[Low Level Module] --> |Correct| D
```

### Module Relationships
- A class can be both:
  - High-level module (for classes depending on it)
  - Low-level module (for classes it depends on)

### Real-World Example: .NET Evolution
```mermaid
graph TD
    subgraph "Old Approach"
    A[.NET Framework] --> B[Windows OS]
    end
    
    subgraph "New Approach"
    C[.NET Development] --> D[OS Abstraction]
    E[Windows] --> D
    F[Linux] --> D
    G[macOS] --> D
    end
```

| Aspect | Old .NET Framework | Modern .NET |
|--------|-------------------|-------------|
| OS Dependency | Direct Windows dependency | OS abstraction |
| Platform Support | Windows-only | Cross-platform |
| Flexibility | Limited | High |

## Dependency Injection in .NET Core

### Overview
- Built-in feature in .NET Core
- Previously required third-party solutions in .NET Framework
- Manages system component dependencies

### Service Collection
```csharp
builder.Services.AddControllers();
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();
```

### How DI Works
1. **Registration**: Services are registered in `IServiceCollection`
2. **Resolution**: DI container resolves dependencies at runtime
3. **Injection**: Dependencies are automatically provided to classes

### Benefits
- Loose coupling
- Better testability
- Modular design
- Simplified dependency management

## Implementation Example Structure
```mermaid
graph TD
    A[Class A] --> |Depends On| D[IService Interface]
    B[Class B Implementation] --> |Implements| D
    C[DI Container] --> |Manages| D
    C --> |Injects| B
```

### Key Principles
1. **Abstraction Dependency**
   - Classes depend on interfaces
   - Implementation details are hidden

2. **Inversion of Control**
   - Object creation handled by DI container
   - Dependencies managed externally

3. **Lifetime Management**
   - Singleton
   - Scoped
   - Transient

## Practical Benefits
1. **Maintainability**
   - Easier to modify implementations
   - Reduced coupling between components

2. **Testability**
   - Easy to mock dependencies
   - Simplified unit testing

3. **Flexibility**
   - Easy to swap implementations
   - Platform-independent code

---
**Note**: Understanding DIP is crucial before implementing DI, as it forms the theoretical foundation for dependency management in modern .NET applications.



# Practical Implementation of Dependency Injection

## Initial Setup (Without DI)

### 1. Controller Creation
```csharp
public class DevelopmentController : ControllerBase
{
    [HttpGet]
    public IActionResult Run()
    {
        var os = new WindowsOsService(); // Direct dependency - Bad practice
        var message = os.RunApp();
        return Ok(message);
    }
}
```

### 2. Service Implementation
```csharp
public class WindowsOsService
{
    public string RunApp()
    {
        return "Running from Windows";
    }
}
```

## Problem Identification
```mermaid
graph TD
    A[DevelopmentController] -->|Direct Dependency| B[WindowsOsService]
    C[High Level Module] -->|Violates DIP| D[Low Level Module]
```

## Implementing DI Pattern

### 1. Interface Definition
```csharp
public interface IOS
{
    string RunApp();
}
```

### 2. Multiple Service Implementations
```csharp
public class WindowsOsService : IOS
{
    public string RunApp() => "Running from Windows";
}

public class MacOsService : IOS
{
    public string RunApp() => "Running from MacOS";
}

public class LinuxService : IOS
{
    public string RunApp() => "Running from Linux";
}
```

### 3. Controller with Constructor Injection
```csharp
public class DevelopmentController : ControllerBase
{
    private readonly IOS _os;

    public DevelopmentController(IOS os)
    {
        _os = os;
    }

    [HttpGet]
    public IActionResult Run()
    {
        var message = _os.RunApp();
        return Ok(message);
    }
}
```

### 4. Service Registration
```csharp
// Program.cs
builder.Services.AddScoped<IOS, WindowsOsService>();
// Or
builder.Services.AddScoped<IOS, MacOsService>();
// Or
builder.Services.AddScoped<IOS, LinuxService>();
```

## Benefits of This Approach

### 1. Dependency Inversion
```mermaid
graph TD
    A[DevelopmentController] -->|Depends On| B[IOS Interface]
    C[WindowsOsService] -->|Implements| B
    D[MacOsService] -->|Implements| B
    E[LinuxService] -->|Implements| B
```

### 2. Advantages Table
| Aspect | Without DI | With DI |
|--------|------------|---------|
| Coupling | Tight | Loose |
| Testability | Difficult | Easy |
| Maintainability | Poor | Good |
| Flexibility | Limited | High |

### 3. Key Benefits
1. **Easy Service Swapping**
   - Change implementation by updating registration
   - No need to modify consumer code

2. **Improved Testing**
   - Easy to mock interfaces
   - Better unit test isolation

3. **Better Maintainability**
   - Centralized service configuration
   - Reduced coupling between components

## Practical Usage

### Swagger Test Flow
1. Launch application
2. Navigate to Swagger UI
3. Locate Development controller
4. Execute GET endpoint
5. Receive 200 status code with OS-specific message

### Service Lifetime Options
```csharp
builder.Services.AddTransient<IOS, WindowsOsService>();  // New instance each time
builder.Services.AddScoped<IOS, WindowsOsService>();     // One instance per request
builder.Services.AddSingleton<IOS, WindowsOsService>();  // One instance for application
```

---
**Note**: This implementation demonstrates how DI helps follow the Dependency Inversion Principle while providing flexibility and maintainability in your application.
