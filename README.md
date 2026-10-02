# Mini E-Commerce Demo Project (MERN Stack)

A clean, modular, and lightweight full-stack E-Commerce demonstration application built using the **MERN** stack (MongoDB, Express.js, React.js, Node.js). 

This project adheres strictly to core e-commerce fundamentals: authentication, catalog management, cart operations, cash-on-delivery (COD) checkout, stock management, and admin order fulfillment without unnecessary third-party bloat.

---

## 📌 Project Overview

- **Architecture:** Client-Server Monorepo (`/client` and `/server`)
- **Frontend:** React.js (Vite), JavaScript, Tailwind CSS, Lucide Icons, Axios
- **Backend:** Node.js, Express.js, Mongoose, JSON Web Tokens (JWT), bcryptjs
- **Database:** MongoDB (Local or Atlas)
- **Payment Method:** Cash on Delivery (COD) Only

---

## 📁 Repository Structure

```text
E-com/
├── README.md                          # Main project documentation & quick start
├── docs/                              # In-depth architectural & technical specs
│   ├── ARCHITECTURE.md                # System design, folder layouts & data flows
│   ├── DATABASE_MODELS.md             # Complete Mongoose schemas & data contracts
│   ├── API_SPECIFICATION.md           # REST API endpoints, payloads & response codes
│   ├── AUTH_AND_SECURITY.md           # JWT auth, hashing & security policies
│   ├── FRONTEND_SPECIFICATION.md      # UI/UX, page routing, Tailwind tokens & components
│   └── WORKFLOW_AND_TESTING.md        # End-to-end demo flow & test scenarios
├── client/                            # React + Vite Frontend (To be implemented)
└── server/                            # Node.js + Express Backend (To be implemented)
```

---

## 📑 Detailed Documentation Index

For detailed engineering and implementation specifications, please refer to the documents in [`docs/`](file:///C:/Users/Snehal/.gemini/antigravity-ide/scratch/E-com/docs):

1. **[System Architecture (`docs/ARCHITECTURE.md`)](file:///C:/Users/Snehal/.gemini/antigravity-ide/scratch/E-com/docs/ARCHITECTURE.md)**
   - High-level client-server topology
   - Directory structure for `/client` and `/server`
   - State management approach and Axios configuration
2. **[Database Models (`docs/DATABASE_MODELS.md`)](file:///C:/Users/Snehal/.gemini/antigravity-ide/scratch/E-com/docs/DATABASE_MODELS.md)**
   - Exactly 4 Mongoose models: `User`, `Category`, `Product`, `Order`
   - Field types, default values, references, indexes, and validation rules
   - Sample JSON documents
3. **[API Specification (`docs/API_SPECIFICATION.md`)](file:///C:/Users/Snehal/.gemini/antigravity-ide/scratch/E-com/docs/API_SPECIFICATION.md)**
   - Complete endpoint catalog (Auth, Categories, Products, Orders, Admin Orders)
   - Query parameter format for filtering (`?category=...&search=...`)
   - Request and response schemas with HTTP status codes
4. **[Authentication & Security (`docs/AUTH_AND_SECURITY.md`)](file:///C:/Users/Snehal/.gemini/antigravity-ide/scratch/E-com/docs/AUTH_AND_SECURITY.md)**
   - JWT generation, validation, and token lifetime
   - Password hashing with bcrypt
   - Route protection middleware (`authMiddleware`, `adminMiddleware`)
   - Critical server-side pricing & stock atomic decrement safeguards
5. **[Frontend Specification (`docs/FRONTEND_SPECIFICATION.md`)](file:///C:/Users/Snehal/.gemini/antigravity-ide/scratch/E-com/docs/FRONTEND_SPECIFICATION.md)**
   - Tailwind CSS design system and responsive breakpoints
   - Public storefront pages: Home, Products, Product Details, Cart, Checkout, Auth, My Orders
   - Admin panel layout: Categories, Products, and Order Management tables
   - Toast alerts, confirmation modals, loading and empty states
6. **[Workflow & Testing Guide (`docs/WORKFLOW_AND_TESTING.md`)](file:///C:/Users/Snehal/.gemini/antigravity-ide/scratch/E-com/docs/WORKFLOW_AND_TESTING.md)**
   - Full 12-step end-to-end demo lifecycle
   - Edge-case validation matrix (out-of-stock, invalid price tampering, unauthorized admin access)

---

## 🚀 Key Features

### 👤 Customer Experience
- **Account Management:** User registration, login with secure credentials, session persistence via JWT.
- **Product Catalog:** Responsive grid view, instant category filtering (`All | Electronics | Fashion | Shoes`), and live keyword search.
- **Product Details:** High-resolution view, category breadcrumbs, live stock indicator, and instant "Add to Cart".
- **Shopping Cart:** Quantity controls with real-time stock cap enforcement, item removal, and auto-calculated subtotal.
- **Checkout:** Simple Cash on Delivery (COD) address form (Name, Phone, Address, City, Pincode).
- **Order Tracking:** "My Orders" dashboard displaying historic orders, status tags, items, and total amount.

### 🛡️ Admin Management
- **Secure Access:** Dedicated admin credential validation and role-protected middleware.
- **Category Control:** Create, view, update, and delete product categories.
- **Product Catalog Management:** Full inventory CRUD (Title, Description, Price, Image URL, Category, Stock count).
- **Order Fulfillment:** Comprehensive order list with customer details, item breakdowns, and status transitions:
  `Pending` ➔ `Confirmed` ➔ `Shipped` ➔ `Delivered` (or `Cancelled`).

---

## 🔒 Security & Validation Highlights

- **Server-Authoritative Pricing:** Frontend product prices in order payloads are strictly ignored. The backend retrieves the authentic price directly from the MongoDB database at order creation.
- **Inventory Protection:** Product stock is verified prior to order creation and atomically decremented. Orders exceeding available stock are rejected.
- **Payload Validation:** Strict field constraints (valid emails, minimum 6-character passwords, non-negative numbers for price and stock).
- **Role Isolation:** Admin endpoints reject requests from non-admin JWT payloads with `403 Forbidden`.

---

## ⚙️ Planned Technology Stack

| Layer | Tool / Library | Purpose |
| :--- | :--- | :--- |
| **Frontend Framework** | React.js (v18+) via Vite | Fast, modern component-based UI rendering |
| **Styling** | Tailwind CSS | Utility-first responsive design system |
| **Icons** | Lucide React | Clean, consistent icons |
| **HTTP Client** | Axios | REST API communication with request interceptors |
| **Backend Framework** | Node.js + Express.js | Scalable, modular REST API server |
| **Database** | MongoDB + Mongoose | Document datastore with schema enforcement |
| **Authentication** | JWT (jsonwebtoken) + bcryptjs | Stateless token auth & secure salted hashing |

---

## 📋 Out of Scope

To preserve the simplicity, maintainability, and clarity of this demo project, the following features are intentionally excluded:
- Third-party online payment gateways (Stripe, Razorpay, PayPal)
- Wishlists, user reviews, or ratings
- Discount coupons, vouchers, or promo codes
- Multi-vendor or supplier portals
- Complex analytics or reporting dashboards
