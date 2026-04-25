# Frontend Integration Guide
## Business Manager API - Complete Reference

**Base URL**: `http://localhost:5000` (adjust for your environment)  
**API Version**: .NET 8  
**Authentication**: JWT Bearer Token

---

## -- Table of Contents

1. [Authentication & Authorization](#authentication--authorization)
2. [Global vs Store-Scoped Endpoints](#global-vs-store-scoped-endpoints)
3. [API Endpoints Reference](#api-endpoints-reference)
4. [Data Models & Enums](#data-models--enums)
5. [Database Constraints & Cascades](#database-constraints--cascades)
6. [Business Flow](#business-flow)
7. [Error Handling](#error-handling)

---

## -- Authentication & Authorization

### JWT Token Management

After successful login/registration, you receive:
```json
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "user": { "id": "guid", "email": "...", "fullName": "..." }
}
```

**Store this token in localStorage/sessionStorage:**
```javascript
localStorage.setItem('authToken', response.token);
```

**Include in all authenticated requests:**
```javascript
headers: {
  'Authorization': `Bearer ${localStorage.getItem('authToken')}`,
  'Content-Type': 'application/json'
}
```

---

## -- Global vs Store-Scoped Endpoints

### GLOBAL Endpoints (NO X-Store-ID header)
- **Authentication**: `/api/auth/*`
- **User Profile**: `/api/users/*`
- **Store Management**: `/api/stores/*`
- **My Teams**: `/api/teams/my`
- **Available Platforms**: `/api/social-platforms/available-platforms`

### STORE-SCOPED Endpoints (REQUIRE X-Store-ID header)
All other endpoints require the store context header:

```javascript
headers: {
  'Authorization': `Bearer ${token}`,
  'X-Store-ID': `${localStorage.getItem('currentStoreId')}`,
  'Content-Type': 'application/json'
}
```

**CRITICAL**: The frontend MUST store and pass the selected store ID for store-scoped operations.

---

## -- API Endpoints Reference

### 1. Authentication (`/api/auth`) - GLOBAL

#### POST `/api/auth/register`
**Purpose**: Register new user account  
**Auth**: None  
**Headers**: `Content-Type: application/json`

**Request Body**:
```json
{
  "email": "user@example.com",
  "password": "SecurePass123!",
  "fullName": "John Doe",
  "phoneNumber": "+1234567890"
}
```

**Response 200**:
```json
{
  "token": "jwt-token-string",
  "user": {
    "id": "guid",
    "email": "user@example.com",
    "fullName": "John Doe",
    "phoneNumber": "+1234567890",
    "ownedStoresCount": 0
  }
}
```

---

#### POST `/api/auth/login`
**Purpose**: Login existing user  
**Auth**: None  
**Headers**: `Content-Type: application/json`

**Request Body**:
```json
{
  "email": "user@example.com",
  "password": "SecurePass123!"
}
```

**Response 200**: Same as register response  
**Response 401**: Invalid credentials  
**Response 404**: User not found

---

#### POST `/api/auth/logout`
**Purpose**: Logout (invalidate token client-side)  
**Auth**: Required  
**Headers**: `Authorization: Bearer {token}`

**Response 200**:
```json
{
  "message": "Logged out successfully"
}
```

---

### 2. User Management (`/api/users`) - GLOBAL

#### GET `/api/users/me`
**Purpose**: Get current authenticated user profile  
**Auth**: Required  
**Headers**: `Authorization: Bearer {token}`

**Response 200**:
```json
{
  "id": "guid",
  "email": "user@example.com",
  "fullName": "John Doe",
  "phoneNumber": "+1234567890",
  "ownedStoresCount": 2
}
```

---

#### GET `/api/users/{userId}`
**Purpose**: Get user by ID  
**Auth**: Required  
**URL Params**: `userId` (guid)

**Response 200**: UserDto (same structure as `/me`)  
**Response 404**: User not found

---

#### GET `/api/users/by-email/{email}`
**Purpose**: Get user by email  
**Auth**: Required  
**URL Params**: `email` (string)

**Response 200**: UserDto  
**Response 404**: User not found

---

#### PUT `/api/users/{userId}`
**Purpose**: Update user profile  
**Auth**: Required

**Request Body**:
```json
{
  "fullName": "Jane Doe",
  "phoneNumber": "+0987654321"
}
```

**Response 200**: Updated UserDto  
**Response 404**: User not found

---

#### DELETE `/api/users/{userId}`
**Purpose**: Delete user account  
**Auth**: Required  
**URL Params**: `userId` (guid)

**Response 204**: No content (success)  
**Response 404**: User not found

---

### 3. Store Management (`/api/stores`) - GLOBAL

#### GET `/api/stores/my`
**Purpose**: Get all stores user has access to (owned + team member)  
**Auth**: Required  
**Headers**: `Authorization: Bearer {token}`

**Response 200**:
```json
[
  {
    "id": "guid",
    "storeName": "My Store",
    "storeDescription": "Store description",
    "storeAddress": "123 Main St",
    "ownerUserId": "guid",
    "createdAt": "2024-01-01T00:00:00Z"
  }
]
```

**-- Frontend Action**: After login, call this endpoint to populate store selection dropdown. Store selected store ID in localStorage:
```javascript
localStorage.setItem('currentStoreId', selectedStore.id);
```

---

#### GET `/api/stores/{storeId}`
**Purpose**: Get store details by ID  
**Auth**: Not required (but recommended)  
**URL Params**: `storeId` (guid)

**Response 200**: StoreDto  
**Response 404**: Store not found

---

#### POST `/api/stores`
**Purpose**: Create new store  
**Auth**: Required  
**Headers**: `Authorization: Bearer {token}`

**Request Body**:
```json
{
  "storeName": "New Store",
  "storeDescription": "Description",
  "storeAddress": "456 Market St"
}
```

**Response 201**:
```json
{
  "id": "guid",
  "storeName": "New Store",
  "storeDescription": "Description",
  "storeAddress": "456 Market St",
  "ownerUserId": "current-user-guid",
  "createdAt": "2024-01-15T10:30:00Z"
}
```

---

#### PUT `/api/stores/{storeId}`
**Purpose**: Update store details  
**Auth**: Required (must be owner)  
**URL Params**: `storeId` (guid)

**Request Body**:
```json
{
  "storeName": "Updated Store Name",
  "storeDescription": "Updated description",
  "storeAddress": "Updated address"
}
```

**Response 200**: Updated StoreDto  
**Response 404**: Store not found  
**Response 403**: Unauthorized (not owner)

---

#### DELETE `/api/stores/{storeId}`
**Purpose**: Delete store  
**Auth**: Required (must be owner)  
**URL Params**: `storeId` (guid)

**Response 204**: No content (success)  
**Response 404**: Store not found  
**Response 400**: Cannot delete (has dependencies)

---

### 4. Team Management (`/api/teams`) - Mixed

#### GET `/api/teams/my` - GLOBAL
**Purpose**: Get all teams user is part of  
**Auth**: Required  
**Headers**: `Authorization: Bearer {token}`

**Response 200**:
```json
[
  {
    "id": "guid",
    "teamName": "Marketing Team",
    "description": "Team description",
    "storeId": "store-guid",
    "storeName": "My Store",
    "memberCount": 5,
    "createdAt": "2024-01-01T00:00:00Z"
  }
]
```

---

#### GET `/api/teams` - STORE-SCOPED
**Purpose**: Get all teams in current store  
**Auth**: Required  
**Headers**: 
```
Authorization: Bearer {token}
X-Store-ID: {storeId}
```

**Response 200**: Array of TeamDto

---

#### GET `/api/teams/{teamId}` - GLOBAL
**Purpose**: Get team details by ID  
**Auth**: Required  
**URL Params**: `teamId` (guid)

**Response 200**: TeamDto  
**Response 404**: Team not found

---

#### POST `/api/teams` - STORE-SCOPED
**Purpose**: Create new team  
**Auth**: Required  
**Headers**: `X-Store-ID: {storeId}`

**Request Body**:
```json
{
  "teamName": "Marketing Team",
  "description": "Handles marketing campaigns"
}
```

**Response 201**: Created TeamDto

---

#### PUT `/api/teams/{teamId}` - STORE-SCOPED
**Purpose**: Update team details  
**Auth**: Required  
**URL Params**: `teamId` (guid)

**Request Body**:
```json
{
  "teamName": "Updated Team Name",
  "description": "Updated description"
}
```

**Response 200**: Updated TeamDto  
**Response 404**: Team not found

---

#### DELETE `/api/teams/{teamId}` - STORE-SCOPED
**Purpose**: Delete team  
**Auth**: Required  
**URL Params**: `teamId` (guid)

**Response 204**: Success  
**Response 404**: Team not found

---

#### GET `/api/teams/{teamId}/members` - STORE-SCOPED
**Purpose**: Get all members of a team  
**Auth**: Required  
**URL Params**: `teamId` (guid)

**Response 200**:
```json
[
  {
    "teamId": "guid",
    "userId": "guid",
    "userName": "John Doe",
    "userEmail": "john@example.com",
    "role": "Owner",
    "joinedAt": "2024-01-01T00:00:00Z"
  }
]
```

**Team Roles**: `Owner`, `Moderator`, `Member`

---

#### POST `/api/teams/{teamId}/members` - STORE-SCOPED
**Purpose**: Add member to team  
**Auth**: Required  
**URL Params**: `teamId` (guid)

**Request Body**:
```json
{
  "userId": "user-guid",
  "role": "Member"
}
```

**Valid Roles**: `Owner` | `Moderator` | `Member`

**Response 201**: TeamMemberDto  
**Response 404**: Team or user not found  
**Response 400**: User already in team

---

#### DELETE `/api/teams/{teamId}/members/{userId}` - STORE-SCOPED
**Purpose**: Remove member from team  
**Auth**: Required  
**URL Params**: 
- `teamId` (guid)
- `userId` (guid)

**Response 204**: Success  
**Response 404**: Team member not found

---

### 5. Product Management (`/api/products`) - STORE-SCOPED

#### GET `/api/products`
**Purpose**: Get all products with optional filtering  
**Auth**: Required  
**Headers**: `X-Store-ID: {storeId}`  
**Query Params**: `inStockOnly` (bool, optional)

**Example**: `GET /api/products-inStockOnly=true`

**Response 200**:
```json
{
  "products": [
    {
      "id": 1,
      "productName": "Laptop",
      "productDescription": "High performance laptop",
      "price": 999.99,
      "currency": "USD",
      "inStock": true,
      "storeId": 1,
      "createdAt": "2024-01-01T00:00:00Z"
    }
  ],
  "totalCount": 1,
  "message": "Products retrieved successfully"
}
```

---

#### GET `/api/products/{productId}`
**Purpose**: Get product by ID  
**Auth**: Required  
**Headers**: `X-Store-ID: {storeId}`  
**URL Params**: `productId` (int)

**Response 200**: ProductDto  
**Response 404**: Product not found

---

#### POST `/api/products`
**Purpose**: Create new product  
**Auth**: Required  
**Headers**: `X-Store-ID: {storeId}`

**Request Body**:
```json
{
  "productName": "Laptop",
  "productDescription": "High performance laptop",
  "price": 999.99,
  "currency": "USD",
  "inStock": true
}
```

**Response 201**: Created ProductDto

---

#### PUT `/api/products/{productId}`
**Purpose**: Update product  
**Auth**: Required  
**Headers**: `X-Store-ID: {storeId}`  
**URL Params**: `productId` (int)

**Request Body**: (all fields optional)
```json
{
  "productName": "Updated Laptop",
  "productDescription": "Updated description",
  "price": 899.99,
  "inStock": false
}
```

**Response 200**: Updated ProductDto  
**Response 404**: Product not found

---

#### DELETE `/api/products/{productId}`
**Purpose**: Delete product  
**Auth**: Required  
**Headers**: `X-Store-ID: {storeId}`  
**URL Params**: `productId` (int)

**Response 204**: Success  
**Response 404**: Product not found  
**Response 400**: Product has dependencies (used in campaigns/orders)

---

### 6. Customer Management (`/api/customers`) - STORE-SCOPED

#### GET `/api/customers`
**Purpose**: Get all customers  
**Auth**: Required  
**Headers**: `X-Store-ID: {storeId}`

**Response 200**:
```json
[
  {
    "id": "guid",
    "customerName": "Alice Johnson",
    "email": "alice@example.com",
    "phoneNumber": "+1234567890",
    "address": "789 Customer Ave",
    "storeId": 1,
    "createdAt": "2024-01-01T00:00:00Z"
  }
]
```

---

#### GET `/api/customers/{customerId}`
**Purpose**: Get customer by ID  
**Auth**: Required  
**Headers**: `X-Store-ID: {storeId}`  
**URL Params**: `customerId` (guid)

**Response 200**: CustomerDto  
**Response 404**: Customer not found

---

#### POST `/api/customers`
**Purpose**: Create new customer  
**Auth**: Required  
**Headers**: `X-Store-ID: {storeId}`

**Request Body**:
```json
{
  "customerName": "Alice Johnson",
  "email": "alice@example.com",
  "phoneNumber": "+1234567890",
  "address": "789 Customer Ave"
}
```

**Response 201**: Created CustomerDto

---

#### PUT `/api/customers/{customerId}`
**Purpose**: Update customer  
**Auth**: Required  
**Headers**: `X-Store-ID: {storeId}`  
**URL Params**: `customerId` (guid)

**Request Body**: (all fields optional)
```json
{
  "customerName": "Alice Smith",
  "email": "alice.smith@example.com"
}
```

**Response 200**: Updated CustomerDto  
**Response 404**: Customer not found

---

#### DELETE `/api/customers/{customerId}`
**Purpose**: Delete customer  
**Auth**: Required  
**Headers**: `X-Store-ID: {storeId}`  
**URL Params**: `customerId` (guid)

**Response 204**: Success  
**Response 404**: Customer not found  
**Response 400**: Customer has orders (cannot delete)

---

### 7. Order Management (`/api/orders`) - STORE-SCOPED

#### GET `/api/orders`
**Purpose**: Get all orders with optional status filter  
**Auth**: Required  
**Headers**: `X-Store-ID: {storeId}`  
**Query Params**: `status` (optional) - Filter by OrderStatus enum value

**Example**: `GET /api/orders-status=Pending`

**Response 200**:
```json
{
  "orders": [
    {
      "id": "guid",
      "orderDate": "2024-01-15T10:00:00Z",
      "totalPrice": 1599.99,
      "status": "Pending",
      "statusDisplayName": "Pending",
      "customerId": "customer-guid",
      "customerName": "Alice Johnson",
      "storeId": 1,
      "createdAt": "2024-01-15T10:00:00Z"
    }
  ],
  "totalCount": 1,
  "message": "Orders retrieved successfully"
}
```

**Order Statuses**: 
- `Pending` - Default for new orders, waiting for admin approval
- `Accepted` - Order accepted by admin
- `Shipped` - Order shipped to customer
- `Delivered` - Order delivered to customer
- `Rejected` - Order rejected by admin
- `Cancelled` - Order cancelled
- `Refunded` - Order refunded

**-- Note**: Status values are stored as **strings** in the database (not integers).

**Example Query**:
```
GET /api/orders-status=Pending
GET /api/orders-status=Accepted
GET /api/orders-status=Rejected
```
---

#### GET `/api/orders/{orderId}`
**Purpose**: Get order by ID  
**Auth**: Required  
**Headers**: `X-Store-ID: {storeId}`  
**URL Params**: `orderId` (guid)

**Response 200**: OrderDto  
**Response 404**: Order not found

---

#### GET `/api/orders/by-customer/{customerId}`
**Purpose**: Get all orders for a customer  
**Auth**: Required  
**Headers**: `X-Store-ID: {storeId}`  
**URL Params**: `customerId` (guid)

**Response 200**: Array of OrderDto

---

#### POST `/api/orders`
**Purpose**: Create new order (defaults to Pending status)  
**Auth**: Required  
**Headers**: `X-Store-ID: {storeId}`

**Request Body**:
```json
{
  "orderDate": "2024-01-15T10:00:00Z",
  "totalPrice": 1599.99,
  "customerId": "customer-guid"
}
```

**Response 201**: Created OrderDto (Status will be Pending)

---

#### PUT `/api/orders/{orderId}`
**Purpose**: Update order  
**Auth**: Required  
**Headers**: `X-Store-ID: {storeId}`  
**URL Params**: `orderId` (guid)

**Request Body**: (all fields optional)
```json
{
  "totalPrice": 1499.99
}
```

**Response 200**: Updated OrderDto  
**Response 404**: Order not found

---

#### DELETE `/api/orders/{orderId}`
**Purpose**: Delete order  
**Auth**: Required  
**Headers**: `X-Store-ID: {storeId}`  
**URL Params**: `orderId` (guid)

**Response 204**: Success  
**Response 404**: Order not found

---

#### PUT `/api/orders/{orderId}/accept`
**Purpose**: Accept a pending order (Admin action)  
**Auth**: Required  
**Headers**: `X-Store-ID: {storeId}`  
**URL Params**: `orderId` (guid)

**Business Rules**:
- Order must exist
- Order status must be Pending
- Status changes from Pending - Accepted
- Order is preserved (not deleted)

**Response 200**: Updated OrderDto with Status = Accepted  
**Response 404**: Order not found  
**Response 400**: Order is not in Pending status

**Example Error**:
```json
{
  "message": "Cannot accept order. Order status is 'Accepted'. Only orders with 'Pending' status can be accepted."
}
```

---

#### PUT `/api/orders/{orderId}/reject`
**Purpose**: Reject a pending order (Admin action)  
**Auth**: Required  
**Headers**: `X-Store-ID: {storeId}`  
**URL Params**: `orderId` (guid)

**Business Rules**:
- Order must exist
- Order status must be Pending
- Status changes from Pending - Rejected
- Order is preserved (not deleted) for record-keeping

**Response 200**: Updated OrderDto with Status = Rejected  
**Response 404**: Order not found  
**Response 400**: Order is not in Pending status

**Example Error**:
```json
{
  "message": "Cannot reject order. Order status is 'Shipped'. Only orders with 'Pending' status can be rejected."
}
```

**-- Frontend Implementation**:
```javascript
// Accept order
const acceptOrder = async (orderId) => {
  const response = await fetch(`/api/orders/${orderId}/accept`, {
    method: 'PUT',
    headers: {
      'Authorization': `Bearer ${token}`,
      'X-Store-ID': storeId
    }
  });
  return await response.json();
};

// Reject order
const rejectOrder = async (orderId) => {
  const response = await fetch(`/api/orders/${orderId}/reject`, {
    method: 'PUT',
    headers: {
      'Authorization': `Bearer ${token}`,
      'X-Store-ID': storeId
    }
  });
  return await response.json();
};

// Get pending orders
const getPendingOrders = async () => {
  const response = await fetch('/api/orders-status=Pending', {
    headers: {
      'Authorization': `Bearer ${token}`,
      'X-Store-ID': storeId
    }
  });
  return await response.json();
};
```

---

### 8. Campaign Management (`/api/campaigns`) - STORE-SCOPED

#### GET `/api/campaigns`
**Purpose**: Get all campaigns  
**Auth**: Required  
**Headers**: `X-Store-ID: {storeId}`

**Response 200**:
```json
[
  {
    "id": "guid",
    "campaignName": "Summer Sale 2024",
    "campaignDescription": "Promotional campaign for summer",
    "campaignStage": "Draft",
    "budget": 5000.00,
    "startDate": "2024-06-01T00:00:00Z",
    "endDate": "2024-08-31T23:59:59Z",
    "assignedProductId": 1,
    "assignedProductName": "Laptop",
    "createdByUserId": "user-guid",
    "createdByUserName": "John Doe",
    "storeId": 1,
    "createdAt": "2024-05-01T00:00:00Z",
    "updatedAt": "2024-05-15T10:00:00Z"
  }
]
```

**Campaign Stages**: `Draft` | `InReview` | `Scheduled` | `Ready` | `Published`

---

#### GET `/api/campaigns/{campaignId}`
**Purpose**: Get campaign by ID  
**Auth**: Required  
**Headers**: `X-Store-ID: {storeId}`  
**URL Params**: `campaignId` (guid)

**Response 200**: CampaignDto  
**Response 404**: Campaign not found

---

#### POST `/api/campaigns`
**Purpose**: Create new campaign  
**Auth**: Required  
**Headers**: `X-Store-ID: {storeId}`

**Request Body**:
```json
{
  "campaignName": "Summer Sale 2024",
  "campaignDescription": "Promotional campaign",
  "campaignStage": "Draft",
  "budget": 5000.00,
  "startDate": "2024-06-01T00:00:00Z",
  "endDate": "2024-08-31T23:59:59Z",
  "assignedProductId": 1
}
```

**Response 201**: Created CampaignDto

---

#### PUT `/api/campaigns/{campaignId}`
**Purpose**: Update campaign  
**Auth**: Required  
**Headers**: `X-Store-ID: {storeId}`  
**URL Params**: `campaignId` (guid)

**Request Body**: (all fields optional)
```json
{
  "campaignName": "Updated Campaign Name",
  "campaignStage": "InReview",
  "budget": 6000.00
}
```

**Response 200**: Updated CampaignDto  
**Response 404**: Campaign not found

---

#### DELETE `/api/campaigns/{campaignId}`
**Purpose**: Delete campaign  
**Auth**: Required  
**Headers**: `X-Store-ID: {storeId}`  
**URL Params**: `campaignId` (guid)

**Response 204**: Success  
**Response 404**: Campaign not found  
**Response 400**: Campaign has posts (cannot delete)

---

### 9. Campaign Posts (`/api/campaign-posts`) - STORE-SCOPED

#### GET `/api/campaign-posts`
**Purpose**: Get all campaign posts  
**Auth**: Required  
**Headers**: `X-Store-ID: {storeId}`

**Response 200**:
```json
[
  {
    "id": "guid",
    "postContent": "Check out our summer sale!",
    "mediaUrl": "https://example.com/image.jpg",
    "scheduledTime": "2024-06-01T09:00:00Z",
    "postStatus": "Scheduled",
    "publishStatus": "Pending",
    "publishedAt": null,
    "lastPublishError": null,
    "campaignId": "campaign-guid",
    "campaignName": "Summer Sale 2024",
    "createdAt": "2024-05-20T10:00:00Z"
  }
]
```

**Post Statuses**: `Draft` | `Scheduled` | `Publishing` | `Published` | `Failed` | `Cancelled`  
**Publish Statuses**: `Pending` | `Publishing` | `Published` | `Failed`

---

#### GET `/api/campaign-posts/{postId}`
**Purpose**: Get campaign post by ID  
**Auth**: Required  
**Headers**: `X-Store-ID: {storeId}`  
**URL Params**: `postId` (guid)

**Response 200**: CampaignPostDto  
**Response 404**: Post not found

---

#### POST `/api/campaign-posts`
**Purpose**: Create new campaign post  
**Auth**: Required  
**Headers**: `X-Store-ID: {storeId}`

**Request Body**:
```json
{
  "postContent": "Check out our summer sale!",
  "mediaUrl": "https://example.com/image.jpg",
  "scheduledTime": "2024-06-01T09:00:00Z",
  "postStatus": "Scheduled",
  "campaignId": "campaign-guid",
  "platformIds": ["platform-guid-1", "platform-guid-2"]
}
```

**-- Important**: `platformIds` array creates platform-specific posts automatically

**Response 201**: Created CampaignPostDto

---

#### PUT `/api/campaign-posts/{postId}`
**Purpose**: Update campaign post  
**Auth**: Required  
**Headers**: `X-Store-ID: {storeId}`  
**URL Params**: `postId` (guid)

**Request Body**: (all fields optional)
```json
{
  "postContent": "Updated content",
  "scheduledTime": "2024-06-02T10:00:00Z",
  "postStatus": "Draft"
}
```

**Response 200**: Updated CampaignPostDto  
**Response 404**: Post not found

---

#### DELETE `/api/campaign-posts/{postId}`
**Purpose**: Delete campaign post  
**Auth**: Required  
**Headers**: `X-Store-ID: {storeId}`  
**URL Params**: `postId` (guid)

**Response 204**: Success  
**Response 404**: Post not found

---

### 10. Social Platforms (`/api/social-platforms`) - Mixed

#### GET `/api/social-platforms/available-platforms` - GLOBAL
**Purpose**: Get list of available social platform types for dropdown  
**Auth**: Required  
**Headers**: `Authorization: Bearer {token}`

**Response 200**:
```json
[
  {
    "value": 0,
    "name": "Facebook",
    "displayName": "Facebook",
    "isOAuthEnabled": true
  },
  {
    "value": 1,
    "name": "Instagram",
    "displayName": "Instagram",
    "isOAuthEnabled": false
  },
  {
    "value": 2,
    "name": "TikTok",
    "displayName": "TikTok",
    "isOAuthEnabled": false
  },
  {
    "value": 3,
    "name": "YouTube",
    "displayName": "YouTube",
    "isOAuthEnabled": false
  }
]
```

**-- Use this for platform selection dropdowns**

---

#### GET `/api/social-platforms/{connectionId}` - STORE-SCOPED
**Purpose**: Get specific platform connection  
**Auth**: Required  
**Headers**: `X-Store-ID: {storeId}`  
**URL Params**: `connectionId` (guid)

**Response 200**:
```json
{
  "id": "guid",
  "platformName": "Facebook",
  "externalPageID": "123456789",
  "pageName": "My Business Page",
  "accessToken": "encrypted-token",
  "isConnected": true,
  "storeId": 1,
  "createdAt": "2024-01-01T00:00:00Z",
  "updatedAt": "2024-01-15T10:00:00Z"
}
```

---

#### POST `/api/social-platforms` - STORE-SCOPED (Testing Only)
**Purpose**: Manually create platform connection (not recommended for production)  
**Auth**: Required  
**Headers**: `X-Store-ID: {storeId}`

**Request Body**:
```json
{
  "platformName": "Facebook",
  "externalPageID": "123456789",
  "pageName": "My Business Page",
  "accessToken": "your-access-token"
}
```

**-- Use OAuth endpoints instead for production**

---

#### POST `/api/social-platforms/facebook/connect` - STORE-SCOPED
**Purpose**: Connect Facebook account via OAuth  
**Auth**: Required  
**Headers**: `X-Store-ID: {storeId}`

**Request Body**:
```json
{
  "accessToken": "facebook-oauth-access-token",
  "externalPageID": "123456789",
  "pageName": "My Business Page"
}
```

**Response 200**: Created SocialPlatformDto  
**Response 400**: Invalid token or connection failed

**-- Frontend Flow**:
1. Initiate Facebook OAuth (get access token from Facebook SDK)
2. Send token to this endpoint
3. Backend validates and stores connection

---

#### POST `/api/social-platforms/instagram/connect` - STORE-SCOPED
**Purpose**: Connect Instagram account (Not Yet Implemented)  
**Auth**: Required  
**Headers**: `X-Store-ID: {storeId}`

**Response 501**: Not Implemented

---

#### PUT `/api/social-platforms/{connectionId}/disconnect` - STORE-SCOPED
**Purpose**: Disconnect platform (soft delete - keeps data)  
**Auth**: Required  
**Headers**: `X-Store-ID: {storeId}`  
**URL Params**: `connectionId` (guid)

**Response 200**: Updated SocialPlatformDto (isConnected = false)  
**Response 404**: Platform not found

---

#### DELETE `/api/social-platforms/{connectionId}` - STORE-SCOPED
**Purpose**: Permanently delete platform connection  
**Auth**: Required  
**Headers**: `X-Store-ID: {storeId}`  
**URL Params**: `connectionId` (guid)

**Response 204**: Success  
**Response 404**: Platform not found

---

### 11. Automation Tasks (`/api/automation-tasks`) - STORE-SCOPED

#### GET `/api/automation-tasks`
**Purpose**: Get all automation tasks  
**Auth**: Required  
**Headers**: `X-Store-ID: {storeId}`

**Response 200**:
```json
[
  {
    "id": 1,
    "taskName": "Auto-respond to comments",
    "taskType": "AutoResponse",
    "taskDescription": "Automatically respond to customer comments",
    "isActive": true,
    "lastRunDate": "2024-01-15T10:00:00Z",
    "relatedCampaignPostId": "post-guid",
    "storeId": 1,
    "createdAt": "2024-01-01T00:00:00Z"
  }
]
```

**Task Types**: `AutoResponse` | `Scheduled` | `Triggered`

---

#### GET `/api/automation-tasks/{taskId}`
**Purpose**: Get task by ID  
**Auth**: Required  
**Headers**: `X-Store-ID: {storeId}`  
**URL Params**: `taskId` (int)

**Response 200**: AutomationTaskDto  
**Response 404**: Task not found

---

#### POST `/api/automation-tasks`
**Purpose**: Create new automation task  
**Auth**: Required  
**Headers**: `X-Store-ID: {storeId}`

**Request Body**:
```json
{
  "taskName": "Auto-respond to comments",
  "taskType": "AutoResponse",
  "taskDescription": "Automatically respond",
  "isActive": true,
  "relatedCampaignPostId": "post-guid"
}
```

**Response 201**: Created AutomationTaskDto

---

#### PUT `/api/automation-tasks/{taskId}`
**Purpose**: Update automation task  
**Auth**: Required  
**Headers**: `X-Store-ID: {storeId}`  
**URL Params**: `taskId` (int)

**Request Body**: (all fields optional)
```json
{
  "taskName": "Updated task name",
  "isActive": false
}
```

**Response 200**: Updated AutomationTaskDto  
**Response 404**: Task not found

---

#### DELETE `/api/automation-tasks/{taskId}`
**Purpose**: Delete automation task  
**Auth**: Required  
**Headers**: `X-Store-ID: {storeId}`  
**URL Params**: `taskId` (int)

**Response 204**: Success  
**Response 404**: Task not found

---

### 12. Chatbot FAQ (`/api/chatbot-faq`) - STORE-SCOPED

#### GET `/api/chatbot-faq`
**Purpose**: Get all FAQs  
**Auth**: Required  
**Headers**: `X-Store-ID: {storeId}`

**Response 200**:
```json
[
  {
    "id": 1,
    "question": "What are your business hours-",
    "answer": "We're open Monday-Friday, 9 AM - 5 PM",
    "messageType": "Text",
    "storeId": 1,
    "createdAt": "2024-01-01T00:00:00Z"
  }
]
```

**Message Types**: `Text` | `Image` | `Video` | `File`

---

#### GET `/api/chatbot-faq/{faqId}`
**Purpose**: Get FAQ by ID  
**Auth**: Required  
**Headers**: `X-Store-ID: {storeId}`  
**URL Params**: `faqId` (int)

**Response 200**: ChatbotFAQDto  
**Response 404**: FAQ not found

---

#### POST `/api/chatbot-faq`
**Purpose**: Create new FAQ  
**Auth**: Required  
**Headers**: `X-Store-ID: {storeId}`

**Request Body**:
```json
{
  "question": "What are your business hours-",
  "answer": "We're open Monday-Friday, 9 AM - 5 PM",
  "messageType": "Text"
}
```

**Response 201**: Created ChatbotFAQDto

---

#### PUT `/api/chatbot-faq/{faqId}`
**Purpose**: Update FAQ  
**Auth**: Required  
**Headers**: `X-Store-ID: {storeId}`  
**URL Params**: `faqId` (int)

**Request Body**: (all fields optional)
```json
{
  "question": "Updated question-",
  "answer": "Updated answer"
}
```

**Response 200**: Updated ChatbotFAQDto  
**Response 404**: FAQ not found

---

#### DELETE `/api/chatbot-faq/{faqId}`
**Purpose**: Delete FAQ  
**Auth**: Required  
**Headers**: `X-Store-ID: {storeId}`  
**URL Params**: `faqId` (int)

**Response 204**: Success  
**Response 404**: FAQ not found

---

## -- Data Models & Enums

### Enums Reference

#### OrderStatus
```typescript
enum OrderStatus {
  Pending = "Pending",       // Order created, waiting for admin approval
  Accepted = "Accepted",     // Order accepted by admin
  Shipped = "Shipped",       // Order shipped to customer
  Delivered = "Delivered",   // Order delivered to customer
  Rejected = "Rejected",     // Order rejected by admin
  Cancelled = "Cancelled",   // Order cancelled (by customer or admin)
  Refunded = "Refunded"      // Order refunded
}
```

#### CampaignStage
```typescript
enum CampaignStage {
  Draft = "Draft",
  InReview = "InReview",
  Scheduled = "Scheduled",
  Ready = "Ready",
  Published = "Published"
}
```

#### PostStatus
```typescript
enum PostStatus {
  Draft = "Draft",
  Scheduled = "Scheduled",
  Publishing = "Publishing",
  Published = "Published",
  Failed = "Failed",
  Cancelled = "Cancelled"
}
```

#### PublishStatus
```typescript
enum PublishStatus {
  Pending = "Pending",
  Publishing = "Publishing",
  Published = "Published",
  Failed = "Failed"
}
```

#### PlatformName
```typescript
enum PlatformName {
  Facebook = "Facebook",
  Instagram = "Instagram",
  TikTok = "TikTok",
  YouTube = "YouTube"
}
```

#### TeamRole
```typescript
enum TeamRole {
  Owner = "Owner",
  Moderator = "Moderator",
  Member = "Member"
}
```

#### MessageType (Chatbot)
```typescript
enum MessageType {
  Text = "Text",
  Image = "Image",
  Video = "Video",
  File = "File"
}
```

**-- Important**: All enums are stored as **strings** in the database, not integers. Use the enum names directly in API requests.

---

## -- Database Constraints & Cascades

### Critical Foreign Key Relationships

#### Store Deletion
**Cascade Deletes** (when store is deleted, these are also deleted):
- - Teams
- - Products
- - Campaigns
- - Campaign Posts
- - Customers
- - Orders
- - Social Platforms
- - Automation Tasks
- - Chatbot FAQs

**-- Frontend Warning**: Deleting a store will permanently remove ALL associated data. Show confirmation dialog.

---

#### Campaign Deletion
**Cascade Deletes**:
- - Campaign Posts (all posts in campaign)
- - Campaign Post Platforms (platform-specific posts)

**Blocked if**:
- Campaign has active scheduled posts

**-- Frontend**: Check if campaign has posts before allowing deletion

---

#### Product Deletion
**Blocked if**:
- - Product is assigned to active campaigns
- - Product is in existing orders

**-- Frontend**: Show error message and list blocking dependencies

---

#### Customer Deletion
**Blocked if**:
- - Customer has orders

**-- Frontend**: Show orders count and prevent deletion if > 0

---

#### Team Deletion
**Cascade Deletes**:
- - Team Members (removes all memberships)

---

#### Campaign Post Deletion
**Cascade Deletes**:
- - Campaign Post Platforms (removes all platform-specific versions)
- - Related Automation Tasks

---

#### Social Platform Deletion
**Blocked if**:
- - Platform has published posts

**-- Frontend**: Use "Disconnect" instead of "Delete" to preserve data

---

## -- Business Flow

### 1. Initial Setup Flow

```
1. User Registration/Login
   -
2. Create Store (or select existing)
   -
3. Store ID saved to localStorage
   -
4. All subsequent requests include X-Store-ID header
```

**-- Frontend Implementation**:
```javascript
// After login
const response = await fetch('/api/auth/login', {
  method: 'POST',
  body: JSON.stringify(credentials)
});
const { token, user } = await response.json();
localStorage.setItem('authToken', token);

// Get user's stores
const storesResponse = await fetch('/api/stores/my', {
  headers: { 'Authorization': `Bearer ${token}` }
});
const stores = await storesResponse.json();

// User selects store
localStorage.setItem('currentStoreId', stores[0].id);
```

---

### 2. Campaign Creation Flow

```
1. Create/Select Product (assign to campaign)
   -
2. Create Campaign (link to product)
   -
3. Connect Social Platforms (Facebook OAuth)
   -
4. Create Campaign Posts (assign platforms)
   -
5. Schedule Posts (set scheduledTime)
   -
6. Background Job publishes at scheduled time
```

**Campaign Stages Progression**:
```
Draft - InReview - Scheduled - Ready - Published
```

**-- Frontend**: Update campaign stage as user completes steps

---

### 3. Social Media Publishing Flow

```
1. User connects Facebook via OAuth
   -
2. Create Campaign Post with platformIds
   -
3. Backend creates CampaignPostPlatform entries
   -
4. Hangfire background job runs every 1 minute
   -
5. Job finds posts with scheduledTime <= now
   -
6. Publishes to each platform
   -
7. Updates PublishStatus (Published/Failed)
```

**Post Status Transitions**:
```
Draft - Scheduled - Publishing - Published/Failed
```

**Publish Status Transitions**:
```
Pending - Publishing - Published/Failed
```

---

### 4. Team Collaboration Flow

```
1. Store Owner creates Team
   -
2. Owner invites Users by email/userId
   -
3. Assign Role (Owner/Moderator/Member)
   -
4. Team members gain access to store resources
```

**-- Frontend**: Implement role-based UI permissions

---

### 5. Order Management Flow

```
1. Create Customer
   -
2. Create Order (link to customer)
   -
3. Add Products to Order (via OrderProducts)
   -
4. Update Order Status (Pending - Processing - Shipped - Delivered)
```

---

## -- Error Handling

### Standard Error Responses

#### 400 Bad Request
```json
{
  "message": "Validation error message",
  "errors": {
    "fieldName": ["Error detail"]
  }
}
```

#### 401 Unauthorized
```json
{
  "message": "Unauthorized. Please login."
}
```

#### 403 Forbidden
```json
{
  "message": "You don't have permission to access this resource."
}
```

#### 404 Not Found
```json
{
  "message": "Resource with ID 'xyz' not found."
}
```

#### 500 Internal Server Error
```json
{
  "message": "An error occurred: [error details]"
}
```

---

### Frontend Error Handling Pattern

```javascript
async function apiRequest(url, options) {
  try {
    const response = await fetch(url, {
      ...options,
      headers: {
        'Authorization': `Bearer ${localStorage.getItem('authToken')}`,
        'X-Store-ID': localStorage.getItem('currentStoreId'),
        'Content-Type': 'application/json',
        ...options.headers
      }
    });

    if (!response.ok) {
      const error = await response.json();
      
      if (response.status === 401) {
        // Redirect to login
        window.location.href = '/login';
      }
      
      throw new Error(error.message || 'Request failed');
    }

    return await response.json();
  } catch (error) {
    console.error('API Error:', error);
    throw error;
  }
}
```

---

## -- Important Notes

### 1. Store Context Management
**CRITICAL**: Always include `X-Store-ID` header for store-scoped endpoints.

```javascript
// Good Practice: Create axios/fetch interceptor
const api = axios.create({
  baseURL: 'http://localhost:5000',
  headers: {
    'Content-Type': 'application/json'
  }
});

api.interceptors.request.use(config => {
  const token = localStorage.getItem('authToken');
  const storeId = localStorage.getItem('currentStoreId');
  
  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }
  
  if (storeId && !config.url.includes('/api/auth') && !config.url.includes('/api/stores')) {
    config.headers['X-Store-ID'] = storeId;
  }
  
  return config;
});
```

---

### 2. Date/Time Handling
All dates are in **UTC** format. Frontend should convert to local timezone.

```javascript
const localDate = new Date(utcDateString).toLocaleString();
```

---

### 3. Pagination & Filtering
Current implementation returns all results. Future versions may implement pagination.

---

### 4. File Upload
Not yet implemented. `mediaUrl` fields expect URLs (hosted externally or uploaded separately).

---

### 5. Background Jobs
Publishing runs every 1 minute via Hangfire. Posts scheduled within past minute will be published.

---

### 6. OAuth Integration
Only Facebook OAuth is fully implemented. Instagram/TikTok/YouTube are placeholders.

---

## -- Recommended Frontend Structure

```
src/
--- api/
-   --- auth.js          // Authentication API calls
-   --- stores.js        // Store management
-   --- campaigns.js     // Campaign operations
-   --- posts.js         // Campaign posts
-   --- products.js      // Product management
-   --- customers.js     // Customer management
-   --- orders.js        // Order management
-   --- teams.js         // Team management
-   --- platforms.js     // Social platform connections
-
--- context/
-   --- AuthContext.js   // Auth state (token, user)
-   --- StoreContext.js  // Current store state
-
--- components/
-   --- auth/
-   --- dashboard/
-   --- campaigns/
-   --- products/
-   --- orders/
-   --- settings/
-
--- utils/
    --- apiClient.js     // Axios/fetch wrapper with interceptors
    --- dateUtils.js     // Date formatting
    --- constants.js     // Enums, API URLs
```

---

## -- Key Frontend Features to Implement

1. **Login/Registration**
2. **Store Selection Dropdown** (persistent in header)
3. **Dashboard** (overview of campaigns, orders, analytics)
4. **Product Management** (CRUD operations)
5. **Campaign Creation Wizard**:
   - Step 1: Campaign Details
   - Step 2: Assign Product
   - Step 3: Connect Platforms
   - Step 4: Create Posts
   - Step 5: Schedule & Review
6. **Social Platform Connection** (Facebook OAuth modal)
7. **Campaign Post Scheduler** (calendar view)
8. **Order Management** (with status tracking)
9. **Team Management** (invite, assign roles)
10. **Settings** (store details, user profile)

---

## -- Support & Documentation

- **API Base URL**: Update in your environment config
- **Swagger Documentation**: Available at `/swagger` (if enabled)
- **Authentication**: JWT tokens expire (check backend config)
- **Rate Limiting**: Not currently implemented

---

**End of Guide**

This document covers all production endpoints excluding internal diagnostic/management endpoints.
