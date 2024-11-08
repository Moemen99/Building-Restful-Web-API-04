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


# Service Lifetimes in ASP.NET Core Dependency Injection

## Overview
Service lifetime determines how long an instance of a registered service remains available in an application. The dependency injection container manages these lifetimes automatically.

## Types of Service Lifetimes

### 1. Transient (⭐)
```csharp
services.AddTransient<IMyService, MyService>();
```

#### Characteristics
- New instance created each time service is requested
- Default lifetime
- Each injection gets its own instance
- Stateless services

#### Best Used For
- Lightweight services
- Stateless operations
- Services that don't share state
- Logging services

```mermaid
graph TD
    A[Request 1] --> B[Instance 1]
    A --> C[Instance 2]
    A --> D[Instance 3]
    E[Request 2] --> F[Instance 4]
    E --> G[Instance 5]
    E --> H[Instance 6]
```

### 2. Scoped (⚪)
```csharp
services.AddScoped<IMyService, MyService>();
```

#### Characteristics
- One instance per request
- Shared within same HTTP request
- New instance for each new request
- Instance disposed after request completion

#### Best Used For
- Per-request operations
- Database contexts
- Request-specific services

```mermaid
graph TD
    A[Request 1] --> B[Instance 1]
    A --> B
    A --> B
    C[Request 2] --> D[Instance 2]
    C --> D
    C --> D
```

### 3. Singleton (⬛)
```csharp
services.AddSingleton<IMyService, MyService>();
```

#### Characteristics
- Single instance for entire application lifetime
- Shared across all requests
- Created at first request
- Lives until application stops

#### Best Used For
- Application-wide services
- Shared resources
- Configuration services
- Caching services

```mermaid
graph TD
    A[Request 1] --> B[Single Instance]
    C[Request 2] --> B
    D[Request 3] --> B
```

## Lifetime Comparison Table

| Lifetime | Instance Creation | Scope | Best For |
|----------|------------------|-------|----------|
| Transient | Every injection | Per injection | Lightweight, stateless services |
| Scoped | Every request | Per HTTP request | Database contexts, request-specific services |
| Singleton | Once | Application-wide | Configuration, caching, shared resources |

## Performance Considerations

### Transient
- More memory usage
- Fresh instance every time
- No state sharing concerns

### Scoped
- Balanced memory usage
- State shared within request
- Most commonly used

### Singleton
- Minimal memory usage
- Application-wide state sharing
- Requires careful consideration

## Usage Guidelines

1. **Choose Transient When**:
   - Service is lightweight
   - State isolation is critical
   - Each operation needs fresh instance

2. **Choose Scoped When**:
   - Service state should persist for request
   - Database contexts needed
   - Request-level caching required

3. **Choose Singleton When**:
   - Service is expensive to create
   - Application-wide sharing needed
   - State should persist across requests

## Warning ⚠️
```mermaid
graph TD
    A[Singleton Service] -->|Should Not Depend On| B[Scoped Service]
    A -->|Should Not Depend On| C[Transient Service]
```

Careful consideration needed when:
- Using Singleton services
- Sharing state across requests
- Managing disposable resources

---
**Note**: The choice of service lifetime can significantly impact application performance and behavior. Choose based on your specific requirements and use case.



# Service Lifetime Implementation Demo

## 1. Interface Setup

```csharp
// Base interface
public interface IOS
{
    string RunApp();
    string OperationId { get; }
}

// Specialized interfaces for different lifetimes
public interface IOperationTransient : IOS { }
public interface IOperationScoped : IOS { }
public interface IOperationSingleton : IOS { }
```

## 2. Service Implementation

```csharp
public class WindowsOsService : IOperationTransient, IOperationScoped, IOperationSingleton
{
    public string OperationId { get; }

    public WindowsOsService()
    {
        OperationId = Guid.NewGuid().ToString()[..4]; // Take first 4 characters
    }

    public string RunApp()
    {
        return "Running from Windows";
    }
}

public class MacOsService : IOperationTransient, IOperationScoped, IOperationSingleton
{
    public string OperationId { get; }

    public MacOsService()
    {
        OperationId = Guid.NewGuid().ToString(); // Full GUID
    }

    public string RunApp()
    {
        return "Running from MacOS";
    }
}
```

## 3. Service Registration

```csharp
// Program.cs
builder.Services.AddTransient<IOperationTransient, WindowsOsService>();
builder.Services.AddScoped<IOperationScoped, WindowsOsService>();
builder.Services.AddSingleton<IOperationSingleton, WindowsOsService>();
```

## 4. Controller Implementation

```csharp
[ApiController]
public class DevelopmentController : ControllerBase
{
    private readonly ILogger<DevelopmentController> _logger;
    private readonly IOperationTransient _operationTransient;
    private readonly IOperationScoped _operationScoped;
    private readonly IOperationSingleton _operationSingleton;

    public DevelopmentController(
        ILogger<DevelopmentController> logger,
        IOperationTransient operationTransient,
        IOperationScoped operationScoped,
        IOperationSingleton operationSingleton)
    {
        _logger = logger;
        _operationTransient = operationTransient;
        _operationScoped = operationScoped;
        _operationSingleton = operationSingleton;
    }

    [HttpGet]
    public IActionResult Run()
    {
        _logger.LogInformation("Transient {0}", _operationTransient.OperationId);
        _logger.LogWarning("Scoped {0}", _operationScoped.OperationId);
        _logger.LogError("Singleton {0}", _operationSingleton.OperationId);
        return Ok();
    }
}
```

## 5. Middleware Implementation for Testing Scoped Services

```csharp
public class MyMiddleware
{
    private readonly RequestDelegate _next;
    private readonly ILogger _logger;
    private readonly IOperationSingleton _operationSingleton;

    public MyMiddleware(
        RequestDelegate next,
        ILogger<MyMiddleware> logger,
        IOperationSingleton operationSingleton)
    {
        _next = next;
        _logger = logger;
        _operationSingleton = operationSingleton;
    }

    public async Task InvokeAsync(
        HttpContext context,
        IOperationTransient operationTransient,
        IOperationScoped operationScoped)
    {
        _logger.LogInformation("Middleware Transient: {0}", operationTransient.OperationId);
        _logger.LogWarning("Middleware Scoped: {0}", operationScoped.OperationId);
        _logger.LogError("Middleware Singleton: {0}", _operationSingleton.OperationId);
        
        await _next(context);
    }
}

// Register in Program.cs
app.UseMiddleware<MyMiddleware>();
```

## Behavior Analysis

### Request Flow
```mermaid
sequenceDiagram
    Client->>+Middleware: HTTP Request
    Middleware->>+Controller: Forward Request
    Controller->>-Middleware: Response
    Middleware->>-Client: HTTP Response
```

### Lifetime Behavior Table

| Service Type | New Request | Same Request Multiple Points | Application Restart |
|-------------|-------------|----------------------------|-------------------|
| Transient | New ID | New ID | New ID |
| Scoped | New ID | Same ID | New ID |
| Singleton | Same ID | Same ID | New ID |

### Testing Results

1. **First Request**:
   - Middleware and Controller show:
     - Different IDs for Transient
     - Same ID for Scoped
     - Same ID for Singleton

2. **Subsequent Requests**:
   - Transient: New IDs every time
   - Scoped: Same ID within request, new ID between requests
   - Singleton: Same ID until application restart

```mermaid
graph TD
    A[Request] --> B[Middleware]
    B --> C[Controller]
    B --> D[Transient ID: Always New]
    B --> E[Scoped ID: Same per Request]
    B --> F[Singleton ID: Same for App Lifetime]
    C --> G[Transient ID: Always New]
    C --> H[Scoped ID: Same as Middleware]
    C --> I[Singleton ID: Same as Middleware]
```

---
**Note**: This implementation demonstrates how different service lifetimes behave in a real application context, showing the practical differences between Transient, Scoped, and Singleton services.


# Keyed Services in .NET 8

## Overview
Keyed Services allow multiple implementations of the same interface to be registered in the dependency injection container, with each implementation identified by a unique key.

## Traditional vs Keyed Registration

### Traditional Service Registration
```csharp
// Only one implementation possible per interface
builder.Services.AddTransient<IOperationTransient, WindowsOsService>();
```

### Keyed Service Registration
```csharp
// Multiple implementations for the same interface
builder.Services.AddKeyedTransient<IOperationTransient, WindowsOsService>("windows");
builder.Services.AddKeyedTransient<IOperationTransient, MacOsService>("macOs");
```

## Implementation Example

### 1. Service Registration
```csharp
public class Startup
{
    public void ConfigureServices(IServiceCollection services)
    {
        // Register multiple implementations
        services.AddKeyedTransient<IOperationTransient, WindowsOsService>("windows");
        services.AddKeyedTransient<IOperationTransient, MacOsService>("macOs");
        
        // Also available for other lifetimes
        services.AddKeyedScoped<IOperationScoped, WindowsOsService>("windows");
        services.AddKeyedSingleton<IOperationSingleton, WindowsOsService>("windows");
    }
}
```

### 2. Controller Usage
```csharp
public class DevelopmentController : ControllerBase
{
    private readonly ILogger _logger;

    public DevelopmentController(ILogger<DevelopmentController> logger)
    {
        _logger = logger;
    }

    [HttpGet]
    public IActionResult Run(
        [FromKeyedServices(Key = "windows")] IOperationTransient windowsService,
        [FromKeyedServices(Key = "macOs")] IOperationTransient macOsService)
    {
        _logger.LogWarning("Windows {0}", windowsService.OperationId);
        _logger.LogError("MacOs {0}", macOsService.OperationId);
        
        return Ok();
    }
}
```

## Key Features

```mermaid
graph TD
    A[Keyed Services] --> B[Multiple Implementations]
    A --> C[Key-based Resolution]
    A --> D[Lifetime Options]
    B --> E[Same Interface]
    C --> F[FromKeyedServices Attribute]
    D --> G[Transient]
    D --> H[Scoped]
    D --> I[Singleton]
```

### Advantages Table

| Feature | Traditional DI | Keyed Services |
|---------|---------------|----------------|
| Implementations per interface | Single | Multiple |
| Implementation selection | At registration | At injection |
| Runtime switching | Limited | Flexible |
| Configuration complexity | Simple | More detailed |

## Use Cases

1. **Multi-platform Services**
   ```mermaid
   graph LR
       A[IOperationService] --> B[Windows Implementation]
       A --> C[MacOS Implementation]
       A --> D[Linux Implementation]
   ```

2. **Environment-specific Implementations**
   - Development vs Production
   - Different cloud providers
   - Various database connections

3. **Feature Variations**
   - Different authentication providers
   - Multiple payment processors
   - Various logging implementations

## Best Practices

1. **Key Naming**
   - Use consistent naming conventions
   - Make keys descriptive and meaningful
   - Consider using constants for keys

2. **Documentation**
   - Document available keys
   - Explain implementation differences
   - Maintain key registry

3. **Usage**
   - Group related implementations
   - Consider using enums for keys
   - Handle missing implementations

---
**Note**: Keyed Services provide a clean way to manage multiple implementations of the same interface, offering greater flexibility in service resolution while maintaining clean dependency injection practices.
