# BusinessManager - Complete Directory Structure

```
BusinessManager/
-
--- -- Domain/                                    # Core Business Domain Layer
-   --- -- Common/
-   -   --- BaseEntity.cs                        # Base entity with soft delete
-   -   --- DomainEvent.cs                       # Domain events base
-   -   --- IHasDomainEvents.cs                  # Domain events interface
-   -
-   --- -- Entities/                             # Domain Entities
-   -   --- AutomationTask.cs                    # Scheduled automation tasks
-   -   --- Campaign.cs                          # Marketing campaign
-   -   --- CampaignPost.cs                      # - Scheduled social media post
-   -   --- CampaignPostPlatform.cs              # - Platform-specific post record
-   -   --- ChatbotFAQ.cs                        # FAQ for chatbot
-   -   --- Customer.cs                          # Customer entity
-   -   --- Order.cs                             # Order entity
-   -   --- OrderProduct.cs                      # Order line items (many-to-many)
-   -   --- Product.cs                           # Product catalog
-   -   --- RefreshToken.cs                      # JWT refresh tokens
-   -   --- SocialPlatform.cs                    # Connected social pages
-   -   --- Store.cs                             # Multi-tenant store (root)
-   -   --- Team.cs                              # Store team
-   -   --- TeamMember.cs                        # Team membership
-   -   --- User.cs                              # User account (ASP.NET Identity)
-   -
-   --- -- Enums/                                # Domain Enumerations
-       --- AutomationTaskType.cs                # Task types
-       --- CampaignStage.cs                     # Campaign stages
-       --- CampaignState.cs                     # Campaign states
-       --- CampaignStatus.cs                    # Campaign status
-       --- MessageType.cs                       # Chatbot message types
-       --- OrderStatus.cs                       # Order status
-       --- PlatformName.cs                      # - Social platforms (Facebook, Instagram, etc.)
-       --- PostStatus.cs                        # Post statuses
-       --- PublishStatus.cs                     # - Publishing status (Pending, Publishing, Published, Failed)
-       --- TaskType.cs                          # Task types
-       --- TeamRole.cs                          # Team member roles
-
--- -- Application/                               # Application/Business Logic Layer
-   --- -- Common/
-   -   --- -- Configuration/
-   -   -   --- JwtOptions.cs                    # JWT configuration + token generation
-   -   -
-   -   --- -- Exceptions/
-   -   -   --- ForbiddenAccessException.cs      # 403 errors
-   -   -   --- NotFoundException.cs             # 404 errors
-   -   -   --- UnAuthorizedException.cs         # 401 errors
-   -   -   --- ValidationException.cs           # Validation errors
-   -   -
-   -   --- -- Interfaces/                       # Application Interfaces
-   -   -   --- IAuthService.cs                  # Authentication service
-   -   -   --- IAutomationTaskRepository.cs     # Automation task repo
-   -   -   --- IAutomationTaskService.cs        # Automation task service
-   -   -   --- ICampaignPostRepository.cs       # - Campaign post repo (includes GetDuePostsAsync)
-   -   -   --- ICampaignPostService.cs          # Campaign post service
-   -   -   --- ICampaignRepository.cs           # Campaign repository
-   -   -   --- ICampaignSchedulingService.cs    # - Campaign scheduling service
-   -   -   --- ICampaignService.cs              # Campaign service
-   -   -   --- IChatbotFAQRepository.cs         # FAQ repository
-   -   -   --- IChatbotFAQService.cs            # FAQ service
-   -   -   --- ICurrentUserService.cs           # Get current authenticated user
-   -   -   --- ICustomerRepository.cs           # Customer repository
-   -   -   --- ICustomerService.cs              # Customer service
-   -   -   --- IDataSeeding.cs                  # Data seeding interface
-   -   -   --- IGenericRepository.cs            # Generic repository pattern
-   -   -   --- IJwtService.cs                   # JWT token generation
-   -   -   --- ILLMService.cs                   # LLM integration (future)
-   -   -   --- IOrderProductRepository.cs       # Order product repo
-   -   -   --- IOrderProductService.cs          # Order product service
-   -   -   --- IOrderRepository.cs              # Order repository
-   -   -   --- IOrderService.cs                 # Order service
-   -   -   --- IProductRepository.cs            # Product repository
-   -   -   --- IProductService.cs               # Product service
-   -   -   --- IServiceManager.cs               # Service manager pattern
-   -   -   --- ISocialPlatformPublisher.cs      # - Platform publisher abstraction
-   -   -   --- ISocialPlatformRepository.cs     # Social platform repo
-   -   -   --- ISocialPlatformService.cs        # Social platform service
-   -   -   --- IStoreAuthorizationService.cs    # Store access validation
-   -   -   --- IStoreContext.cs                 # Store context provider
-   -   -   --- IStoreRepository.cs              # Store repository
-   -   -   --- IStoreService.cs                 # Store service
-   -   -   --- ITeamMemberService.cs            # Team member service
-   -   -   --- ITeamRepository.cs               # Team repository
-   -   -   --- ITeamService.cs                  # Team service
-   -   -   --- IUnitOfWork.cs                   # Unit of Work pattern
-   -   -   --- IUserRepository.cs               # User repository
-   -   -   --- IUserService.cs                  # User service
-   -   -
-   -   --- -- Mapping/
-   -       --- MappingProfile.cs                # AutoMapper configuration
-   -
-   --- -- DTOs/                                 # Data Transfer Objects
-   -   --- -- Auth/
-   -   -   --- AuthResponseDto.cs               # Login/register response
-   -   -   --- LoginRequestDto.cs               # Login request
-   -   -   --- RegisterRequestDto.cs            # Registration request
-   -   -   --- UserResponseDto.cs               # User response
-   -   --- -- AutomationTasks/
-   -   -   --- AutomationTaskDto.cs
-   -   --- -- CampaignPosts/
-   -   -   --- CampaignPostDto.cs
-   -   --- -- Campaigns/
-   -   -   --- CampaignDto.cs
-   -   --- -- ChatbotFAQ/
-   -   -   --- ChatbotFAQDto.cs
-   -   --- -- ChatbotFAQs/
-   -   -   --- ChatbotFAQDto.cs
-   -   --- -- Customers/
-   -   -   --- CustomerDto.cs
-   -   --- -- OrderProducts/
-   -   -   --- OrderProductDto.cs
-   -   --- -- Orders/
-   -   -   --- OrderDto.cs
-   -   --- -- Products/
-   -   -   --- ProductDto.cs
-   -   --- -- SocialPlatforms/
-   -   -   --- SocialPlatformDto.cs
-   -   --- -- Stores/
-   -   -   --- StoreDto.cs
-   -   --- -- Teams/
-   -   -   --- TeamDto.cs
-   -   --- -- Users/
-   -       --- UserDto.cs
-   -
-   --- -- ExternalServices/                     # External Service Integrations
-   -   --- CampaignAutomationService.cs         # Campaign automation workflows
-   -   --- TargetingRecommendationService.cs    # AI targeting recommendations
-   -
-   --- -- Services/                             # Application Services (Business Logic)
-   -   --- AuthService.cs                       # Authentication logic
-   -   --- AutomationTaskService.cs             # Task automation
-   -   --- CampaignPostService.cs               # Post management
-   -   --- CampaignSchedulingService.cs         # - Post scheduling & publishing
-   -   --- CampaignService.cs                   # Campaign management
-   -   --- ChatbotFAQService.cs                 # FAQ management
-   -   --- CustomerService.cs                   # Customer management
-   -   --- OrderProductService.cs               # Order products
-   -   --- OrderService.cs                      # Order management
-   -   --- ProductService.cs                    # Product catalog
-   -   --- ServiceManager.cs                    # Service aggregator
-   -   --- SocialPlatformService.cs             # Social platform connections
-   -   --- StoreService.cs                      # Store management
-   -   --- TeamMemberService.cs                 # Team member management
-   -   --- TeamService.cs                       # Team management
-   -   --- UserService.cs                       # User management
-   -
-   --- DependencyInjection.cs                   # Application layer DI registration
-
--- -- Infrastructure/                            # Infrastructure/Data Access Layer
-   --- -- BackgroundJobs/                       # Hangfire Background Jobs
-   -   --- CampaignSchedulerJob.cs              # - Recurring job (every 5 min)
-   -
-   --- -- Persistence/                          # Database Context & Migrations
-   -   --- -- Configurations/                   # EF Core Entity Configurations
-   -   -   --- AutomationTaskConfiguration.cs
-   -   -   --- CampaignConfiguration.cs
-   -   -   --- CampaignPostConfiguration.cs
-   -   -   --- CampaignPostPlatformConfiguration.cs
-   -   -   --- ChatbotFAQConfiguration.cs
-   -   -   --- CustomerConfiguration.cs
-   -   -   --- OrderConfiguration.cs
-   -   -   --- OrderProductConfiguration.cs
-   -   -   --- ProductConfiguration.cs
-   -   -   --- RefreshTokenConfiguration.cs
-   -   -   --- SocialPlatformConfiguration.cs
-   -   -   --- StoreConfiguration.cs
-   -   -   --- TeamConfiguration.cs
-   -   -   --- TeamMemberConfiguration.cs
-   -   -   --- UserConfiguration.cs
-   -   -
-   -   --- -- Migrations/                       # EF Core Migrations
-   -   -   --- 20241201_InitialCreate.cs
-   -   -   --- 20241214_AddCampaignSchedulingAndPublishing.cs
-   -   -   --- ...other migrations...
-   -   -
-   -   --- DataSeeding.cs                       # Database seeding from JSON
-   -   --- SaasDbContext.cs                     # - Main DbContext (multi-tenant)
-   -
-   --- -- Repositories/                         # Repository Implementations
-   -   --- AutomationTaskRepository.cs
-   -   --- CampaignPostRepository.cs            # - Includes GetDuePostsAsync
-   -   --- CampaignRepository.cs
-   -   --- ChatbotFAQRepository.cs
-   -   --- CustomerRepository.cs
-   -   --- GenericRepository.cs                 # Base generic repo
-   -   --- OrderProductRepository.cs
-   -   --- OrderRepository.cs
-   -   --- ProductRepository.cs
-   -   --- SocialPlatformRepository.cs
-   -   --- StoreRepository.cs
-   -   --- TeamRepository.cs
-   -   --- UnitOfWork.cs                        # - Coordinates all repositories
-   -   --- UserRepository.cs
-   -
-   --- -- Services/                             # Infrastructure Services
-   -   --- CurrentUserService.cs                # Extracts user from JWT
-   -   --- DataSeeding.cs                       # Seeds database
-   -   --- FacebookPublisher.cs                 # - Facebook API publishing
-   -   --- StoreAuthorizationService.cs         # Store access validation
-   -   --- StoreContext.cs                      # Store context provider
-   -
-   --- DependencyInjection.cs                   # Infrastructure layer DI registration
-
--- -- Presentation/                              # API/Presentation Layer
    --- -- Common/
    -   --- ApiResponse.cs                       # Standardized API responses
    -
    --- -- Controllers/                          # API Controllers
    -   --- AuthController.cs                    # /api/auth (login, register, refresh)
    -   --- AutomationTaskController.cs          # /api/automation-tasks (store-scoped)
    -   --- CampaignController.cs                # /api/campaigns (store-scoped)
    -   --- CampaignPostController.cs            # /api/campaign-posts (store-scoped)
    -   --- ChatbotFAQController.cs              # /api/chatbot-faqs (store-scoped)
    -   --- CustomerController.cs                # /api/customers (store-scoped)
    -   --- OrderController.cs                   # /api/orders (store-scoped)
    -   --- OrderProductController.cs            # /api/order-products (store-scoped)
    -   --- ProductController.cs                 # /api/products (store-scoped)
    -   --- SocialPlatformController.cs          # /api/social-platforms (mixed scope)
    -   --- StoreController.cs                   # /api/stores (global + store-scoped)
    -   --- TeamController.cs                    # /api/teams (store-scoped)
    -   --- UsersController.cs                   # /api/users (global)
    -
    --- -- Middleware/                           # Custom Middleware
    -   --- ExceptionMiddleware.cs               # Global exception handling
    -   --- HangfireAuthorizationFilter.cs       # Hangfire dashboard auth
    -   --- StoreContextMiddleware.cs            # Extracts X-Store-ID header
    -   --- StoreValidationMiddleware.cs         # Validates store access
    -   --- SwaggerStoreIdHeaderOperationFilter.cs # Adds X-Store-ID to Swagger
    -
    --- -- Middlewares/                          # (Duplicate folder - legacy)
    -   --- ExceptionMiddleware.cs
    -   --- StoreContextMiddleware.cs
    -
    --- Program.cs                               # - Application entry point + Hangfire config
    --- appsettings.json                         # - Configuration (DB, JWT, Facebook, etc.)
    --- appsettings.Development.json             # Development-specific config

```

---

## -- Key Files for Campaign Scheduling

### Entry Points
1. **`Program.cs`** - Configures Hangfire recurring job
2. **`CampaignSchedulerJob.cs`** - Hangfire job entry point
3. **`CampaignSchedulingService.cs`** - Core scheduling logic
4. **`FacebookPublisher.cs`** - Facebook API integration

### Critical Interfaces
- **`ICampaignSchedulingService`** - Scheduling service contract
- **`ISocialPlatformPublisher`** - Publisher abstraction
- **`ICampaignPostRepository`** - Includes `GetDuePostsAsync()`

### Database Context
- **`SaasDbContext.cs`** - Multi-tenant DbContext with store filtering

---

## -- Middleware Pipeline Order (Program.cs)

```
1. ExceptionMiddleware         - Global error handling
2. CORS                         - Allow origins
3. StoreContextMiddleware       - Extract X-Store-ID header
4. Authentication               - Validate JWT token
5. Authorization                - Role-based access
6. StoreValidationMiddleware    - Validate user has store access
7. Controllers                  - Process request
```

---

## --- Database Tables (Primary)

### Multi-Tenancy
- `Stores` - Tenant root
- `Teams` / `TeamMembers` - Collaboration

### E-commerce
- `Products` - Catalog
- `Orders` / `OrderProducts` - Orders
- `Customers` - Customer data

### Marketing
- `Campaigns` - Marketing campaigns
- `CampaignPosts` - - Scheduled posts
- `CampaignPostPlatforms` - - Platform-specific publishing
- `SocialPlatforms` - Connected pages
- `AutomationTasks` - Automation rules

### Authentication
- `AspNetUsers` - User accounts
- `RefreshTokens` - JWT refresh tokens

### Chatbot
- `ChatbotFAQs` - FAQ responses

---

## -- Quick Navigation

| Feature | Controller | Service | Repository |
|---------|-----------|---------|------------|
| Authentication | `AuthController.cs` | `AuthService.cs` | `UserRepository.cs` |
| Stores | `StoreController.cs` | `StoreService.cs` | `StoreRepository.cs` |
| Campaigns | `CampaignController.cs` | `CampaignService.cs` | `CampaignRepository.cs` |
| **Scheduling** | - | `CampaignSchedulingService.cs` | `CampaignPostRepository.cs` |
| **Publishing** | - | `FacebookPublisher.cs` | - |
| Products | `ProductController.cs` | `ProductService.cs` | `ProductRepository.cs` |
| Orders | `OrderController.cs` | `OrderService.cs` | `OrderRepository.cs` |
| Teams | `TeamController.cs` | `TeamService.cs` | `TeamRepository.cs` |
| Social Platforms | `SocialPlatformController.cs` | `SocialPlatformService.cs` | `SocialPlatformRepository.cs` |

---

## -- Configuration Files

| File | Purpose |
|------|---------|
| `appsettings.json` | Main configuration (DB, JWT, Facebook keys) |
| `Program.cs` | Startup configuration (DI, middleware, Hangfire) |
| `DependencyInjection.cs` (Application) | Application layer services |
| `DependencyInjection.cs` (Infrastructure) | Infrastructure services + repositories |

---

## -- Notes

- **- Symbol** indicates critical files for campaign scheduling
- **Store-scoped** endpoints require `X-Store-ID` header
- **Global** endpoints don't require store context
- **Hangfire** dashboard accessible at `/hangfire`
- **Swagger** documentation at `/swagger`

---

**Generated**: December 2024  
**Architecture**: Clean Architecture  
**Pattern**: Service-Repository with Unit of Work  
**Framework**: .NET 8
