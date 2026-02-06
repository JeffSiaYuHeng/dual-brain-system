# Food Ordering System - Project Structure

## Overview
This is a simple food ordering system that allows customers to browse menus, place orders, and track their delivery status.

## Directory Structure

```
food-ordering-system/
├── src/
│   ├── components/          # React components
│   │   ├── Menu/           # Menu display components
│   │   ├── Cart/           # Shopping cart components
│   │   ├── Order/          # Order management components
│   │   └── Auth/           # Authentication components
│   ├── pages/              # Next.js pages
│   │   ├── index.tsx       # Home page with restaurant list
│   │   ├── menu/[id].tsx   # Restaurant menu page
│   │   ├── cart.tsx        # Shopping cart page
│   │   ├── checkout.tsx    # Checkout page
│   │   └── orders.tsx      # Order history page
│   ├── lib/                # Utility functions and helpers
│   │   ├── db.ts           # Database connection
│   │   ├── auth.ts         # Authentication utilities
│   │   └── validation.ts   # Input validation
│   ├── actions/            # Server actions
│   │   ├── menu.ts         # Menu-related actions
│   │   ├── order.ts        # Order-related actions
│   │   └── user.ts         # User-related actions
│   └── types/              # TypeScript type definitions
│       ├── menu.ts
│       ├── order.ts
│       └── user.ts
├── public/                 # Static assets
│   ├── images/            # Food images
│   └── icons/             # UI icons
└── prisma/                # Database schema
    └── schema.prisma
```

## Core Features

### 1. Restaurant & Menu Management
- Browse available restaurants
- View restaurant menus with categories
- Search and filter menu items
- View item details (description, price, ingredients)

### 2. Shopping Cart
- Add/remove items from cart
- Adjust item quantities
- Apply discount codes
- Calculate totals with tax and delivery fee

### 3. Order Management
- Place orders with delivery details
- Track order status (pending, preparing, delivering, completed)
- View order history
- Cancel orders (if not yet preparing)

### 4. User Authentication
- User registration and login
- Save delivery addresses
- Manage payment methods
- View order history

## Component Architecture

### Menu Components
- `RestaurantList`: Displays all available restaurants
- `MenuCategory`: Shows menu items by category
- `MenuItem`: Individual menu item card
- `ItemDetail`: Detailed view with customization options

### Cart Components
- `CartSummary`: Shows cart items and totals
- `CartItem`: Individual cart item with quantity controls
- `CheckoutForm`: Delivery and payment information form

### Order Components
- `OrderCard`: Summary of a single order
- `OrderStatus`: Real-time order tracking
- `OrderHistory`: List of past orders

## Data Flow

1. **Browse**: User browses restaurants and menus
2. **Select**: User adds items to cart
3. **Review**: User reviews cart and applies discounts
4. **Checkout**: User provides delivery and payment info
5. **Confirm**: Order is placed and confirmed
6. **Track**: User can track order status
7. **Complete**: Order is delivered and marked complete

## State Management

- **Cart State**: Managed via React Context or Zustand
- **User State**: Managed via authentication provider
- **Order State**: Fetched from server and cached

## API Routes

- `GET /api/restaurants` - List all restaurants
- `GET /api/menu/:restaurantId` - Get restaurant menu
- `POST /api/cart/add` - Add item to cart
- `POST /api/order/create` - Create new order
- `GET /api/order/:orderId` - Get order details
- `PATCH /api/order/:orderId/status` - Update order status
