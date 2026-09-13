# Tukupedia Backend (.NET 8)

Backend untuk Tukupedia menggunakan ASP.NET Core 8 dengan arsitektur layering sederhana.

## 📁 Folder Structure

```
backend/
├── Tukupedia.API/              # Main API project (entry point)
│   ├── Controllers/            # REST API endpoints
│   ├── GraphQL/                # GraphQL schema & resolvers
│   ├── Services/               # Business logic
│   ├── Models/                 # Request/Response DTOs
│   ├── Middleware/             # Custom middleware
│   ├── appsettings.json        # Configuration
│   ├── Program.cs              # Application startup
│   └── Tukupedia.API.csproj    # Project file
│
├── Tukupedia.Core/             # Domain models & interfaces
│   ├── Entities/               # Database entities
│   ├── DTOs/                   # Data Transfer Objects
│   ├── Interfaces/             # Service interfaces
│   └── Tukupedia.Core.csproj
│
├── Tukupedia.Data/             # Data access layer
│   ├── Contexts/               # EF Core DbContext
│   ├── Repositories/           # Repository pattern
│   ├── Migrations/             # EF Core migrations
│   └── Tukupedia.Data.csproj
│
├── Tukupedia.Service/          # Business logic services
│   ├── Services/               # Service implementations
│   └── Tukupedia.Service.csproj
│
└── Tukupedia.sln               # Solution file
```

## 🚀 Quick Start

### Prerequisites
- .NET 8 SDK
- PostgreSQL 15+
- Redis (optional)

### Setup

```bash
# Restore packages
dotnet restore

# Apply migrations
dotnet ef database update --project Tukupedia.Data --startup-project Tukupedia.API

# Run server
dotnet run --project Tukupedia.API

# Server at: http://localhost:5000
# GraphQL: http://localhost:5000/graphql
# API: http://localhost:5000/api
```

## 📦 Dependencies

### Core Packages
- `Microsoft.EntityFrameworkCore` - ORM
- `Microsoft.EntityFrameworkCore.PostgreSQL` - PostgreSQL provider
- `HotChocolate.AspNetCore` - GraphQL server
- `Microsoft.AspNetCore.Identity` - Authentication
- `System.IdentityModel.Tokens.Jwt` - JWT tokens
- `StackExchange.Redis` - Redis client
- `Serilog` - Logging

### Development Packages
- `Microsoft.EntityFrameworkCore.Tools` - EF CLI tools
- `xunit` - Unit testing
- `Moq` - Mocking library

## 🏗️ Architecture

### Layering
```
Controllers (REST) + GraphQL Resolvers
          ↓
    Services (Business Logic)
          ↓
    Repositories (Data Access)
          ↓
    DbContext (Entity Framework)
          ↓
    PostgreSQL Database
```

### Key Files

**Program.cs** - Startup configuration
- Database connection
- Dependency injection
- GraphQL setup
- Middleware registration

**Controllers/** - REST endpoints
- ProductController
- OrderController
- UserController
- AuthController

**GraphQL/** - GraphQL schema
- ProductType, OrderType, etc
- Resolvers (queries, mutations)
- Subscriptions

**Services/** - Business logic
- ProductService
- OrderService
- PaymentService (mock)
- etc

**Repositories/** - Data access
- Product/Order/User repositories
- Unit of Work pattern

## 📝 Example: Creating a Service

```csharp
// Services/ProductService.cs
public class ProductService : IProductService
{
    private readonly IProductRepository _productRepo;
    private readonly ILogger<ProductService> _logger;
    
    public ProductService(IProductRepository productRepo, ILogger<ProductService> logger)
    {
        _productRepo = productRepo;
        _logger = logger;
    }
    
    public async Task<IEnumerable<ProductDTO>> GetProductsAsync(string search, Guid categoryId)
    {
        return await _productRepo.SearchAsync(search, categoryId);
    }
}
```

## 🧪 Testing

```bash
# Run all tests
dotnet test

# Run specific project
dotnet test --project Tukupedia.Tests.Unit

# With coverage
dotnet test /p:CollectCoverage=true
```

## 🔍 Debugging

```bash
# Debug mode
dotnet run --project Tukupedia.API --configuration Debug

# View logs
dotnet run --project Tukupedia.API -- --verbose
```

## 📚 Resources

- [ASP.NET Core Docs](https://docs.microsoft.com/aspnet/core)
- [Entity Framework Core](https://docs.microsoft.com/ef/core)
- [HotChocolate GraphQL](https://chillicream.com/docs/hotchocolate)

## 🤝 Contributing

Follow these guidelines:
- Use dependency injection for all services
- Add logging to important operations
- Write tests for business logic
- Follow C# naming conventions
- Keep methods focused and testable

---

**Lebih lanjut**: Lihat [ARCHITECTURE.md](../docs/ARCHITECTURE.md) dan [SETUP.md](../docs/SETUP.md)
