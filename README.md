# Med Man Sandbox Hub 🧪

This repo is a lightweight hub for active prototypes, spikes, and real client exploration
related to Shopify, Checkout, Functions, Deployments and platform experiments.

Nothing here is production-ready (yet).  

This exists to:
- Track what's been explored
- Link working prototypes
- Capture feasibility learnings
- Keep context while ramping

Eventually, this may evolve into a monorepo once direction stabilizes.

---

## Active Workstreams

### 1) Extended Shopify Discount Functionality
**Repo:** https://github.com/mssalemi/discount-manager-app  
**Status:** ✅ Core functionality validated  

**What's proven:**
- Cart attributes flow storefront → checkout → Discount Function
- Discount Functions can apply logic based on merchant-defined attributes
- Metafield-based configuration is viable

**What's not possible:**
- Discounts based on payment method (checkout execution order blocker)

**Supported patterns:**
- Platform-specific discounts (via cart attribute)
- Acquisition channel discounts (via cart attribute)
- Loyalty tier discounts (via customer metafield)
- Location / shipping speed discounts (native data)

---

### 2) Checkout Extensibility (Checkout UI Extensions)
**Repo:** https://github.com/mssalemi/checkout-ui-add-order-metafields  
**Status:** ⚠️ In Progress

**What's proven:**
- Custom checkout fields (gift message, delivery notes)
- Checkout UI Extension → Order Metafields persistence
- `applyAttributeChange` can update cart attributes mid-checkout
- Cart attribute changes trigger Discount Function re-evaluation

**Extension target:** `purchase.checkout.block.render`

**Unlocked pattern:**
```
Checkout UI Extension → applyAttributeChange → Discount Function reads attribute
```
Enables: loyalty point redemption, user-selected discounts, experiment variants.

**Caveat:** Doesn't work with accelerated checkout (Apple Pay, Google Pay, Meta Pay).

---

### 3) Customer Accounts / Order History
**Repo:** https://github.com/mssalemi/customer-accounts-headless-oauth-example  
**Status:** ✅ Auth Flow + Customer Accounts API usage validated

**What's proven:**
- Headless customer accounts via OAuth + PKCE
- Authenticated customer context for order history access
- Aligns with various clients Customer Accounts / Order History patterns
- Compatible with existing storefront deployment flows

---

### 4) Fulfillment
**Repo:** https://github.com/mssalemi/3pl-mock-vercel  
**Status:** ✅ Working prototype  

**What's proven:**
- Register fulfillment service via Admin API
- Create fulfillment groups + assign products to locations
- Callback URL for fulfillment requests
- Fulfillment routing can be modeled upstream of Functions

*can connect with 3PL apis for various client fulfillments

---

### 5) Self-Serve Returns
**Repo:** https://github.com/mssalemi/selfserve-returns-shopify-app-mock  
**Status:** ✅ Working prototype  

**What's proven:**
- Merchant-facing self-serve returns app
- Approve, reject, restock, refund flow hooks
- Full control via Admin API + webhooks can be added for automation
- Return workflows can be automated via webhook-driven logic

---

## Key Learnings

| Principle | Detail |
|-----------|--------|
| **Schema is truth** | Functions only see what's in their input query. Everything else must be injected upstream. |
| **Cart attributes = storefront context** | Platform, UTM, acquisition channel, experiments. |
| **Metafields = durable business state** | Loyalty tiers, fulfillment channel, product flags. |
| **Checkout execution order matters** | Cart → Discounts → Delivery → Payment → Validation. Payment method discounts = impossible. |
| **`applyAttributeChange` unlocks mid-checkout logic** | Checkout UI can write cart attributes → triggers Discount Function re-run. |

---

## Environment Strategy

| Env | Purpose | Deployment |
|-----|---------|------------|
| Team | Dev sandbox | Auto on PR merge |
| Alpha | Integration testing | Auto on PR merge |
| Beta | Staging / UAT | Tag-based promotion |
| Prod | Live | Tag-based promotion |

**Pattern:** Single branch (`main`) + env-specific config files. Avoid long-lived env branches.

---

## Status Legend

| Icon | Meaning |
|------|---------|
| ✅ | Proven / working |
| 🟡 | In progress / needs upstream work |
| 🔴 | Not feasible with current Shopify model |
```
