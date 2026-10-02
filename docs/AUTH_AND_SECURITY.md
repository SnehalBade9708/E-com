# Authentication & Security Specification

This document details the authentication mechanism, role-based authorization, cryptographic hashing, and security principles implemented in the **Mini E-Commerce Demo Project**.

---

## 1. Authentication Architecture

The application uses stateless **JSON Web Tokens (JWT)** for authenticating HTTP requests.

```text
┌─────────────────┐                                  ┌─────────────────┐
│     Client      │                                  │   Node/Express  │
└────────┬────────┘                                  └────────┬────────┘
         │                                                    │
         │─── 1. POST /api/auth/login { email, password } ───>│
         │                                                    │─── Verify Email
         │                                                    │─── bcrypt.compare()
         │                                                    │─── Sign JWT with Secret
         │<── 2. Response: { token, user: { id, role } } ─────│
         │                                                    │
   Store Token in                                             │
    localStorage                                              │
         │                                                    │
         │─── 3. GET /api/orders/my-orders ──────────────────>│
         │       Header: Authorization: Bearer <token>        │─── authMiddleware:
         │                                                    │    jwt.verify()
         │                                                    │    req.user = decoded
         │<── 4. Response: { orders: [...] } ─────────────────│
         │                                                    │
```

---

## 2. Password Hashing with bcryptjs

To prevent credential leakage in the event of database compromise:
- Passwords are never stored in plain text.
- Passwords are salted and hashed using **bcryptjs** with a cost factor (salt rounds) of `10`.
- The user schema specifies `select: false` on the `password` field so that standard Mongoose queries (`User.find()`, `User.findById()`) omit the hash unless explicitly overridden (`.select('+password')`).

### Implementation Reference
```javascript
const bcrypt = require('bcryptjs');

// Hash password before saving
async function hashPassword(plainTextPassword) {
  const salt = await bcrypt.genSalt(10);
  return await bcrypt.hash(plainTextPassword, salt);
}

// Compare submitted password against stored hash
async function verifyPassword(plainTextPassword, hashedPassword) {
  return await bcrypt.compare(plainTextPassword, hashedPassword);
}
```

---

## 3. JWT Signing & Payload

### 3.1 Token Payload
The JWT payload is kept lightweight and contains only essential identity attributes:
```json
{
  "id": "65b8ef109a24d5a71c890001",
  "role": "customer",
  "iat": 1727856000,
  "exp": 1728460800
}
```

### 3.2 Secret and Expiration
- Secret is loaded securely via environment variable `JWT_SECRET`.
- Expiration is set to `7d` (7 days) for a seamless demo experience.

---

## 4. Route Protection Middleware

### 4.1 Authentication Middleware (`authMiddleware.js`)
Validates the presence and signature of the Bearer token. Upon verification, attaches the user object to `req.user`.

```javascript
const jwt = require('jsonwebtoken');
const User = require('../models/User');

const protect = async (req, res, next) => {
  let token;

  if (
    req.headers.authorization &&
    req.headers.authorization.startsWith('Bearer ')
  ) {
    token = req.headers.authorization.split(' ')[1];
  }

  if (!token) {
    return res.status(401).json({
      success: false,
      message: 'Access denied. No token provided.',
    });
  }

  try {
    const decoded = jwt.verify(token, process.env.JWT_SECRET);
    const user = await User.findById(decoded.id).select('-password');

    if (!user) {
      return res.status(401).json({
        success: false,
        message: 'Invalid token. User no longer exists.',
      });
    }

    req.user = user;
    next();
  } catch (error) {
    return res.status(401).json({
      success: false,
      message: 'Invalid or expired token.',
    });
  }
};

module.exports = { protect };
```

### 4.2 Admin Authorization Middleware (`adminMiddleware.js`)
Ensures that the authenticated user possesses administrative privileges. Must execute after `protect`.

```javascript
const requireAdmin = (req, res, next) => {
  if (!req.user || req.user.role !== 'admin') {
    return res.status(403).json({
      success: false,
      message: 'Access forbidden: Administrator privileges required.',
    });
  }
  next();
};

module.exports = { requireAdmin };
```

---

## 5. Critical E-Commerce Security Safeguards

### 5.1 Server-Authoritative Price Calculation
> **CRITICAL RULE:** The server NEVER trusts product prices submitted by the client during checkout.

#### Threat Model
A malicious actor could intercept the `POST /api/orders` HTTP request and alter a product price from `$899.99` to `$0.01`.

#### Mitigation Logic
1. The client request body only submits an array of `{ product: productId, quantity }`.
2. The server queries MongoDB for each `productId`.
3. The server multiplies the verified database price by `quantity`.
4. The server computes the final `totalAmount`.
5. Snapshot names and prices are captured directly from MongoDB records into the new `Order` document.

### 5.2 Stock Race Condition & Depletion Safeguards
> **CRITICAL RULE:** Stock cannot drop below zero, and orders must be prevented if stock is insufficient.

#### Mitigation Logic
When handling `POST /api/orders`:
1. Check that `product.stock >= item.quantity` for all ordered items. If any item fails, abort with `400 Bad Request` without creating the order.
2. Atomically decrement stock:
```javascript
await Product.findByIdAndUpdate(item.productId, {
  $inc: { stock: -item.quantity }
});
```

---

## 6. Validation Matrix

| Field / Operation | Client-Side Validation | Server-Side Validation |
| :--- | :--- | :--- |
| **Email** | HTML5 `type="email"` + Regex format | Sanitized lowercase + RFC 5322 regex + Unique index |
| **Password** | Min 6 chars, match `confirmPassword` | Min 6 chars length check |
| **Product Price** | HTML5 `min="0.01"`, numeric check | Mongoose `min: [0.01]` |
| **Product Stock** | Numeric integer >= 0 | Integer check + `min: 0` |
| **Category** | Form select dropdown | Must exist in `categories` collection |
| **Order Quantity** | Clamped to available stock in cart UI | Verified against live DB stock count |
| **Admin Routes** | Admin UI view restriction | HTTP 401/403 middleware verification |
| **Shipping Address** | Required: Name, Phone, Address, City, Pincode | All fields required in Mongoose sub-schema |
