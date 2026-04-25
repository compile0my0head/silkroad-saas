# Business Manager - Multi-Tenant SaaS Platform

## -- Project Overview

**Business Manager** is a comprehensive multi-tenant SaaS platform designed to help businesses manage their online presence, campaigns, products, orders, and social media marketing. The system supports multiple stores per user, team collaboration, and automated campaign scheduling with social media integration.

### Key Features
- -- **Multi-Tenant Store Management** - Users can create and manage multiple business stores
- -- **Team Collaboration** - Store owners can invite team members with role-based access
- -- **Social Media Integration** - Connect Facebook and Instagram pages for automated posting
- -- **Campaign Management** - Create, schedule, and track marketing campaigns
- -- **Automated Scheduling** - Background jobs handle scheduled post publishing via Hangfire
- --- **E-commerce Features** - Product catalog, order management, customer tracking
- -- **Chatbot FAQ Management** - Manage automated responses for customer inquiries
- -- **JWT Authentication** - Secure authentication with refresh token support
- -- **AI-Powered Targeting** - Campaign audience targeting recommendations

---

## --- Architecture

This solution follows **Clean Architecture** principles with clear separation of concerns:

```
BusinessManager/
--- Domain/                    # Core business entities and enums
--- Application/              # Business logic and interfaces
--- Infrastructure/           # Data access and external services
--- Presentation/            # API controllers and middleware
```

### Technology Stack
- **.NET 8** - Latest .NET framework
- **Entity Framework Core** - ORM for database access
- **SQL Server** - Primary database
- **Hangfire** - Background job processing
- **AutoMapper** - Object-to-object mapping
- **JWT** - Authentication and authorization
- **Swagger/OpenAPI** - API documentation

---

## -- Project Structure

### 1-- Domain Layer (`Domain/`)

**Purpose**: Contains core business entities, enums, and domain logic. No dependencies on other layers.

#### Entities (`Domain/Entities/`)
- **User.cs** - User accounts with Identity integration
- **Store.cs** - Business store (multi-tenant root)
- **Team.cs** & **TeamMember.cs** - Team collaboration
- **Product.cs** - Product catalog items
- **Order.cs** & **OrderProduct.cs** - E-commerce orders
- **Customer.cs** - Customer information with PSID tracking
- **Campaign.cs** - Marketing campaigns
- **CampaignPost.cs** - Individual scheduled posts
- **CampaignPostPlatform.cs** - Platform-specific post tracking
- **SocialPlatform.cs** - Connected social media pages
- **AutomationTask.cs** - Scheduled automation tasks
- **ChatbotFAQ.cs** - FAQ responses for chatbot
- **RefreshToken.cs** - JWT refresh token management

#### Enums (`Domain/Enums/`)
- **CampaignStage.cs** - Campaign lifecycle stages
- **CampaignStatus.cs** - Campaign status values
- **PublishStatus.cs** - Post publishing states (Pending, Publishing, Published, Failed)
- **PlatformName.cs** - Supported social platforms
- **TeamRole.cs** - Team member roles
- **TaskType.cs** - Automation task types
- **MessageType.cs** - Chatbot message types
- **OrderStatus.cs** - Order fulfillment status

#### Common (`Domain/Common/`)
- **BaseEntity.cs** - Base class with soft delete support
- **DomainEvent.cs** - Domain event base class
- **IHasDomainEvents.cs** - Domain event interface

---

### 2-- Application Layer (`Application/`)

**Purpose**: Contains business logic, DTOs, interfaces, and services. Orchestrates domain objects using **Service-Repository pattern**.

#### Services (`Application/Services/`)
Core business services implementing use cases:
- **AuthService.cs** - User authentication and registration
- **StoreService.cs** - Store CRUD operations
- **TeamService.cs** & **TeamMemberService.cs** - Team management
- **ProductService.cs** - Product catalog management
- **OrderService.cs** & **OrderProductService.cs** - Order processing
- **CustomerService.cs** - Customer management
- **CampaignService.cs** - Campaign lifecycle management
- **CampaignPostService.cs** - Post scheduling and management
- **CampaignSchedulingService.cs** - - Background post publishing logic
- **SocialPlatformService.cs** - Social platform connection management
- **AutomationTaskService.cs** - Task automation
- **ChatbotFAQService.cs** - FAQ management
- **ServiceManager.cs** - Service aggregator pattern

#### External Services (`Application/ExternalServices/`)
- **CampaignAutomationService.cs** - Campaign automation workflows
- **TargetingRecommendationService.cs** - AI-powered audience targeting

#### DTOs (`Application/DTOs/`)
Data transfer objects organized by feature:
- `Auth/` - Login, Register, User response DTOs
- `Stores/` - Store DTOs
- `Teams/` - Team and member DTOs
- `Products/` - Product DTOs
- `Orders/` - Order and order product DTOs
- `Customers/` - Customer DTOs
- `Campaigns/` - Campaign DTOs
- `CampaignPosts/` - Campaign post DTOs
- `SocialPlatforms/` - Social platform DTOs
- `AutomationTasks/` - Automation task DTOs
- `ChatbotFAQs/` - FAQ DTOs
- `Users/` - User DTOs

#### Interfaces (`Application/Common/Interfaces/`)

**Repository Interfaces:**
- `IUnitOfWork.cs` - Unit of Work pattern
- `IGenericRepository<T>.cs` - Generic repository pattern
- `IProductRepository.cs`
- `IOrderRepository.cs`
- `ICustomerRepository.cs`
- `IStoreRepository.cs`
- `IUserRepository.cs`
- `ICampaignRepository.cs`
- `ICampaignPostRepository.cs` - - Includes GetDuePostsAsync()
- `ITeamRepository.cs`
- `ISocialPlatformRepository.cs`
- `IAutomationTaskRepository.cs`
- `IChatbotFAQRepository.cs`

**Service Interfaces:**
- `IAuthService.cs`
- `IStoreService.cs`
- `ITeamService.cs`
- `IProductService.cs`
- `IOrderService.cs`
- `ICustomerService.cs`
- `ICampaignService.cs`
- `ICampaignPostService.cs`
- `ICampaignSchedulingService.cs` - - Background scheduling
- `ISocialPlatformService.cs`
- `ISocialPlatformPublisher.cs` - - Platform publishing abstraction
- `IAutomationTaskService.cs`
- `IChatbotFAQService.cs`
- `IServiceManager.cs`

**Infrastructure Interfaces:**
- `ICurrentUserService.cs` - Gets current authenticated user
- `IStoreContext.cs` - Provides current store context from header
- `IStoreAuthorizationService.cs` - Validates user access to stores
- `IJwtService.cs` - JWT token generation
- `IDataSeeding.cs` - Database seeding

#### Configuration (`Application/Common/Configuration/`)
- **JwtOptions.cs** - JWT configuration and token generation

#### Exceptions (`Application/Common/Exceptions/`)
- **NotFoundException.cs** - 404 errors
- **ValidationException.cs** - Validation errors
- **UnAuthorizedException.cs** - 401 errors
- **ForbiddenAccessException.cs** - 403 errors

#### Mapping (`Application/Common/Mapping/`)
- **MappingProfile.cs** - AutoMapper configuration

---

### 3-- Infrastructure Layer (`Infrastructure/`)

**Purpose**: Implements interfaces from Application layer. Handles data persistence, external APIs, and background jobs.

#### Persistence (`Infrastructure/Persistence/`)
- **SaasDbContext.cs** - Main EF Core DbContext with:
  - Global query filters for soft delete
  - Store-scoped filtering via IStoreContext
  - Entity configurations
  - Audit fields (CreatedAt, DeletedAt, etc.)

#### Configurations (`Infrastructure/Persistence/Configurations/`)
Entity-specific EF Core configurations:
- `ProductConfiguration.cs`
- `OrderConfiguration.cs`
- `CustomerConfiguration.cs`
- `CampaignConfiguration.cs`
- `CampaignPostConfiguration.cs`
- `SocialPlatformConfiguration.cs`
- `AutomationTaskConfiguration.cs`
- And more...

#### Migrations (`Infrastructure/Persistence/Migrations/`)
EF Core database migrations including:
- Initial schema creation
- Campaign scheduling fields
- Social platform integration
- Team management
- Soft delete support

#### Repositories (`Infrastructure/Repositories/`)
Implementation of repository interfaces:
- **UnitOfWork.cs** - Coordinates multiple repositories
- **GenericRepository.cs** - Base repository with LINQ support
- **ProductRepository.cs**
- **OrderRepository.cs**
- **CustomerRepository.cs**
- **StoreRepository.cs**
- **UserRepository.cs**
- **CampaignRepository.cs**
- **CampaignPostRepository.cs** - - Implements GetDuePostsAsync()
- **TeamRepository.cs**
- **SocialPlatformRepository.cs**
- **AutomationTaskRepository.cs**
- **ChatbotFAQRepository.cs**

#### Services (`Infrastructure/Services/`)
- **CurrentUserService.cs** - Extracts user from JWT claims
- **StoreContext.cs** - Manages current store context
- **StoreAuthorizationService.cs** - Validates store access
- **DataSeeding.cs** - Seeds initial data from JSON files
- **FacebookPublisher.cs** - - Publishes posts to Facebook Pages

#### Background Jobs (`Infrastructure/BackgroundJobs/`)
- **CampaignSchedulerJob.cs** - - Hangfire job that processes due campaign posts

---

### 4-- Presentation Layer (`Presentation/`)

**Purpose**: ASP.NET Core Web API. Handles HTTP requests, authentication, and API documentation.

#### Controllers (`Presentation/Controllers/`)
RESTful API endpoints:
- **AuthController.cs** - `/api/auth` - Login, register, refresh tokens
- **UsersController.cs** - `/api/users` - User management (global)
- **StoreController.cs** - `/api/stores` - Store CRUD
- **TeamController.cs** - `/api/teams` - Team management (store-scoped)
- **ProductController.cs** - `/api/products` - Product catalog (store-scoped)
- **OrderController.cs** - `/api/orders` - Order management (store-scoped)
- **CustomerController.cs** - `/api/customers` - Customer management (store-scoped)
- **CampaignController.cs** - `/api/campaigns` - Campaign CRUD (store-scoped)
- **CampaignPostController.cs** - `/api/campaign-posts` - Post scheduling (store-scoped)
- **SocialPlatformController.cs** - `/api/social-platforms` - Social media connections
- **AutomationTaskController.cs** - `/api/automation-tasks` - Task automation
- **ChatbotFAQController.cs** - `/api/chatbot-faqs` - FAQ management

#### Middleware (`Presentation/Middleware/`)
- **ExceptionMiddleware.cs** - Global exception handling
- **StoreContextMiddleware.cs** - Extracts `X-Store-ID` header
- **StoreValidationMiddleware.cs** - Validates user has store access
- **SwaggerStoreIdHeaderOperationFilter.cs** - Adds X-Store-ID to Swagger UI
- **HangfireAuthorizationFilter.cs** - Controls Hangfire dashboard access

#### Common (`Presentation/Common/`)
- **ApiResponse.cs** - Standardized API response wrapper

#### Configuration Files
- **Program.cs** - Application startup and middleware pipeline
- **appsettings.json** - Configuration (connection strings, JWT, Facebook/Instagram keys)

---

## -- Campaign Scheduling Flow

### How Automated Post Publishing Works

#### 1. **User Creates Campaign Post**
```
POST /api/campaign-posts
{
  "campaignId": "guid",
  "postCaption": "Check out our new product!",
  "postImageUrl": "https://...",
  "scheduledAt": "2024-12-20T10:00:00Z"
}
```

#### 2. **Database Storage**
- Post saved to `CampaignPosts` table
- `PublishStatus` = `PublishStatus.Pending` (stored as "Pending" string)
- `ScheduledAt` = user's desired publish time
- Associated platforms stored in `CampaignPostPlatforms`

#### 3. **Hangfire Background Job**
- **Frequency**: Runs every 5 minutes (cron: `*/5 * * * *`)
- **Entry Point**: `CampaignSchedulerJob.ExecuteAsync()`
- **Service**: `CampaignSchedulingService.ProcessDueCampaignPostsAsync()`

#### 4. **Post Processing Logic**
```csharp
// Query due posts (uses PublishStatus enum)
var duePosts = await _campaignPostRepository.GetDuePostsAsync(DateTime.UtcNow);

foreach (var post in duePosts)
{
    // Validate campaign is active
    if (!campaign.IsSchedulingEnabled) continue;
    
    // Update status to Publishing (using enum)
    post.PublishStatus = PublishStatus.Publishing.ToString();
    
    // Publish to each platform (Facebook, Instagram, etc.)
    foreach (var platformPost in post.PlatformPosts)
    {
        var publisher = GetPublisher(platformPost.Platform.PlatformName);
        var externalId = await publisher.PublishPostAsync(post, platform);
        
        platformPost.ExternalPostId = externalId;
        platformPost.PublishStatus = PublishStatus.Published.ToString();
    }
    
    // Mark overall post as Published (using enum)
    post.PublishStatus = PublishStatus.Published.ToString();
    post.PublishedAt = DateTime.UtcNow;
}
```

#### 5. **Facebook Publishing**
- **Service**: `FacebookPublisher.cs`
- **API**: Facebook Graph API v18.0
- **Endpoints**:
  - Photos: `/{page-id}/photos` (for posts with images)
  - Feed: `/{page-id}/feed` (for text-only posts)
- **Authentication**: Uses stored page access token

#### 6. **Error Handling**
- Platform-specific errors stored in `CampaignPostPlatforms.ErrorMessage`
- Overall post errors in `CampaignPosts.LastPublishError`
- Failed posts marked with `PublishStatus.Failed` for manual review
- Comprehensive logging via `ILogger`

---

## -- Authentication & Authorization

### Multi-Layered Security

#### 1. **JWT Authentication**
- **Login**: `POST /api/auth/login` - Returns access token + refresh token
- **Token Validation**: Every request validates JWT signature
- **Claims**: UserId, Email extracted from token

#### 2. **Store-Level Authorization**
- **Header**: `X-Store-ID: {guid}` required for store-scoped endpoints
- **Middleware Pipeline**:
  1. `StoreContextMiddleware` - Extracts store ID from header
  2. `AuthenticationMiddleware` - Validates JWT token
  3. `StoreValidationMiddleware` - Checks user belongs to store

#### 3. **Access Control**
Users can access a store if:
- They are the store **owner** (Store.OwnerUserId)
- They are a **team member** (TeamMember record exists)

---

## --- Database Schema

### Key Tables

#### Users & Authentication
- `AspNetUsers` - User accounts (ASP.NET Identity)
- `AspNetRoles`, `AspNetUserRoles` - Role management
- `RefreshTokens` - JWT refresh tokens

#### Multi-Tenancy
- `Stores` - Business stores (tenant root)
- `Teams` - Store teams
- `TeamMembers` - User-to-team associations

#### E-commerce
- `Products` - Product catalog
- `Orders` - Customer orders
- `OrderProducts` - Order line items (many-to-many)
- `Customers` - Customer information

#### Marketing & Campaigns
- `Campaigns` - Marketing campaigns
- `CampaignPosts` - Scheduled social media posts
- `CampaignPostPlatforms` - Platform-specific publishing records
- `SocialPlatforms` - Connected Facebook/Instagram pages
- `AutomationTasks` - Scheduled automation tasks

#### Chatbot
- `ChatbotFAQs` - FAQ question-answer pairs

### Important Indexes
- `CampaignPosts.ScheduledAt` - For efficient due post queries
- `CampaignPosts.PublishStatus` - For status filtering
- `SocialPlatforms.ExternalPageID` - For platform lookups
- `Customers.PSID` - For Facebook Messenger integration

---

## -- Getting Started

### Prerequisites
- .NET 8 SDK
- SQL Server (local or remote)
- Visual Studio 2022 or VS Code

### Setup Steps

1. **Clone Repository**
```bash
git clone <repository-url>
cd BusinessManager
```

2. **Update Connection String**
Edit `Presentation/appsettings.json`:
```json
{
  "ConnectionStrings": {
    "ApplicationSqlConnection": "Server=your-server;Database=BusinessManager;..."
  }
}
```

3. **Apply Migrations**
```bash
cd Infrastructure
dotnet ef database update --startup-project ../Presentation
```

4. **Configure Social Media Apps**
Update `appsettings.json`:
```json
{
  "Facebook": {
    "AppId": "your-facebook-app-id",
    "AppSecret": "your-facebook-app-secret",
    "RedirectUri": "https://localhost:5001/api/social-platforms/facebook/callback"
  }
}
```

5. **Run Application**
```bash
cd Presentation
dotnet run
```

6. **Access Swagger UI**
Navigate to: `https://localhost:5001/swagger`

7. **Access Hangfire Dashboard**
Navigate to: `https://localhost:5001/hangfire`

---

## -- API Workflow Examples

### Complete Campaign Creation Flow

#### Step 1: Register & Login
```http
POST /api/auth/register
{
  "fullName": "John Doe",
  "email": "john@example.com",
  "password": "SecurePass123!"
}

POST /api/auth/login
{
  "email": "john@example.com",
  "password": "SecurePass123!"
}
```
**Response**: `{ "token": "eyJhbGc...", "refreshToken": "..." }`

#### Step 2: Create Store
```http
POST /api/stores
Authorization: Bearer eyJhbGc...
{
  "storeName": "My Fashion Store"
}
```
**Response**: `{ "id": "store-guid", "storeName": "My Fashion Store" }`

#### Step 3: Connect Facebook Page
```http
GET /api/social-platforms/facebook/auth-url-storeId=store-guid
```
User authorizes - Facebook redirects - Page connected

#### Step 4: Create Campaign
```http
POST /api/campaigns
Authorization: Bearer eyJhbGc...
X-Store-ID: store-guid
{
  "campaignName": "Spring Sale 2024",
  "isSchedulingEnabled": true,
  "scheduledStartAt": "2024-03-01T00:00:00Z",
  "scheduledEndAt": "2024-03-31T23:59:59Z"
}
```

#### Step 5: Schedule Post
```http
POST /api/campaign-posts
Authorization: Bearer eyJhbGc...
X-Store-ID: store-guid
{
  "campaignId": "campaign-guid",
  "postCaption": "-- Spring Sale! 50% off all dresses!",
  "postImageUrl": "https://example.com/spring-dress.jpg",
  "scheduledAt": "2024-03-05T10:00:00Z"
}
```

#### Step 6: Automatic Publishing
- Hangfire job runs every 5 minutes
- At 2024-03-05 10:00 UTC, post is published to Facebook
- `PublishStatus` - `PublishStatus.Published` (stored as "Published")
- `PublishedAt` - actual publish time
- `ExternalPostId` - Facebook post ID

---

## -- Testing

### Unit Testing
- Test services in isolation with mocked repositories
- Use xUnit, Moq, or NSubstitute

### Integration Testing
- Test API endpoints with WebApplicationFactory
- Use in-memory database or test database

### Manual Testing
- Use Swagger UI for interactive testing
- Monitor Hangfire dashboard for background jobs
- Check logs for debugging

---

## -- NuGet Packages

### Core Packages
- `Microsoft.EntityFrameworkCore.SqlServer` - EF Core SQL Server provider
- `Microsoft.AspNetCore.Identity.EntityFrameworkCore` - Identity framework
- `Microsoft.AspNetCore.Authentication.JwtBearer` - JWT authentication
- `Hangfire.AspNetCore` - Background job processing
- `Hangfire.SqlServer` - Hangfire SQL Server storage
- `AutoMapper.Extensions.Microsoft.DependencyInjection` - Object mapping
- `Swashbuckle.AspNetCore` - Swagger/OpenAPI

---

## -- Configuration Reference

### appsettings.json Structure
```json
{
  "ConnectionStrings": {
    "ApplicationSqlConnection": "Server=...;Database=...;..."
  },
  "JwtOptions": {
    "SecretKey": "your-256-bit-secret-key",
    "Issuer": "https://localhost:5001",
    "Audience": "https://localhost:5001",
    "DurationInDays": "1"
  },
  "Facebook": {
    "AppId": "your-facebook-app-id",
    "AppSecret": "your-facebook-app-secret",
    "RedirectUri": "https://localhost:5001/api/social-platforms/facebook/callback"
  },
  "Instagram": {
    "AppId": "your-instagram-app-id",
    "AppSecret": "your-instagram-app-secret",
    "RedirectUri": "https://localhost:5001/api/social-platforms/instagram/callback"
  },
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  }
}
```

---

## -- Common Issues & Solutions

### Issue: "Store ID is required"
**Solution**: Include `X-Store-ID` header in store-scoped endpoints

### Issue: Hangfire jobs not running
**Solution**: 
- Check Hangfire dashboard at `/hangfire`
- Verify SQL Server connection
- Check Hangfire tables in database

### Issue: Facebook publishing fails
**Solution**:
- Verify page access token is valid
- Check Facebook App settings
- Review error in `CampaignPostPlatforms.ErrorMessage`

### Issue: Soft-deleted entities still appear
**Solution**: EF Core global query filters should handle this automatically. Check `SaasDbContext.OnModelCreating`

---

## -- Future Enhancements

### Planned Features
- - Instagram Direct Publishing
- - Twitter/X Integration
- - LinkedIn Publishing
- - TikTok API Integration
- - Advanced Analytics Dashboard
- - AI-Generated Post Captions
- - Multi-Language Support
- - WhatsApp Business Integration
- - Email Marketing Campaigns
- - SMS Notifications
- - Calendar View for Scheduled Posts
- - Approval Workflow for Team Posts
- - Post Performance Tracking
- - A/B Testing for Campaigns

---

## -- Team Collaboration Workflow

### For Store Owners
1. Create store
2. Invite team members via email
3. Assign roles (Admin, Editor, Viewer)
4. Monitor team activity

### For Team Members
1. Receive invitation email
2. Accept invitation
3. Access store with assigned permissions
4. Collaborate on campaigns and posts

---

## -- Monitoring & Logging

### Hangfire Dashboard
- View running jobs
- Retry failed jobs
- Monitor job history
- See scheduled jobs

### Application Logs
- Structured logging with `ILogger`
- Log levels: Information, Warning, Error
- Integration with Application Insights (optional)

---

## -- Security Best Practices

### Implemented
- - JWT token authentication
- - Refresh token rotation
- - HTTPS enforcement
- - CORS configuration
- - SQL injection prevention (EF Core parameterized queries)
- - Soft delete for data retention
- - Store-level data isolation

### Recommended for Production
- -- Rate limiting
- -- API key authentication for webhooks
- -- Data encryption at rest
- -- Regular security audits
- -- OWASP Top 10 compliance
- -- Penetration testing

---

## -- Support & Documentation

### Resources
- **API Documentation**: `/swagger` endpoint
- **Hangfire Dashboard**: `/hangfire` endpoint
- **Database Schema**: See EF Core migrations
- **Code Documentation**: XML comments in code

### Contact
For questions or issues, create a GitHub issue or contact the development team.

---

## -- License

[Specify your license here - MIT, Apache 2.0, etc.]

---

## -- Summary for AI Agents

**Key Points:**
1. **Architecture**: Clean Architecture with 4 layers (Domain, Application, Infrastructure, Presentation)
2. **Pattern**: Service-Repository pattern with Unit of Work
3. **Multi-Tenancy**: Store-based with X-Store-ID header for scoping
4. **Authentication**: JWT with refresh tokens
5. **Background Jobs**: Hangfire processes scheduled posts every 5 minutes
6. **Social Media**: Facebook Graph API integration, extensible for Instagram/Twitter
7. **Database**: SQL Server with EF Core, soft delete support
8. **API**: RESTful with Swagger documentation
9. **Entry Points**: 
   - API: `Presentation/Program.cs`
   - Scheduling: `Infrastructure/BackgroundJobs/CampaignSchedulerJob.cs`
   - Publishing: `Infrastructure/Services/FacebookPublisher.cs`

**Critical Files for Campaign Scheduling:**
- `CampaignSchedulingService.cs` - Core scheduling logic
- `CampaignPostRepository.cs` - GetDuePostsAsync() query
- `FacebookPublisher.cs` - Facebook API integration
- `Program.cs` - Hangfire configuration

**Enum Usage:**
- All status values use enums (`PublishStatus`, `PlatformName`, etc.)
- Enums converted to strings when stored in database
- Code always uses enum values, not string literals

**Database Context:**
- Primary: `SaasDbContext.cs`
- Connection String: `appsettings.json` - `ApplicationSqlConnection`

**To Add New Social Platform:**
1. Create `{Platform}Publisher.cs` implementing `ISocialPlatformPublisher`
2. Register in `Infrastructure/DependencyInjection.cs`
3. Add platform enum to `Domain/Enums/PlatformName.cs`
4. Implement OAuth callback in `SocialPlatformController.cs`

---

**Generated**: December 2024  
**Version**: 1.0  
**Framework**: .NET 8  
**Architecture**: Clean Architecture  
**Pattern**: Service-Repository with Unit of Work
