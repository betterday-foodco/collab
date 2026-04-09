# What's Next — Coupon Module

**Last updated:** 2026-04-09 by the coupons chat
**Status:** Migration 5 (coupon_power_up) landed. Schema is complete. Engine and UI not yet built.

---

## 🎯 Where we are right now

| Layer | Status |
|---|---|
| **Schema** | ✅ Complete — Migration 5 added 42 Coupon fields, 4 CustomerCoupon fields, 2 new child tables (CouponTier, CouponAttachedProduct), 4 new enums. Applied to `betterday-commerce` Neon dev branch. |
| **HTML design prototypes** | ✅ Two standalone prototypes live in culinary-ops: `conner/coupon-features-picker.html` (admin form prototype) and `conner/dotw-scheduler-mockup.html` (DOTW scheduler). |
| **Backend engine** | ❌ Nothing built. No validator, no apply/remove endpoints, no DOTW rule. |
| **Admin UI (real)** | ❌ Nothing built. Only HTML mockups. |
| **Customer UX** | ❌ No checkout input, no clip cards, no "my coupons" page. |
| **Reporting** | ❌ Denormalized analytics skipped per picker selection. Queries + UI TBD. |

---

## 🚀 Phase 1 — Make coupons actually work at checkout

**Goal:** A customer can type a code at checkout, the system validates it, applies the discount, and the cart total drops. Nothing fancy yet — no admin UI, no clippable cards, no reporting. Just the minimum viable "coupons are live."

**Four backend leaves, in order:**

### 1. `coupons.engine.validate.service` — the core validator

Write `CouponValidationService` in the NestJS commerce backend. Takes a cart and a code, returns:

```typescript
{
  valid: boolean
  reason?: 'CODE_NOT_FOUND' | 'EXPIRED' | 'NOT_YET_ACTIVE' | 'MIN_ORDER_NOT_MET' | ...
  meta?: { required?: number, actual?: number, shortfall?: number, mealNames?: string[] }
  customerMessage?: string
  savings?: number
}
```

Implements the validation rules in order:
1. Code exists + is_active
2. Date range (starts_at / expires_at)
3. Usage limits (max_uses, max_uses_per_customer, max_uses_per_household)
4. Order value thresholds (min_order_value, max_order_value)
5. Product/category include/exclude arrays
6. Email allowlist / blocklist
7. Customer segment targeting (tags, LTV, member_since)
8. Subscription restriction
9. Order count restrictions
10. DOTW delivery week match (see next leaf)

### 2. `coupons.dotw.engine.validate-week-match` — the DOTW rule

One line of code structurally, but the most important piece for preventing the "pre-order leak" edge case:

```typescript
if (coupon.purpose === 'deal_of_the_week') {
  if (coupon.delivery_week_sunday !== order.delivery_week_sunday) {
    return {
      valid: false,
      reason: 'WRONG_DELIVERY_WEEK',
      customerMessage: `This Deal of the Week is for ${formatWeek(coupon.delivery_week_sunday)} delivery only.`,
    }
  }
}
```

See `project_dotw_preorder_rules` memory for the full three-rule explanation (week binding, visibility filter, cart price ceiling).

### 3. `coupons.engine.apply.endpoint` — POST /api/cart/apply-coupon

Calls `CouponValidationService`. On success, persists the coupon on `CustomerCoupon` with `status=applied` and `applied_at=now()`. On failure, returns the structured error from the validator (the frontend renders `customerMessage` directly).

### 4. `coupons.engine.apply.remove` — POST /api/cart/remove-coupon

Sets `CustomerCoupon.status = available`, clears `applied_at`. Re-computes cart total without the coupon.

---

## 🎨 Parallel — Customer-facing error message catalog

Side-project that can happen at the same time as Phase 1: write `backend/src/commerce/coupons/error-messages.ts` — a single file mapping error codes to warm, actionable copy in BetterDay voice.

```typescript
export const couponErrorMessages = {
  MIN_ORDER_NOT_MET: (meta) =>
    `Almost there! Add $${meta.shortfall.toFixed(2)} more and this ${meta.discount} kicks in.`,
  CODE_NOT_FOUND: () =>
    `Hmm, we don't recognize that code — double-check for typos?`,
  EXPIRED: (meta) =>
    `This one expired on ${formatDate(meta.expiredAt)}. [See this week's deals →]`,
  // ... etc
}
```

See the coupons chat conversation (2026-04-08) for the full error catalog with warm/cold examples and the gatekeeping vs silent-error categorization.

---

## 🗓️ Phase 2 preview — Admin create/edit (day 2 or 3)

Once validation works, wire up the admin form:

- Backend: Coupon CRUD endpoints (`coupons.engine.create.api`)
- Frontend: Real admin list + form at `/admin/coupons` (using `conner/coupon-features-picker.html` as the design reference)

---

## 🗓️ Phase 3 preview — DOTW scheduler (day 3 or 4)

- Backend: DOTW-specific endpoints (`coupons.dotw.api.scheduler-backend`) — thin wrapper over Coupon CRUD with locked presets
- Frontend: Real scheduler UI at `/admin/dotw` (using `conner/dotw-scheduler-mockup.html` as the reference)

---

## 🗓️ Phase 4 preview — Customer-facing coupon UX (day 4 or 5)

- Frontend: Coupon input + applied chip on checkout page (`coupons.ui.checkout.input`, `coupons.ui.checkout.applied`)
- Frontend: Clippable coupon cards on cart (`coupons.ui.checkout.clippable`)
- Frontend: "My coupons" tab in account hub (`coupons.ui.account.my-coupons`)

---

## 🗓️ Phase 5 preview — Reporting (week 2+)

- Backend: cohort analysis queries joining `Coupon ← CustomerCoupon ← Order ← Customer` for redemptions, revenue, LTV, retention — sliceable by category/tag
- Frontend: admin reporting dashboard (`coupons.ui.admin.stats`)
- Optional: coupon validation telemetry table (log every validation attempt, for tuning thresholds based on top failure reasons)

---

## 🧭 Files to reference tomorrow

| File | What it is |
|---|---|
| `backend/prisma/commerce/schema.prisma` | Live Coupon + CustomerCoupon models with all 42 new fields (culinary-ops repo) |
| `backend/prisma/commerce/migrations/20260409060452_coupon_power_up/migration.sql` | The applied migration SQL (culinary-ops repo) |
| `conner/coupon-features-picker.html` | Admin form design prototype |
| `conner/dotw-scheduler-mockup.html` | DOTW scheduler design prototype |
| `conner/deferred-decisions.md` | Edge cases, pending decisions, and future ideas to check before building |
| Memory: `project_dotw_preorder_rules` | Three DOTW rules (week binding, visibility filter, cart price ceiling) |
| Memory: `project_coupon_stacking_policy` | Why `stackable` defaults to true |

---

## 🛑 No blocking open questions

The schema is set. The stacking policy is decided. The DOTW rules are documented. **Start building when you're ready.**

---

*This file is a snapshot of tomorrow's priorities. Update or delete when Phase 1 is in progress. Lives in `project-scope/` next to `data.json` and `HOW_TO_USE.md`.*
