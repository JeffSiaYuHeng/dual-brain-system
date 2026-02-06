# Food Ordering System - Server Actions

## Overview
This document describes the server actions used in the food ordering system. Server actions handle data mutations and business logic on the server side.

## Authentication Actions

### `registerUser`
Creates a new user account.

```typescript
'use server'

import { hash } from 'bcrypt';
import { prisma } from '@/lib/db';
import { z } from 'zod';

const registerSchema = z.object({
  email: z.string().email(),
  password: z.string().min(8),
  name: z.string().min(2),
  phone: z.string().optional(),
});

export async function registerUser(formData: FormData) {
  const data = {
    email: formData.get('email'),
    password: formData.get('password'),
    name: formData.get('name'),
    phone: formData.get('phone'),
  };

  // Validate input
  const validated = registerSchema.parse(data);

  // Check if user exists
  const existing = await prisma.user.findUnique({
    where: { email: validated.email },
  });

  if (existing) {
    throw new Error('User already exists');
  }

  // Hash password
  const passwordHash = await hash(validated.password, 10);

  // Create user
  const user = await prisma.user.create({
    data: {
      email: validated.email,
      passwordHash,
      name: validated.name,
      phone: validated.phone,
    },
  });

  return { success: true, userId: user.id };
}
```

**Usage:**
```tsx
<form action={registerUser}>
  <input name="email" type="email" required />
  <input name="password" type="password" required />
  <input name="name" type="text" required />
  <input name="phone" type="tel" />
  <button type="submit">Register</button>
</form>
```

---

### `loginUser`
Authenticates a user and creates a session.

```typescript
'use server'

import { compare } from 'bcrypt';
import { prisma } from '@/lib/db';
import { createSession } from '@/lib/auth';
import { redirect } from 'next/navigation';

export async function loginUser(formData: FormData) {
  const email = formData.get('email') as string;
  const password = formData.get('password') as string;

  // Find user
  const user = await prisma.user.findUnique({
    where: { email },
  });

  if (!user) {
    throw new Error('Invalid credentials');
  }

  // Verify password
  const isValid = await compare(password, user.passwordHash);

  if (!isValid) {
    throw new Error('Invalid credentials');
  }

  // Create session
  await createSession(user.id);

  redirect('/');
}
```

---

## Menu Actions

### `getRestaurants`
Fetches all active restaurants.

```typescript
'use server'

import { prisma } from '@/lib/db';

export async function getRestaurants() {
  const restaurants = await prisma.restaurant.findMany({
    where: { isActive: true },
    orderBy: { rating: 'desc' },
  });

  return restaurants;
}
```

---

### `getRestaurantMenu`
Fetches a restaurant's menu with categories and items.

```typescript
'use server'

import { prisma } from '@/lib/db';

export async function getRestaurantMenu(restaurantId: string) {
  const restaurant = await prisma.restaurant.findUnique({
    where: { id: restaurantId },
    include: {
      categories: {
        include: {
          menuItems: {
            where: { isAvailable: true },
            orderBy: { name: 'asc' },
          },
        },
        orderBy: { sortOrder: 'asc' },
      },
    },
  });

  if (!restaurant) {
    throw new Error('Restaurant not found');
  }

  return restaurant;
}
```

---

### `searchMenuItems`
Searches for menu items across all restaurants.

```typescript
'use server'

import { prisma } from '@/lib/db';

export async function searchMenuItems(query: string) {
  const items = await prisma.menuItem.findMany({
    where: {
      AND: [
        { isAvailable: true },
        {
          OR: [
            { name: { contains: query, mode: 'insensitive' } },
            { description: { contains: query, mode: 'insensitive' } },
          ],
        },
      ],
    },
    include: {
      restaurant: true,
      category: true,
    },
    take: 20,
  });

  return items;
}
```

---

## Order Actions

### `createOrder`
Creates a new order from cart items.

```typescript
'use server'

import { prisma } from '@/lib/db';
import { getCurrentUser } from '@/lib/auth';
import { z } from 'zod';

const orderSchema = z.object({
  addressId: z.string(),
  restaurantId: z.string(),
  items: z.array(z.object({
    menuItemId: z.string(),
    quantity: z.number().min(1),
  })),
  specialInstructions: z.string().optional(),
});

export async function createOrder(data: z.infer<typeof orderSchema>) {
  const user = await getCurrentUser();
  
  if (!user) {
    throw new Error('Not authenticated');
  }

  // Validate input
  const validated = orderSchema.parse(data);

  // Get menu items with prices
  const menuItems = await prisma.menuItem.findMany({
    where: {
      id: { in: validated.items.map(item => item.menuItemId) },
    },
  });

  // Calculate totals
  const subtotal = validated.items.reduce((sum, item) => {
    const menuItem = menuItems.find(mi => mi.id === item.menuItemId);
    return sum + (menuItem?.price || 0) * item.quantity;
  }, 0);

  const restaurant = await prisma.restaurant.findUnique({
    where: { id: validated.restaurantId },
  });

  const deliveryFee = restaurant?.deliveryFee || 0;
  const tax = subtotal * 0.08; // 8% tax
  const total = subtotal + deliveryFee + tax;

  // Create order
  const order = await prisma.order.create({
    data: {
      userId: user.id,
      addressId: validated.addressId,
      restaurantId: validated.restaurantId,
      status: 'PENDING',
      subtotal,
      deliveryFee,
      tax,
      total,
      specialInstructions: validated.specialInstructions,
      items: {
        create: validated.items.map(item => {
          const menuItem = menuItems.find(mi => mi.id === item.menuItemId);
          return {
            menuItemId: item.menuItemId,
            quantity: item.quantity,
            priceAtTime: menuItem?.price || 0,
          };
        }),
      },
    },
    include: {
      items: {
        include: {
          menuItem: true,
        },
      },
    },
  });

  return order;
}
```

**Usage:**
```tsx
const handleCheckout = async () => {
  const order = await createOrder({
    addressId: selectedAddress.id,
    restaurantId: cart.restaurantId,
    items: cart.items.map(item => ({
      menuItemId: item.id,
      quantity: item.quantity,
    })),
    specialInstructions: instructions,
  });
  
  router.push(`/orders/${order.id}`);
};
```

---

### `getOrderDetails`
Fetches details of a specific order.

```typescript
'use server'

import { prisma } from '@/lib/db';
import { getCurrentUser } from '@/lib/auth';

export async function getOrderDetails(orderId: string) {
  const user = await getCurrentUser();
  
  if (!user) {
    throw new Error('Not authenticated');
  }

  const order = await prisma.order.findUnique({
    where: { id: orderId },
    include: {
      items: {
        include: {
          menuItem: true,
        },
      },
      address: true,
    },
  });

  if (!order || order.userId !== user.id) {
    throw new Error('Order not found');
  }

  return order;
}
```

---

### `updateOrderStatus`
Updates the status of an order (admin/restaurant use).

```typescript
'use server'

import { prisma } from '@/lib/db';
import { OrderStatus } from '@prisma/client';

export async function updateOrderStatus(
  orderId: string,
  status: OrderStatus
) {
  const order = await prisma.order.update({
    where: { id: orderId },
    data: { status },
  });

  return order;
}
```

---

### `cancelOrder`
Cancels an order if it hasn't started preparing.

```typescript
'use server'

import { prisma } from '@/lib/db';
import { getCurrentUser } from '@/lib/auth';

export async function cancelOrder(orderId: string) {
  const user = await getCurrentUser();
  
  if (!user) {
    throw new Error('Not authenticated');
  }

  const order = await prisma.order.findUnique({
    where: { id: orderId },
  });

  if (!order || order.userId !== user.id) {
    throw new Error('Order not found');
  }

  if (order.status !== 'PENDING' && order.status !== 'CONFIRMED') {
    throw new Error('Cannot cancel order in current status');
  }

  const updated = await prisma.order.update({
    where: { id: orderId },
    data: { status: 'CANCELLED' },
  });

  return updated;
}
```

---

### `getUserOrders`
Fetches all orders for the current user.

```typescript
'use server'

import { prisma } from '@/lib/db';
import { getCurrentUser } from '@/lib/auth';

export async function getUserOrders() {
  const user = await getCurrentUser();
  
  if (!user) {
    throw new Error('Not authenticated');
  }

  const orders = await prisma.order.findMany({
    where: { userId: user.id },
    include: {
      items: {
        include: {
          menuItem: true,
        },
      },
      address: true,
    },
    orderBy: { createdAt: 'desc' },
  });

  return orders;
}
```

---

## Address Actions

### `addAddress`
Adds a new delivery address for the user.

```typescript
'use server'

import { prisma } from '@/lib/db';
import { getCurrentUser } from '@/lib/auth';
import { z } from 'zod';

const addressSchema = z.object({
  label: z.string(),
  street: z.string(),
  city: z.string(),
  state: z.string(),
  zipCode: z.string(),
  isDefault: z.boolean().optional(),
});

export async function addAddress(data: z.infer<typeof addressSchema>) {
  const user = await getCurrentUser();
  
  if (!user) {
    throw new Error('Not authenticated');
  }

  const validated = addressSchema.parse(data);

  // If this is set as default, unset other defaults
  if (validated.isDefault) {
    await prisma.address.updateMany({
      where: { userId: user.id, isDefault: true },
      data: { isDefault: false },
    });
  }

  const address = await prisma.address.create({
    data: {
      ...validated,
      userId: user.id,
    },
  });

  return address;
}
```

---

## Error Handling

All server actions should handle errors gracefully:

```typescript
export async function someAction(data: unknown) {
  try {
    // Validate input
    const validated = schema.parse(data);
    
    // Perform action
    const result = await performAction(validated);
    
    return { success: true, data: result };
  } catch (error) {
    if (error instanceof z.ZodError) {
      return { success: false, error: 'Invalid input', details: error.errors };
    }
    
    return { success: false, error: 'Something went wrong' };
  }
}
```

## Best Practices

1. **Always validate input** using Zod or similar
2. **Check authentication** for protected actions
3. **Use transactions** for multi-step operations
4. **Return typed responses** with success/error states
5. **Log errors** for debugging
6. **Rate limit** sensitive actions
7. **Sanitize user input** to prevent injection attacks
