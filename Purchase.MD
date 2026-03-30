---
title: Purchase & Checkout — March 31  Early Day status
updated: 2026-03-31
---

# Purchase & Checkout — Honest Status

## Cart & Checkout: ✅ Fully Working

The entire purchase flow — add to cart through to Shopify checkout — is implemented and functional end-to-end.

### What actually works

| Feature | Component | Status |
|---------|-----------|--------|
| Add to Cart (product page) | `ProductForm.tsx` + `AddToCartButton` | ✅ Works |
| Add to Cart (collection cards) | `ProductItem.tsx` | ✅ Works |
| Add to Cart (AppleSlider) | `AppleSlider.tsx` → `AddToCartCTA` | ✅ Works |
| Cart drawer open/close | `PageLayout.tsx` + GSAP animations | ✅ Works |
| Cart items display (image, title, price, options) | `CartLineItem.tsx` | ✅ Works |
| Quantity +/− controls | `CartLineItem.tsx` | ✅ Works |
| Remove line item | `CartLineItem.tsx` | ✅ Works |
| Optimistic updates | `useOptimisticCart()` | ✅ Works |
| Discount code apply/remove | `CartSummary.tsx` | ✅ Works |
| Gift card apply/remove | `CartSummary.tsx` | ✅ Works |
| Cart subtotal display | `CartSummary.tsx` | ✅ Works |
| Checkout redirect | `cart.checkoutUrl` → Shopify hosted | ✅ Works |
| Cart page (`/cart`) | `($locale).cart.tsx` | ✅ Works |
| All cart mutations (add/update/remove) | `($locale).cart.tsx` action handler | ✅ Works |
| Order history (logged-in) | `account.orders._index.tsx` | ✅ Works — Shopify Kit, not custom |

---

## What is NOT done

### ❌ Custom thank-you / order confirmation page
After Shopify processes payment, the user lands on **Shopify's own post-purchase screen**. There is no custom page on this domain. This is an acceptable gap for launch — Shopify handles it — but means zero branding control after checkout.

**To fix:** Configure Shopify Admin → Settings → Checkout → Post-purchase to redirect to a custom route, then build `app/routes/($locale).orders.confirmation.tsx`.

### ❌ Branded order confirmation email
Shopify sends its default transactional email. There are no custom templates.

**To fix:** `app/routes/webhooks.orders.create.tsx` already receives the `orders/create` webhook and syncs to GoHighLevel. A GHL email automation could be triggered from there — low effort if you already use GHL for email.

### ⚠️ Shopify Kit UI (account, addresses, login) — not styled
The customer account routes (`account.*`) are Shopify Kit scaffolding. They work but use plain Shopify Kit markup — not styled to match the dark design system. Kept as pending because they ship with Hydrogen and aren't broken, just unstyled.

---

## Pre-launch checklist (Shopify Admin, not code)

- [ ] Shopify Payments activated + bank connected
- [ ] Shipping zones and rates configured
- [ ] Tax settings correct for your region
- [ ] `SHOPIFY_STORE_DOMAIN` + `SHOPIFY_STOREFRONT_ACCESS_TOKEN` in production env
- [ ] `SESSION_SECRET` set in production env
- [ ] Webhook registered: `POST https://yourdomain.com/webhooks/orders/create`
- [ ] Custom domain pointed to deployment
- [ ] Test a full add-to-cart → checkout cycle on production before going live

---

## Key files

| File | Purpose |
|------|---------|
| `app/components/CartMain.tsx` | Cart item list |
| `app/components/CartLineItem.tsx` | Line item: image, qty, remove |
| `app/components/CartSummary.tsx` | Totals, discount codes, checkout button |
| `app/components/PageLayout.tsx` | Cart drawer (GSAP open/close) |
| `app/hooks/useCartStore.ts` | Drawer open/close state |
| `app/routes/($locale).cart.tsx` | All cart mutations server-side |
| `app/routes/webhooks.orders.create.tsx` | Order webhook → GHL CRM sync |
| `app/lib/fragments.ts` | Cart GraphQL fragment (includes `checkoutUrl`) |
