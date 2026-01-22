# Med Man Sandbox Hub 🧪

## Quick Links
| Workstream | Repo |
|------------|------|
| [Discount Functions](#1-extended-shopify-discount-functionality) | [discount-manager-app](https://github.com/mssalemi/discount-manager-app) |
| [Checkout UI Extensions](#2-checkout-extensibility-checkout-ui-extensions) | [checkout-ui-add-order-metafields](https://github.com/mssalemi/checkout-ui-add-order-metafields) |
| [Customer Accounts](#3-customer-accounts--order-history) | [customer-accounts-headless-oauth-example](https://github.com/mssalemi/customer-accounts-headless-oauth-example) |
| [Fulfillment](#4-fulfillment) | [3pl-mock-vercel](https://github.com/mssalemi/3pl-mock-vercel) |
| [Self-Serve Returns](#5-self-serve-returns) | [selfserve-returns-shopify-app-mock](https://github.com/mssalemi/selfserve-returns-shopify-app-mock) |
| [Metafield Definition Syncer](#6-metafield-definition-syncer) | [meta-sync](https://github.com/mssalemi/mock-meta-syncer) |
| [SCAS-Lite OAuth Broker](#7-scas-lite-oauth-broker) | [scas-lite-broker](https://github.com/mssalemi/scas-lite-broker) |

---

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
**Status:** ✅ Core functionality validated

**What's proven:**
- Custom checkout fields (gift message, delivery notes) → Order Metafields
- Thank You page extensions with merchant-configurable settings
- `useSignalEffect` for reactive settings in Preact extensions
- `applyMetafieldChange` persists data to order metafields
- `applyAttributeChange` can update cart attributes mid-checkout
- Cart attribute changes trigger Discount Function re-evaluation
- Read cart attributes set via Storefront API
- Add items to cart mid-checkout via `applyCartLinesChange`
- Apply discount codes via `applyDiscountCodeChange`
- External 3P API calls with `network_access = true`

**Extensions built:**
| Extension | Target | Purpose |
|-----------|--------|---------|
| custom-checkout-fields | `purchase.checkout.block.render` | Collect gift message & delivery notes |
| cart-attributes-debug | `purchase.checkout.block.render` | Read & display cart attributes set via Storefront API |
| upsell-prototype | `purchase.checkout.block.render` | Add upsell item to cart via `applyCartLinesChange` |
| loyalty-points-mock | `purchase.checkout.reductions.render-before` | Mock loyalty: 3P API fetch + discount code pattern |
| post-purchase-order-metafields | `customer-account.order-status.block.render` | Display metafields + configurable returns link |
| thank-you-api-test | `purchase.thank-you.block.render` | External API calls (Cat Facts demo) |
| order-status-api-test | `customer-account.order-status.block.render` | External API calls + session token demo |

**Unlocked patterns:**
```
Checkout UI Extension → applyMetafieldChange → Order Metafields
Checkout UI Extension → applyAttributeChange → Discount Function reads attribute
Checkout UI Extension → applyCartLinesChange → Add items to cart (upsells)
Checkout UI Extension → applyDiscountCodeChange → Apply discount codes
Checkout UI Extension → shopify.attributes.value → Read cart attributes from Storefront API
Checkout UI Extension → sessionToken.get() → JWT with customer ID → secure 3P API calls
Checkout UI Extension → fetch() with network_access → 3P API integration
```
Enables: loyalty point redemption, user-selected discounts, upsells/cross-sells, experiment variants, custom order data, authenticated 3P integrations.

**Technical notes:**
- Requires `tsconfig.json` with `jsxImportSource: "preact"` for JSX
- Settings require `useSignalEffect` from `@preact/signals` to be reactive
- Metafields need definitions in Shopify Admin (Settings > Custom data > Orders)
- `network_access = true` required in toml for external fetch() calls
- Session token (`shopify.sessionToken.get()`) returns signed JWT with customer ID in `sub` claim
- Cart attributes set via Storefront API are readable via `shopify.attributes.value`

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

### 6) Metafield Definition Syncer
**Repo:** https://github.com/mssalemi/mock-meta-syncer
**Status:** ✅ Production-ready

A CLI tool for managing Shopify metafield and metaobject definitions as code. Single source of truth for custom data structures across environments.

**What's proven:**
- Define metafields/metaobjects in JSON config files
- Sync definitions across dev → uat → prod environments
- Version tracking via Shop metafield prevents drift
- Smart diff detection: create, update, recreate (validation changes), delete
- Safe by default: protected namespaces (`shopify`, `checkoutblocks`), diff-only mode

**Supported owner types:** PRODUCT, CUSTOMER, ORDER, COMPANY, ARTICLE, DRAFTORDER, COLLECTION, PRODUCTVARIANT, SHOP, + more (15 total)

**CLI usage:**
```bash
# Preview changes (safe)
npx tsx src/sync/index.ts --store-domain mystore.myshopify.com --access-token shpat_... --config-path ./src --diff-only

# Apply sync
npx tsx src/sync/index.ts --store-domain mystore.myshopify.com --access-token shpat_... --config-path ./src --enable-deletes
```

**Key pattern:**
```
JSON Config → Diff Detection → GraphQL Mutations → Version Update
```

**Stack:** TypeScript, `@shopify/admin-api-client`, Shopify Admin API 2025-10

---

### 7) SCAS-Lite OAuth Broker
**Repo:** https://github.com/mssalemi/scas-lite-broker  
**Status:** ✅ Production-ready, deployed to AWS  
**Deployment:** `https://vhmh0yyf7k.execute-api.us-east-1.amazonaws.com/`

A centralized OAuth broker for enterprise clients with multiple Shopify apps. Register apps once, any service can access Shopify APIs without touching secrets.

**What's proven:**
- OAuth install flow with HMAC verification
- Expiring offline tokens with automatic refresh (90-day refresh tokens)
- GraphQL proxy with automatic token injection
- Multi-app, multi-shop token management
- Embedded app support with app-specific status pages (iframe-ready)
- Client example library with reusable `ScasClient` class
- Compatible with `@shopify/admin-api-client` SDK
- Shop domain validation, request timeouts, comprehensive error handling
- Security best practices (token masking, production guidelines)

**Security model:**
```
┌─────────────────────────────────────────────────────┐
│  YOUR SERVICES (Fulfillment, Order Sync, etc.)      │
│  Know: broker URL, internal API key, app ID         │
│  Don't know: clientSecret, accessToken              │
└─────────────────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────┐
│  SCAS-LITE BROKER                                   │
│  Stores all Shopify secrets in DynamoDB             │
│  Handles token refresh automatically                │
│  Embedded app status pages per app                  │
└─────────────────────────────────────────────────────┘
                         │
                         ▼
                    Shopify API
```

**Usage patterns:**

**Option A: GraphQL Proxy (Recommended)**
```typescript
// Tokens never leave the broker
const res = await fetch(`${SCAS_BROKER_URL}/proxy/graphql`, {
  method: 'POST',
  headers: {
    'x-api-key': SCAS_API_KEY,
    'x-app-id': SCAS_APP_ID,
    'x-shop-domain': 'store.myshopify.com',
  },
  body: JSON.stringify({ query: `{ shop { name } }` }),
});
```

**Option B: Get Token + Use Official SDK**
```typescript
// Get token from broker
const { accessToken } = await fetch(`${SCAS_BROKER_URL}/internal/token`, {
  method: 'POST',
  headers: { 'x-api-key': SCAS_API_KEY },
  body: JSON.stringify({ appId: SCAS_APP_ID, shop: 'store.myshopify.com' }),
}).then(r => r.json());

// Use with official Shopify client
const client = createAdminApiClient({
  storeDomain: 'store.myshopify.com',
  apiVersion: '2026-01',
  accessToken,
});
```

**Client Example Library:**
- Reusable `ScasClient` class in `examples/client-example/`
- Features: shop domain validation, request timeouts (30s), comprehensive error handling
- Example scripts:
  - `get-token.ts` - Fetch access tokens
  - `use-proxy.ts` - GraphQL proxy usage
  - `use-shopify-client.ts` - Official SDK integration
  - `test-products.ts` - Production test script (fetches real products)
  - `full-example.ts` - Complete flow with error handling
- Security: Token masking utility, SECURITY.md with best practices
- Production-ready with TypeScript types and error codes

**Key endpoints:**
| Endpoint | Purpose | Auth |
|----------|---------|------|
| `POST /internal/apps` | Register a Shopify app (one-time) | API key |
| `GET /oauth/install` | Initiate OAuth flow (per shop) | Public |
| `POST /proxy/graphql` | Proxy GraphQL (broker injects token) | API key |
| `POST /internal/token` | Get access token (auto-refreshes) | API key |
| `POST /internal/token-exchange` | Exchange session token for offline token | API key |
| `GET /apps/:appId/shops/:shop/status` | Check installation status | Public |
| `GET /apps/:appId/shops` | List all shops for an app | Public |
| `GET /` | Embedded app status page (app-specific filtering) | Public |
| `POST /webhooks/app-uninstalled` | Handle app uninstall webhook | Public (HMAC verified) |

**Stack:** Fastify, TypeScript, AWS Lambda, DynamoDB, CDK, `@shopify/admin-api-client`

**Recent additions:**
- Embedded app support with iframe-ready status pages
- Client example library with 5 working example scripts
- Security documentation and token masking utilities
- Production test scripts for real API validation
- Comprehensive error handling and validation

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
