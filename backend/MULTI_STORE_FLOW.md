# Multi-Store Context - Business Flow Documentation

## -- Business Scenario

### User Journey
1. **User creates account** - `/api/auth/register` (NO StoreId)
2. **User signs in** - `/api/auth/login` (NO StoreId) - Receives JWT token
3. **User views accessible stores** - `/api/store/my-stores` (NO StoreId) - Returns owned stores + team member stores
4. **User selects a store** - Frontend stores StoreId in localStorage
5. **User accesses store features** - All requests include `X-Store-ID` header - Backend filters data by StoreId
6. **User exits store** - Frontend clears StoreId from localStorage

## -- Endpoint Classification

### - Non-Store-Scoped Endpoints (NO X-Store-ID header needed)

#### Authentication
- `POST /api/auth/register` - Create account
- `POST /api/auth/login` - Sign in
- `POST /api/auth/refresh` - Refresh token
- `POST /api/auth/logout` - Sign out

#### User Management
- `GET /api/users` - Get all users (admin)
- `GET /api/users/{id}` - Get user details
- `POST /api/users` - Create user
- `PUT /api/users/{id}` - Update user
- `DELETE /api/users/{id}` - Delete user

#### Store Management
- `GET /api/store/my-stores` - Get user's accessible stores (owned + team member)
- `GET /api/store` - Get all stores
- `GET /api/store/{id}` - Get store details
- `POST /api/store` - Create new store
- `PUT /api/store/{id}` - Update store details
- `DELETE /api/store/{id}` - Delete store

### - Store-Scoped Endpoints (X-Store-ID header REQUIRED)

All other endpoints require X-Store-ID header and data is automatically filtered:

#### Products
- `GET /api/product` - Get products (filtered by StoreId)
- `POST /api/product` - Create product in current store

#### Teams
- `GET /api/team` - Get teams (filtered by StoreId)
- `POST /api/team` - Create team in current store

#### Campaigns
- `GET /api/campaigns/store/{storeId}` - Get campaigns for store
- `POST /api/campaigns` - Create campaign in current store

#### Orders
- `GET /api/order` - Get orders (filtered by StoreId)
- `POST /api/order` - Create order in current store

#### Customers
- `GET /api/customer` - Get customers (filtered by StoreId)
- `POST /api/customer` - Create customer in current store

#### Social Platforms
- `GET /api/social-platforms/store/{storeId}` - Get connected platforms
- `POST /api/social-platforms` - Connect platform to current store

#### ChatbotFAQ
- `GET /api/chatbotfaq/store/{storeId}` - Get FAQs for store
- `POST /api/chatbotfaq` - Create FAQ in current store

#### Automation Tasks
- `GET /api/automationtask/store/{storeId}` - Get tasks for store
- `POST /api/automationtask` - Create task in current store

## -- Request Flow

### Non-Store-Scoped Request
```
User - POST /api/auth/login
Headers: Authorization: Bearer {token}
Body: { "email": "...", "password": "..." }
-
No X-Store-ID header - Middleware does nothing
-
AuthController.Login()
-
Returns JWT token
```

### Store-Scoped Request (Valid Store)
```
User - GET /api/product
Headers: 
  - Authorization: Bearer {token}
  - X-Store-ID: 550e8400-e29b-41d4-a716-446655440000 (valid store user has access to)
-
StoreContextMiddleware: Extracts StoreId from header - Sets StoreContext.StoreId
-
Authentication: Validates JWT token - Sets UserId
-
StoreValidationMiddleware: 
  1. Checks store exists - - Found
  2. Checks user belongs to store - - Owner or team member
-
ProductController.GetAllProducts()
-
EF Core applies global filter: WHERE StoreId = '550e8400-...' AND !IsDeleted
-
Returns products for selected store only
```

### Store-Scoped Request (Non-Existent Store)
```
User - GET /api/campaigns/3fa85f64-5717-4562-b3fc-2c963f66afa7
Headers: 
  - Authorization: Bearer {token}
  - X-Store-ID: 3fa85f64-5717-4562-b3fc-2c963f66afa7 (FAKE store ID)
-
StoreContextMiddleware: Extracts StoreId from header - Sets StoreContext.StoreId
-
Authentication: Validates JWT token - Sets UserId
-
StoreValidationMiddleware: 
  1. Checks store exists - - NOT FOUND
-
Returns: 404 Not Found
{
  "error": "Store Not Found",
  "message": "Store with ID 3fa85f64-5717-4562-b3fc-2c963f66afa7 does not exist."
}
```

### Store-Scoped Request (No Access)
```
User - GET /api/product
Headers: 
  - Authorization: Bearer {token}
  - X-Store-ID: 550e8400-e29b-41d4-a716-446655440000 (valid store but user is NOT member)
-
StoreContextMiddleware: Extracts StoreId from header - Sets StoreContext.StoreId
-
Authentication: Validates JWT token - Sets UserId
-
StoreValidationMiddleware: 
  1. Checks store exists - - Found
  2. Checks user belongs to store - - NOT owner, NOT team member
-
Returns: 403 Forbidden
{
  "error": "Forbidden",
  "message": "You do not have access to store 550e8400-.... You must be the store owner or a team member."
}
```

## --- Authorization Rules

### Store Access Validation
User can access a store if:
1. **User is the store owner** (`Store.OwnerUserId == User.Id`)
2. **User is a team member** (exists in `TeamMember` table for any team in that store)

### Implementation
```csharp
// Check if user belongs to store
var hasAccess = await _storeAuthorizationService.UserBelongsToStoreAsync(storeId, userId);
if (!hasAccess)
{
    throw new UnauthorizedAccessException(); // Returns 403 Forbidden
}
```

## -- Frontend Implementation (Angular)

### Login Flow
```typescript
// 1. User logs in
authService.login(email, password).subscribe(response => {
  localStorage.setItem('token', response.token);
  // NO StoreId stored yet
  router.navigate(['/stores']); // Show store selection
});

// 2. Load user's accessible stores
storeService.getMyStores().subscribe(stores => {
  // Display list of owned stores + team member stores
  this.stores = stores;
});

// 3. User selects a store
selectStore(storeId: string) {
  localStorage.setItem('selectedStoreId', storeId);
  router.navigate(['/dashboard']); // Enter store context
}

// 4. HTTP Interceptor adds X-Store-ID to requests
export class StoreContextInterceptor implements HttpInterceptor {
  intercept(req: HttpRequest<any>, next: HttpHandler): Observable<HttpEvent<any>> {
    const storeId = localStorage.getItem('selectedStoreId');
    
    // Skip auth endpoints
    if (req.url.includes('/auth/')) {
      return next.handle(req);
    }
    
    // Skip store management endpoints
    if (req.url.includes('/store/my-stores') || req.url.includes('/store')) {
      return next.handle(req);
    }
    
    // Add X-Store-ID to all other requests
    if (storeId) {
      req = req.clone({
        setHeaders: { 'X-Store-ID': storeId }
      });
    }
    
    return next.handle(req);
  }
}

// 5. User exits store
exitStore() {
  localStorage.removeItem('selectedStoreId');
  router.navigate(['/stores']); // Back to store selection
}
```

## -- Testing in Swagger

### Step 1: Register & Login (NO X-Store-ID)
```
POST /api/auth/register
Body: { "email": "user@example.com", "password": "Password123!", "fullName": "John Doe" }
-
POST /api/auth/login
Body: { "email": "user@example.com", "password": "Password123!" }
-
Copy JWT token - Click "Authorize" - Paste token
```

### Step 2: Create or View Stores (NO X-Store-ID)
```
GET /api/store/my-stores
Returns: [
  { "id": "550e8400-...", "storeName": "My First Store", ... }
]
-
Copy Store ID
```

### Step 3: Access Store Features (WITH X-Store-ID)
```
GET /api/product
Headers: X-Store-ID: 550e8400-e29b-41d4-a716-446655440000
Returns: Products for that store only
```

## -- Important Notes

1. **StoreId is NOT in JWT claims** - It's a UI concern only
2. **StoreId is NOT stored in database** - Only in browser localStorage
3. **Global query filters are automatic** - No manual filtering needed in code
4. **Authorization is enforced** - Users can only access stores they belong to
5. **Non-scoped endpoints work without X-Store-ID** - Auth, user management, store selection

## -- Troubleshooting

### Error: "Invalid Store ID format" (400 Bad Request)
- **Cause**: X-Store-ID header value is not a valid GUID
- **Solution**: Ensure frontend sends valid GUID string (e.g., "550e8400-e29b-41d4-a716-446655440000")

### Error: "Store Not Found" (404 Not Found)
- **Cause**: The StoreId in X-Store-ID header doesn't exist in the database
- **Solution**: 
  1. Verify the store was created: `GET /api/store/my-stores`
  2. Check that you're using a valid Store ID from the response
  3. Ensure store wasn't deleted (soft delete)

### Error: "Forbidden - You do not have access to store" (403 Forbidden)
- **Cause**: User is not the store owner and not a team member
- **Solution**: 
  1. Create a team in that store: `POST /api/team`
  2. Add user to the team: `POST /api/team/{teamId}/members`
  3. Or use a different user who owns the store

### Error: "Campaign with ID {id} not found" (404 Not Found)
- **Cause**: The specific campaign doesn't exist in the current store
- **This is CORRECT** - Store exists and user has access, but the specific resource wasn't found
- **Solution**: Verify the campaign ID is correct for this store

### Error: No data returned for store-scoped endpoint (Empty array)
- **Cause**: Store exists and user has access, but no data exists yet
- **This is CORRECT** - Empty result means no products/campaigns/etc. have been created yet
- **Solution**: Create data using POST endpoints

### Error: "Authentication required for store-scoped operations" (401 Unauthorized)
- **Cause**: JWT token is missing or invalid
- **Solution**: Login first, then add Bearer token to Authorization header

---

**Summary**: Store acts as a "door" to all features. Users authenticate first, select a store, then the frontend sends X-Store-ID with every request. Backend automatically filters data by StoreId using EF Core global query filters.

## -- Error Response Hierarchy

```
Request with X-Store-ID header
-
1. Is GUID valid- 
   NO - 400 Bad Request: "Invalid Store ID format"
   -
2. Is user authenticated-
   NO - 401 Unauthorized: "Authentication required"
   -
3. Does store exist-
   NO - 404 Not Found: "Store Not Found"
   -
4. Does user belong to store-
   NO - 403 Forbidden: "You do not have access to store"
   -
5. Does specific resource exist-
   NO - 404 Not Found: "Campaign/Product/etc. not found"
   -
- SUCCESS - 200 OK: Returns data
```

This ensures proper error messages at each validation layer!
