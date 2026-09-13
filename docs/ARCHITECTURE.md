# Tukupedia Architecture Documentation

## 📐 System Architecture Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                    Client Layer (PWA)                           │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │  Next.js 14+ (React, App Router, Service Worker)         │  │
│  │  - Katalog & pencarian produk                            │  │
│  │  - Keranjang & checkout                                  │  │
│  │  - User account & order tracking                         │  │
│  │  - Admin dashboard                                       │  │
│  └──────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
                              │
                    (GraphQL + REST)
                              │
┌─────────────────────────────────────────────────────────────────┐
│                 API Gateway Layer (.NET 8)                      │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │  ASP.NET Core + HotChocolate (GraphQL)                   │  │
│  │  ├─ GraphQL Schema (Products, Orders, Auth)             │  │
│  │  └─ REST Controllers (Webhooks, Auth, Admin)            │  │
│  └──────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
                              │
        ┌─────────────────────┼─────────────────────┐
        │                     │                     │
        ▼                     ▼                     ▼
┌──────────────────┐ ┌──────────────────┐ ┌──────────────────┐
│  Business Logic  │ │  Cache Layer     │ │  Storage Layer   │
│  (Services)      │ │  (Redis)         │ │  (S3/MinIO)      │
│                  │ │                  │ │                  │
│  - Product Svc   │ │  - Sessions      │ │  - Product imgs  │
│  - Order Svc     │ │  - Cart cache    │ │  - Review imgs   │
│  - User Svc      │ │  - Query cache   │ │  - Banner        │
│  - Auth Svc      │ │  - Rate limit    │ │                  │
���  - Payment Mock  │ │                  │ │                  │
└──────────────────┘ └──────────────────┘ └──────────────────┘
        │
        └─────────────────────────────────────────┐
                                                  │
                                                  ▼
                                        ┌──────────────────┐
                                        │  Data Layer      │
                                        │  (PostgreSQL)    │
                                        │                  │
                                        │  - Users         │
                                        │  - Products      │
                                        │  - Orders        │
                                        │  - Transactions  │
                                        │  - Reviews       │
                                        └──────────────────┘
```

## 🎯 Design Patterns

### 1. Layered Architecture (.NET Backend)

```
Presentation Layer (Controllers/GraphQL)
           ↓
Application Layer (Services/DTOs)
           ↓
Domain Layer (Models/Interfaces)
           ↓
Data Access Layer (Repositories/DbContext)
           ↓
Database (PostgreSQL)
```

**Advantages:**
- Clear separation of concerns
- Easy to test each layer independently
- Scalable & maintainable

### 2. GraphQL + REST Hybrid

```
GraphQL Endpoint (Primary)
├─ Products Query
│  ├─ search
│  ├─ filter by category/price/rating
│  └─ paginated results
│
├─ Cart Mutation
│  ├─ addToCart
│  ├─ updateQuantity
│  └─ removeItem
│
└─ Order Subscription
   └─ orderStatusChanged

REST Endpoints (Secondary)
├─ POST /api/webhooks/payment
├─ POST /api/webhooks/shipping
└─ POST /api/auth/refresh-token
```

**Rationale:**
- GraphQL untuk data queries yang kompleks & fleksibel
- REST untuk webhooks & non-graph operations

### 3. PWA Architecture

```
Service Worker
├─ Cache Strategies
│  ├─ Cache-first (statics: CSS, JS, fonts)
│  ├─ Network-first (data: products, prices)
│  └─ Stale-while-revalidate (images)
│
├─ Background Sync
│  └─ Sync cart/order actions when online
│
└─ Push Notifications
   └─ Order status updates
```

## 🔐 Authentication & Authorization

### Flow
```
User Input (email/password or OAuth)
           ↓
ASP.NET Identity (user validation)
           ↓
Generate JWT Token (HS256 signature)
           ↓
Return token to client
           ↓
Store in localStorage/sessionStorage
           ↓
Include in Authorization header
           ↓
GraphQL/REST validates JWT
           ↓
Extract user claims (id, email, role)
```

### Roles
- **USER**: Regular customer
- **MERCHANT**: Can sell products
- **ADMIN**: System administrator

## 📊 Data Flow Examples

### Product Browsing Flow
```
1. User opens /products (Next.js SSR)
2. Server fetches from GraphQL: query { products(categoryId, filter, sort) }
3. GraphQL resolver calls ProductService.GetProducts()
4. Service checks Redis cache first
5. If cache miss, query PostgreSQL via Repository
6. Cache result in Redis (TTL: 1 hour)
7. Return to client with ISR (Incremental Static Regeneration)
8. Client renders with Tailwind CSS
9. Service Worker caches page for offline
```

### Order Checkout Flow
```
1. User fills checkout form (Next.js form)
2. Submit mutation: checkout(addressId, paymentMethod, voucherCode)
3. GraphQL validates input & calls OrderService.CheckoutAsync()
4. Service validates:
   - Address ownership
   - Product availability
   - Voucher validity
5. Create Order & OrderItems in PostgreSQL
6. Trigger payment mock webhook
7. Update order status → PENDING_PAYMENT
8. Return order details with payment URL
9. Client receives WebSocket subscription: orderStatusChanged
10. When payment mock returns, update status
11. Send WebSocket update to client → UI updates real-time
```

### Search Flow
```
1. User types in search box (Client-side)
2. Debounce (300ms) & autocomplete via GraphQL
3. query { productAutocomplete(term: "laptop") }
4. Search in SQLite FTS index
5. Return suggestions (10 items)
6. User selects or presses Enter
7. Navigate to /products?search=laptop
8. GraphQL query: products(search: "laptop", page, limit)
9. FTS search → ranking by relevance
10. Return paginated results
11. Server-side sort/filter via PostgreSQL
12. Cache in Redis
```

## 🗄️ Database Design

### Key Tables & Relationships

```
users (1) ──────────┐
                    │ (N)
                    └── addresses
                    │
                    └── carts (1) ────────┐
                                          │ (N)
                                          └── cart_items (N) ──┐
                                                                │
                    └── orders (1) ────────┐                    │
                                           │ (N)                │
                                           └── order_items (N)──┤
                                                                 │
                                                        (1) ─────┤
categories (1) ──┐ (N)                                          │
                 └── products (1) ────────┐                     │
                                          │ (N)                 │
                          ┌───────────────┤                     │
                          │               └── product_variants ◄─┘
                          │
                    (1) ──┤ (N)
                          └── product_images

                    (1) ──┐ (N)
                          └── reviews

brands ──────────────────────────────────┐
                                         │ (N)
                                         └── products

vouchers ───────────────────────────────┐
                                        │ (N)
                                        └── orders

orders (1) ───────┐
                  │ (1)
                  └── payments

orders (1) ───────┐
                  │ (1)
                  └── shipments
```

### Key Indexes (Performance Optimization)

```sql
-- Products
CREATE INDEX idx_products_slug ON products(slug);
CREATE INDEX idx_products_category_id ON products(category_id);
CREATE INDEX idx_products_active ON products(is_active);

-- Variants
CREATE INDEX idx_variants_sku ON product_variants(sku);
CREATE INDEX idx_variants_product_id ON product_variants(product_id);

-- Orders
CREATE INDEX idx_orders_user_id ON orders(user_id);
CREATE INDEX idx_orders_status ON orders(status);
CREATE INDEX idx_orders_created_at ON orders(created_at DESC);

-- Reviews
CREATE INDEX idx_reviews_product_id ON reviews(product_id);
CREATE INDEX idx_reviews_rating ON reviews(rating);

-- Categories
CREATE INDEX idx_categories_parent_id ON categories(parent_id);
CREATE INDEX idx_categories_slug ON categories(slug);
```

## 🔄 Caching Strategy

### Redis Cache Layers

```
┌─────────────────────────────────────────────────────┐
│           Redis Cache Structure                    │
├─────────────────────────────────────────────────────┤
│                                                     │
│  user:<id>:session                                  │
│  ├─ userId, email, role                           │
│  └─ TTL: 24 hours                                 │
│                                                     │
│  cart:<user_id>                                    │
│  ├─ Items array (variant_id, qty)                │
│  └─ TTL: 7 days                                   │
│                                                     │
│  product:<id>                                      │
│  ├─ name, price, stock, images                   │
│  └─ TTL: 1 hour                                   │
│                                                     │
│  products:category:<cat_id>:page:<n>              │
│  ├─ Paginated product list                       │
│  └─ TTL: 2 hours                                  │
│                                                     │
│  search:<term>:page:<n>                           │
│  ├─ Search results cache                         │
│  └─ TTL: 4 hours                                 │
│                                                     │
│  rate_limit:<user_id>:<endpoint>                  │
│  ├─ Request count                                │
│  └─ TTL: 1 minute                                │
│                                                     │
└─────────────────────────────────────────────────────┘
```

## 🚀 Deployment Architecture

### Local Development (Docker Compose)
```
docker-compose up
├─ PostgreSQL:5432
├─ Redis:6379
├─ Backend API:5000
└─ Frontend Next.js:3000
```

### Production Ready (Kubernetes)
```
Kubernetes Cluster
├─ Frontend Service (Next.js container)
├─ Backend Service (.NET container)
├─ PostgreSQL StatefulSet
├─ Redis Cache
├─ Ingress (SSL/TLS)
└─ Storage (S3/MinIO)
```

## 📈 Scalability Considerations

### Horizontal Scaling
- Backend API (stateless) → multiple replicas
- Frontend (CDN) → distributed via Vercel/Cloudflare
- Database read replicas → for read-heavy queries
- Elasticsearch sharding → for large dataset searches

### Vertical Scaling
- Increase Redis memory (for more cache)
- PostgreSQL optimization (indexes, partitioning)
- Database connection pooling (PgBouncer)

### Performance Optimization
- GraphQL query complexity limits
- Rate limiting per user
- Pagination (page size: 20-50)
- Database query optimization
- Redis cache warming on startup

## 🔐 Security Considerations

- JWT token expiration: 15 minutes (access token), 7 days (refresh token)
- Password hashing: bcrypt (ASP.NET Identity)
- CORS: Whitelist frontend domain
- SQL Injection: Parameterized queries (Entity Framework)
- XSS Protection: Sanitize user input, CSP headers
- CSRF: Token-based CSRF protection
- Rate Limiting: Per IP & per user
- HTTPS: Required in production

## 📝 API Versioning Strategy

```
v1 (Current)
├─ /graphql/v1
├─ /api/v1/...

v2 (Future)
├─ /graphql/v2
├─ /api/v2/...

Backward Compatibility
├─ Deprecation warnings
└─ Migration period: 6 months
```

## 🧪 Testing Strategy

```
Frontend Tests
├─ Unit: Jest + React Testing Library
├─ E2E: Playwright
└─ Visual: Percy (optional)

Backend Tests
├─ Unit: xUnit + Moq
├─ Integration: TestContainers
└─ Load: k6 (optional)
```

---

**Last Updated**: 2024
**Architecture Version**: 1.0
