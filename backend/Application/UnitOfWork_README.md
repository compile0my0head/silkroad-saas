# Unit of Work Pattern - Implementation Guide

## -- Overview

The Unit of Work pattern has been implemented to manage database transactions across multiple repositories. It ensures data consistency by grouping multiple operations into a single transaction.

---

## --- Structure

### Files Created:
1. **`Application/Common/Interfaces/IUnitOfWork.cs`** - Interface definition
2. **`Infrastructure/Persistence/UnitOfWork.cs`** - Implementation
3. **`Application/Services/ProductService.cs`** - Example: Simple CRUD operations
4. **`Application/Services/OrderServiceExample.cs`** - Example: Complex multi-repository transactions

---

## -- Key Concepts

### What is Unit of Work-
- **Groups multiple repository operations** into a single transaction
- **Ensures atomicity** - all operations succeed or all fail together
- **Single SaveChanges()** call at the end
- **Manages DbContext lifecycle**

### When to Use-
- - **Multi-repository operations** (e.g., creating order + order items)
- - **Complex business transactions** (e.g., transfer money between accounts)
- - **Ensuring data consistency** across related entities
- - Simple single-entity CRUD (repository alone is fine)

---

## -- Usage Examples

### Example 1: Simple Operation (Single Repository)

```csharp
public class ProductService : IProductService
{
    private readonly IUnitOfWork _unitOfWork;

    public ProductService(IUnitOfWork unitOfWork)
    {
        _unitOfWork = unitOfWork;
    }

    public async Task<ProductDto> CreateAsync(CreateProductRequest request, CancellationToken cancellationToken)
    {
        // Create entity
        var product = new Product { /* properties */ };

        // Add through repository
        await _unitOfWork.Products.AddAsync(product, cancellationToken);

        // Save changes (EF Core handles transaction automatically)
        await _unitOfWork.SaveChangesAsync(cancellationToken);

        return MapToDto(product);
    }
}
```

### Example 2: Complex Transaction (Multiple Repositories)

```csharp
public async Task<OrderDto> CreateOrderWithProductsAsync(...)
{
    // BEGIN explicit transaction
    await _unitOfWork.BeginTransactionAsync(cancellationToken);

    try
    {
        // Step 1: Create order
        var order = await _unitOfWork.Orders.AddAsync(newOrder, cancellationToken);

        // Step 2: Add products
        foreach (var item in products)
        {
            var product = await _unitOfWork.Products.GetByIdAsync(item.ProductId, cancellationToken);
            var orderProduct = new OrderProduct { /* ... */ };
            await _unitOfWork.OrderProducts.AddAsync(orderProduct, cancellationToken);
        }

        // Step 3: Update order total
        order.TotalAmount = CalculateTotal();
        await _unitOfWork.Orders.UpdateAsync(order, cancellationToken);

        // COMMIT - All changes saved together
        await _unitOfWork.CommitTransactionAsync(cancellationToken);

        return MapToDto(order);
    }
    catch
    {
        // ROLLBACK - Nothing is saved
        await _unitOfWork.RollbackTransactionAsync(cancellationToken);
        throw;
    }
}
```

---

## -- Available Repositories in UnitOfWork

```csharp
_unitOfWork.Products
_unitOfWork.Orders
_unitOfWork.OrderProducts
_unitOfWork.Customers
_unitOfWork.Stores
_unitOfWork.Users
_unitOfWork.Campaigns
_unitOfWork.CampaignPosts
_unitOfWork.Teams
_unitOfWork.SocialPlatforms
_unitOfWork.AutomationTasks
_unitOfWork.ChatbotFAQs
```

---

## -- Available Methods

### Transaction Control:
```csharp
// Save changes (automatic transaction)
await _unitOfWork.SaveChangesAsync(cancellationToken);

// Explicit transaction control (for complex scenarios)
await _unitOfWork.BeginTransactionAsync(cancellationToken);
await _unitOfWork.CommitTransactionAsync(cancellationToken);
await _unitOfWork.RollbackTransactionAsync(cancellationToken);
```

### Disposal:
```csharp
// Properly dispose resources
_unitOfWork.Dispose();
```

---

## -- When to Use SaveChangesAsync vs Explicit Transactions

### Use `SaveChangesAsync()` for:
- - Single entity operations
- - Simple CRUD
- - Operations within same aggregate
- **EF Core automatically wraps in transaction**

```csharp
var product = await _unitOfWork.Products.AddAsync(product);
await _unitOfWork.SaveChangesAsync(); // - Automatic transaction
```

### Use `BeginTransaction/Commit/Rollback` for:
- - Multiple unrelated entities
- - Complex multi-step workflows
- - Need to rollback on business rule violations
- - Custom error handling per step

```csharp
await _unitOfWork.BeginTransactionAsync();
try {
    // Multiple operations
    await _unitOfWork.CommitTransactionAsync();
} catch {
    await _unitOfWork.RollbackTransactionAsync();
}
```

---

## -- Migration Guide for Existing Services

### Before (Direct Repository Injection):
```csharp
public class ProductService : IProductService
{
    private readonly IProductRepository _productRepository;

    public ProductService(IProductRepository productRepository)
    {
        _productRepository = productRepository;
    }

    public async Task CreateAsync(...)
    {
        await _productRepository.AddAsync(product);
        // No explicit SaveChanges - repository does it
    }
}
```

### After (Using Unit of Work):
```csharp
public class ProductService : IProductService
{
    private readonly IUnitOfWork _unitOfWork;

    public ProductService(IUnitOfWork unitOfWork)
    {
        _unitOfWork = unitOfWork;
    }

    public async Task CreateAsync(...)
    {
        await _unitOfWork.Products.AddAsync(product);
        await _unitOfWork.SaveChangesAsync(); // - Explicit save
    }
}
```

---

## -- Important Notes

### 1. Repository SaveChanges
-- **Current repositories call `SaveChangesAsync()` internally**

You need to **update repositories** to NOT call SaveChanges:

```csharp
// OLD (in repository)
public async Task<Product> AddAsync(Product product)
{
    _context.Products.Add(product);
    await _context.SaveChangesAsync(); // - Remove this
    return product;
}

// NEW (repository doesn't save)
public async Task<Product> AddAsync(Product product)
{
    _context.Products.Add(product);
    return product; // Let UnitOfWork handle SaveChanges
}
```

### 2. Lazy Initialization
Repositories are created **only when accessed** for better performance.

### 3. DbContext Lifecycle
- UnitOfWork manages **single DbContext** instance
- All repositories share the **same context**
- Disposed automatically by DI container

---

## - Next Steps

1. **Update remaining services** to use `IUnitOfWork`
2. **Refactor repositories** to remove internal `SaveChangesAsync()` calls
3. **Add validation** before SaveChanges in services
4. **Implement complex transactions** where needed (e.g., Campaign + Posts)
5. **Add logging** around transaction boundaries

---

## -- Additional Resources

- [Unit of Work Pattern - Martin Fowler](https://martinfowler.com/eaaCatalog/unitOfWork.html)
- [EF Core Transactions](https://learn.microsoft.com/en-us/ef/core/saving/transactions)
- [Repository Pattern Best Practices](https://learn.microsoft.com/en-us/dotnet/architecture/microservices/microservice-ddd-cqrs-patterns/infrastructure-persistence-layer-design)

---

**Created by:** GitHub Copilot  
**Date:** 2025-01-04
