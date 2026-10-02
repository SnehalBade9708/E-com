# Frontend UI/UX Specification

This document defines the layout specifications, UI components, responsive breakpoints, Tailwind CSS styling tokens, and user interaction patterns for the **Mini E-Commerce Demo Project**.

---

## 1. Design System & Styling Tokens

### 1.1 Styling Framework
- Built strictly with **Tailwind CSS**.
- Design aesthetic: **Clean, modern, professional, and minimal** (no visual clutter).

### 1.2 Color Palette
| Token | Hex / Class | Purpose |
| :--- | :--- | :--- |
| **Primary** | `indigo-600` (`#4F46E5`) | Brand accent, primary buttons, active links, highlights |
| **Primary Hover** | `indigo-700` (`#4338CA`) | Interactive hover states on buttons |
| **Background (App)** | `gray-50` (`#F9FAFB`) | Application background |
| **Surface (Cards/Modals)** | `white` (`#FFFFFF`) | Cards, tables, modals, input backgrounds |
| **Border** | `gray-200` (`#E5E7EB`) | Subtle dividers, card borders, input borders |
| **Text Primary** | `gray-900` (`#111827`) | Headings, titles, prices |
| **Text Secondary** | `gray-500` (`#6B7280`) | Subtitles, meta tags, labels |
| **Success** | `emerald-600` (`#059669`)| Delivered status, stock available, toast success |
| **Warning** | `amber-500` (`#F59E0B`) | Pending / Confirmed status, low stock warning |
| **Danger** | `rose-600` (`#E11D48`) | Delete buttons, cancelled status, errors |

### 1.3 Typography
- **Font Family:** Inter or system sans-serif (`font-sans`).
- Clean hierarchy:
  - `h1`: `text-2xl md:text-3xl font-bold text-gray-900`
  - `h2`: `text-xl md:text-2xl font-semibold text-gray-900`
  - `h3`: `text-lg font-semibold text-gray-800`
  - `body`: `text-sm md:text-base text-gray-600`

---

## 2. Global Layout & Components

### 2.1 Public Navigation Bar (`Navbar.jsx`)
- **Brand Logo:** "MiniStore" with shopping bag icon.
- **Navigation Links:**
  - `Home` (`/`)
  - `Products` (`/products`)
  - `My Orders` (`/my-orders` - if authenticated)
- **Cart Icon Button:**
  - Displays dynamic badge counter showing total cart item count.
  - Links to `/cart`.
- **User Authentication Action:**
  - If unauthenticated: "Login" button linking to `/login`.
  - If authenticated: Greeting dropdown with "Logout" and, if admin, link to "Admin Panel".
- **Responsive Mobile Drawer:** Hamburger toggle on screens `< 768px`.

### 2.2 Admin Layout (`AdminSidebar.jsx` & `AdminHeader.jsx`)
- **Left Sidebar Navigation:**
  - Dashboard Overview (`/admin`)
  - Categories (`/admin/categories`)
  - Products (`/admin/products`)
  - Orders (`/admin/orders`)
  - Switch to Public Store (`/`)
- **Header:** Displays current view title and logged-in admin badge.

---

## 3. Public Storefront Pages

### 3.1 Home Page (`HomePage.jsx`)
- **Hero Section:** Clean banner with bold headline ("Quality Essentials Delivered to Your Door"), concise subheading, and "Shop Now" call-to-action button linking to `/products`.
- **Category Highlights:** Quick access cards for key categories (Electronics, Fashion, Shoes).
- **Featured Products Grid:** Displays top 4 recently added products.

### 3.2 Products Catalog Page (`ProductsPage.jsx`)
- **Search Bar:** Real-time input box with search icon. Triggers debounced query `?search=<keyword>`.
- **Category Filter Pills:** Horizontal pill list:
  ```text
  [ All ] [ Electronics ] [ Fashion ] [ Shoes ]
  ```
  Active filter receives `bg-indigo-600 text-white`, inactive pills receive `bg-white text-gray-700 hover:bg-gray-100 border`.
- **Product Grid:** Responsive CSS grid:
  - Mobile: `grid-cols-1`
  - Tablet: `grid-cols-2`
  - Desktop: `grid-cols-3` or `grid-cols-4`
- **Product Card (`ProductCard.jsx`):**
  - Aspect ratio square image with hover zoom (`group-hover:scale-105 transition`).
  - Category pill badge.
  - Product title with truncation.
  - Formatted price.
  - Stock indicator (In Stock vs Out of Stock).
  - "Add to Cart" button (disabled if `stock === 0`).

### 3.3 Product Details Page (`ProductDetailPage.jsx`)
- **Breadcrumbs:** `Home / Products / <Category> / <Product Name>`.
- **Layout:** Two-column split on desktop (Image gallery on left, details on right).
- **Details:** Full title, description, category tag, price in large text.
- **Stock Availability:** Shows remaining units (`Only 3 units left!` or `In Stock: 15`).
- **Quantity Selector:** `[-] [Qty] [+]` clamped between 1 and available stock.
- **Action Button:** "Add to Cart" with toast confirmation.

### 3.4 Shopping Cart Page (`CartPage.jsx`)
- **Cart Item Table / List:**
  - Product thumbnail, name, unit price.
  - Quantity controls (`-` button, quantity text, `+` button).
    - `+` button automatically disabled when quantity equals available stock.
  - Subtotal per item (`price * quantity`).
  - Trash can icon button to remove item.
- **Order Summary Card (Sticky on desktop):**
  - Items Subtotal.
  - Shipping fee: "FREE" (COD).
  - Total Price in bold.
  - "Proceed to Checkout" button.
- **Empty State:** Friendly graphic with "Your cart is empty" and "Start Shopping" button.

### 3.5 Checkout Page (`CheckoutPage.jsx`)
- **Payment Method Badge:** Explicitly marked: `Cash on Delivery (COD) Only`.
- **Shipping Address Form:**
  - Full Name (`name`)
  - Phone Number (`phone`)
  - Street Address (`address`)
  - City (`city`)
  - Pincode / Postal Code (`pincode`)
- **Review Items Accordion:** Brief itemized list with total price.
- **Place Order Button:** Validates all fields, sends `POST /api/orders`, flushes cart on success, and routes to `/my-orders`.

### 3.6 Authentication Pages (`LoginPage.jsx` & `RegisterPage.jsx`)
- Centered card layout on soft gray background.
- Clear error display for invalid credentials or mismatched passwords.
- Redirect link between login and register.

### 3.7 My Orders Page (`MyOrdersPage.jsx`)
- List of cards representing historic customer orders.
- Each order card shows:
  - Order ID & Placed Date.
  - Status badge with color coding:
    - `Pending`: Yellow badge
    - `Confirmed`: Blue badge
    - `Shipped`: Purple badge
    - `Delivered`: Green badge
    - `Cancelled`: Red badge
  - Item list with snapshot thumbnails, names, and purchase price.
  - Total amount paid upon delivery.

---

## 4. Admin Dashboard Pages

### 4.1 Categories Management (`AdminCategoriesPage.jsx`)
- **Top Actions:** "Add Category" button opening modal.
- **Categories Table:**
  - Column 1: Category Name
  - Column 2: Description
  - Column 3: Created Date
  - Column 4: Actions (Edit icon, Delete icon)
- **Add / Edit Modal (`CategoryModal.jsx`):** Inputs for Name and Description.
- **Delete Confirmation Dialog:** Requires explicit confirmation before deletion.

### 4.2 Products Management (`AdminProductsPage.jsx`)
- **Top Actions:** "Add Product" button opening modal.
- **Products Table:**
  - Thumbnail Image
  - Product Name
  - Category Name
  - Price ($)
  - Current Stock (styled red if 0, yellow if < 5, green otherwise)
  - Actions (Edit, Delete)
- **Add / Edit Product Modal (`ProductModal.jsx`):**
  - Name, Description, Price, Image URL, Category dropdown, Stock count.

### 4.3 Orders Management (`AdminOrdersPage.jsx`)
- **Orders Table:**
  - Order ID
  - Customer Name & Email
  - Items Count & Summary
  - Total Amount
  - Status Badge with live dropdown
  - Date Placed
- **Change Status Action:**
  - Dropdown containing `Pending`, `Confirmed`, `Shipped`, `Delivered`, `Cancelled`.
  - Dispatches `PATCH /api/admin/orders/:id/status` on selection.
  - Shows success toast upon confirmation.

---

## 5. Feedback & Micro-Interactions

### 5.1 Toast Notification Component (`Toast.jsx`)
- Dispatches floating alerts in top-right or bottom-center screen.
- Types: `success`, `error`, `info`.
- Auto-dismisses after 3 seconds.

### 5.2 Confirmation Modal (`ConfirmModal.jsx`)
- Replaces native browser `window.confirm()`.
- Title: "Are you sure?"
- Content: Explanatory warning text (e.g., "Deleting this product cannot be undone.").
- Buttons: "Cancel" (gray) and "Confirm Delete" (red).

### 5.3 Loading States (`LoadingSpinner.jsx`)
- Clean circular SVG spinner with `animate-spin text-indigo-600`.
- Shown during product fetching, order submission, and authentication validation.

### 5.4 Empty States (`EmptyState.jsx`)
- Used for:
  - No products found in search or category filter.
  - Empty shopping cart.
  - No orders placed yet in "My Orders".
  - Empty admin tables.
