# -- Architecture Diagrams: Product Embedding & Chatbot Orders

## --- System Overview

```
-------------------------------------------------------------------
-                         PRESENTATION LAYER                       -
-------------------------------------------------------------------
-                                                                  -
-  --------------------              -----------------------     -
-  - ProductController-              -  OrderController     -     -
-  -                  -              -                      -     -
-  - POST /products   -              - POST /orders/chatbot -     -
-  - PUT /products/:id-              -   [AllowAnonymous]   -     -
-  --------------------              -----------------------     -
-           -                                    -                 -
--------------------------------------------------------------------
            -                                    -
            -                                    -
-------------------------------------------------------------------
-                        APPLICATION LAYER                         -
-------------------------------------------------------------------
-                                                                  -
-  --------------------              -----------------------     -
-  -  ProductService  -              - ChatbotOrderService -     -
-  -                  -              -                      -     -
-  - CreateAsync()    -              - ProcessChatbotOrder()-     -
-  - UpdateAsync()    -              -                      -     -
-  --------------------              -----------------------     -
-           -                                    -                 -
-           -  -----------------------          -                 -
-           ----IProductEmbeddingServ-          -                 -
-              -                     -          -                 -
-              -  EmbedProductAsync()-          -                 -
-              -----------------------          -                 -
-                                               -                 -
-              ------------------------------------------------- -
-              -         IUnitOfWork             -             - -
-              -                                 -             - -
-              -  ---------------  ------------ -             - -
-              -  -Products     -  -Customers - -             - -
-              -  ---------------  ------------ -             - -
-              -  ---------------  ------------ -             - -
-              -  -Orders       -  -Platforms - -             - -
-              -  ---------------  ------------ -             - -
-              -----------------------------------             - -
-------------------------------------------------------------------
            -                                    -
            -                                    -
-------------------------------------------------------------------
-                      INFRASTRUCTURE LAYER                        -
-------------------------------------------------------------------
-                                                                  -
-  ------------------------         ------------------------     -
-  -ProductEmbeddingService-         -    Repositories       -     -
-  -                       -         -                       -     -
-  - HttpClient           -         - CustomerRepository    -     -
-  - -- POST to n8n       -         - ProductRepository     -     -
-  - -- 10s timeout       -         - SocialPlatformRepo    -     -
-  - -- Error logging     -         - OrderRepository       -     -
-  -------------------------         ------------------------     -
-              -                                 -                 -
-              -                                 -                 -
-  ------------------------         ------------------------     -
-  -  n8n Webhook         -         -   SQL Server DB       -     -
-  -  (External)          -         -                       -     -
-  ------------------------         ------------------------     -
-                                                                  -
-------------------------------------------------------------------
```

---

## -- Product Embedding Flow

```
--------------
-  Frontend  -
-            -
- Creates or -
- Updates    -
- Product    -
--------------
      - POST /api/products
      - Authorization: Bearer {token}
      - X-Store-ID: {storeId}
      -
      -
-------------------
-ProductController-
-------------------
         -
         -
-------------------
- ProductService  -
-                 -
- 1. Map request  -
- 2. Add to DB    -
- 3. SaveChanges()- -------
-------------------        -
         -                 -
         - Fire-and-forget -
         -                 -
----------------------------
- Task.Run(async () =>    --
-   await Embed()         --
- )                       --
----------------------------
         -                 -
         -                 -
------------------------------------------
- ProductEmbeddingService  -             -
-                          -             -
- 1. Build payload         -   Main      -
- 2. POST to n8n webhook   -   Thread    -
- 3. Handle errors         -   Returns   -
- 4. Log result            -   Here      -
------------------------------------------
         -
         -
----------------------------
-   n8n Webhook            -
-   (Vector Embedding)     -
-                          -
-   Stores product data    -
-   for semantic search    -
----------------------------
```

**Key Points**:
- - Non-blocking
- - Failures don't affect product operations
- - Background execution
- - Detailed logging

---

## -- Chatbot Order Flow

```
----------------
-   n8n        -
-   Workflow   -
-              -
- Facebook     -
- Messenger    -
----------------
       - POST /api/orders/chatbot
       - {customer, items, pageId}
       - NO AUTHENTICATION
       -
       -
--------------------
- OrderController  -
- [AllowAnonymous] -
--------------------
         -
         -
-----------------------------------------------
-         ChatbotOrderService                  -
-                                              -
-  ------------------------------------------ -
-  - Step 1: Resolve Store                  - -
-  - pageId - SocialPlatforms.ExternalPageID- -
-  -        - StoreId                        - -
-  ------------------------------------------ -
-                                              -
-  ------------------------------------------ -
-  - Step 2: Find/Create Customer           - -
-  -                                         - -
-  - ---------------------------------------- -
-  - - Try: Find by PSID                   -- -
-  - ---------------------------------------- -
-  -                                         - -
-  - ---------------------------------------- -
-  - - If not found: Find by Phone         -- -
-  - -               Update PSID            -- -
-  - ---------------------------------------- -
-  -                                         - -
-  - ---------------------------------------- -
-  - - If not found: Create New            -- -
-  - -               (name or "Anonymous") -- -
-  - ---------------------------------------- -
-  ------------------------------------------ -
-                                              -
-  ------------------------------------------ -
-  - Step 3: Match Products                 - -
-  -                                         - -
-  - For each item:                          - -
-  -   Search by name (LIKE)                 - -
-  -   If found: Add to order                - -
-  -   If not:   Skip, log warning           - -
-  ------------------------------------------ -
-                                              -
-  ------------------------------------------ -
-  - Step 4: Create Order                   - -
-  -                                         - -
-  - Status: Pending (always)                - -
-  - TotalPrice: Sum of matched products     - -
-  ------------------------------------------ -
-                                              -
-  ------------------------------------------ -
-  - Step 5: Create OrderProducts           - -
-  -                                         - -
-  - For each matched product:               - -
-  -   Create OrderProduct record            - -
-  ------------------------------------------ -
-                                              -
------------------------------------------------
             -
             -
--------------------------------------------------
-              Database                           -
-                                                 -
-  ------------  ---------  ----------------   -
-  -Customers -  -Orders -  -OrderProducts -   -
-  ------------  ---------  ----------------   -
-                                                 -
---------------------------------------------------
```

---

## --- Database Relationship Diagram

```
-----------------------
-   SocialPlatforms   -
-                     -
- � ExternalPageID  ---------
- � StoreId           -     -
- � PlatformName      -     -
-----------------------     -
                            -
                            - Used to
                            - resolve
                            - StoreId
                            -
-----------------------     -
-      Customers      -     -
-                     -     -
- � PSID (indexed)    -     -
- � Phone             -     -
- � CustomerName      -     -
- � BillingAddress    -     -
- � StoreI-------------------
-----------------------
           -
           - 1:N
           -
           -
-----------------------
-       Orders        -
-                     -
- � Status (Pending)  -
- � TotalPrice        -
- � CustomerId        -
- � StoreId           -
-----------------------
           -
           - 1:N
           -
           -
-----------------------       -----------------------
-   OrderProducts     - N:1   -      Products       -
-                     ---------                     -
- � Quantity          -       - � ProductName       -
- � UnitPrice         -       - � ProductPrice      -
- � OrderId           -       - � StoreId           -
- � ProductId         -     -------------------------
-----------------------
```

---

## -- Security & Validation Flow

```
---------------------------------------------------------------
-                    Product Embedding                         -
---------------------------------------------------------------
-                                                              -
-  ----------------                                           -
-  - JWT Token    - - Required                               -
-  ----------------                                           -
-                                                              -
-  ----------------                                           -
-  - X-Store-ID   - - Required                               -
-  ----------------                                           -
-                                                              -
-  ----------------                                           -
-  - Product Data - - Validated via DTOs                     -
-  ----------------                                           -
-                                                              -
---------------------------------------------------------------

---------------------------------------------------------------
-                    Chatbot Orders                            -
---------------------------------------------------------------
-                                                              -
-  ----------------                                           -
-  - Authentication- - None (Public endpoint)                -
-  ----------------                                           -
-                                                              -
-  ----------------                                           -
-  - PageId       - - Must exist in SocialPlatforms          -
-  ----------------                                           -
-                                                              -
-  ----------------                                           -
-  - Customer PSID- - Required                               -
-  ----------------                                           -
-                                                              -
-  ----------------                                           -
-  - Items array  - - Min 1, Quantity > 0                    -
-  ----------------                                           -
-                                                              -
-  ----------------                                           -
-  - Validation   - - DataAnnotations + ModelState           -
-  ----------------                                           -
-                                                              -
---------------------------------------------------------------
```

---

## - Error Handling Flow

```
---------------------------------------------------------------
-                Product Embedding Errors                      -
---------------------------------------------------------------
-                                                              -
-  HTTP Timeout (>10s)                                         -
-      -                                                       -
-      --- Log Warning --- Continue                          -
-                                                              -
-  Network Error                                               -
-      -                                                       -
-      --- Log Error ----- Continue                          -
-                                                              -
-  Webhook Returns 4xx/5xx                                     -
-      -                                                       -
-      --- Log Warning --- Continue                          -
-                                                              -
-  - Product operation ALWAYS succeeds                        -
-                                                              -
---------------------------------------------------------------

---------------------------------------------------------------
-                Chatbot Order Errors                          -
---------------------------------------------------------------
-                                                              -
-  Invalid PageId                                              -
-      -                                                       -
-      --- 400 Bad Request                                    -
-      --- "Page not connected to any store"                  -
-                                                              -
-  Validation Error                                            -
-      -                                                       -
-      --- 400 Bad Request                                    -
-      --- ModelState errors                                  -
-                                                              -
-  Product Not Found                                           -
-      -                                                       -
-      --- Log Warning --- Skip item --- Continue            -
-                                                              -
-  Database Error                                              -
-      -                                                       -
-      --- 500 Internal Server Error                         -
-      --- Error logged                                       -
-                                                              -
---------------------------------------------------------------
```

---

## -- Request/Response Flow

### Product Creation with Embedding

```
Client                Controller           Service              Infrastructure      External
  -                       -                  -                       -               -
  --POST /products--------                  -                       -               -
  - {productData}        -                  -                       -               -
  -                      -                  -                       -               -
  -                      --CreateAsync()----                       -               -
  -                      -                  -                       -               -
  -                      -                  --AddAsync()------------               -
  -                      -                  -                       -               -
  -                      -                  --SaveChanges()---------               -
  -                      -                  -                       -               -
  -                      -                  --Task.Run(Embed)-------               -
  -                      -                  -     (non-blocking)   -               -
  -                      -                  -                       -               -
  -                      ---ProductDto-------                       -               -
  -                      -                  -                       -               -
  ---201 Created----------                  -                       -               -
  - {productDto}         -                  -                       -               -
  -                      -                  -                       -               -
  -                      -                  -                  ---------------      -
  -                      -                  -                  - Background         -
  -                      -                  -                  - Thread             -
  -                      -                  -                  -                    -
  -                      -                  -                  --POST webhook--------
  -                      -                  -                  - {productData}      -
  -                      -                  -                  -                    -
  -                      -                  -                  ---200 OK-------------
  -                      -                  -                  -                    -
  -                      -                  -                  --Log success        -
  -                      -                  -                                       -
```

### Chatbot Order Creation

```
n8n                 Controller           Service              Repositories         Database
  -                       -                  -                       -               -
  --POST /chatbot----------                  -                       -               -
  - {orderRequest}       -                  -                       -               -
  -                      -                  -                       -               -
  -                      --ProcessOrder()----                       -               -
  -                      -                  -                       -               -
  -                      -                  --GetByPageId()----------               -
  -                      -                  -                       --SELECT----------
  -                      -                  -                       -----Platform-----
  -                      -                  ---SocialPlatform--------               -
  -                      -                  -  (has StoreId)        -               -
  -                      -                  -                       -               -
  -                      -                  --GetByPSID()------------               -
  -                      -                  -                       --SELECT----------
  -                      -                  -                       -----Customer-----
  -                      -                  ---Customer (or null)----               -
  -                      -                  -                       -               -
  -                      -                  --GetByName()------------               -
  -                      -                  -  (for each item)      -               -
  -                      -                  -                       --SELECT----------
  -                      -                  -                       -----Product------
  -                      -                  ---Product (or null)-----               -
  -                      -                  -                       -               -
  -                      -                  --AddAsync(Order)--------               -
  -                      -                  -                       --INSERT----------
  -                      -                  -                       -               -
  -                      -                  --SaveChanges()----------               -
  -                      -                  -                       -               -
  -                      -                  --AddAsync(OrderProd)----               -
  -                      -                  -  (for each match)     -               -
  -                      -                  -                       --INSERT----------
  -                      -                  -                       -               -
  -                      -                  --SaveChanges()----------               -
  -                      -                  -                       -               -
  -                      ---OrderDto---------                       -               -
  -                      -                  -                       -               -
  ---201 Created----------                  -                       -               -
  - {success, orderId}   -                  -                       -               -
  -                      -                  -                       -               -
```

---

## --- Dependency Injection Structure

```
------------------------------------------------------------------
-                     DI Container                                -
------------------------------------------------------------------
-                                                                 -
-  Application Layer (AddApplicationServices)                     -
-  -- IProductService - ProductService                            -
-  -- IOrderService - OrderService                                -
-  -- ChatbotOrderService (concrete, no interface)               -
-  -- AutoMapper                                                  -
-                                                                 -
-  Infrastructure Layer (AddInfrastructureServices)               -
-  -- IProductEmbeddingService - ProductEmbeddingService          -
-  -- IUnitOfWork - UnitOfWork                                    -
-  -- IProductRepository - ProductRepository                      -
-  -- ICustomerRepository - CustomerRepository                    -
-  -- ISocialPlatformRepository - SocialPlatformRepository        -
-  -- IOrderRepository - OrderRepository                          -
-  -- IHttpClientFactory (built-in)                               -
-  -- DbContext                                                   -
-                                                                 -
-------------------------------------------------------------------
```

---

**All diagrams show the complete architecture and flow of both implemented features.**

For code-level details, see implementation files and documentation.
