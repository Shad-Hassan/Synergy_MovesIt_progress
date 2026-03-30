---
title: Compare Products — March 31  Early Day status
updated: 2026-03-31
---

# Compare Products — Honest Status

## TL;DR: The compare feature is a dead end right now.

The store, the animated bar, and the buttons all exist and look great. But clicking "Compare" takes the user to a **404**. The AppleSlider compare button does literally nothing except `console.log`. No one can actually compare products today.

---

## What actually works

| Part | Status | Notes |
|------|--------|-------|
| `useCompareStore.ts` — state store | ✅ Works | Toggle, clear, isSelected, max 3 limit all correct |
| `CompareBar.tsx` — floating bottom bar | ✅ Renders | Animations, thumbnails, dismiss button all work |
| `ProductItem.tsx` — "Add to Compare" button | ✅ Works | Fires `toggle()`, visual selected/unselected state correct |
| `PageLayout.tsx` — CompareBar mounted globally | ✅ Works | Bar appears at the bottom of every page |

---

## What is broken

### ❌ No `/compare` route — the destination is a 404

`CompareBar.tsx` line 79 builds the link:
```tsx
<Link to={`/compare?ids=${items.map((p) => p.id).join(',')}`}>
```

There is **no route file** at `app/routes/($locale).compare.tsx` or anywhere else. Every user who follows through will hit a 404.

**To fix:** Create `app/routes/($locale).compare.tsx` with:
1. A loader that reads `?ids=` from the URL and batch-fetches product data
2. A comparison table UI

### ❌ AppleSlider "Compare" button does nothing

`app/components/sliders/AppleSlider.tsx` — `CompareButton` component:
```tsx
onClick={(e) => {
  e.preventDefault();
  e.stopPropagation();
  console.log('Compare triggered for:', handle); // ← that's it
}}
```

The `useCompareStore` hook is imported in the file but never called inside `CompareButton`. Clicking this button has zero effect on the compare store or the bar.

**To fix:** Wire the hook and build the payload (10 minutes of work):
```tsx
const { toggle } = useCompareStore();
// onClick:
toggle({ id: product.id, handle, title, imageUrl, imageAlt, price, currencyCode });
```

### ❌ No localStorage persistence

The compare selection is wiped on every page reload. A user adds 2 products, navigates to a product detail page, and their selections are gone when they come back.

**To fix:** Read/write `localStorage` in `useCompareStore.ts` on init and every toggle.

### ❌ No comparison UI built at all

Even if the route existed, there is no component that shows a side-by-side product comparison. This is the bulk of the remaining work.

---

## What needs to be built, in order

```
1. Wire AppleSlider compare button to store        ~15 min   — easy fix
2. Add localStorage persistence to store           ~20 min   — easy fix
3. Create /compare route + loader                  ~1 hr     — fetches products by ID
4. Build CompareTable component                    ~4-6 hrs  — the actual comparison UI
```

### Step 3 — Route sketch
```tsx
// app/routes/($locale).compare.tsx
export async function loader({ request, context }) {
  const ids = new URL(request.url).searchParams.get('ids')?.split(',') ?? [];
  // batch fetch via storefront.query nodes(ids: [...])
  return { products };
}
```

### Step 4 — Comparison data available without extra GraphQL work
Using `deriveSpecs()` from `~/lib/productUtils` already gives:
- Voltage, capacity (Ah), output amps, bank count
- Bluetooth, Marine Grade, LiFePO4 flags

What is NOT available without Shopify metafield setup:
- Exact weight / dimensions
- Warranty details
- Certifications

---

## Key files

| File | Status | Purpose |
|------|--------|---------|
| `app/hooks/useCompareStore.ts` | ✅ Works | State — needs localStorage added |
| `app/components/catalog/CompareBar.tsx` | ✅ Renders | Links to non-existent route |
| `app/components/ProductItem.tsx` | ✅ Works | Toggle button wired correctly |
| `app/components/sliders/AppleSlider.tsx` | ❌ Broken | Button only logs, store never called |
| `app/routes/($locale).compare.tsx` | ❌ Missing | Doesn't exist — must be created |
| `app/components/catalog/CompareTable.tsx` | ❌ Missing | Doesn't exist — must be created |
