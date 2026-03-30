---
title: Purchase & Checkout — Deployment Readiness
updated: 2026-03-30
---

# Purchase & Checkout — What's Left Before You Can Sell

## TL;DR

The core purchase flow is **production-ready**. Add to cart, cart drawer, variant selection, discount codes, and Shopify-hosted checkout all work. There are three gaps that are nice-to-have but not blockers for launch.

---

## What's Working Right Now

| Area | Status | Notes |
|------|--------|-------|
| Add to Cart | ✅ | Works from product pages, AppleSlider, and collection cards |
| Cart Drawer | ✅ | GSAP-animated, opens on add, closable via backdrop/X/ESC |
| Variant Selection | ✅ | Swatch UI, URL-persisted, disables unavailable combos |
| Cart CRUD | ✅ | Add, update qty, remove — all with optimistic updates |
| Discount Codes | ✅ | Add & remove in cart summary |
| Gift Cards | ✅ | Add & remove in cart summary |
| Checkout Redirect | ✅ | `cart.checkoutUrl` → Shopify hosted checkout |
| Quick Cart URL | ✅ | `/cart/variantId:qty?discount=CODE` deep-link works |
| Order History | ✅ | Logged-in customers can view past orders |
| Webhook → CRM | ✅ | Orders sync to GoHighLevel on creation |

---

## Gaps — Grouped by Priority

### 🟡 Nice-to-Have Before Launch

#### 1. Custom Thank-You / Order Confirmation Page
**What's missing:** After Shopify processes payment, users are redirected back but there's no custom confirmation page on this domain. Shopify handles its own post-purchase screen.

**Why it matters:** You lose the opportunity for upsells, brand reinforcement, and customer trust signals after purchase.

**Files to create:**
- `app/routes/($locale).orders.confirmation.tsx` — or configure Shopify admin to redirect to a custom URL post-purchase

**Effort:** Medium — requires Shopify checkout settings + a new route.

---

#### 2. Post-Purchase Order Confirmation Email
**What's missing:** Shopify sends its default transactional email. There are no custom branded templates.

**Why it matters:** Default Shopify emails are plain and generic. A branded email reinforces trust.

**What exists:** `app/routes/webhooks.orders.create.tsx` already receives the order webhook and syncs to GHL — you could trigger a GHL email automation from there.

**Effort:** Low if using GHL automations (the webhook already fires). Medium if building custom email templates.

---

#### 3. Real-Time Order Status / Tracking
**What's missing:** No order tracking page for unfulfilled orders. Customers can see order history but not live fulfillment status.

**What exists:** `app/routes/($locale).account.orders._index.tsx` and `($locale).account.orders.$id.tsx` show past orders with financial and fulfillment status.

**Effort:** Low — Shopify's fulfillment status is already in the order query. Just surface it more prominently in the order detail page.

---

### 🔴 Not a Blocker — Already Handled by Shopify

- **Payment processing** — Shopify Payments handles everything on their hosted checkout
- **Tax calculation** — handled by Shopify
- **Shipping rates** — configured in Shopify admin, shown at checkout
- **Fraud protection** — Shopify built-in

---

## Deployment Checklist

Before going live, verify these in Shopify Admin (not code changes):

- [ ] Shopify Payments activated and bank account connected
- [ ] Shipping zones and rates configured
- [ ] Tax settings correct for your region
- [ ] `SHOPIFY_STORE_DOMAIN` and `SHOPIFY_STOREFRONT_ACCESS_TOKEN` env vars set in production
- [ ] `SESSION_SECRET` set in production environment
- [ ] Webhook endpoint registered: `https://yourdomain.com/webhooks/orders/create`
- [ ] Custom domain pointed to deployment (Oxygen / Vercel / Cloudflare)
- [ ] Cart `checkoutUrl` points to correct Shopify store (verify with a test add-to-cart)

---

## Key Files Reference

| File | Purpose |
|------|---------|
| `app/hooks/useCartStore.ts` | Cart drawer open/close state |
| `app/components/CartMain.tsx` | Cart item list (drawer + page) |
| `app/components/CartLineItem.tsx` | Individual line item with qty controls |
| `app/components/CartSummary.tsx` | Totals, discount codes, checkout button |
| `app/routes/($locale).cart.tsx` | All cart mutations (server-side) |
| `app/routes/($locale).cart.$lines.tsx` | Quick cart creation + checkout redirect |
| `app/routes/($locale).products.$handle.tsx` | Product page with variant selection |
| `app/components/ProductForm.tsx` | Variant picker + Add to Cart button |
| `app/routes/webhooks.orders.create.tsx` | Order webhook → GHL CRM sync |
| `app/lib/fragments.ts` | Cart GraphQL fragment (includes `checkoutUrl`) |
