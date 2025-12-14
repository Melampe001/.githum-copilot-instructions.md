# E-commerce Template - Online Store Platform

## 🛒 Ejemplo de uso

```typescript
// PROYECTO: Tienda online de productos artesanales con carrito y pagos
// Stack: Next.js + Stripe + Prisma + PostgreSQL + Tailwind CSS
```

## 📦 Stack Generado

- **Frontend**: Next.js 14 with App Router
- **Backend**: Next.js API Routes
- **Lenguaje**: TypeScript 5.x
- **Database**: PostgreSQL + Prisma
- **Payments**: Stripe / PayPal / Mercadopago
- **Styling**: Tailwind CSS + shadcn/ui
- **State**: Zustand / Redux Toolkit
- **Email**: SendGrid / Nodemailer
- **Search**: Algolia / MeiliSearch
- **Testing**: Jest + Playwright
- **Deployment**: Vercel / Docker

## 📁 Estructura

```
ecommerce-project/
├── src/
│   ├── app/
│   │   ├── (shop)/
│   │   │   ├── products/
│   │   │   ├── cart/
│   │   │   └── checkout/
│   │   ├── (admin)/
│   │   │   ├── dashboard/
│   │   │   ├── products/
│   │   │   └── orders/
│   │   ├── api/
│   │   │   ├── products/
│   │   │   ├── cart/
│   │   │   ├── orders/
│   │   │   └── payments/
│   │   ├── layout.tsx
│   │   └── page.tsx
│   ├── components/
│   │   ├── shop/
│   │   ├── admin/
│   │   └── ui/
│   ├── lib/
│   │   ├── stripe.ts
│   │   ├── prisma.ts
│   │   └── email.ts
│   ├── hooks/
│   ├── store/
│   ├── types/
│   └── utils/
├── prisma/
│   ├── schema.prisma
│   └── seed.ts
├── public/
│   └── products/
├── tests/
└── package.json
```

## 🎯 Características Incluidas

### Shop Features
- Product catalog with categories
- Product search and filters
- Shopping cart (session/persistent)
- Checkout process
- Multiple payment methods
- Order tracking
- User accounts
- Wishlist
- Product reviews
- Inventory management

### Admin Features
- Product CRUD
- Order management
- Customer management
- Analytics dashboard
- Inventory tracking
- Discount/coupon codes
- Email notifications
- Export reports

### Payment Integration
- Stripe checkout
- PayPal integration
- Webhook handling
- Refund management
- Invoice generation

## 📝 Ejemplo: Next.js E-commerce

### Product Page
```typescript
// src/app/(shop)/products/[id]/page.tsx
import { Metadata } from 'next';
import { notFound } from 'next/navigation';
import { getProductById } from '@/lib/api/products';
import { ProductDetails } from '@/components/shop/ProductDetails';
import { AddToCartButton } from '@/components/shop/AddToCartButton';
import { ProductReviews } from '@/components/shop/ProductReviews';

interface ProductPageProps {
  params: { id: string };
}

export async function generateMetadata({
  params,
}: ProductPageProps): Promise<Metadata> {
  const product = await getProductById(params.id);

  if (!product) {
    return { title: 'Product Not Found' };
  }

  return {
    title: product.name,
    description: product.description,
    openGraph: {
      images: [product.images[0]],
    },
  };
}

export default async function ProductPage({ params }: ProductPageProps) {
  const product = await getProductById(params.id);

  if (!product) {
    notFound();
  }

  return (
    <div className="container mx-auto px-4 py-8">
      <div className="grid grid-cols-1 md:grid-cols-2 gap-8">
        <ProductDetails product={product} />
        
        <div>
          <h1 className="text-3xl font-bold mb-4">{product.name}</h1>
          <p className="text-2xl font-semibold mb-4">
            ${product.price.toFixed(2)}
          </p>
          
          <p className="text-gray-600 mb-6">{product.description}</p>
          
          <AddToCartButton product={product} />
          
          {product.stock > 0 ? (
            <p className="text-green-600 mt-2">In Stock ({product.stock})</p>
          ) : (
            <p className="text-red-600 mt-2">Out of Stock</p>
          )}
        </div>
      </div>
      
      <ProductReviews productId={product.id} />
    </div>
  );
}
```

### Shopping Cart
```typescript
// src/store/cart.ts
import { create } from 'zustand';
import { persist } from 'zustand/middleware';

interface CartItem {
  id: string;
  name: string;
  price: number;
  quantity: number;
  image: string;
}

interface CartStore {
  items: CartItem[];
  addItem: (item: CartItem) => void;
  removeItem: (id: string) => void;
  updateQuantity: (id: string, quantity: number) => void;
  clearCart: () => void;
  getTotalPrice: () => number;
  getTotalItems: () => number;
}

export const useCartStore = create<CartStore>()(
  persist(
    (set, get) => ({
      items: [],

      addItem: (item) => {
        set((state) => {
          const existingItem = state.items.find((i) => i.id === item.id);

          if (existingItem) {
            return {
              items: state.items.map((i) =>
                i.id === item.id
                  ? { ...i, quantity: i.quantity + item.quantity }
                  : i
              ),
            };
          }

          return { items: [...state.items, item] };
        });
      },

      removeItem: (id) => {
        set((state) => ({
          items: state.items.filter((item) => item.id !== id),
        }));
      },

      updateQuantity: (id, quantity) => {
        set((state) => ({
          items: state.items.map((item) =>
            item.id === id ? { ...item, quantity } : item
          ),
        }));
      },

      clearCart: () => set({ items: [] }),

      getTotalPrice: () => {
        return get().items.reduce(
          (total, item) => total + item.price * item.quantity,
          0
        );
      },

      getTotalItems: () => {
        return get().items.reduce((total, item) => total + item.quantity, 0);
      },
    }),
    {
      name: 'cart-storage',
    }
  )
);
```

### Checkout API
```typescript
// src/app/api/checkout/route.ts
import { NextRequest, NextResponse } from 'next/server';
import Stripe from 'stripe';
import { prisma } from '@/lib/prisma';
import { getServerSession } from 'next-auth';

const stripe = new Stripe(process.env.STRIPE_SECRET_KEY!, {
  apiVersion: '2023-10-16',
});

export async function POST(req: NextRequest) {
  try {
    const session = await getServerSession();
    if (!session?.user) {
      return NextResponse.json(
        { error: 'Authentication required' },
        { status: 401 }
      );
    }

    const { items, shippingAddress } = await req.json();

    // Validate items and calculate total
    const productIds = items.map((item: any) => item.productId);
    const products = await prisma.product.findMany({
      where: { id: { in: productIds } },
    });

    let total = 0;
    const lineItems = products.map((product) => {
      const item = items.find((i: any) => i.productId === product.id);
      total += product.price * item.quantity;

      return {
        price_data: {
          currency: 'usd',
          product_data: {
            name: product.name,
            images: [product.images[0]],
          },
          unit_amount: Math.round(product.price * 100),
        },
        quantity: item.quantity,
      };
    });

    // Create order in database
    const order = await prisma.order.create({
      data: {
        userId: session.user.id,
        total,
        status: 'PENDING',
        shippingAddress,
        items: {
          create: items.map((item: any) => ({
            productId: item.productId,
            quantity: item.quantity,
            price: products.find((p) => p.id === item.productId)!.price,
          })),
        },
      },
    });

    // Create Stripe checkout session
    const checkoutSession = await stripe.checkout.sessions.create({
      payment_method_types: ['card'],
      line_items: lineItems,
      mode: 'payment',
      success_url: `${process.env.NEXT_PUBLIC_URL}/order/${order.id}/success`,
      cancel_url: `${process.env.NEXT_PUBLIC_URL}/cart`,
      metadata: {
        orderId: order.id,
      },
    });

    return NextResponse.json({ url: checkoutSession.url });
  } catch (error) {
    console.error('Checkout error:', error);
    return NextResponse.json(
      { error: 'Failed to create checkout session' },
      { status: 500 }
    );
  }
}
```

### Stripe Webhook
```typescript
// src/app/api/webhooks/stripe/route.ts
import { NextRequest, NextResponse } from 'next/server';
import Stripe from 'stripe';
import { prisma } from '@/lib/prisma';
import { sendOrderConfirmationEmail } from '@/lib/email';

const stripe = new Stripe(process.env.STRIPE_SECRET_KEY!, {
  apiVersion: '2023-10-16',
});

const webhookSecret = process.env.STRIPE_WEBHOOK_SECRET!;

export async function POST(req: NextRequest) {
  const body = await req.text();
  const signature = req.headers.get('stripe-signature')!;

  let event: Stripe.Event;

  try {
    event = stripe.webhooks.constructEvent(body, signature, webhookSecret);
  } catch (err) {
    console.error('Webhook signature verification failed:', err);
    return NextResponse.json(
      { error: 'Invalid signature' },
      { status: 400 }
    );
  }

  if (event.type === 'checkout.session.completed') {
    const session = event.data.object as Stripe.Checkout.Session;
    const orderId = session.metadata?.orderId;

    if (orderId) {
      // Update order status
      const order = await prisma.order.update({
        where: { id: orderId },
        data: {
          status: 'PAID',
          stripePaymentId: session.payment_intent as string,
        },
        include: {
          user: true,
          items: {
            include: {
              product: true,
            },
          },
        },
      });

      // Update inventory
      for (const item of order.items) {
        await prisma.product.update({
          where: { id: item.productId },
          data: {
            stock: {
              decrement: item.quantity,
            },
          },
        });
      }

      // Send confirmation email
      await sendOrderConfirmationEmail(order);
    }
  }

  return NextResponse.json({ received: true });
}
```

### Prisma Schema
```prisma
// prisma/schema.prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model User {
  id            String    @id @default(uuid())
  email         String    @unique
  name          String
  password      String
  role          Role      @default(CUSTOMER)
  orders        Order[]
  reviews       Review[]
  createdAt     DateTime  @default(now())
  updatedAt     DateTime  @updatedAt
}

model Product {
  id          String      @id @default(uuid())
  name        String
  slug        String      @unique
  description String
  price       Float
  images      String[]
  stock       Int         @default(0)
  categoryId  String
  category    Category    @relation(fields: [categoryId], references: [id])
  orderItems  OrderItem[]
  reviews     Review[]
  createdAt   DateTime    @default(now())
  updatedAt   DateTime    @updatedAt

  @@index([slug])
  @@index([categoryId])
}

model Category {
  id       String    @id @default(uuid())
  name     String
  slug     String    @unique
  products Product[]
}

model Order {
  id              String      @id @default(uuid())
  userId          String
  user            User        @relation(fields: [userId], references: [id])
  items           OrderItem[]
  total           Float
  status          OrderStatus @default(PENDING)
  shippingAddress Json
  stripePaymentId String?
  createdAt       DateTime    @default(now())
  updatedAt       DateTime    @updatedAt

  @@index([userId])
}

model OrderItem {
  id        String  @id @default(uuid())
  orderId   String
  order     Order   @relation(fields: [orderId], references: [id])
  productId String
  product   Product @relation(fields: [productId], references: [id])
  quantity  Int
  price     Float

  @@index([orderId])
}

model Review {
  id        String   @id @default(uuid())
  productId String
  product   Product  @relation(fields: [productId], references: [id])
  userId    String
  user      User     @relation(fields: [userId], references: [id])
  rating    Int
  comment   String
  createdAt DateTime @default(now())

  @@index([productId])
}

enum Role {
  CUSTOMER
  ADMIN
}

enum OrderStatus {
  PENDING
  PAID
  PROCESSING
  SHIPPED
  DELIVERED
  CANCELLED
}
```

## 📦 Dependencies

```json
{
  "dependencies": {
    "next": "^14.0.0",
    "react": "^18.2.0",
    "react-dom": "^18.2.0",
    "typescript": "^5.0.0",
    "@prisma/client": "^5.5.0",
    "prisma": "^5.5.0",
    "stripe": "^14.0.0",
    "next-auth": "^4.24.0",
    "zustand": "^4.4.0",
    "zod": "^3.22.0",
    "react-hook-form": "^7.48.0",
    "@hookform/resolvers": "^3.3.0",
    "tailwindcss": "^3.3.0",
    "clsx": "^2.0.0",
    "lucide-react": "^0.292.0"
  },
  "devDependencies": {
    "@types/react": "^18.2.0",
    "@types/node": "^20.8.0",
    "jest": "^29.7.0",
    "@testing-library/react": "^14.0.0",
    "playwright": "^1.40.0",
    "eslint": "^8.50.0",
    "prettier": "^3.0.0"
  }
}
```

## 🚀 Scripts

```json
{
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "lint": "next lint",
    "test": "jest",
    "test:e2e": "playwright test",
    "prisma:generate": "prisma generate",
    "prisma:migrate": "prisma migrate dev",
    "prisma:seed": "ts-node prisma/seed.ts",
    "prisma:studio": "prisma studio"
  }
}
```

## 🔐 Environment Variables

```env
# .env.example
DATABASE_URL=postgresql://user:password@localhost:5432/ecommerce
NEXT_PUBLIC_URL=http://localhost:3000

# Stripe
STRIPE_SECRET_KEY=sk_test_...
NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY=pk_test_...
STRIPE_WEBHOOK_SECRET=whsec_...

# Auth
NEXTAUTH_SECRET=your-secret-here
NEXTAUTH_URL=http://localhost:3000

# Email
SENDGRID_API_KEY=your-sendgrid-key
EMAIL_FROM=noreply@yourstore.com
```

## 🧪 Tests

```typescript
// tests/e2e/checkout.spec.ts
import { test, expect } from '@playwright/test';

test.describe('Checkout Flow', () => {
  test('complete purchase', async ({ page }) => {
    // Add product to cart
    await page.goto('/products/1');
    await page.click('button:has-text("Add to Cart")');
    
    // Go to cart
    await page.click('a:has-text("Cart")');
    await expect(page.locator('.cart-item')).toHaveCount(1);
    
    // Proceed to checkout
    await page.click('button:has-text("Checkout")');
    
    // Fill shipping info
    await page.fill('input[name="address"]', '123 Main St');
    await page.fill('input[name="city"]', 'New York');
    await page.fill('input[name="zipCode"]', '10001');
    
    // Complete payment (test mode)
    await page.click('button:has-text("Pay Now")');
    
    // Verify success
    await expect(page).toHaveURL(/\/order\/.*\/success/);
    await expect(page.locator('text=Order Confirmed')).toBeVisible();
  });
});
```

## 🐳 Docker

```dockerfile
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
COPY prisma ./prisma/
RUN npm ci
RUN npx prisma generate
COPY . .
RUN npm run build

FROM node:20-alpine AS runner
WORKDIR /app
ENV NODE_ENV production
COPY --from=builder /app/next.config.js ./
COPY --from=builder /app/public ./public
COPY --from=builder /app/.next ./.next
COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app/package.json ./package.json
COPY --from=builder /app/prisma ./prisma
EXPOSE 3000
CMD ["sh", "-c", "npx prisma migrate deploy && npm start"]
```

## 📊 Admin Dashboard

```typescript
// src/app/(admin)/dashboard/page.tsx
import { getOrderStats, getRevenueStats } from '@/lib/api/stats';
import { Card } from '@/components/ui/Card';
import { Chart } from '@/components/ui/Chart';

export default async function AdminDashboard() {
  const orderStats = await getOrderStats();
  const revenueStats = await getRevenueStats();

  return (
    <div className="container mx-auto px-4 py-8">
      <h1 className="text-3xl font-bold mb-8">Admin Dashboard</h1>
      
      <div className="grid grid-cols-1 md:grid-cols-3 gap-6 mb-8">
        <Card>
          <h3 className="text-lg font-semibold">Total Orders</h3>
          <p className="text-3xl font-bold">{orderStats.total}</p>
        </Card>
        
        <Card>
          <h3 className="text-lg font-semibold">Revenue (Month)</h3>
          <p className="text-3xl font-bold">
            ${revenueStats.monthly.toFixed(2)}
          </p>
        </Card>
        
        <Card>
          <h3 className="text-lg font-semibold">Pending Orders</h3>
          <p className="text-3xl font-bold">{orderStats.pending}</p>
        </Card>
      </div>
      
      <Chart data={revenueStats.daily} />
    </div>
  );
}
```

## ✅ Checklist de Funcionalidades

- [x] Product catalog con búsqueda
- [x] Shopping cart persistente
- [x] Checkout con Stripe
- [x] Autenticación de usuarios
- [x] Panel de administración
- [x] Order management
- [x] Email notifications
- [x] Webhook handling
- [x] Inventory tracking
- [x] Product reviews
- [x] Mobile responsive
- [x] SEO optimizado
- [x] Tests E2E
- [x] Docker ready
- [x] Deploy a Vercel
