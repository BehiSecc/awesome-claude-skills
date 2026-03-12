---
name: whop
description: Whop platform expert for digital products, memberships, and community monetization. Covers memberships API, payments, courses, forums, webhooks, OAuth apps, and checkout integration. Build SaaS, course platforms, and gated communities. Triggers on Whop, memberships, digital products, course platform, community monetization, Whop API, license keys.
allowed-tools: Read, Write, Edit, Grep, Glob, Bash
model: sonnet
license: MIT
metadata:
  author: raintree
  version: "1.0"
---

# Whop Platform Expert

Whop is a platform for selling digital products, memberships, courses, and community access.

## When to Use

- Building membership/subscription products
- Integrating Whop checkout and payments
- Managing courses and educational content
- Gating content by membership status
- Handling webhooks for payment events
- Building Whop apps with OAuth

## Quick Start

### Authentication

```typescript
const headers = {
  Authorization: `Bearer ${process.env.WHOP_API_KEY}`,
  "Content-Type": "application/json",
};

const response = await fetch("https://api.whop.com/api/v5/me", { headers });
```

**API Key Types:**
- **Company API Keys** - Your own company data (Dashboard → Settings → API Keys)
- **App API Keys** - Multi-tenant apps (Dashboard → Developer → Your App)

## Core Operations

### Create Product

```typescript
const product = await fetch("https://api.whop.com/api/v5/products", {
  method: "POST",
  headers,
  body: JSON.stringify({
    title: "Premium Membership",
    description: "Access to all content",
    visibility: "visible",
  }),
});
```

### Create Plan

```typescript
const plan = await fetch("https://api.whop.com/api/v5/plans", {
  method: "POST",
  headers,
  body: JSON.stringify({
    product_id: "prod_xxx",
    billing_period: 1,
    billing_period_unit: "month",
    price: 2999, // $29.99 in cents
    currency: "usd",
  }),
});
```

### Create Checkout

```typescript
const checkout = await fetch(
  "https://api.whop.com/api/v5/checkout-configurations",
  {
    method: "POST",
    headers,
    body: JSON.stringify({
      plan_id: "plan_xxx",
      success_url: "https://yourapp.com/success",
      cancel_url: "https://yourapp.com/cancel",
      metadata: { user_id: "123" },
    }),
  }
);

const { url } = await checkout.json();
// Redirect user to url
```

## Membership Management

### List Memberships

```typescript
const response = await fetch(
  "https://api.whop.com/api/v5/memberships?valid=true",
  { headers }
);

const { data: memberships } = await response.json();
```

### Check User Access

```typescript
async function checkAccess(userId: string, productId: string): Promise<boolean> {
  const response = await fetch(
    `https://api.whop.com/api/v5/memberships?user_id=${userId}&product_id=${productId}&valid=true`,
    { headers }
  );

  const { data } = await response.json();
  return data.length > 0;
}
```

### Validate License Key

```typescript
async function validateLicense(licenseKey: string): Promise<boolean> {
  const response = await fetch(
    `https://api.whop.com/api/v5/memberships?license_key=${licenseKey}`,
    { headers }
  );

  const { data } = await response.json();
  return data.length > 0 && data[0].valid;
}
```

### Cancel Membership

```typescript
await fetch(
  `https://api.whop.com/api/v5/memberships/${membershipId}/cancel`,
  {
    method: "POST",
    headers,
  }
);
```

## Payments

### Get Payment

```typescript
const payment = await fetch(
  `https://api.whop.com/api/v5/payments/${paymentId}`,
  { headers }
);
```

### Refund Payment

```typescript
await fetch(`https://api.whop.com/api/v5/payments/${paymentId}/refund`, {
  method: "POST",
  headers,
  body: JSON.stringify({
    amount: 1000, // Optional: partial refund in cents
  }),
});
```

## Courses

### Create Course

```typescript
const course = await fetch("https://api.whop.com/api/v5/courses", {
  method: "POST",
  headers,
  body: JSON.stringify({
    product_id: "prod_xxx",
    title: "Complete Web Development",
    description: "Learn fullstack from scratch",
    visibility: "visible",
  }),
});
```

### Create Chapter

```typescript
const chapter = await fetch("https://api.whop.com/api/v5/course-chapters", {
  method: "POST",
  headers,
  body: JSON.stringify({
    course_id: "course_xxx",
    title: "Introduction to JavaScript",
    order: 1,
  }),
});
```

### Create Lesson

```typescript
const lesson = await fetch("https://api.whop.com/api/v5/course-lessons", {
  method: "POST",
  headers,
  body: JSON.stringify({
    chapter_id: "chapter_xxx",
    title: "Variables and Data Types",
    content: "Lesson content in markdown...",
    type: "video", // or 'text', 'quiz', 'assignment'
    video_url: "https://youtube.com/watch?v=...",
    order: 1,
  }),
});
```

### Mark Lesson Complete

```typescript
await fetch(
  `https://api.whop.com/api/v5/course-lessons/${lessonId}/mark-as-completed`,
  {
    method: "POST",
    headers: {
      Authorization: `Bearer ${userAccessToken}`,
    },
  }
);
```

## Webhooks

### Setup Endpoint

```typescript
import crypto from "crypto";

export async function POST(req: Request) {
  const body = await req.text();
  const signature = req.headers.get("x-whop-signature");

  // Verify signature
  const hash = crypto
    .createHmac("sha256", process.env.WHOP_WEBHOOK_SECRET!)
    .update(body)
    .digest("hex");

  if (hash !== signature) {
    return new Response("Invalid signature", { status: 401 });
  }

  const event = JSON.parse(body);

  switch (event.action) {
    case "payment.succeeded":
      await handlePaymentSuccess(event.data);
      break;

    case "membership.activated":
      await grantAccess(event.data);
      break;

    case "membership.deactivated":
      await revokeAccess(event.data);
      break;

    case "dispute.created":
      await handleDispute(event.data);
      break;
  }

  return Response.json({ received: true });
}
```

**Key webhook events:**
| Event | Trigger |
|-------|---------|
| `payment.succeeded` | Payment completed |
| `payment.failed` | Payment failed |
| `membership.activated` | Membership started |
| `membership.deactivated` | Membership ended |
| `invoice.paid` | Recurring payment succeeded |
| `dispute.created` | Chargeback initiated |

## OAuth Apps

### Redirect to Whop OAuth

```typescript
const authUrl = new URL("https://whop.com/oauth");
authUrl.searchParams.set("client_id", process.env.WHOP_CLIENT_ID!);
authUrl.searchParams.set("redirect_uri", "https://yourapp.com/callback");
authUrl.searchParams.set("scope", "memberships:read payments:read");

// Redirect user
window.location.href = authUrl.toString();
```

### Handle Callback

```typescript
export async function GET(req: Request) {
  const url = new URL(req.url);
  const code = url.searchParams.get("code");

  const response = await fetch("https://api.whop.com/api/v5/oauth/token", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({
      client_id: process.env.WHOP_CLIENT_ID!,
      client_secret: process.env.WHOP_CLIENT_SECRET!,
      code,
      grant_type: "authorization_code",
      redirect_uri: "https://yourapp.com/callback",
    }),
  });

  const { access_token, refresh_token } = await response.json();
  // Store tokens securely
}
```

## Middleware (Access Control)

```typescript
// middleware.ts
import { NextResponse } from "next/server";
import type { NextRequest } from "next/server";

export async function middleware(request: NextRequest) {
  const token = request.cookies.get("whop_user_token")?.value;

  if (!token) {
    return NextResponse.redirect(new URL("/login", request.url));
  }

  const response = await fetch("https://api.whop.com/api/v5/me/memberships", {
    headers: { Authorization: `Bearer ${token}` },
  });

  const { data } = await response.json();
  const hasAccess = data.some((m: any) => m.status === "active" && m.valid);

  if (!hasAccess) {
    return NextResponse.redirect(new URL("/subscribe", request.url));
  }

  return NextResponse.next();
}

export const config = {
  matcher: ["/dashboard/:path*", "/premium/:path*"],
};
```

## TypeScript Types

```typescript
interface WhopMembership {
  id: string;
  user_id: string;
  product_id: string;
  plan_id: string;
  status: "active" | "paused" | "canceled" | "expired";
  valid: boolean;
  license_key: string;
  renews_at: string | null;
  expires_at: string | null;
  cancel_at_period_end: boolean;
}

interface WhopPayment {
  id: string;
  amount: number;
  currency: string;
  status: "succeeded" | "pending" | "failed";
  user_id: string;
  product_id: string;
}
```

## Error Handling

```typescript
async function safeWhopRequest(url: string, options: RequestInit) {
  const response = await fetch(url, options);

  if (!response.ok) {
    const error = await response.json();
    throw new Error(error.message || "Whop API error");
  }

  return response.json();
}
```

| Error | Meaning |
|-------|---------|
| `401 Unauthorized` | Invalid API key |
| `403 Forbidden` | Insufficient permissions |
| `404 Not Found` | Resource doesn't exist |
| `429 Too Many Requests` | Rate limited |

## Security Best Practices

**DO:**
- Verify webhook signatures
- Store tokens encrypted
- Check membership status server-side
- Use HTTPS everywhere

**DON'T:**
- Expose API keys client-side
- Trust client-sent membership data
- Skip webhook verification
- Log sensitive data

## Implementation Checklist

- [ ] Create Whop account and company
- [ ] Get API keys from Dashboard
- [ ] Set environment variables
- [ ] Create products and plans
- [ ] Implement checkout flow
- [ ] Add membership access checking
- [ ] Set up webhook endpoint
- [ ] Handle payment events
- [ ] Test end-to-end flow

## Resources

- **Dashboard:** https://whop.com/dashboard
- **API Docs:** https://docs.whop.com
- **API Base:** https://api.whop.com/api/v5
