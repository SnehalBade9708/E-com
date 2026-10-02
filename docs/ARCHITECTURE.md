# System Architecture Specification

This document details the architectural layout, directory structures, communication protocols, and design patterns for the **Mini E-Commerce Demo Project**.

---

## 1. High-Level Architecture Topology

The application follows the decoupled **Client-Server Architecture** pattern using the MERN stack:

```text
┌─────────────────────────────────────────────────────────────┐
│                    Client (Browser / React)                 │
│  - React.js 18 + Vite (SPA)                                 │
│  - Tailwind CSS UI Components                               │
│  - React Router DOM (Public & Protected Routes)             │
│  - AuthContext & CartContext (State Management)             │
│  - Axios Client with JWT Authorization Interceptor          │
└──────────────────────────────┬──────────────────────────────┘
                               │
                        HTTPS / JSON (REST API)
                               │
┌──────────────────────────────▼──────────────────────────────┐
│                    Server (Node.js + Express)               │
│  - Express REST API (/api)                                  │
│  - CORS & Body Parser Middleware                            │
│  - JWT Authentication & Admin Authorization Middleware      │
│  - Controllers & Business Logic Layer                       │
│  - Server-Authoritative Price & Stock Validation            │
└──────────────────────────────┬──────────────────────────────┘
                               │
                         Mongoose ODM
                               │
┌──────────────────────────────▼──────────────────────────────┐
│                    Database (MongoDB)                       │
│  - users collection                                         │
│  - categories collection                                    │
│  - products collection                                      │
│  - orders collection                                        │
└─────────────────────────────────────────────────────────────┘
```

---

## 2. Directory Layout

The project is structured into two self-contained sub-projects: `/client` and `/server`.

```text
E-com/
├── README.md
├── docs/
│   ├── ARCHITECTURE.md
│   ├── DATABASE_MODELS.md
│   ├── API_SPECIFICATION.md
│   ├── AUTH_AND_SECURITY.md
│   ├── FRONTEND_SPECIFICATION.md
│   └── WORKFLOW_AND_TESTING.md
│
├── client/                               # Frontend Single Page Application
│   ├── index.html
│   ├── package.json
│   ├── vite.config.js
│   ├── tailwind.config.js
│   ├── postcss.config.js
│   ├── .env.example
│   └── src/
│       ├── main.jsx                      # Application bootstrap
│       ├── App.jsx                       # Routing & provider wrapping
│       ├── index.css                     # Tailwind directives & base styles
│       ├── api/
│       │   └── axiosClient.js            # Configured Axios instance with interceptors
│       ├── context/
│       │   ├── AuthContext.jsx           # Global user auth state & tokens
│       │   └── CartContext.jsx           # Global cart state (with stock limits)
│       ├── components/
│       │   ├── common/
│       │   │   ├── Navbar.jsx            # Responsive navigation header
│       │   │   ├── Footer.jsx            # Footer component
│       │   │   ├── Toast.jsx             # Toast notification component
│       │   │   ├── ConfirmModal.jsx      # Delete confirmation dialog
│       │   │   ├── LoadingSpinner.jsx    # Loading state indicator
│       │   │   └── EmptyState.jsx        # Empty state placeholder
│       │   ├── admin/
│       │   │   ├── AdminSidebar.jsx      # Admin navigation drawer/sidebar
│       │   │   ├── AdminHeader.jsx       # Admin top bar
│       │   │   ├── CategoryModal.jsx     # Add/Edit Category modal
│       │   │   ├── ProductModal.jsx      # Add/Edit Product modal
│       │   │   └── OrderStatusModal.jsx  # Change order status dropdown modal
│       │   └── shop/
│       │       ├── ProductCard.jsx       # Grid item card with stock badge
│       │       ├── CategoryFilter.jsx    # Category pills selector
│       │       └── SearchBar.jsx         # Product search input
│       └── pages/
│           ├── HomePage.jsx              # Landing banner & featured items
│           ├── ProductsPage.jsx          # Catalog with filters & search
│           ├── ProductDetailPage.jsx     # Detailed view with stock check
│           ├── CartPage.jsx              # Cart items & quantity adjustment
│           ├── CheckoutPage.jsx          # Shipping address & COD order placement
│           ├── LoginPage.jsx             # Customer / Admin Login
│           ├── RegisterPage.jsx          # Customer Registration
│           ├── MyOrdersPage.jsx          # Customer Order History
│           └── admin/
│               ├── AdminDashboardPage.jsx# Admin metrics overview
│               ├── AdminCategoriesPage.jsx# Category CRUD table
│               ├── AdminProductsPage.jsx # Product CRUD table
│               └── AdminOrdersPage.jsx   # All customer orders table
│
└── server/                               # Backend REST API Service
    ├── package.json
    ├── .env.example
    ├── server.js                         # Application entrypoint & DB connection
    └── src/
        ├── config/
        │   └── db.js                     # MongoDB connection via Mongoose
        ├── models/
        │   ├── User.js                   # User Mongoose model
        │   ├── Category.js               # Category Mongoose model
        │   ├── Product.js                # Product Mongoose model
        │   └── Order.js                  # Order Mongoose model
        ├── middleware/
        │   ├── authMiddleware.js         # JWT verification & req.user attachment
        │   ├── adminMiddleware.js        # Admin role enforcement
        │   └── errorHandler.js          # Centralized error handler
        ├── controllers/
        │   ├── authController.js         # Registration, login logic
        │   ├── categoryController.js     # Category CRUD logic
        │   ├── productController.js      # Product CRUD, search & filter logic
        │   └── orderController.js        # Order placement & admin status updates
        └── routes/
            ├── authRoutes.js             # /api/auth routes
            ├── categoryRoutes.js         # /api/categories routes
            ├── productRoutes.js          # /api/products routes
            └── orderRoutes.js            # /api/orders & /api/admin/orders routes
```

---

## 3. Communication & State Management

### 3.1 HTTP Client (`axiosClient.js`)
- Standardized base URL pointing to backend API (e.g. `http://localhost:5000/api`).
- **Request Interceptor:** Automatically inspects `localStorage` for `token` and injects `Authorization: Bearer <token>` into headers.
- **Response Interceptor:** Gracefully traps global errors, including `401 Unauthorized` (clearing credentials and redirecting to login when session expires).

### 3.2 Global State Architecture
- **`AuthContext`**:
  - `user`: Holds authenticated user profile `{ _id, name, email, role }`.
  - `token`: JWT token string.
  - `login(email, password)`: Submits credentials, persists JWT to `localStorage`, and updates state.
  - `logout()`: Clears token and resets user state.
  - `isAdmin`: Boolean helper computed from `user.role === 'admin'`.

- **`CartContext`**:
  - `cart`: Array of items `[{ product: { _id, name, price, image, stock }, quantity }]`.
  - Persisted in browser `localStorage` for seamless reload experience.
  - `addToCart(product, quantity)`: Appends or increases quantity, rejecting increments beyond `product.stock`.
  - `updateQuantity(productId, quantity)`: Modifies quantity within bounds `[1, product.stock]`.
  - `removeFromCart(productId)`: Removes an item from the cart.
  - `clearCart()`: Flushes all items upon successful checkout.
  - `totalPrice`: Memoized computation of total cart cost.

---

## 4. End-to-End Data Flow

### 4.1 Order Placement Lifecycle

```text
Customer Browser                  Server (Express)                Database (MongoDB)
     │                                   │                                 │
     │── 1. POST /api/orders ───────────>│                                 │
     │     { products, shippingAddress } │                                 │
     │                                   │── 2. authMiddleware verifies JWT│
     │                                   │                                 │
     │                                   │── 3. Fetch Products from DB ───>│
     │                                   │<── Returns current price/stock ─│
     │                                   │                                 │
     │                                   │── 4. Verify Stock Availability  │
     │                                   │   & Calculate Total Amount      │
     │                                   │                                 │
     │                                   │── 5. Decrement Stock ──────────>│
     │                                   │── 6. Save Order Record ────────>│
     │                                   │<── Order Created Successfully ──│
     │                                   │                                 │
     │<── 7. HTTP 201 { order } ─────────│                                 │
     │                                   │                                 │
     │── 8. CartContext.clearCart()      │                                 │
     │── 9. Navigate to /my-orders       │                                 │
```

---

## 5. Environment Configuration

### Client (`/client/.env`)
```env
VITE_API_BASE_URL=http://localhost:5000/api
```

### Server (`/server/.env`)
```env
PORT=5000
MONGO_URI=mongodb://localhost:27017/ecom_demo
JWT_SECRET=supersecret_jwt_key_change_in_production
JWT_EXPIRES_IN=7d
NODE_ENV=development
```
