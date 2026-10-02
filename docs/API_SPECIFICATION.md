# REST API Specification

This document provides the complete API reference for the **Mini E-Commerce Demo Project**. All endpoints respond with `Content-Type: application/json`.

---

## 1. Global Standards & Conventions

### 1.1 Base URL
```text
http://localhost:5000/api
```

### 1.2 Authentication Header
Endpoints requiring authentication must supply the JWT token in the `Authorization` HTTP header:
```text
Authorization: Bearer <jwt_token>
```

### 1.3 Standard Error Format
All 4xx and 5xx errors return a unified response shape:
```json
{
  "success": false,
  "message": "Descriptive error message",
  "errors": [] // Optional field-specific validation errors
}
```

### 1.4 Access Control Levels
- 🌐 **Public:** Accessible without authentication.
- 👤 **Protected (Customer):** Requires valid JWT token (`authMiddleware`).
- 🛡️ **Protected (Admin):** Requires valid JWT token with `role: 'admin'` (`authMiddleware` + `adminMiddleware`).

---

## 2. API Endpoints Matrix

| Domain | Method | Endpoint | Access | Description |
| :--- | :--- | :--- | :--- | :--- |
| **Auth** | `POST` | `/api/auth/register` | 🌐 Public | Register new customer account |
| **Auth** | `POST` | `/api/auth/login` | 🌐 Public | Authenticate customer or admin |
| **Categories** | `GET` | `/api/categories` | 🌐 Public | List all categories |
| **Categories** | `POST` | `/api/categories` | 🛡️ Admin | Create new category |
| **Categories** | `PUT` | `/api/categories/:id` | 🛡️ Admin | Update category details |
| **Categories** | `DELETE`| `/api/categories/:id` | 🛡️ Admin | Remove category |
| **Products** | `GET` | `/api/products` | 🌐 Public | List products (with filter & search) |
| **Products** | `GET` | `/api/products/:id` | 🌐 Public | Fetch single product by ID |
| **Products** | `POST` | `/api/products` | 🛡️ Admin | Create new product |
| **Products** | `PUT` | `/api/products/:id` | 🛡️ Admin | Update product details |
| **Products** | `DELETE`| `/api/products/:id` | 🛡️ Admin | Delete product |
| **Orders** | `POST` | `/api/orders` | 👤 Customer | Create new COD order |
| **Orders** | `GET` | `/api/orders/my-orders` | 👤 Customer | Fetch logged-in user's orders |
| **Orders** | `GET` | `/api/admin/orders` | 🛡️ Admin | Fetch all orders system-wide |
| **Orders** | `PATCH` | `/api/admin/orders/:id/status`| 🛡️ Admin | Update order status |

---

## 3. Authentication Endpoints

### 3.1 Register Customer
- **Method:** `POST`
- **Path:** `/api/auth/register`
- **Access:** 🌐 Public
- **Request Body:**
```json
{
  "name": "Snehal Bade",
  "email": "snehal@example.com",
  "password": "password123",
  "confirmPassword": "password123"
}
```
- **Validation Rules:**
  - `name`: Required, min 2 characters.
  - `email`: Required, valid email format, unique in database.
  - `password`: Required, min 6 characters.
  - `confirmPassword`: Required, must strictly match `password`.
- **Success Response (`201 Created`):**
```json
{
  "success": true,
  "message": "User registered successfully",
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "user": {
    "_id": "65b8ef109a24d5a71c890001",
    "name": "Snehal Bade",
    "email": "snehal@example.com",
    "role": "customer"
  }
}
```
- **Error Response (`400 Bad Request`):**
```json
{
  "success": false,
  "message": "Email already in use"
}
```

---

### 3.2 Login (Customer & Admin)
- **Method:** `POST`
- **Path:** `/api/auth/login`
- **Access:** 🌐 Public
- **Request Body:**
```json
{
  "email": "snehal@example.com",
  "password": "password123"
}
```
- **Validation Rules:**
  - `email`: Required, valid format.
  - `password`: Required.
- **Success Response (`200 OK`):**
```json
{
  "success": true,
  "message": "Login successful",
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "user": {
    "_id": "65b8ef109a24d5a71c890001",
    "name": "Snehal Bade",
    "email": "snehal@example.com",
    "role": "customer"
  }
}
```
- **Error Response (`401 Unauthorized`):**
```json
{
  "success": false,
  "message": "Invalid email or password"
}
```

---

## 4. Category Endpoints

### 4.1 Get All Categories
- **Method:** `GET`
- **Path:** `/api/categories`
- **Access:** 🌐 Public
- **Success Response (`200 OK`):**
```json
{
  "success": true,
  "count": 3,
  "categories": [
    {
      "_id": "65b8ef209a24d5a71c890002",
      "name": "Electronics",
      "description": "Smartphones, audio, and gadgets",
      "createdAt": "2026-10-02T08:05:00.000Z"
    },
    {
      "_id": "65b8ef209a24d5a71c890003",
      "name": "Fashion",
      "description": "Apparel, clothing, and accessories",
      "createdAt": "2026-10-02T08:06:00.000Z"
    }
  ]
}
```

---

### 4.2 Create Category
- **Method:** `POST`
- **Path:** `/api/categories`
- **Access:** 🛡️ Admin (`Bearer <token>`)
- **Request Body:**
```json
{
  "name": "Shoes",
  "description": "Sneakers, running shoes, and formal footwear"
}
```
- **Success Response (`201 Created`):**
```json
{
  "success": true,
  "message": "Category created successfully",
  "category": {
    "_id": "65b8ef209a24d5a71c890004",
    "name": "Shoes",
    "description": "Sneakers, running shoes, and formal footwear",
    "createdAt": "2026-10-02T08:20:00.000Z"
  }
}
```

---

### 4.3 Update Category
- **Method:** `PUT`
- **Path:** `/api/categories/:id`
- **Access:** 🛡️ Admin (`Bearer <token>`)
- **Request Body:**
```json
{
  "name": "Footwear & Shoes",
  "description": "Updated category description"
}
```
- **Success Response (`200 OK`):**
```json
{
  "success": true,
  "message": "Category updated successfully",
  "category": {
    "_id": "65b8ef209a24d5a71c890004",
    "name": "Footwear & Shoes",
    "description": "Updated category description"
  }
}
```

---

### 4.4 Delete Category
- **Method:** `DELETE`
- **Path:** `/api/categories/:id`
- **Access:** 🛡️ Admin (`Bearer <token>`)
- **Success Response (`200 OK`):**
```json
{
  "success": true,
  "message": "Category deleted successfully"
}
```

---

## 5. Product Endpoints

### 5.1 Get Products (Catalog, Search & Filter)
- **Method:** `GET`
- **Path:** `/api/products`
- **Query Parameters:**
  - `category` *(optional)*: Filter by category name or category ID (e.g. `?category=electronics`)
  - `search` *(optional)*: Case-insensitive keyword search on product name or description (e.g. `?search=phone`)
- **Example Request:**
```text
GET /api/products?category=electronics&search=phone
```
- **Access:** 🌐 Public
- **Success Response (`200 OK`):**
```json
{
  "success": true,
  "count": 1,
  "products": [
    {
      "_id": "65b8ef309a24d5a71c890003",
      "name": "Smartphone Pro Max",
      "description": "Flagship 5G smartphone with 120Hz OLED screen",
      "price": 899.99,
      "image": "https://images.unsplash.com/photo-1511707171634-5f897ff02aa9",
      "category": {
        "_id": "65b8ef209a24d5a71c890002",
        "name": "Electronics"
      },
      "stock": 15,
      "createdAt": "2026-10-02T08:10:00.000Z"
    }
  ]
}
```

---

### 5.2 Get Product By ID
- **Method:** `GET`
- **Path:** `/api/products/:id`
- **Access:** 🌐 Public
- **Success Response (`200 OK`):**
```json
{
  "success": true,
  "product": {
    "_id": "65b8ef309a24d5a71c890003",
    "name": "Smartphone Pro Max",
    "description": "Flagship 5G smartphone with 120Hz OLED screen",
    "price": 899.99,
    "image": "https://images.unsplash.com/photo-1511707171634-5f897ff02aa9",
    "category": {
      "_id": "65b8ef209a24d5a71c890002",
      "name": "Electronics"
    },
    "stock": 15
  }
}
```

---

### 5.3 Create Product
- **Method:** `POST`
- **Path:** `/api/products`
- **Access:** 🛡️ Admin (`Bearer <token>`)
- **Request Body:**
```json
{
  "name": "Running Athletic Shoes",
  "description": "Breathable lightweight running sneakers with cushioned soles.",
  "price": 79.50,
  "image": "https://images.unsplash.com/photo-1542291026-7eec264c27ff",
  "category": "65b8ef209a24d5a71c890004",
  "stock": 40
}
```
- **Validation Rules:**
  - `name`: Required.
  - `description`: Required.
  - `price`: Required, number > 0.
  - `image`: Required URL.
  - `category`: Required, valid MongoDB Category ObjectId.
  - `stock`: Required, integer >= 0.
- **Success Response (`201 Created`):**
```json
{
  "success": true,
  "message": "Product created successfully",
  "product": {
    "_id": "65b8ef309a24d5a71c890010",
    "name": "Running Athletic Shoes",
    "price": 79.50,
    "stock": 40
  }
}
```

---

### 5.4 Update Product
- **Method:** `PUT`
- **Path:** `/api/products/:id`
- **Access:** 🛡️ Admin (`Bearer <token>`)
- **Request Body:** Same schema as create product. Partial updates supported.
- **Success Response (`200 OK`):**
```json
{
  "success": true,
  "message": "Product updated successfully",
  "product": { ... }
}
```

---

### 5.5 Delete Product
- **Method:** `DELETE`
- **Path:** `/api/products/:id`
- **Access:** 🛡️ Admin (`Bearer <token>`)
- **Success Response (`200 OK`):**
```json
{
  "success": true,
  "message": "Product deleted successfully"
}
```

---

## 6. Order Endpoints

### 6.1 Create Order (Checkout)
- **Method:** `POST`
- **Path:** `/api/orders`
- **Access:** 👤 Protected Customer (`Bearer <token>`)
- **Security Check:** Backend **never** trusts client price. The controller extracts each `product` ID, fetches current DB price and stock, calculates `totalAmount`, verifies stock >= quantity, decrements stock atomically, and creates the order.
- **Request Body:**
```json
{
  "products": [
    {
      "product": "65b8ef309a24d5a71c890003",
      "quantity": 2
    }
  ],
  "shippingAddress": {
    "name": "Snehal Bade",
    "phone": "+91 9876543210",
    "address": "Flat 3B, Tech Park residency",
    "city": "Pune",
    "pincode": "411001"
  }
}
```
- **Success Response (`201 Created`):**
```json
{
  "success": true,
  "message": "Order placed successfully",
  "order": {
    "_id": "65b8ef409a24d5a71c890004",
    "user": "65b8ef109a24d5a71c890001",
    "products": [
      {
        "product": "65b8ef309a24d5a71c890003",
        "name": "Smartphone Pro Max",
        "price": 899.99,
        "quantity": 2
      }
    ],
    "totalAmount": 1799.98,
    "shippingAddress": {
      "name": "Snehal Bade",
      "phone": "+91 9876543210",
      "address": "Flat 3B, Tech Park residency",
      "city": "Pune",
      "pincode": "411001"
    },
    "status": "Pending",
    "createdAt": "2026-10-02T08:30:00.000Z"
  }
}
```
- **Error Response - Insufficient Stock (`400 Bad Request`):**
```json
{
  "success": false,
  "message": "Insufficient stock for product 'Smartphone Pro Max'. Available: 1, Requested: 2"
}
```

---

### 6.2 Get Customer's Orders
- **Method:** `GET`
- **Path:** `/api/orders/my-orders`
- **Access:** 👤 Protected Customer (`Bearer <token>`)
- **Success Response (`200 OK`):**
```json
{
  "success": true,
  "count": 1,
  "orders": [
    {
      "_id": "65b8ef409a24d5a71c890004",
      "products": [
        {
          "product": "65b8ef309a24d5a71c890003",
          "name": "Smartphone Pro Max",
          "price": 899.99,
          "quantity": 2
        }
      ],
      "totalAmount": 1799.98,
      "shippingAddress": { ... },
      "status": "Pending",
      "createdAt": "2026-10-02T08:30:00.000Z"
    }
  ]
}
```

---

### 6.3 Admin: Get All Orders
- **Method:** `GET`
- **Path:** `/api/admin/orders`
- **Access:** 🛡️ Admin (`Bearer <token>`)
- **Success Response (`200 OK`):**
```json
{
  "success": true,
  "count": 15,
  "orders": [
    {
      "_id": "65b8ef409a24d5a71c890004",
      "user": {
        "_id": "65b8ef109a24d5a71c890001",
        "name": "Snehal Bade",
        "email": "snehal@example.com"
      },
      "products": [ ... ],
      "totalAmount": 1799.98,
      "shippingAddress": { ... },
      "status": "Pending",
      "createdAt": "2026-10-02T08:30:00.000Z"
    }
  ]
}
```

---

### 6.4 Admin: Update Order Status
- **Method:** `PATCH`
- **Path:** `/api/admin/orders/:id/status`
- **Access:** 🛡️ Admin (`Bearer <token>`)
- **Request Body:**
```json
{
  "status": "Confirmed"
}
```
- **Allowed Status Values:**
  `Pending`, `Confirmed`, `Shipped`, `Delivered`, `Cancelled`
- **Success Response (`200 OK`):**
```json
{
  "success": true,
  "message": "Order status updated to Confirmed",
  "order": {
    "_id": "65b8ef409a24d5a71c890004",
    "status": "Confirmed",
    "updatedAt": "2026-10-02T08:45:00.000Z"
  }
}
```
