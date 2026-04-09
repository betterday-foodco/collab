# What's Next — Meal Edit Module

**Last updated:** 2026-04-09 by the meal-edit chat
**Status:** Diet-plan classifier shipped. Meal edit page refactored. Tier 1 new-meal form built. Diet selector HTML prototype designed on client-website. Customer backend join and public menu-cats endpoint are the remaining gaps.

---

## 🎯 Where we are right now

| Layer | Status |
|---|---|
| **Schema — meal side** | ✅ Complete. `MealRecipe.diet_plan_id` is a NOT NULL FK to `SystemTag(type='diets')`. 159 meals classified on the Neon `conner-local-dev` branch (88 Omnivore + 71 Plant-Based), 0 nulls. 47 meat→plant variant pairings established one-directionally via `linked_meal_id`. |
| **Admin UI — meal edit page** | ✅ Shipped via PR #10. Diet Plan segmented toggle in left aside, menu-cats Category dropdown, conditional plant-based variant picker, smart word-match suggestions, merged Allergens & Dislikes section with ingredient rollup, collapsed Internal Name admin disclosure. |
| **Admin UI — new meal create form** | ✅ Shipped via PR #10. Tier 1 rewrite: Display Name / Diet Plan / Category / Sell Price. Legacy skeleton form (name/yield/components editor) replaced. Backend auto-mirrors `display_name → name` when blank. |
| **Menu builder pair modal** | ✅ 3-bug fix shipped via PR #10 (wrong URL, wrong token key, silent error swallow). Loading state added. |
| **Category classifier** | ✅ SystemTag(type='menu-cats') is now the single source of truth. Entree 142, Breakfast 10, Snacks 7 after the bulk backfill. |
| **Diet selector HTML prototype** | ✅ Designed. Lives at `conner/client-website/onboarding/index.html` on culinary-ops. Uses SystemTag slugs (`omnivore` / `vegan`), real images from `/brand/photos/`, `brand/tokens.css`. Next.js implementation pending. See `subscriptions.signup.onboarding.dietary` node. |
| **Schema — customer side** | ❌ `Customer.diet_plan_id` not added yet. The selector has nowhere to persist the choice. |
| **Public menu-cats endpoint** | ❌ `/api/tags` is JWT-guarded. Client-website can't read the menu categories without admin auth. |
| **Customer-facing menu filter** | ❌ `GET /api/menu/filter?diet=vegan` doesn't exist. Node is `designed` (data layer exists) but endpoint not built. |

---

## 🚀 Phase 1 — Close the customer-side loop (backend + commerce schema)

**Goal:** The diet selector HTML prototype can actually persist a choice and the menu page can actually filter by it. Three backend leaves, no new UI work — the HTML already exists.

### 1. `Customer.diet_plan_id` on `schema.commerce.prisma`

Add the field to the commerce `Customer` model. Mirrors `MealRecipe.diet_plan_id`:

```prisma
model Customer {
  // ... existing fields ...
  diet_plan_id  String   // FK to SystemTag(type='diets'). No cross-DB relation
                         // (SystemTag lives in culinary-ops DB, Customer in commerce
                         // DB), so this is an app-enforced reference not a hard FK.
}
```

Migration: `npx prisma migrate dev --name add_customer_diet_plan_id`.

**Important:** the `SystemTag` table lives in the **culinary-ops Postgres**, and `Customer` lives in the **betterday-commerce Postgres** (two separate Neon projects). So `diet_plan_id` cannot be a hard Prisma FK — it's a soft reference validated by the app layer. See the two-DB-split ADR for why this is the right tradeoff.

The valid values are the two hardcoded UUIDs from the culinary-ops SystemTag table:
- `fc0a70f3-644b-4248-b9c1-65882cc503de` — Omnivore
- `9c68ba40-f59d-40a8-8210-bdc1f3cd3973` — Plant-Based

Nullable during migration, flip to NOT NULL after customer backfill (customers without a choice get defaulted to Omnivore, matching the majority of production meals).

### 2. `GET /api/public/menu-categories` — unauthenticated menu-cats read

New controller (or extend the existing `SystemConfigPublicController` pattern from the 2026-04-08 sprint). Returns an array of menu-cats SystemTag rows filtered to `visible=true`, sorted by `sort_order` then `name`. Fields: `id`, `name`, `slug`, `emoji`, `sort_order`.

Same pattern as the existing `GET /api/system-config/public` endpoint — no JWT required, read-only, small surface area.

### 3. `POST /api/commerce/customers/me/diet-plan` — persist the selection

Takes `{ diet_plan_id: string }`, validates against the two hardcoded UUIDs above, writes to `Customer.diet_plan_id`, returns the updated customer record. 10 lines of NestJS.

Also update the customer profile GET endpoint to include `diet_plan_id` in the response so the frontend can render which choice is currently selected on the account hub.

---

## 🗓️ Phase 2 — Customer-facing filter endpoint (day 2)

`GET /api/menu/filter?diet=vegan` — the node that's already marked `designed` on the scope board. Reads from the culinary-ops Postgres (via the culinary Prisma client, not commerce), filters by `diet_plan_id`, joins with `MealComponent` and `PortionSpec` for the full card payload. Should also support `?category=entrees|breakfast|snacks|sandwich-wraps|protein-pack` filter via the menu-cats slugs.

Caches in Redis or in-memory for the weekly menu window. See `menu.source.bridge.fetch` and `menu.source.bridge.photo-urls` scope nodes.

---

## 🗓️ Phase 3 — Wire the diet selector HTML to the backend (day 2 or 3)

The HTML prototype at `conner/client-website/onboarding/index.html` currently writes the choice to `localStorage` only. Replace with an actual `POST /api/commerce/customers/me/diet-plan` call after Phase 1 ships. On success, redirect to `/menu` with the selected plan already applied as a query param. On failure, show the customer error (probably rare — the only valid failure is "not logged in yet," in which case fall back to localStorage and apply at signup).

---

## 🗓️ Phase 4 preview — Cleanup debt

Deferred items from PR #10 that aren't blocking Phase 1 but should be addressed before the customer website goes live:

- [ ] **Drop `MealRecipe.net_weight_kg`** — 9 orphan references, zero load-bearing use. Documented in the ADR as bundled cleanup. Needs Gurleen's review before the migration runs.
- [ ] **Fix menu builder diet inference** at `frontend/app/(dashboard)/menu-builder/page.tsx:511-514` — currently does `cat.includes('vegan')` which works by accident. Should read `diet_plan_id === PLANT_BASED_ID` directly.
- [ ] **Filter `getSuggestedVariants`** server-side to the opposite diet plan. Currently client-side filtering catches most cases but the endpoint itself should enforce.
- [ ] **Protected SystemTag rows** — tags service should refuse to delete the two diet-plan classifier IDs. Currently nothing prevents an admin from deleting them and breaking every MealRecipe FK.
- [ ] **Tier 2 fields on the create form** — photo + tagline. Deferred to keep Tier 1 focused; add when the admin flow feels too spartan.
- [ ] **Production diet_plan backfill.** The Neon dev branch has the full 159-meal classification. Same SQL can run against the production branch after Gurleen approves the ADR.

---

## 🧭 Files to reference tomorrow

| File | What it is |
|---|---|
| `backend/prisma/schema.commerce.prisma` | Where `Customer.diet_plan_id` gets added (culinary-ops repo) |
| `backend/src/modules/system-config/system-config.public.controller.ts` | Reference pattern for the new public menu-categories controller |
| `backend/src/modules/meals/meals.service.ts` | Where the meal-side diet_plan_id logic lives (validator, create, update) |
| `conner/client-website/onboarding/index.html` | The designed HTML prototype that Phase 3 will wire to the new endpoint |
| `conner/data-model/decisions/2026-04-08-mandatory-diet-plan-on-dishes.md` | Full ADR with the rationale, backfill plan, and open questions |
| `conner/data-model/flows/meal-variants.md` | N-to-1 variant linking reference — relevant when building the menu filter's meat↔plant swap feature |
| Memory: `project_diet_plan_rule` | The mandatory-diet-plan rule in plain English |
| Memory: `project_two_db_split` | Why `Customer.diet_plan_id` can't be a hard FK |

---

## 🛑 Open questions before Phase 1

1. **Default diet plan for existing customers.** Commerce DB already has customers (from the commerce-customers backend that shipped today). When `Customer.diet_plan_id` is added, those customers need a value. Default to Omnivore? Default to null and force re-selection on next login? Conner's call.
2. **Hardcoded SystemTag UUIDs vs. config.** The two diet-plan UUIDs are currently hardcoded in both the culinary-ops frontend AND the commerce backend's validator. Should they move to `SystemConfig` keys (`public.diet_plan.omnivore_id`, `public.diet_plan.plant_based_id`) so admins can rename the SystemTag rows without a code deploy? Low priority but worth considering.

---

*This file is a snapshot of tomorrow's priorities for the meal-edit / diet-plan module. Update or delete when Phase 1 is in progress. Lives in `project-scope/` next to `data.json`, `HOW_TO_USE.md`, and the coupons chat's `NEXT_UP.md`.*
