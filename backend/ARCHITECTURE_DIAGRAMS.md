# Architecture Diagrams and System Design

This document explains how the current backend is organized and how requests move through the system.

It is written as a quick technical walkthrough for reviewers.

---

## 1) System at a Glance

Simple flow:

Facebook -> n8n -> Backend (.NET) -> Database -> Frontend

```mermaid
flowchart LR
  Client[Web Frontend / Admin UI]
  N8N[n8n Workflows]

  subgraph API[ASP.NET Core API]
    MW[Middleware Pipeline]
    CTR[Controllers]
    APP[Application Services]
  end

  subgraph Infra[Infrastructure]
    UOW[Unit of Work + Repositories]
    DB[(SQL Server)]
    BG[Hangfire Jobs]
    EXT[External Integrations]
  end

  Client --> MW --> CTR --> APP --> UOW --> DB
  APP --> EXT
  N8N -->|POST /api/orders/chatbot| CTR
  BG --> APP
```

---

## 2) Layered Architecture

```mermaid
flowchart TB
  subgraph Presentation
    P1[Program.cs]
    P2[Controllers]
    P3[Exception + Store Middleware]
  end

  subgraph Application
    A1[Use-case Services]
    A2[DTOs + Mapping]
    A3[Interfaces: IUnitOfWork, Repositories, Services]
  end

  subgraph Domain
    D1[Entities]
    D2[Enums]
    D3[BaseEntity + Soft Delete]
  end

  subgraph Infrastructure
    I1[EF Core DbContext]
    I2[Repository Implementations]
    I3[UnitOfWork Implementation]
    I4[External Services + Publishers]
  end

  Presentation --> Application
  Application --> Domain
  Infrastructure --> Domain
  Presentation --> Infrastructure
  Application --> Infrastructure
```

### Responsibility split

- **Presentation**: HTTP contract, middleware, auth flow, endpoint exposure.
- **Application**: business rules, orchestration, transaction boundary coordination.
- **Domain**: core business model independent of frameworks.
- **Infrastructure**: persistence and external systems.

---

## 3) Request Pipeline (Store-Scoped APIs)

```mermaid
sequenceDiagram
  participant C as Client
  participant M1 as ExceptionMiddleware
  participant M2 as StoreContextMiddleware
  participant A as AuthN/AuthZ
  participant M3 as StoreValidationMiddleware
  participant CT as Controller
  participant S as Application Service
  participant R as UnitOfWork/Repository
  participant DB as SQL Server

  C->>M1: HTTP request
  M1->>M2: pass through
  M2->>M2: read X-Store-ID
  M2->>A: pass through
  A->>M3: authenticated context
  M3->>M3: validate user-store access
  M3->>CT: continue
  CT->>S: execute application service
  S->>R: fetch data (EF Core)
  R->>DB: EF Core operations
  DB-->>R: result
  R-->>S: entities
  S-->>CT: map to DTO
  CT-->>C: HTTP response
```

---

## 3.1) System Flow

1. Customer sends a message via Facebook.
2. n8n processes the message and prepares a structured order payload.
3. Backend receives the order through `POST /api/orders/chatbot`.
4. The order is stored in SQL Server and linked to the correct store.
5. Admin reviews the new order in the dashboard.

---

## 4) Product Flow with Async Embedding

```mermaid
sequenceDiagram
  participant C as Frontend
  participant PC as ProductController
  participant PS as ProductService
  participant U as UnitOfWork
  participant DB as SQL Server
  participant ES as ProductEmbeddingService
  participant W as n8n Webhook

  C->>PC: POST /api/products + JWT + X-Store-ID
  PC->>PS: CreateAsync(request)
  PS->>PS: inject StoreId from StoreContext
  PS->>U: Products.AddAsync
  PS->>U: SaveChangesAsync
  U->>DB: INSERT Product
  DB-->>U: committed
  U-->>PS: created product
  PS-->>PC: ProductDto
  PC-->>C: 201 Created

  Note over PS,ES: Fire-and-forget embedding call
  PS->>ES: EmbedProductAsync(product)
  ES->>W: POST product payload
```

Why this matters:

- API latency is not blocked by embedding webhook execution.
- Product write path remains resilient even if integration is unavailable.

---

## 5) Chatbot Order Intake (n8n -> API)

```mermaid
sequenceDiagram
  participant N as n8n
  participant OC as OrderController
  participant CS as ChatbotOrderService
  participant U as UnitOfWork
  participant DB as SQL Server

  N->>OC: POST /api/orders/chatbot
  OC->>CS: ProcessChatbotOrderAsync

  CS->>U: SocialPlatforms.GetByPageId
  U->>DB: resolve StoreId from ExternalPageID
  DB-->>U: SocialPlatform

  CS->>U: find/create Customer (PSID/Phone)
  CS->>U: match Products by name
  CS->>U: create Order (Status=Pending)
  CS->>U: create OrderProducts
  CS->>U: SaveChangesAsync

  U->>DB: transaction write
  DB-->>U: committed
  CS-->>OC: Order response
  OC-->>N: 201 Created
```

Design intent:

- n8n handles conversation logic.
- API handles persistence, validation, and order lifecycle initiation.

---

## 6) Scheduled Publishing Architecture

```mermaid
flowchart LR
  H[Hangfire Recurring Job: platform-publisher]
  J[PlatformPublisherJob]
  S[PlatformPublishingService]
  U[UnitOfWork]
  P[ISocialPlatformPublisher Implementations]
  F[Facebook Graph API]

  H --> J --> S --> U
  S --> P --> F
```

Key behavior:

- Processes only due, pending platform posts.
- Marks records as `Publishing` before external calls to reduce duplicate execution risk.
- Updates per-platform and aggregate post status after execution.

---

## 7) Data Isolation and Security Model

```mermaid
flowchart TB
  Req[Incoming Request]
  JWT[JWT Authentication]
  SC[X-Store-ID Context]
  Access[Store Access Validation]
  EF[EF Core Global Query Filters]
  Data[(Tenant-Scoped Data)]

  Req --> JWT --> SC --> Access --> EF --> Data
```

Controls in place:

- JWT-based authentication and authorization.
- Store membership/ownership validation before store-scoped operations.
- EF Core global filters for soft-delete and store scoping.

---

## 8) Core Technical Decisions

1. **Repository + Unit of Work on EF Core**
   - Keeps data access consistent and transaction boundaries explicit.

2. **Middleware-first tenant resolution**
   - Centralizes tenant context instead of repeating logic in each endpoint.

3. **Service-oriented application layer**
   - Business rules remain outside controllers and persistence layer.

4. **Background processing for scheduled work**
   - Time-based publishing runs independently from request/response paths.

5. **Integration boundaries behind interfaces**
   - External platforms are isolated through `ISocialPlatformPublisher` and service abstractions.

---

## 9) What This Demonstrates to Reviewers

- Clear separation of concerns across a real multi-project .NET backend.
- Practical multi-tenant API design with layered enforcement.
- Production-oriented handling of async integrations and scheduled jobs.
- Maintainable structure that supports feature growth without controller bloat.

This is the architecture currently implemented in the repository.
