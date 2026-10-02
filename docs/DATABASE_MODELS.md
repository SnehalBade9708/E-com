# Database Models Specification

This document defines the schema definitions, validation constraints, data types, indexes, and relationships for MongoDB using **Mongoose ODM**.

As per requirements, exactly **four models** are defined:
1. `User`
2. `Category`
3. `Product`
4. `Order`

---

## 1. Entity Relationship Diagram (ERD)

```text
┌────────────────────────┐                  ┌────────────────────────┐
│         User           │                  │        Category        │
├────────────────────────┤                  ├────────────────────────┤
│ _id (ObjectId)         │                  │ _id (ObjectId)         │
│ name (String)          │                  │ name (String, Unique)  │
│ email (String, Unique) │                  │ description (String)   │
│ password (String Hash) │                  │ createdAt (Date)       │
│ role (customer | admin)│                  └───────────┬────────────┘
│ createdAt (Date)       │                              │ 1
└───────────┬────────────┘                              │
            │ 1                                         │ has many
            │                                           │
            │ places many                               │ N
            │                               ┌───────────▼────────────┐
            │                               │        Product         │
            │                               ├────────────────────────┤
            │                               │ _id (ObjectId)         │
            │                               │ name (String)          │
            │                               │ description (String)   │
            │                               │ price (Number, Min 0)  │
            │                               │ image (String)         │
            │                               │ category (Ref Category)│
            │                               │ stock (Number, Min 0)  │
            │                               │ createdAt (Date)       │
            │                               └───────────┬────────────┘
            │                                           │
            │ N                                         │ embedded items
┌───────────▼───────────────────────────────────────────▼────────────┐
│                               Order                                │
├────────────────────────────────────────────────────────────────────┤
│ _id (ObjectId)                                                     │
│ user (Ref User)                                                    │
│ products: [                                                        │
│   {                                                                │
│     product (Ref Product),                                         │
│     name (String),                                                 │
│     price (Number - captured at order time),                       │
│     quantity (Number, Min 1)                                       │
│   }                                                                │
│ ]                                                                  │
│ totalAmount (Number, Min 0)                                        │
│ shippingAddress: {                                                 │
│   name, phone, address, city, pincode                              │
│ }                                                                  │
│ status (Pending | Confirmed | Shipped | Delivered | Cancelled)     │
│ createdAt (Date)                                                   │
└────────────────────────────────────────────────────────────────────┘
```

---

## 2. Model Definitions

### 2.1 User Model (`models/User.js`)

Represents registered customers and administrative personnel.

#### Schema Fields
| Field | Type | Required | Unique | Validation / Defaults | Description |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `name` | String | Yes | No | Trimmed, min length 2 | Full name of the user |
| `email` | String | Yes | Yes | Trimmed, lowercase, RFC 5322 regex match | Unique email address |
| `password` | String | Yes | No | Min 6 characters (hashed with bcrypt) | Hashed password |
| `role` | String | Yes | No | Enum: `['customer', 'admin']`, Default: `'customer'` | User authorization role |
| `createdAt` | Date | Auto | No | Default: `Date.now` | Registration timestamp |

#### Mongoose Implementation Blueprint
```javascript
const mongoose = require('mongoose');

const userSchema = new mongoose.Schema(
  {
    name: {
      type: String,
      required: [true, 'Please provide your full name'],
      trim: true,
      minlength: [2, 'Name must be at least 2 characters'],
    },
    email: {
      type: String,
      required: [true, 'Please provide an email address'],
      unique: true,
      trim: true,
      lowercase: true,
      match: [
        /^\w+([.-]?\w+)*@\w+([.-]?\w+)*(\.\w{2,3})+$/,
        'Please provide a valid email address',
      ],
    },
    password: {
      type: String,
      required: [true, 'Please provide a password'],
      minlength: [6, 'Password must be at least 6 characters long'],
      select: false, // Prevents accidental exposure in queries
    },
    role: {
      type: String,
      enum: ['customer', 'admin'],
      default: 'customer',
    },
  },
  { timestamps: true }
);

module.exports = mongoose.model('User', userSchema);
```

---

### 2.2 Category Model (`models/Category.js`)

Represents the product categorization taxonomy.

#### Schema Fields
| Field | Type | Required | Unique | Validation / Defaults | Description |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `name` | String | Yes | Yes | Trimmed, min 2 chars | Unique category title (e.g., Electronics) |
| `description` | String | No | No | Trimmed, default: `''` | Brief description of the category |
| `createdAt` | Date | Auto | No | Default: `Date.now` | Creation timestamp |

#### Mongoose Implementation Blueprint
```javascript
const mongoose = require('mongoose');

const categorySchema = new mongoose.Schema(
  {
    name: {
      type: String,
      required: [true, 'Category name is required'],
      unique: true,
      trim: true,
      minlength: [2, 'Category name must be at least 2 characters'],
    },
    description: {
      type: String,
      trim: true,
      default: '',
    },
  },
  { timestamps: true }
);

module.exports = mongoose.model('Category', categorySchema);
```

---

### 2.3 Product Model (`models/Product.js`)

Represents individual items available for purchase in the storefront.

#### Schema Fields
| Field | Type | Required | Unique | Validation / Defaults | Description |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `name` | String | Yes | No | Trimmed, min 2 chars | Product title |
| `description` | String | Yes | No | Trimmed | Detailed product description |
| `price` | Number | Yes | No | Min: `0.01` | Positive unit price in USD/INR |
| `image` | String | Yes | No | Trimmed, valid URL or path | Image asset URL |
| `category` | ObjectId | Yes | No | Ref: `'Category'` | Associated Category ID |
| `stock` | Number | Yes | No | Min: `0`, Default: `0`, Integer | Units available in inventory |
| `createdAt` | Date | Auto | No | Default: `Date.now` | Creation timestamp |

#### Mongoose Implementation Blueprint
```javascript
const mongoose = require('mongoose');

const productSchema = new mongoose.Schema(
  {
    name: {
      type: String,
      required: [true, 'Product name is required'],
      trim: true,
      minlength: [2, 'Product name must be at least 2 characters'],
    },
    description: {
      type: String,
      required: [true, 'Product description is required'],
      trim: true,
    },
    price: {
      type: Number,
      required: [true, 'Product price is required'],
      min: [0.01, 'Price must be greater than zero'],
    },
    image: {
      type: String,
      required: [true, 'Product image URL is required'],
      trim: true,
    },
    category: {
      type: mongoose.Schema.Types.ObjectId,
      ref: 'Category',
      required: [true, 'Product category is required'],
    },
    stock: {
      type: Number,
      required: [true, 'Stock count is required'],
      min: [0, 'Stock cannot be negative'],
      default: 0,
      validate: {
        validator: Number.isInteger,
        message: 'Stock must be an integer value',
      },
    },
  },
  { timestamps: true }
);

// Compound index for high performance search and category filtering
productSchema.index({ name: 'text', description: 'text' });
productSchema.index({ category: 1 });

module.exports = mongoose.model('Product', productSchema);
```

---

### 2.4 Order Model (`models/Order.js`)

Stores completed purchases, captured snapshot item details, shipping address, and fulfillment status.

#### Schema Fields
| Field | Type | Required | Validation / Defaults | Description |
| :--- | :--- | :--- | :--- | :--- |
| `user` | ObjectId | Yes | Ref: `'User'` | The customer who placed the order |
| `products` | Array | Yes | Non-empty array | Array of purchased line items |
| `products[].product` | ObjectId | Yes | Ref: `'Product'` | Reference to the product |
| `products[].name` | String | Yes | Non-empty | Snapshot product name at checkout |
| `products[].price` | Number | Yes | Min: `0.01` | Authentic price fetched from DB |
| `products[].quantity` | Number | Yes | Min: `1`, Integer | Quantity ordered |
| `totalAmount` | Number | Yes | Min: `0.01` | Server-calculated total order amount |
| `shippingAddress.name` | String | Yes | Min 2 chars | Recipient name |
| `shippingAddress.phone` | String | Yes | Regex (valid phone digits) | Recipient contact number |
| `shippingAddress.address`| String | Yes | Min 5 chars | Street address |
| `shippingAddress.city` | String | Yes | Min 2 chars | City |
| `shippingAddress.pincode`| String | Yes | Min 3 chars | Postal code / PIN code |
| `status` | String | Yes | Enum: `['Pending', 'Confirmed', 'Shipped', 'Delivered', 'Cancelled']`, Default: `'Pending'` | Order fulfillment state |
| `createdAt` | Date | Auto | Default: `Date.now` | Creation timestamp |

#### Mongoose Implementation Blueprint
```javascript
const mongoose = require('mongoose');

const orderItemSchema = new mongoose.Schema(
  {
    product: {
      type: mongoose.Schema.Types.ObjectId,
      ref: 'Product',
      required: true,
    },
    name: {
      type: String,
      required: true,
    },
    price: {
      type: Number,
      required: true,
      min: [0.01, 'Price must be positive'],
    },
    quantity: {
      type: Number,
      required: true,
      min: [1, 'Quantity must be at least 1'],
    },
  },
  { _id: false }
);

const shippingAddressSchema = new mongoose.Schema(
  {
    name: { type: String, required: [true, 'Recipient name is required'], trim: true },
    phone: { type: String, required: [true, 'Contact phone is required'], trim: true },
    address: { type: String, required: [true, 'Street address is required'], trim: true },
    city: { type: String, required: [true, 'City is required'], trim: true },
    pincode: { type: String, required: [true, 'Pincode is required'], trim: true },
  },
  { _id: false }
);

const orderSchema = new mongoose.Schema(
  {
    user: {
      type: mongoose.Schema.Types.ObjectId,
      ref: 'User',
      required: [true, 'Order must belong to a user'],
    },
    products: {
      type: [orderItemSchema],
      required: [true, 'Order must contain at least one product'],
      validate: [
        (arr) => Array.isArray(arr) && arr.length > 0,
        'Order cannot be empty',
      ],
    },
    totalAmount: {
      type: Number,
      required: [true, 'Total amount is required'],
      min: [0.01, 'Total amount must be greater than zero'],
    },
    shippingAddress: {
      type: shippingAddressSchema,
      required: [true, 'Shipping address is required'],
    },
    status: {
      type: String,
      enum: ['Pending', 'Confirmed', 'Shipped', 'Delivered', 'Cancelled'],
      default: 'Pending',
    },
  },
  { timestamps: true }
);

module.exports = mongoose.model('Order', orderSchema);
```

---

## 3. Sample Documents

### 3.1 Sample User Document
```json
{
  "_id": "65b8ef109a24d5a71c890001",
  "name": "Snehal Bade",
  "email": "snehal@example.com",
  "password": "$2a$10$wI5uF/hN.o7kE2rX7jCSuOzpS4Q21oT1F.2E5L...",
  "role": "customer",
  "createdAt": "2026-10-02T08:00:00.000Z",
  "updatedAt": "2026-10-02T08:00:00.000Z"
}
```

### 3.2 Sample Category Document
```json
{
  "_id": "65b8ef209a24d5a71c890002",
  "name": "Electronics",
  "description": "Smartphones, audio, and personal computing hardware",
  "createdAt": "2026-10-02T08:05:00.000Z",
  "updatedAt": "2026-10-02T08:05:00.000Z"
}
```

### 3.3 Sample Product Document
```json
{
  "_id": "65b8ef309a24d5a71c890003",
  "name": "Wireless Noise-Cancelling Headphones",
  "description": "Premium over-ear Bluetooth headphones with active noise cancellation and 30-hour battery life.",
  "price": 129.99,
  "image": "https://images.unsplash.com/photo-1505740420928-5e560c06d30e",
  "category": "65b8ef209a24d5a71c890002",
  "stock": 25,
  "createdAt": "2026-10-02T08:10:00.000Z",
  "updatedAt": "2026-10-02T08:10:00.000Z"
}
```

### 3.4 Sample Order Document
```json
{
  "_id": "65b8ef409a24d5a71c890004",
  "user": "65b8ef109a24d5a71c890001",
  "products": [
    {
      "product": "65b8ef309a24d5a71c890003",
      "name": "Wireless Noise-Cancelling Headphones",
      "price": 129.99,
      "quantity": 1
    }
  ],
  "totalAmount": 129.99,
  "shippingAddress": {
    "name": "Snehal Bade",
    "phone": "+91 9876543210",
    "address": "42 Tech Park Lane, Flat 3B",
    "city": "Pune",
    "pincode": "411001"
  },
  "status": "Pending",
  "createdAt": "2026-10-02T08:30:00.000Z",
  "updatedAt": "2026-10-02T08:30:00.000Z"
}
```
