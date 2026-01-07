# Mining Application - Complete Architecture

## Table of Contents
1. [System Overview](#system-overview)
2. [Technology Stack](#technology-stack)
3. [Architecture Diagram](#architecture-diagram)
4. [Database Design](#database-design)
5. [API Architecture](#api-architecture)
6. [Frontend Architecture](#frontend-architecture)
7. [Authentication & Authorization](#authentication--authorization)
8. [Wallet System Design](#wallet-system-design)
9. [Security Considerations](#security-considerations)
10. [Deployment Architecture](#deployment-architecture)

---

## 1. System Overview

### High-Level Architecture
```
┌─────────────────────────────────────────────────────────────┐
│                      CLIENT LAYER                            │
│  ┌────────────────────────────────────────────────────────┐ │
│  │           Next.js Frontend Application                  │ │
│  │  - SSR/SSG Pages                                        │ │
│  │  - React Components                                     │ │
│  │  - State Management (React Query/Zustand)              │ │
│  │  - Tailwind CSS                                         │ │
│  └────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
                            ↕ HTTPS/REST
┌─────────────────────────────────────────────────────────────┐
│                    API GATEWAY LAYER                         │
│  ┌────────────────────────────────────────────────────────┐ │
│  │              Laravel Backend API                        │ │
│  │  - RESTful API Endpoints                               │ │
│  │  - Laravel Sanctum Authentication                      │ │
│  │  - Request Validation                                  │ │
│  │  - Rate Limiting                                       │ │
│  └────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
                            ↕
┌─────────────────────────────────────────────────────────────┐
│                  BUSINESS LOGIC LAYER                        │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐     │
│  │   Services   │  │ Repositories │  │   Events     │     │
│  │  - Wallet    │  │  - Data      │  │  - Listeners │     │
│  │  - Product   │  │    Access    │  │  - Jobs      │     │
│  │  - Purchase  │  │    Layer     │  │  - Queues    │     │
│  └──────────────┘  └──────────────┘  └──────────────┘     │
└─────────────────────────────────────────────────────────────┘
                            ↕
┌─────────────────────────────────────────────────────────────┐
│                    DATA LAYER                                │
│  ┌────────────────────────────────────────────────────────┐ │
│  │              PostgreSQL Database                        │ │
│  │  - User Data                                           │ │
│  │  - Products                                            │ │
│  │  - Wallet Transactions (Ledger)                       │ │
│  │  - Audit Logs                                          │ │
│  └────────────────────────────────────────────────────────┘ │
│  ┌────────────────────────────────────────────────────────┐ │
│  │              File Storage (S3/Local)                    │ │
│  │  - Product Images                                      │ │
│  │  - Payment Proofs                                      │ │
│  └────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
```

---

## 2. Technology Stack

### Frontend Stack
```yaml
Framework: Next.js 16+ (App Router)
Language: TypeScript
Styling: Tailwind CSS
State Management:
  - React Query (Server State)
  - Zustand (Client State)
Form Handling: React Hook Form
Validation: Zod
HTTP Client: Axios
Authentication: Next-Auth (optional) or Custom with Sanctum
UI Components: shadcn/ui or Headless UI
Icons: Lucide React or Heroicons
```

### Backend Stack
```yaml
Framework: Laravel 11.x
Language: PHP 8.2+
Authentication: Laravel Sanctum
Database: PostgreSQL 15+
ORM: Eloquent
Validation: Form Requests
File Storage: Laravel Storage (Local/S3)
Queue: Redis (optional)
Cache: Redis
API Documentation: Scribe or OpenAPI
Testing: PHPUnit, Pest
```

### DevOps & Infrastructure
```yaml
Containerization: Docker
Web Server: Nginx
Process Manager: PHP-FPM
Reverse Proxy: Nginx
SSL: Let's Encrypt
CI/CD: GitHub Actions
Monitoring: Laravel Telescope (dev), Sentry (prod)
```

---

## 3. Architecture Diagram

### System Component Diagram
```
┌────────────────────────────────────────────────────────────────┐
│                        FRONTEND (Next.js)                       │
├────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐        │
│  │ Public Pages │  │  User Pages  │  │ Admin Pages  │        │
│  ├──────────────┤  ├──────────────┤  ├──────────────┤        │
│  │ - Home       │  │ - Dashboard  │  │ - Dashboard  │        │
│  │ - Products   │  │ - Wallet     │  │ - Products   │        │
│  │ - Product    │  │ - Purchases  │  │ - Users      │        │
│  │   Detail     │  │ - Profile    │  │ - Approvals  │        │
│  │ - Login      │  │              │  │ - Logs       │        │
│  │ - Register   │  │              │  │              │        │
│  └──────────────┘  └──────────────┘  └──────────────┘        │
│                                                                 │
│  ┌───────────────────────────────────────────────────────┐    │
│  │              Shared Components                         │    │
│  │  - Header/Footer  - Forms  - Cards  - Modals         │    │
│  └───────────────────────────────────────────────────────┘    │
│                                                                 │
│  ┌───────────────────────────────────────────────────────┐    │
│  │              State Management                          │    │
│  │  - Auth Context  - API Client  - Query Cache          │    │
│  └───────────────────────────────────────────────────────┘    │
└────────────────────────────────────────────────────────────────┘
                              ↓ HTTP/REST API
┌────────────────────────────────────────────────────────────────┐
│                      BACKEND (Laravel)                          │
├────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐  │
│  │                     Routes Layer                         │  │
│  │  ┌───────────┐  ┌───────────┐  ┌───────────┐          │  │
│  │  │   API     │  │   Auth    │  │   Admin   │          │  │
│  │  │  Routes   │  │  Routes   │  │  Routes   │          │  │
│  │  └───────────┘  └───────────┘  └───────────┘          │  │
│  └─────────────────────────────────────────────────────────┘  │
│                              ↓                                  │
│  ┌─────────────────────────────────────────────────────────┐  │
│  │                  Middleware Layer                        │  │
│  │  - Sanctum Auth  - Rate Limit  - CORS  - Logging       │  │
│  └─────────────────────────────────────────────────────────┘  │
│                              ↓                                  │
│  ┌─────────────────────────────────────────────────────────┐  │
│  │                  Controller Layer                        │  │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  │  │
│  │  │   Product    │  │    Wallet    │  │    Admin     │  │  │
│  │  │  Controller  │  │  Controller  │  │  Controller  │  │  │
│  │  └──────────────┘  └──────────────┘  └──────────────┘  │  │
│  └─────────────────────────────────────────────────────────┘  │
│                              ↓                                  │
│  ┌─────────────────────────────────────────────────────────┐  │
│  │                   Service Layer                          │  │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  │  │
│  │  │   Product    │  │    Wallet    │  │   Purchase   │  │  │
│  │  │   Service    │  │   Service    │  │   Service    │  │  │
│  │  └──────────────┘  └──────────────┘  └──────────────┘  │  │
│  └─────────────────────────────────────────────────────────┘  │
│                              ↓                                  │
│  ┌─────────────────────────────────────────────────────────┐  │
│  │                 Repository Layer                         │  │
│  │  - Data Access  - Query Building  - Eloquent Models     │  │
│  └─────────────────────────────────────────────────────────┘  │
│                              ↓                                  │
│  ┌─────────────────────────────────────────────────────────┐  │
│  │                    Model Layer                           │  │
│  │  - User  - Product  - WalletTransaction  - Purchase     │  │
│  └─────────────────────────────────────────────────────────┘  │
└────────────────────────────────────────────────────────────────┘
                              ↓
┌────────────────────────────────────────────────────────────────┐
│                    DATABASE (PostgreSQL)                        │
└────────────────────────────────────────────────────────────────┘
```

---

## 4. Database Design

### Complete Database Schema

#### 4.1 Users Table
```sql
CREATE TABLE users (
    id BIGSERIAL PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    email_verified_at TIMESTAMP NULL,
    password VARCHAR(255) NOT NULL,
    phone VARCHAR(20) NULL,
    status VARCHAR(20) DEFAULT 'active', -- active, inactive
    last_login_at TIMESTAMP NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_status ON users(status);
```

#### 4.2 Roles Table
```sql
CREATE TABLE roles (
    id BIGSERIAL PRIMARY KEY,
    name VARCHAR(50) UNIQUE NOT NULL, -- admin, user
    description TEXT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE role_user (
    id BIGSERIAL PRIMARY KEY,
    user_id BIGINT NOT NULL,
    role_id BIGINT NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
    FOREIGN KEY (role_id) REFERENCES roles(id) ON DELETE CASCADE,
    UNIQUE(user_id, role_id)
);

CREATE INDEX idx_role_user_user_id ON role_user(user_id);
CREATE INDEX idx_role_user_role_id ON role_user(role_id);
```

#### 4.3 Products Table
```sql
CREATE TABLE products (
    id BIGSERIAL PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    slug VARCHAR(255) UNIQUE NOT NULL,
    description TEXT NULL,
    short_description VARCHAR(500) NULL,
    price DECIMAL(15, 2) NOT NULL CHECK (price >= 0),
    image_url VARCHAR(500) NULL,
    images JSONB NULL, -- Gallery images
    status VARCHAR(20) DEFAULT 'active', -- active, inactive
    created_by BIGINT NULL,
    updated_by BIGINT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (created_by) REFERENCES users(id) ON DELETE SET NULL,
    FOREIGN KEY (updated_by) REFERENCES users(id) ON DELETE SET NULL
);

CREATE INDEX idx_products_status ON products(status);
CREATE INDEX idx_products_slug ON products(slug);
CREATE INDEX idx_products_created_at ON products(created_at DESC);
```

#### 4.4 Wallet Transactions Table (CRITICAL - Ledger Pattern)
```sql
CREATE TABLE wallet_transactions (
    id BIGSERIAL PRIMARY KEY,
    user_id BIGINT NOT NULL,
    type VARCHAR(20) NOT NULL, -- CREDIT, DEBIT
    amount DECIMAL(15, 2) NOT NULL CHECK (amount > 0),
    balance_after DECIMAL(15, 2) NOT NULL CHECK (balance_after >= 0),
    reference_type VARCHAR(50) NULL, -- wallet_request, purchase
    reference_id BIGINT NULL,
    description TEXT NULL,
    metadata JSONB NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
);

CREATE INDEX idx_wallet_transactions_user_id ON wallet_transactions(user_id);
CREATE INDEX idx_wallet_transactions_type ON wallet_transactions(type);
CREATE INDEX idx_wallet_transactions_created_at ON wallet_transactions(created_at DESC);
CREATE INDEX idx_wallet_transactions_reference ON wallet_transactions(reference_type, reference_id);

-- Critical: Index for calculating balance
CREATE INDEX idx_wallet_balance_calc ON wallet_transactions(user_id, id DESC);
```

#### 4.5 Wallet Requests Table
```sql
CREATE TABLE wallet_requests (
    id BIGSERIAL PRIMARY KEY,
    user_id BIGINT NOT NULL,
    amount DECIMAL(15, 2) NOT NULL CHECK (amount > 0),
    payment_proof_url VARCHAR(500) NOT NULL,
    status VARCHAR(20) DEFAULT 'pending', -- pending, approved, rejected
    admin_notes TEXT NULL,
    approved_by BIGINT NULL,
    approved_at TIMESTAMP NULL,
    rejected_at TIMESTAMP NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
    FOREIGN KEY (approved_by) REFERENCES users(id) ON DELETE SET NULL
);

CREATE INDEX idx_wallet_requests_user_id ON wallet_requests(user_id);
CREATE INDEX idx_wallet_requests_status ON wallet_requests(status);
CREATE INDEX idx_wallet_requests_created_at ON wallet_requests(created_at DESC);
```

#### 4.6 Purchases Table
```sql
CREATE TABLE purchases (
    id BIGSERIAL PRIMARY KEY,
    user_id BIGINT NOT NULL,
    product_id BIGINT NOT NULL,
    quantity INTEGER DEFAULT 1 CHECK (quantity > 0),
    price DECIMAL(15, 2) NOT NULL CHECK (price >= 0), -- Price at purchase time
    total_amount DECIMAL(15, 2) NOT NULL CHECK (total_amount >= 0),
    wallet_transaction_id BIGINT NULL,
    status VARCHAR(20) DEFAULT 'completed', -- completed, cancelled
    metadata JSONB NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
    FOREIGN KEY (product_id) REFERENCES products(id) ON DELETE RESTRICT,
    FOREIGN KEY (wallet_transaction_id) REFERENCES wallet_transactions(id) ON DELETE SET NULL
);

CREATE INDEX idx_purchases_user_id ON purchases(user_id);
CREATE INDEX idx_purchases_product_id ON purchases(product_id);
CREATE INDEX idx_purchases_created_at ON purchases(created_at DESC);
CREATE INDEX idx_purchases_status ON purchases(status);
```

#### 4.7 Login Activities Table
```sql
CREATE TABLE login_activities (
    id BIGSERIAL PRIMARY KEY,
    user_id BIGINT NOT NULL,
    ip_address VARCHAR(45) NOT NULL,
    user_agent TEXT NULL,
    country VARCHAR(100) NULL,
    city VARCHAR(100) NULL,
    device_type VARCHAR(50) NULL, -- desktop, mobile, tablet
    browser VARCHAR(100) NULL,
    status VARCHAR(20) DEFAULT 'success', -- success, failed
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
);

CREATE INDEX idx_login_activities_user_id ON login_activities(user_id);
CREATE INDEX idx_login_activities_created_at ON login_activities(created_at DESC);
CREATE INDEX idx_login_activities_ip ON login_activities(ip_address);
```

#### 4.8 Admin Logs Table
```sql
CREATE TABLE admin_logs (
    id BIGSERIAL PRIMARY KEY,
    admin_id BIGINT NOT NULL,
    action VARCHAR(100) NOT NULL, -- login, approve_wallet, reject_wallet, etc.
    entity_type VARCHAR(50) NULL, -- product, user, wallet_request
    entity_id BIGINT NULL,
    description TEXT NULL,
    ip_address VARCHAR(45) NULL,
    metadata JSONB NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (admin_id) REFERENCES users(id) ON DELETE CASCADE
);

CREATE INDEX idx_admin_logs_admin_id ON admin_logs(admin_id);
CREATE INDEX idx_admin_logs_action ON admin_logs(action);
CREATE INDEX idx_admin_logs_created_at ON admin_logs(created_at DESC);
CREATE INDEX idx_admin_logs_entity ON admin_logs(entity_type, entity_id);
```

#### 4.9 Password Reset Tokens (Laravel Default)
```sql
CREATE TABLE password_reset_tokens (
    email VARCHAR(255) PRIMARY KEY,
    token VARCHAR(255) NOT NULL,
    created_at TIMESTAMP NULL
);

CREATE INDEX idx_password_reset_tokens_email ON password_reset_tokens(email);
```

#### 4.10 Personal Access Tokens (Laravel Sanctum)
```sql
CREATE TABLE personal_access_tokens (
    id BIGSERIAL PRIMARY KEY,
    tokenable_type VARCHAR(255) NOT NULL,
    tokenable_id BIGINT NOT NULL,
    name VARCHAR(255) NOT NULL,
    token VARCHAR(64) UNIQUE NOT NULL,
    abilities TEXT NULL,
    last_used_at TIMESTAMP NULL,
    expires_at TIMESTAMP NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_pat_tokenable ON personal_access_tokens(tokenable_type, tokenable_id);
CREATE INDEX idx_pat_token ON personal_access_tokens(token);
```

### Database Relationships Diagram
```
users (1) ──────────── (N) wallet_transactions
  │                           │
  │                           │ (1)
  │                           │
  │ (1)                       ├──── purchases (N)
  │                           │
  ├───────── (N) wallet_requests
  │
  ├───────── (N) purchases
  │                │
  │                │ (N)
  │                │
  │                └──────── (1) products
  │
  ├───────── (N) login_activities
  │
  ├───────── (N) admin_logs
  │
  └───────── (N) role_user ──────── (N) roles
```

---

## 5. API Architecture

### 5.1 API Endpoints Structure

#### Public Endpoints (No Authentication)
```
POST   /api/register
POST   /api/login
POST   /api/logout

GET    /api/products              # List active products
GET    /api/products/{slug}       # Product detail
```

#### User Endpoints (Sanctum Auth Required)
```
GET    /api/user                  # Current user info
PUT    /api/user/profile          # Update profile
PUT    /api/user/password         # Change password

# Wallet
GET    /api/wallet/balance        # Get current balance
GET    /api/wallet/transactions   # Transaction history
POST   /api/wallet/request        # Request to add amount
GET    /api/wallet/requests       # User's wallet requests

# Purchases
POST   /api/purchases             # Buy product
GET    /api/purchases             # Purchase history
GET    /api/purchases/{id}        # Purchase detail
```

#### Admin Endpoints (Admin Role Required)
```
# Dashboard
GET    /api/admin/dashboard       # Stats

# Products
GET    /api/admin/products        # All products (active + inactive)
POST   /api/admin/products        # Create product
GET    /api/admin/products/{id}   # Product detail
PUT    /api/admin/products/{id}   # Update product
DELETE /api/admin/products/{id}   # Delete product
PATCH  /api/admin/products/{id}/status  # Toggle status

# Wallet Approvals
GET    /api/admin/wallet-requests # Pending requests
POST   /api/admin/wallet-requests/{id}/approve
POST   /api/admin/wallet-requests/{id}/reject

# Users
GET    /api/admin/users           # All users
GET    /api/admin/users/{id}      # User detail
PATCH  /api/admin/users/{id}/status  # Activate/deactivate
GET    /api/admin/users/{id}/activities  # Login history

# Logs
GET    /api/admin/logs            # Admin activity logs
```

### 5.2 API Response Structure

#### Success Response
```json
{
  "success": true,
  "message": "Operation successful",
  "data": {
    // Response data
  },
  "meta": {
    "timestamp": "2024-01-02T10:30:00Z"
  }
}
```

#### Paginated Response
```json
{
  "success": true,
  "data": [],
  "meta": {
    "current_page": 1,
    "per_page": 15,
    "total": 100,
    "last_page": 7
  },
  "links": {
    "first": "...",
    "last": "...",
    "prev": null,
    "next": "..."
  }
}
```

#### Error Response
```json
{
  "success": false,
  "message": "Validation failed",
  "errors": {
    "email": ["The email field is required."]
  },
  "meta": {
    "timestamp": "2024-01-02T10:30:00Z"
  }
}
```

### 5.3 HTTP Status Codes
```
200 OK                  - Success
201 Created             - Resource created
400 Bad Request         - Validation error
401 Unauthorized        - Not authenticated
403 Forbidden           - Not authorized
404 Not Found           - Resource not found
422 Unprocessable       - Validation failed
429 Too Many Requests   - Rate limit exceeded
500 Internal Error      - Server error
```

---

## 6. Frontend Architecture

### 6.1 Directory Structure
```
src/
├── app/                          # Next.js App Router
│   ├── (auth)/                   # Auth group
│   │   ├── login/
│   │   └── register/
│   ├── (public)/                 # Public group
│   │   ├── page.tsx              # Home page
│   │   └── products/
│   │       ├── page.tsx          # Products list
│   │       └── [slug]/
│   │           └── page.tsx      # Product detail
│   ├── (user)/                   # User dashboard group
│   │   ├── dashboard/
│   │   ├── wallet/
│   │   └── purchases/
│   ├── (admin)/                  # Admin panel group
│   │   ├── admin/
│   │   │   ├── dashboard/
│   │   │   ├── products/
│   │   │   ├── users/
│   │   │   └── wallet-requests/
│   ├── api/                      # API routes (if needed)
│   ├── layout.tsx
│   └── globals.css
├── components/                   # React components
│   ├── ui/                       # Base UI components
│   │   ├── button.tsx
│   │   ├── card.tsx
│   │   ├── input.tsx
│   │   └── modal.tsx
│   ├── shared/                   # Shared components
│   │   ├── Header.tsx
│   │   ├── Footer.tsx
│   │   ├── Sidebar.tsx
│   │   └── LoadingSpinner.tsx
│   ├── products/                 # Product components
│   │   ├── ProductCard.tsx
│   │   ├── ProductList.tsx
│   │   └── ProductGallery.tsx
│   ├── wallet/                   # Wallet components
│   │   ├── WalletBalance.tsx
│   │   ├── AddFundsForm.tsx
│   │   └── TransactionHistory.tsx
│   └── admin/                    # Admin components
│       ├── ProductForm.tsx
│       ├── WalletApproval.tsx
│       └── UserManagement.tsx
├── lib/                          # Utilities
│   ├── api.ts                    # API client
│   ├── auth.ts                   # Auth utilities
│   ├── utils.ts                  # Helper functions
│   └── validators.ts             # Zod schemas
├── hooks/                        # Custom hooks
│   ├── useAuth.ts
│   ├── useWallet.ts
│   ├── useProducts.ts
│   └── usePurchases.ts
├── store/                        # State management
│   ├── authStore.ts              # Zustand store
│   └── cartStore.ts
├── types/                        # TypeScript types
│   ├── api.ts
│   ├── user.ts
│   ├── product.ts
│   └── wallet.ts
└── config/                       # Configuration
    ├── constants.ts
    └── routes.ts
```

### 6.2 Key Frontend Features

#### Authentication Flow
```typescript
// lib/api.ts - API Client with Sanctum
import axios from 'axios';

const apiClient = axios.create({
  baseURL: process.env.NEXT_PUBLIC_API_URL,
  withCredentials: true, // Important for Sanctum
  headers: {
    'Accept': 'application/json',
    'Content-Type': 'application/json',
  },
});

// Request interceptor
apiClient.interceptors.request.use(
  (config) => {
    const token = localStorage.getItem('auth_token');
    if (token) {
      config.headers.Authorization = `Bearer ${token}`;
    }
    return config;
  },
  (error) => Promise.reject(error)
);

// Response interceptor
apiClient.interceptors.response.use(
  (response) => response,
  async (error) => {
    if (error.response?.status === 401) {
      // Handle unauthorized
      localStorage.removeItem('auth_token');
      window.location.href = '/login';
    }
    return Promise.reject(error);
  }
);
```

#### State Management Strategy
```typescript
// Using React Query for server state
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';

// Using Zustand for client state
import { create } from 'zustand';

interface AuthState {
  user: User | null;
  token: string | null;
  setAuth: (user: User, token: string) => void;
  logout: () => void;
}

export const useAuthStore = create<AuthState>((set) => ({
  user: null,
  token: null,
  setAuth: (user, token) => {
    localStorage.setItem('auth_token', token);
    set({ user, token });
  },
  logout: () => {
    localStorage.removeItem('auth_token');
    set({ user: null, token: null });
  },
}));
```

### 6.3 Page Components

#### Home Page Structure
```tsx
// app/(public)/page.tsx
export default async function HomePage() {
  return (
    <>
      <Hero />
      <ActiveProducts />
      <Features />
      <CallToAction />
    </>
  );
}
```

#### Product Detail Page
```tsx
// app/(public)/products/[slug]/page.tsx
export default async function ProductDetailPage({ 
  params 
}: { 
  params: { slug: string } 
}) {
  const product = await getProduct(params.slug);
  
  return (
    <div className="container mx-auto">
      <ProductGallery images={product.images} />
      <ProductInfo product={product} />
      <BuyButton product={product} />
    </div>
  );
}
```

#### User Dashboard
```tsx
// app/(user)/dashboard/page.tsx
export default function UserDashboard() {
  const { data: walletBalance } = useWalletBalance();
  const { data: recentPurchases } = useRecentPurchases();
  
  return (
    <div className="grid grid-cols-1 md:grid-cols-3 gap-6">
      <WalletCard balance={walletBalance} />
      <PurchasesSummary purchases={recentPurchases} />
      <QuickActions />
    </div>
  );
}
```

---

## 7. Authentication & Authorization

### 7.1 Laravel Sanctum Configuration

#### config/sanctum.php
```php
return [
    'stateful' => explode(',', env('SANCTUM_STATEFUL_DOMAINS', 
        'localhost,localhost:3000,127.0.0.1,127.0.0.1:8000,::1'
    )),
    
    'guard' => ['web'],
    
    'expiration' => null, // Token never expires (or set minutes)
    
    'middleware' => [
        'verify_csrf_token' => App\Http\Middleware\VerifyCsrfToken::class,
        'encrypt_cookies' => App\Http\Middleware\EncryptCookies::class,
    ],
];
```

#### CORS Configuration
```php
// config/cors.php
return [
    'paths' => ['api/*', 'sanctum/csrf-cookie'],
    'allowed_methods' => ['*'],
    'allowed_origins' => [env('FRONTEND_URL', 'http://localhost:3000')],
    'allowed_origins_patterns' => [],
    'allowed_headers' => ['*'],
    'exposed_headers' => [],
    'max_age' => 0,
    'supports_credentials' => true,
];
```

### 7.2 Authentication Flow

```
1. Frontend: GET /sanctum/csrf-cookie
   ├─ Backend: Set XSRF-TOKEN cookie
   └─ Response: 204 No Content

2. Frontend: POST /api/login
   ├─ Body: { email, password }
   ├─ Backend: Validate credentials
   ├─ Backend: Create personal access token
   └─ Response: { token, user }

3. Frontend: Store token in localStorage
   └─ Set Authorization: Bearer {token} in headers

4. Subsequent requests
   ├─ Header: Authorization: Bearer {token}
   └─ Middleware: Sanctum authenticates user
```

### 7.3 Authorization Middleware

#### Custom Admin Middleware
```php
// app/Http/Middleware/EnsureUserIsAdmin.php
namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;

class EnsureUserIsAdmin
{
    public function handle(Request $request, Closure $next)
    {
        if (!$request->user() || !$request->user()->hasRole('admin')) {
            return response()->json([
                'success' => false,
                'message' => 'Unauthorized. Admin access required.'
            ], 403);
        }
        
        return $next($request);
    }
}
```

#### Route Protection
```php
// routes/api.php
Route::middleware('auth:sanctum')->group(function () {
    // User routes
    Route::get('/user', [UserController::class, 'show']);
    Route::get('/wallet/balance', [WalletController::class, 'balance']);
    
    // Admin routes
    Route::middleware('admin')->prefix('admin')->group(function () {
        Route::apiResource('products', AdminProductController::class);
        Route::post('wallet-requests/{id}/approve', [AdminWalletController::class, 'approve']);
    });
});
```

### 7.4 Role-Based Access Control

#### User Model with Roles
```php
// app/Models/User.php
class User extends Authenticatable
{
    use HasApiTokens, HasRoles;
    
    public function hasRole(string $role): bool
    {
        return $this->roles()->where('name', $role)->exists();
    }
    
    public function isAdmin(): bool
    {
        return $this->hasRole('admin');
    }
    
    public function roles()
    {
        return $this->belongsToMany(Role::class);
    }
}
```

---

## 8. Wallet System Design (CRITICAL)

### 8.1 Ledger-Based Wallet Architecture

#### Core Principles
```
1. NEVER store balance directly in a column
2. Calculate balance from transaction ledger
3. Use database transactions for atomicity
4. Implement row-level locking
5. Maintain complete audit trail
```

### 8.2 Wallet Service Implementation

```php
// app/Services/WalletService.php
namespace App\Services;

use App\Models\User;
use App\Models\WalletTransaction;
use Illuminate\Support\Facades\DB;
use Exception;

class WalletService
{
    /**
     * Get current wallet balance for user
     */
    public function getBalance(User $user): float
    {
        return WalletTransaction::where('user_id', $user->id)
            ->orderBy('id', 'desc')
            ->value('balance_after') ?? 0.00;
    }
    
    /**
     * Add credit to wallet (Admin approval)
     */
    public function addCredit(
        User $user, 
        float $amount, 
        string $referenceType = null,
        int $referenceId = null,
        string $description = null
    ): WalletTransaction {
        return DB::transaction(function () use ($user, $amount, $referenceType, $referenceId, $description) {
            // Lock user's wallet for update
            $currentBalance = WalletTransaction::where('user_id', $user->id)
                ->lockForUpdate()
                ->orderBy('id', 'desc')
                ->value('balance_after') ?? 0.00;
            
            $newBalance = $currentBalance + $amount;
            
            // Create credit transaction
            return WalletTransaction::create([
                'user_id' => $user->id,
                'type' => 'CREDIT',
                'amount' => $amount,
                'balance_after' => $newBalance,
                'reference_type' => $referenceType,
                'reference_id' => $referenceId,
                'description' => $description ?? 'Wallet credited',
            ]);
        });
    }
    
    /**
     * Deduct from wallet (Purchase)
     */
    public function deductBalance(
        User $user,
        float $amount,
        string $referenceType = null,
        int $referenceId = null,
        string $description = null
    ): WalletTransaction {
        return DB::transaction(function () use ($user, $amount, $referenceType, $referenceId, $description) {
            // Lock user's wallet for update
            $currentBalance = WalletTransaction::where('user_id', $user->id)
                ->lockForUpdate()
                ->orderBy('id', 'desc')
                ->value('balance_after') ?? 0.00;
            
            // Check sufficient balance
            if ($currentBalance < $amount) {
                throw new Exception('Insufficient wallet balance');
            }
            
            $newBalance = $currentBalance - $amount;
            
            // Create debit transaction
            return WalletTransaction::create([
                'user_id' => $user->id,
                'type' => 'DEBIT',
                'amount' => $amount,
                'balance_after' => $newBalance,
                'reference_type' => $referenceType,
                'reference_id' => $referenceId,
                'description' => $description ?? 'Wallet debited',
            ]);
        });
    }
    
    /**
     * Get transaction history
     */
    public function getTransactionHistory(User $user, int $perPage = 15)
    {
        return WalletTransaction::where('user_id', $user->id)
            ->orderBy('created_at', 'desc')
            ->paginate($perPage);
    }
}
```

### 8.3 Wallet Request Approval Flow

```php
// app/Services/WalletRequestService.php
namespace App\Services;

use App\Models\WalletRequest;
use App\Models\User;
use Illuminate\Support\Facades\DB;
use Exception;

class WalletRequestService
{
    public function __construct(
        private WalletService $walletService
    ) {}
    
    /**
     * Approve wallet request
     */
    public function approve(WalletRequest $request, User $admin): void
    {
        if ($request->status !== 'pending') {
            throw new Exception('Only pending requests can be approved');
        }
        
        DB::transaction(function () use ($request, $admin) {
            // Update request status
            $request->update([
                'status' => 'approved',
                'approved_by' => $admin->id,
                'approved_at' => now(),
            ]);
            
            // Add credit to user's wallet
            $this->walletService->addCredit(
                user: $request->user,
                amount: $request->amount,
                referenceType: 'wallet_request',
                referenceId: $request->id,
                description: "Wallet request #{$request->id} approved"
            );
            
            // Log admin action
            AdminLog::create([
                'admin_id' => $admin->id,
                'action' => 'approve_wallet_request',
                'entity_type' => 'wallet_request',
                'entity_id' => $request->id,
                'description' => "Approved wallet request of ${$request->amount}",
            ]);
        });
    }
    
    /**
     * Reject wallet request
     */
    public function reject(WalletRequest $request, User $admin, string $reason = null): void
    {
        if ($request->status !== 'pending') {
            throw new Exception('Only pending requests can be rejected');
        }
        
        DB::transaction(function () use ($request, $admin, $reason) {
            $request->update([
                'status' => 'rejected',
                'admin_notes' => $reason,
                'approved_by' => $admin->id,
                'rejected_at' => now(),
            ]);
            
            // Log admin action
            AdminLog::create([
                'admin_id' => $admin->id,
                'action' => 'reject_wallet_request',
                'entity_type' => 'wallet_request',
                'entity_id' => $request->id,
                'description' => "Rejected wallet request of ${$request->amount}",
                'metadata' => ['reason' => $reason],
            ]);
        });
    }
}
```

### 8.4 Purchase Flow with Wallet

```php
// app/Services/PurchaseService.php
namespace App\Services;

use App\Models\User;
use App\Models\Product;
use App\Models\Purchase;
use Illuminate\Support\Facades\DB;
use Exception;

class PurchaseService
{
    public function __construct(
        private WalletService $walletService
    ) {}
    
    /**
     * Process product purchase
     */
    public function processPurchase(User $user, Product $product, int $quantity = 1): Purchase
    {
        // Validate product is active
        if ($product->status !== 'active') {
            throw new Exception('Product is not available for purchase');
        }
        
        $totalAmount = $product->price * $quantity;
        
        // Check wallet balance
        $currentBalance = $this->walletService->getBalance($user);
        if ($currentBalance < $totalAmount) {
            throw new Exception('Insufficient wallet balance');
        }
        
        return DB::transaction(function () use ($user, $product, $quantity, $totalAmount) {
            // Deduct from wallet
            $walletTransaction = $this->walletService->deductBalance(
                user: $user,
                amount: $totalAmount,
                referenceType: 'purchase',
                referenceId: null, // Will be updated after purchase is created
                description: "Purchase: {$product->name}"
            );
            
            // Create purchase record
            $purchase = Purchase::create([
                'user_id' => $user->id,
                'product_id' => $product->id,
                'quantity' => $quantity,
                'price' => $product->price,
                'total_amount' => $totalAmount,
                'wallet_transaction_id' => $walletTransaction->id,
                'status' => 'completed',
            ]);
            
            // Update wallet transaction reference
            $walletTransaction->update([
                'reference_id' => $purchase->id,
            ]);
            
            return $purchase;
        });
    }
}
```

---

## 9. Security Considerations

### 9.1 Backend Security

#### Input Validation
```php
// app/Http/Requests/PurchaseRequest.php
class PurchaseRequest extends FormRequest
{
    public function authorize(): bool
    {
        return auth()->check();
    }
    
    public function rules(): array
    {
        return [
            'product_id' => 'required|exists:products,id',
            'quantity' => 'required|integer|min:1|max:100',
        ];
    }
}
```

#### SQL Injection Prevention
```php
// Always use Eloquent or Query Builder with parameter binding
// GOOD
$users = User::where('email', $email)->first();

// BAD - Never do this
$users = DB::select("SELECT * FROM users WHERE email = '$email'");
```

#### File Upload Security
```php
// app/Http/Controllers/WalletController.php
public function requestAddFunds(Request $request)
{
    $validated = $request->validate([
        'amount' => 'required|numeric|min:1|max:100000',
        'payment_proof' => 'required|file|mimes:jpg,jpeg,png,pdf|max:5120', // 5MB
    ]);
    
    // Store with random filename
    $path = $request->file('payment_proof')->store('payment-proofs', 'private');
    
    WalletRequest::create([
        'user_id' => auth()->id(),
        'amount' => $validated['amount'],
        'payment_proof_url' => $path,
    ]);
}
```

#### Rate Limiting
```php
// app/Http/Kernel.php
protected $middlewareGroups = [
    'api' => [
        'throttle:60,1', // 60 requests per minute
    ],
];

// Custom rate limiting for specific routes
Route::middleware('throttle:10,1')->group(function () {
    Route::post('/wallet/request', [WalletController::class, 'requestAddFunds']);
});
```

### 9.2 Frontend Security

#### XSS Prevention
```tsx
// Always escape user input
import DOMPurify from 'dompurify';

function ProductDescription({ html }: { html: string }) {
  const sanitized = DOMPurify.sanitize(html);
  return <div dangerouslySetInnerHTML={{ __html: sanitized }} />;
}
```

#### CSRF Protection
```typescript
// API client automatically handles CSRF with Sanctum
await apiClient.get('/sanctum/csrf-cookie');
await apiClient.post('/api/login', credentials);
```

#### Secure Token Storage
```typescript
// Store tokens securely
class AuthService {
  setToken(token: string) {
    // Option 1: localStorage (simpler but less secure)
    localStorage.setItem('auth_token', token);
    
    // Option 2: httpOnly cookie (more secure, set by backend)
    // Backend sets: Set-Cookie: token=xxx; HttpOnly; Secure; SameSite=Strict
  }
  
  getToken(): string | null {
    return localStorage.getItem('auth_token');
  }
}
```

### 9.3 Environment Variables

#### Backend (.env)
```env
APP_NAME="Mining App"
APP_ENV=production
APP_DEBUG=false
APP_URL=https://api.yourdomain.com

DB_CONNECTION=pgsql
DB_HOST=127.0.0.1
DB_PORT=5432
DB_DATABASE=mining_app
DB_USERNAME=postgres
DB_PASSWORD=your_secure_password

SANCTUM_STATEFUL_DOMAINS=yourdomain.com
FRONTEND_URL=https://yourdomain.com

SESSION_DRIVER=redis
CACHE_DRIVER=redis
QUEUE_CONNECTION=redis

REDIS_HOST=127.0.0.1
REDIS_PASSWORD=null
REDIS_PORT=6379

FILESYSTEM_DISK=s3
AWS_ACCESS_KEY_ID=
AWS_SECRET_ACCESS_KEY=
AWS_DEFAULT_REGION=us-east-1
AWS_BUCKET=
```

#### Frontend (.env.local)
```env
NEXT_PUBLIC_API_URL=https://api.yourdomain.com
NEXT_PUBLIC_APP_NAME=Mining App
```

---

## 10. Deployment Architecture

### 10.1 Production Infrastructure

```
┌─────────────────────────────────────────────────────────┐
│                    Load Balancer (Nginx)                 │
│                         SSL/TLS                          │
└─────────────────────────────────────────────────────────┘
                          │
        ┌─────────────────┴─────────────────┐
        │                                   │
┌───────▼──────────┐              ┌────────▼─────────┐
│  Frontend Server │              │  Backend Server  │
│    (Next.js)     │              │    (Laravel)     │
│                  │              │                  │
│  - Node 18+      │◄────REST────►│  - PHP 8.2       │
│  - PM2           │              │  - PHP-FPM       │
│  - Port 3000     │              │  - Nginx         │
└──────────────────┘              │  - Port 8000     │
                                  └──────┬───────────┘
                                         │
                    ┌────────────────────┼────────────────────┐
                    │                    │                    │
            ┌───────▼────────┐  ┌────────▼──────┐  ┌────────▼────────┐
            │   PostgreSQL   │  │     Redis     │  │   File Storage  │
            │                │  │               │  │      (S3)       │
            │  - Primary     │  │  - Cache      │  │                 │
            │  - Replica     │  │  - Sessions   │  │  - Images       │
            └────────────────┘  │  - Queue      │  │  - Documents    │
                                └───────────────┘  └─────────────────┘
```

### 10.2 Docker Deployment

#### docker-compose.yml
```yaml
version: '3.8'

services:
  # Frontend (Next.js)
  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile
    ports:
      - "3000:3000"
    environment:
      - NEXT_PUBLIC_API_URL=http://backend:8000
    depends_on:
      - backend
    restart: unless-stopped

  # Backend (Laravel)
  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile
    ports:
      - "8000:8000"
    environment:
      - DB_HOST=postgres
      - DB_DATABASE=mining_app
      - DB_USERNAME=postgres
      - DB_PASSWORD=secret
      - REDIS_HOST=redis
    depends_on:
      - postgres
      - redis
    volumes:
      - ./backend/storage:/var/www/html/storage
    restart: unless-stopped

  # PostgreSQL
  postgres:
    image: postgres:15-alpine
    environment:
      - POSTGRES_DB=mining_app
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=secret
    volumes:
      - postgres_data:/var/lib/postgresql/data
    restart: unless-stopped

  # Redis
  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
    restart: unless-stopped

  # Nginx (Reverse Proxy)
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/conf.d:/etc/nginx/conf.d
      - ./nginx/ssl:/etc/nginx/ssl
    depends_on:
      - frontend
      - backend
    restart: unless-stopped

volumes:
  postgres_data:
  redis_data:
```

### 10.3 CI/CD Pipeline (GitHub Actions)

#### Backend Pipeline
```yaml
# .github/workflows/backend.yml
name: Backend CI/CD

on:
  push:
    branches: [ main ]
    paths:
      - 'backend/**'

jobs:
  test:
    runs-on: ubuntu-latest
    
    services:
      postgres:
        image: postgres:15
        env:
          POSTGRES_DB: test_db
          POSTGRES_USER: postgres
          POSTGRES_PASSWORD: password
        ports:
          - 5432:5432
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup PHP
        uses: shivammathur/setup-php@v2
        with:
          php-version: '8.2'
          extensions: pgsql, pdo_pgsql, redis
      
      - name: Install dependencies
        run: |
          cd backend
          composer install --no-interaction --prefer-dist
      
      - name: Run tests
        run: |
          cd backend
          php artisan test
      
      - name: Deploy to production
        if: success()
        run: |
          # SSH and deployment commands
```

#### Frontend Pipeline
```yaml
# .github/workflows/frontend.yml
name: Frontend CI/CD

on:
  push:
    branches: [ main ]
    paths:
      - 'frontend/**'

jobs:
  build-deploy:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
      
      - name: Install dependencies
        run: |
          cd frontend
          npm ci
      
      - name: Build
        run: |
          cd frontend
          npm run build
      
      - name: Deploy
        run: |
          # Deployment commands
```

---

## Summary

This architecture provides:

1. **Scalable Structure**: Separation of concerns with Next.js frontend and Laravel backend
2. **Secure Authentication**: Laravel Sanctum with token-based auth
3. **Robust Wallet System**: Ledger-based with ACID compliance
4. **Comprehensive Admin Panel**: Full control over products, users, and wallet approvals
5. **Production-Ready**: Docker deployment, CI/CD pipelines, monitoring
6. **Database Integrity**: Proper relationships, indexes, and constraints
7. **Security**: Input validation, rate limiting, file upload security, CSRF protection

The system is designed to handle:
- Multiple concurrent wallet transactions
- Secure payment proof handling
- Admin approval workflows
- Complete audit trails
- User activity tracking
- Role-based access control

Ready for development and production deployment!




T9yD14Nj9j7xAB4dbGeiX9h8unkKHxuWwb
TFj5g6A1zdbZz3xR2qM5y8wQ4wJ9z8wQ4w
TX1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6q