# Architecture Overview

This backend is a layered .NET 8 API built around clean separation of concerns, with explicit handling for multi-tenant store context.

The design is practical rather than academic: controllers stay thin, business logic lives in application services, and data access is isolated in repositories behind a Unit of Work.

## Solution Structure

- **Presentation**
  - ASP.NET Core Web API entry point and HTTP pipeline.
  - Controllers define endpoints and delegate work to application services.
  - Middleware handles cross-cutting concerns such as exception handling and store context validation.

- **Application**
  - Use-case oriented services (`ProductService`, `OrderService`, `CampaignService`, etc.).
  - DTO contracts and mapping profiles.
  - Interfaces that define boundaries (`IUnitOfWork`, repository interfaces, service interfaces).
  - Business orchestration for publishing workflows.

- **Domain**
  - Core entities (`Store`, `Product`, `Order`, `Campaign`, ...).
  - Enums and base entity model (including soft-delete fields).
  - No dependency on infrastructure frameworks.

- **Infrastructure**
  - Entity Framework Core implementation (`SaasDbContext`) and SQL Server persistence.
  - Repository implementations and Unit of Work implementation.
  - External integrations (social publishers, embedding webhook, current user/store services).
  - Background job host components (Hangfire jobs).

## Runtime Architecture

### Request Pipeline

1. Request enters ASP.NET Core pipeline.
2. `ExceptionMiddleware` catches and normalizes unhandled errors.
3. `StoreContextMiddleware` reads `X-Store-ID` (when present) and stores it in scoped `StoreContext`.
4. JWT authentication and authorization run.
5. `StoreValidationMiddleware` confirms the authenticated user can access the selected store for store-scoped routes.
6. Controller executes and calls application service.
7. Application service uses Unit of Work/repositories and returns DTO response.

### Multi-Tenant Data Isolation

The project uses two layers of tenant protection:

- **Request-level validation**
  - `StoreValidationMiddleware` checks whether the current user owns the store or is a team member.

- **Data-level filtering**
  - `SaasDbContext` applies global query filters that combine:
    - soft delete (`IsDeleted == false`)
    - store scoping (`entity.StoreId == current StoreId` when store context exists)

This means most repository queries stay simple while still respecting tenant boundaries.

## Data Access Pattern

The application uses **Repository + Unit of Work** over EF Core:

- Repositories expose aggregate-specific operations (for example, product search or due campaign posts).
- `IUnitOfWork` provides a consistent access point to repositories and a single `SaveChangesAsync` boundary.
- Services coordinate business operations and persistence in one place.

This keeps controllers lightweight and avoids leaking EF Core concerns into the API layer.

## Example Flow: Products Endpoint

`GET /api/products`

- `ProductController` receives the request.
- `ProductService.GetAllAsync()` executes business rules (like optional `inStockOnly` filtering).
- `UnitOfWork.Products.GetAllAsync()` loads data.
- EF global query filters automatically apply store and soft-delete constraints.
- Service maps entities to DTOs and returns a response model.

For writes (`POST`/`PUT`), `ProductService` also triggers a non-blocking embedding webhook call after persistence.

## Background Processing and Integrations

- **Hangfire** is configured for recurring background work.
- `PlatformPublisherJob` runs on a schedule and delegates publishing logic to `PlatformPublishingService`.
- Platform-specific publishing is abstracted behind `ISocialPlatformPublisher` implementations.
- External HTTP integrations use `IHttpClientFactory`.

This keeps heavy or time-based operations out of synchronous request paths.

## Security and API Concerns

- JWT Bearer authentication with configured issuer/audience/signing key.
- Identity integration via `IdentityDbContext<User, IdentityRole<Guid>, Guid>`.
- Swagger/OpenAPI with bearer security definition.
- Store header support in API docs via operation filter.

## Why This Architecture Works Well

- Clear boundaries between HTTP, business logic, domain model, and persistence.
- Strong tenant isolation strategy for a multi-store SaaS model.
- Predictable and testable service/repository composition.
- Extensible integration model for social publishing and external automation.
- Background processing for long-running or scheduled tasks without blocking API requests.

In short, the system is organized for maintainability and production operations: straightforward request handling, explicit business orchestration, and controlled infrastructure dependencies.