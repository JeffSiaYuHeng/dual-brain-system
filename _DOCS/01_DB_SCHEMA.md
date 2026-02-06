# Food Ordering System - Database Schema

## Overview
This document defines the database schema for the food ordering system using Prisma ORM with PostgreSQL.

## Schema Diagram

```
User ──< Address
  │
  └──< Order ──< OrderItem >── MenuItem
                                   │
Restaurant ──< MenuItem            │
  │            │                   │
  └──< Category ──< MenuItem       │
```

## Tables

### User
Stores customer information and authentication details.

```prisma
model User {
  id            String    @id @default(cuid())
  email         String    @unique
  passwordHash  String
  name          String
  phone         String?
  createdAt     DateTime  @default(now())
  updatedAt     DateTime  @updatedAt
  
  addresses     Address[]
  orders        Order[]
}
```

**Fields:**
- `id`: Unique identifier
- `email`: User's email (used for login)
- `passwordHash`: Hashed password
- `name`: Full name
- `phone`: Contact number
- `createdAt`: Account creation timestamp
- `updatedAt`: Last update timestamp

---

### Address
Stores delivery addresses for users.

```prisma
model Address {
  id          String   @id @default(cuid())
  userId      String
  label       String   // e.g., "Home", "Work"
  street      String
  city        String
  state       String
  zipCode     String
  isDefault   Boolean  @default(false)
  createdAt   DateTime @default(now())
  
  user        User     @relation(fields: [userId], references: [id], onDelete: Cascade)
  orders      Order[]
  
  @@index([userId])
}
```

**Fields:**
- `label`: User-friendly name for the address
- `isDefault`: Whether this is the default delivery address
- `street`, `city`, `state`, `zipCode`: Address components

---

### Restaurant
Stores restaurant information.

```prisma
model Restaurant {
  id              String     @id @default(cuid())
  name            String
  description     String?
  imageUrl        String?
  address         String
  phone           String
  rating          Float      @default(0)
  deliveryFee     Float
  minOrderAmount  Float      @default(0)
  isActive        Boolean    @default(true)
  createdAt       DateTime   @default(now())
  updatedAt       DateTime   @updatedAt
  
  categories      Category[]
  menuItems       MenuItem[]
}
```

**Fields:**
- `rating`: Average customer rating (0-5)
- `deliveryFee`: Delivery charge
- `minOrderAmount`: Minimum order value required
- `isActive`: Whether restaurant is accepting orders

---

### Category
Menu categories within a restaurant.

```prisma
model Category {
  id            String     @id @default(cuid())
  restaurantId  String
  name          String
  description   String?
  sortOrder     Int        @default(0)
  
  restaurant    Restaurant @relation(fields: [restaurantId], references: [id], onDelete: Cascade)
  menuItems     MenuItem[]
  
  @@index([restaurantId])
}
```

**Fields:**
- `sortOrder`: Display order in the menu
- `name`: Category name (e.g., "Appetizers", "Main Course")

---

### MenuItem
Individual food items available for order.

```prisma
model MenuItem {
  id            String      @id @default(cuid())
  restaurantId  String
  categoryId    String
  name          String
  description   String?
  price         Float
  imageUrl      String?
  isAvailable   Boolean     @default(true)
  isVegetarian  Boolean     @default(false)
  isVegan       Boolean     @default(false)
  spicyLevel    Int         @default(0) // 0-3
  createdAt     DateTime    @default(now())
  updatedAt     DateTime    @updatedAt
  
  restaurant    Restaurant  @relation(fields: [restaurantId], references: [id], onDelete: Cascade)
  category      Category    @relation(fields: [categoryId], references: [id], onDelete: Cascade)
  orderItems    OrderItem[]
  
  @@index([restaurantId])
  @@index([categoryId])
}
```

**Fields:**
- `isAvailable`: Whether item is currently available
- `isVegetarian`, `isVegan`: Dietary information
- `spicyLevel`: Spice level indicator (0=none, 1=mild, 2=medium, 3=hot)

---

### Order
Customer orders.

```prisma
model Order {
  id              String      @id @default(cuid())
  userId          String
  addressId       String
  restaurantId    String
  status          OrderStatus @default(PENDING)
  subtotal        Float
  deliveryFee     Float
  tax             Float
  discount        Float       @default(0)
  total           Float
  specialInstructions String?
  createdAt       DateTime    @default(now())
  updatedAt       DateTime    @updatedAt
  
  user            User        @relation(fields: [userId], references: [id])
  address         Address     @relation(fields: [addressId], references: [id])
  items           OrderItem[]
  
  @@index([userId])
  @@index([status])
}

enum OrderStatus {
  PENDING
  CONFIRMED
  PREPARING
  OUT_FOR_DELIVERY
  DELIVERED
  CANCELLED
}
```

**Fields:**
- `status`: Current order status
- `subtotal`: Sum of all items before fees
- `deliveryFee`: Delivery charge
- `tax`: Calculated tax
- `discount`: Applied discount amount
- `total`: Final amount to pay
- `specialInstructions`: Customer notes for the restaurant

---

### OrderItem
Individual items within an order.

```prisma
model OrderItem {
  id          String   @id @default(cuid())
  orderId     String
  menuItemId  String
  quantity    Int
  priceAtTime Float    // Price when ordered (for historical accuracy)
  
  order       Order    @relation(fields: [orderId], references: [id], onDelete: Cascade)
  menuItem    MenuItem @relation(fields: [menuItemId], references: [id])
  
  @@index([orderId])
}
```

**Fields:**
- `quantity`: Number of items ordered
- `priceAtTime`: Captures the price at order time (in case menu prices change)

---

## Relationships

1. **User → Address**: One-to-many (a user can have multiple addresses)
2. **User → Order**: One-to-many (a user can place multiple orders)
3. **Restaurant → Category**: One-to-many (a restaurant has multiple categories)
4. **Restaurant → MenuItem**: One-to-many (a restaurant has multiple menu items)
5. **Category → MenuItem**: One-to-many (a category contains multiple items)
6. **Order → OrderItem**: One-to-many (an order contains multiple items)
7. **MenuItem → OrderItem**: One-to-many (a menu item can appear in multiple orders)

## Indexes

- `userId` on Address, Order
- `restaurantId` on Category, MenuItem
- `categoryId` on MenuItem
- `orderId` on OrderItem
- `status` on Order (for efficient status-based queries)

## Sample Queries

### Get all menu items for a restaurant with categories
```typescript
const menu = await prisma.restaurant.findUnique({
  where: { id: restaurantId },
  include: {
    categories: {
      include: {
        menuItems: {
          where: { isAvailable: true }
        }
      },
      orderBy: { sortOrder: 'asc' }
    }
  }
});
```

### Get user's order history
```typescript
const orders = await prisma.order.findMany({
  where: { userId },
  include: {
    items: {
      include: {
        menuItem: true
      }
    },
    address: true
  },
  orderBy: { createdAt: 'desc' }
});
```

### Create a new order
```typescript
const order = await prisma.order.create({
  data: {
    userId,
    addressId,
    restaurantId,
    status: 'PENDING',
    subtotal,
    deliveryFee,
    tax,
    total,
    items: {
      create: cartItems.map(item => ({
        menuItemId: item.id,
        quantity: item.quantity,
        priceAtTime: item.price
      }))
    }
  }
});
```
