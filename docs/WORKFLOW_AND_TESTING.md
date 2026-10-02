# Workflow & Testing Guide

This document outlines the step-by-step demonstration walkthrough, testing procedures, seed data setup, and validation checks for the **Mini E-Commerce Demo Project**.

---

## 1. End-to-End Demo Workflow

The primary end-to-end demonstration flow tests every critical subsystem of the MERN architecture:

```text
  ┌────────────────────────────────────────────────────────┐
  │                    1. Admin Login                      │
  └───────────────────────────┬────────────────────────────┘
                              │
  ┌───────────────────────────▼────────────────────────────┐
  │                   2. Add Category                      │
  │     e.g., "Electronics", "Fashion", "Shoes"            │
  └───────────────────────────┬────────────────────────────┘
                              │
  ┌───────────────────────────▼────────────────────────────┐
  │                   3. Add Product                       │
  │  Set Name, Description, Price, Image, Stock: 10        │
  └───────────────────────────┬────────────────────────────┘
                              │
  ┌───────────────────────────▼────────────────────────────┐
  │          4. Product Appears on Public Website          │
  └───────────────────────────┬────────────────────────────┘
                              │
  ┌───────────────────────────▼────────────────────────────┐
  │                5. User Register / Login                │
  └───────────────────────────┬────────────────────────────┘
                              │
  ┌───────────────────────────▼────────────────────────────┐
  │            6. Browse Catalog & Filter                  │
  │     Filter by category pill or search by keyword       │
  └───────────────────────────┬────────────────────────────┘
                              │
  ┌───────────────────────────▼────────────────────────────┐
  │                   7. Add to Cart                       │
  │       Select item quantity & verify stock cap          │
  └───────────────────────────┬────────────────────────────┘
                              │
  ┌───────────────────────────▼────────────────────────────┐
  │             8. Proceed to Checkout                     │
  │   Enter Name, Phone, Address, City, Pincode (COD)      │
  └───────────────────────────┬────────────────────────────┘
                              │
  ┌───────────────────────────▼────────────────────────────┐
  │                   9. Place Order                       │
  │  Server validates price & decrements stock (10 -> 8)   │
  └───────────────────────────┬────────────────────────────┘
                              │
  ┌───────────────────────────▼────────────────────────────┐
  │        10. Order Saved in MongoDB & Cart Cleared       │
  └───────────────────────────┬────────────────────────────┘
                              │
  ┌───────────────────────────▼────────────────────────────┐
  │          11. User Views Order in "My Orders"           │
  │               Status shows as "Pending"                │
  └───────────────────────────┬────────────────────────────┘
                              │
  ┌───────────────────────────▼────────────────────────────┐
  │            12. Admin Updates Order Status              │
  │  Admin transitions status: Pending -> Confirmed        │
  │                          -> Shipped -> Delivered       │
  └────────────────────────────────────────────────────────┘
```

---

## 2. Step-by-Step Walkthrough Guide

### Step 1: Administrator Login
1. Navigate to `/login`.
2. Enter pre-seeded administrative credentials:
   - **Email:** `admin@example.com`
   - **Password:** `admin123`
3. Click **Sign In**.
4. Confirm redirection to the **Admin Dashboard** (`/admin`).

### Step 2: Create Product Categories
1. In the Admin Sidebar, click **Categories** (`/admin/categories`).
2. Click **+ Add Category**.
3. Create the following standard taxonomy:
   - **Category 1:** Name: `Electronics`, Description: `Gadgets and hardware`
   - **Category 2:** Name: `Fashion`, Description: `Apparel and clothing`
   - **Category 3:** Name: `Shoes`, Description: `Footwear and sneakers`
4. Verify all categories appear in the admin table.

### Step 3: Add Inventory Products
1. In the Admin Sidebar, click **Products** (`/admin/products`).
2. Click **+ Add Product**.
3. Create a test item:
   - **Name:** `Wireless Noise-Cancelling Headphones`
   - **Description:** `High-fidelity wireless Bluetooth headphones with 30-hour battery life.`
   - **Price:** `129.99`
   - **Image:** `https://images.unsplash.com/photo-1505740420928-5e560c06d30e`
   - **Category:** Select `Electronics`
   - **Stock:** `10`
4. Click **Save Product**. Verify the item renders in the product table with green stock badge.

### Step 4: Verify Public Storefront Visibility
1. Click the **View Store** link or navigate to `/products`.
2. Confirm the newly created headphones appear in the grid.
3. Test the category filter:
   - Click `Shoes` ➔ Headphones disappear, empty state renders.
   - Click `Electronics` ➔ Headphones reappear.
   - Click `All` ➔ All items render.
4. Test the search bar:
   - Type `Headphones` ➔ Product remains visible.
   - Type `Laptop` ➔ Empty state renders.

### Step 5: Customer Registration
1. In the navbar, click **Logout** (exiting admin session).
2. Click **Register** (`/register`).
3. Fill out registration details:
   - **Name:** `John Doe`
   - **Email:** `john@example.com`
   - **Password:** `password123`
   - **Confirm Password:** `password123`
4. Submit form. Confirm instant login with customer navigation options.

### Step 6: Add to Cart & Stock Clamping
1. Click on the headphones product card to view `/products/:id`.
2. Test quantity limit:
   - Increase quantity to `10`.
   - Attempt to click `+` again ➔ Disabled button prevents exceeding `10`.
3. Set quantity to `2`.
4. Click **Add to Cart**. A confirmation toast appears, and the navbar cart badge updates to `2`.

### Step 7: Cart Review
1. Click the Cart icon in the navbar (`/cart`).
2. Verify:
   - Item row with thumbnail, title, price (`$129.99`), quantity (`2`), and subtotal (`$259.98`).
   - Order total accurately displays `$259.98`.
3. Click **Proceed to Checkout**.

### Step 8: Checkout & Order Placement
1. Navigate to `/checkout`.
2. Notice the payment indicator: **Cash on Delivery (COD)**.
3. Complete the shipping address form:
   - **Full Name:** `John Doe`
   - **Phone:** `+1 555-0199`
   - **Address:** `123 Elm Street, Apt 4B`
   - **City:** `Springfield`
   - **Pincode:** `97477`
4. Click **Place Order**.
5. System displays success toast, cart flushes to 0, and user is routed to `/my-orders`.

### Step 9: Customer Order Verification
1. On `/my-orders`, verify the new order card:
   - **Status:** `Pending` (amber badge).
   - **Total Amount:** `$259.98`.
   - **Items:** 2x `Wireless Noise-Cancelling Headphones`.

### Step 10: Admin Order Fulfillment
1. Log out from customer account.
2. Log back in with Admin credentials (`admin@example.com`).
3. Navigate to **Orders** (`/admin/orders`).
4. Find John Doe's order.
5. In the status dropdown, change status from `Pending` ➔ `Confirmed` ➔ `Shipped` ➔ `Delivered`.
6. Navigate to **Products** (`/admin/products`).
7. Confirm that stock for headphones has automatically decremented from `10` to `8`.

---

## 3. Test Scenarios & Edge Cases

| Test Case | Scenario / Steps | Expected Result |
| :--- | :--- | :--- |
| **TC-01: Duplicate Email** | Register customer with an email already present in DB | `400 Bad Request`: "Email already in use". Form displays clear error. |
| **TC-02: Password Mismatch** | Register with `password` != `confirmPassword` | Client validation blocks submission; displays mismatch warning. |
| **TC-03: Unauthorized Admin Route** | Customer token attempts `POST /api/products` | `403 Forbidden`: "Access forbidden: Administrator privileges required". |
| **TC-04: Negative Stock Rejection** | Admin submits product with `stock = -5` | `400 Bad Request`: "Stock cannot be negative". |
| **TC-05: Price Tampering Defense** | Attacker tampers request payload `price: 0.01` | Backend ignores submitted price, fetches DB price, calculates valid total. |
| **TC-06: Stock Depletion Guard** | Customer orders 9 units when only 8 remain in stock | `400 Bad Request`: "Insufficient stock...". Order rejected. |
| **TC-07: Cart Stock Clamp** | User clicks `+` in Cart when quantity == stock | Button is disabled; quantity cannot exceed stock. |
| **TC-08: Empty Cart Checkout** | Direct URL visit to `/checkout` with empty cart | Redirects user back to `/cart` or `/products` with empty notification. |
