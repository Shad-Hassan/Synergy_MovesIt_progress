---
title: Compare Products — Implementation Status
updated: 2026-03-30
---

# Compare Products — What's Left to Build

## TL;DR

The foundations exist: a working state store, animated CompareBar UI, and toggle buttons on product cards. What's missing is the actual compare page, the AppleSlider button wiring, and localStorage persistence.

---

## What Already Exists

| Area | Status | File |
|------|--------|------|
| Compare store (max 3 products) | ✅ | `app/hooks/useCompareStore.ts` |
| CompareBar (floating bottom widget) | ✅ | `app/components/catalog/CompareBar.tsx` |
| Toggle button on collection cards | ✅ | `app/components/ProductItem.tsx` |
| Quick Compare button in AppleSlider | ⚠️ UI only | `app/components/sliders/AppleSlider.tsx` |
| CompareBar wired into PageLayout | ✅ | `app/components/PageLayout.tsx` |
| `deriveSpecs()` for badge/tag parsing | ✅ | `app/lib/productUtils.ts` |

---

## What's Missing — Ordered by Dependency

### 1. Wire Up AppleSlider Quick Compare Button
**File:** `app/components/sliders/AppleSlider.tsx` — lines ~118–149

The button renders and has the `GitCompareArrows` icon but the store hook is commented out and the `onClick` only calls `console.log`.

**What to do:**
```tsx
// Uncomment and implement:
const { toggle: toggleCompare } = useCompareStore();

// In the button onClick:
toggleCompare({
  id: product.id,
  handle: product.handle,
  title: product.title,
  imageUrl: product.featuredImage?.url ?? null,
  imageAlt: product.featuredImage?.altText ?? null,
  price: product.priceRange.minVariantPrice.amount,
  currencyCode: product.priceRange.minVariantPrice.currencyCode,
});
```

**Effort:** ~10 minutes. Just uncomment, build the payload, call toggle.

---

### 2. Add localStorage Persistence to the Store
**File:** `app/hooks/useCompareStore.ts`

Currently the compare list is lost on every page refresh. This is a poor UX — users build a comparison, navigate to a product page, and lose their selection.

**What to do:** Initialise the store from `localStorage` and write back on every change:
```ts
// On init
const stored = localStorage.getItem('synergy_compare');
let state: CompareProduct[] = stored ? JSON.parse(stored) : [];

// After every toggle/clear
localStorage.setItem('synergy_compare', JSON.stringify(state));
```

**Effort:** ~20 minutes.

---

### 3. Create the `/compare` Route
**File to create:** `app/routes/($locale).compare.tsx`

This is the critical missing piece. The CompareBar already links to `/compare?ids=id1,id2,id3` — that URL currently 404s.

**What the route needs:**
- Read `ids` from URL searchParams
- Fetch full product data for each ID via Storefront API
- Render a side-by-side comparison table

**Loader sketch:**
```tsx
export async function loader({ request, context }: LoaderFunctionArgs) {
  const ids = new URL(request.url).searchParams.get('ids')?.split(',') ?? [];
  // Fetch products by IDs using storefront.query()
  // Return { products }
}
```

**GraphQL:** Use `nodes(ids: [...])` query to batch-fetch all products in one request.

**Effort:** Medium (~2–4 hours including UI).

---

### 4. Build the Comparison Table UI
**File to create:** `app/components/catalog/CompareTable.tsx` (used inside the compare route)

The table should show products as columns, attributes as rows.

**Suggested rows using data already available:**

| Row | Data source |
|-----|------------|
| Product image | `featuredImage` |
| Title | `title` |
| Price | `priceRange.minVariantPrice` |
| Availability | `variants.nodes[0].availableForSale` |
| Specs/tags | `deriveSpecs(title, tags)` from `productUtils` |
| Description | `description` (plain text, truncated) |
| Vendor | `vendor` |
| Add to Cart | Direct CTA per column |

**What's not available without metafields:**
- Weight / dimensions
- Exact capacity (Ah) beyond what `deriveSpecs` regex extracts from title/tags
- Warranty details
- Certifications

**Effort:** Medium (~3–5 hours for a polished table).

---

### 5. Extend Product Data for Richer Comparisons (Optional)
**File:** `app/lib/fragments.ts` — add to `PRODUCT_ITEM_FRAGMENT`

Currently the fragment used by sliders/cards only fetches: `id, handle, title, tags, featuredImage, priceRange, variants(first:1)`.

To show vendor and description on the compare page, add:
```graphql
vendor
description
```

For truly detailed specs (battery chemistry, BMS specs, weight, etc.), you'd need Shopify metafields:
```graphql
metafields(identifiers: [
  { namespace: "specs", key: "capacity_ah" },
  { namespace: "specs", key: "weight_kg" },
  { namespace: "specs", key: "warranty_years" }
]) {
  key
  value
}
```
This requires setting up metafield definitions in Shopify Admin first.

**Effort:** Low for vendor/description. High for metafields (requires Shopify Admin setup + query changes).

---

## Implementation Order

```
1. Wire AppleSlider button      (~10 min)  — low effort, visible fix
2. localStorage persistence     (~20 min)  — prevents UX frustration
3. /compare route + loader      (~1 hr)    — makes CompareBar functional
4. CompareTable component       (~3-5 hrs) — the actual compare experience
5. Extend product fragment      (~30 min)  — richer data once table exists
6. Metafields (optional)        (varies)   — only if detailed specs needed
```

---

## Key Files Reference

| File | Purpose |
|------|---------|
| `app/hooks/useCompareStore.ts` | State store — toggle, clear, isSelected |
| `app/components/catalog/CompareBar.tsx` | Floating bottom bar, links to `/compare` |
| `app/components/ProductItem.tsx` | Toggle button on collection grid cards |
| `app/components/sliders/AppleSlider.tsx` | Quick Compare button (needs wiring) |
| `app/components/PageLayout.tsx` | Mounts CompareBar globally |
| `app/lib/productUtils.ts` | `deriveSpecs()` — reuse this in compare table |
| `app/lib/fragments.ts` | GraphQL fragments — extend for compare data |
