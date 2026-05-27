---
name: add-capability
description: "Wire up a new capability (data.fetch, data.query, context.read, actions.toast, actions.invoke) in this extension. Use when adding a new platform-mediated API."
---

# Add a Capability

Wire up a new capability in this extension. Follow these steps exactly:

## 1. Determine the capability
Ask which capability to add. Valid capabilities:
- `data.query` — host-mediated data requests (action name + params → host returns data)
- `data.fetch` — direct HTTP requests from the sandbox (requires allowedDomains)
- `context.read` — read host-provided context (customerId, customerEmail, messaging.conversationId, etc.)
- `actions.toast` — show toast notifications (success, error, info, warning)
- `actions.invoke` — invoke host actions (e.g., open new conversation)
- `identity.extend` — enrich identity JWT claims before signing
- `events:identity` — subscribe to identity events (login, logout, refresh, expired)
- `events:messaging` — subscribe to messaging events (postback button clicks)
- `events:activity` — subscribe to activity events (page views, clicks, purchases)

## 2. Add permission to manifest.json
Add the corresponding permission to `packages/extension/public/manifest.json`:
- `data.query` → `"data:query"`
- `data.fetch` → `"data:fetch"`
- `context.read` → `"context:read"`
- `actions.toast` → `"actions:toast"`
- `actions.invoke` → `"actions:invoke"`
- `identity.extend` → `"identity:extend"`
- `events:identity` → `"events:identity"` (also add entries to `events` array, e.g. `["identity:login", "identity:logout"]`)
- `events:messaging` → `"events:messaging"` (also add entries to `events` array, e.g. `["messaging:postback:Buy Now"]`)
- `events:activity` → `"events:activity"` (also add entries to `events` array, e.g. `["activity:product_view"]`)

Only add if not already declared.

**Note:** Identity state is available via `context.read()` → `identity` field (requires `context:read`, no separate permission).

## 3. If data.fetch — add allowedDomains
Ask for the API domain(s) and add them to the `allowedDomains` array in manifest.json.

## 4. If data.fetch — create API wrapper
Create `packages/extension/src/lib/api.ts` with the API wrapper pattern:
```tsx
type FetchFn = (url: string, init?: FetchRequestInit) => Promise<FetchResponse>

export function createApi(fetch: FetchFn) {
  return {
    async getData(): Promise<DataType> {
      const result = await fetch('https://api.example.com/data', { method: 'GET' })
      if (!result.ok) throw new Error(`Request failed: ${result.status}`)
      return result.data as DataType
    },
  }
}
```

## 5. Use the capability in a surface

### For data.*, context.*, and actions.* capabilities — use `useCapabilities()`:
```tsx
import { useCapabilities } from '@stackable-labs/sdk-extension-react'

const capabilities = useCapabilities()
// data.query:    capabilities.data.query({ action: 'getItems', ... })
// data.fetch:    const api = createApi(capabilities.data.fetch)
// context.read:  capabilities.context.read()  (or use useContextData() hook)
// actions.toast: capabilities.actions.toast({ message: 'Done!', type: 'success' })
// actions.invoke: capabilities.actions.invoke('newConversation', { tags: ['order'], fields: [{ id: 'field_id', value: 'val' }] })
```

### Special handling (events + identity.extend):

#### For events — ALWAYS use dedicated hooks (INSTEAD of useCapabilities direct):
Events are subscribed via `useIdentityEvent` / `useMessagingEvent` / `useActivityEvent` — never use `capabilities.events.*` directly (events are not part of the `capabilities` object).

#### For identity.extend — choose the CORRECT option:
- **`useExtendIdentity(handler)`** — synchronous hook, fires only once at initial login. ALWAYS use for known-at-login enrichment.
- **`capabilities.identity.extend(patch)`** — imperative call via `useCapabilities()`, fires post-login (after async verification, webhook callbacks, user-triggered flows). Re-signs the JWT and broadcasts `identity:refresh`.

```tsx
// events:identity — use useIdentityEvent hook
import { useIdentityEvent } from '@stackable-labs/sdk-extension-react'

// manifest events: ["identity:login", "identity:logout", "identity:refresh"]
useIdentityEvent('login', (event) => {
  // event.data.state.user.metadata is populated with any enrichment from sibling
  // extensions with identity:extend (declared in their manifest.identityClaims)
  console.log('User logged in:', event.data.state.user?.email, event.data.state.user?.metadata)
})
useIdentityEvent('logout', () => {
  console.log('User logged out')
})
// identity:refresh fires after any extension calls capabilities.identity.extend({...}).
// Listen here to react to post-login enrichment (verification, tier upgrades, etc.).
useIdentityEvent('refresh', (event) => {
  console.log('Identity refreshed — metadata:', event.data.state.user?.metadata)
})
```

```tsx
// events:messaging — use useMessagingEvent hook
import { useMessagingEvent } from '@stackable-labs/sdk-extension-react'

useMessagingEvent('postback:Buy Now', (event) => {
  console.log('Postback:', event.data.actionName, event.data.conversationId)
})
```

```tsx
// events:activity — use useActivityEvent hook
import { useActivityEvent } from '@stackable-labs/sdk-extension-react'

useActivityEvent('product_view', (event) => {
  console.log('Activity:', event.eventName, event.data)
})
```

```tsx
// identity.extend — login-time useExtendIdentity hook OR post-login imperative capabilities.identity.extend
// Enrich identity JWT claims before signing.
// The host sends base claims (external_id, email, name); your handler returns
// additional claims to merge. Returned keys are filtered against manifest.identityClaims:
//   - Standard JWT claims (external_id, email, name) are exempt — always allowed
//   - Custom claims must be declared in manifest.identityClaims
//   - Undeclared keys are dropped with a console.warn
// Merged claims land in BOTH identityState.user.metadata (for sibling extensions)
// AND the signed JWT's custom_claims (for downstream JWT consumers).
// Pattern: name the handler with useCallback<ExtendIdentityHandler> for stable
// reference + clean separation; pass the named const to useExtendIdentity.
//
// useExtendIdentity fires ONCE at initial login — return what's known synchronously.
// For post-login async updates (e.g., after verification completes via a webhook
// or polling), use capabilities.identity.extend(patch) — see the 'identity.extend'
// capability for the imperative path.

import { useCallback } from 'react'
import { useExtendIdentity } from '@stackable-labs/sdk-extension-react'
import type { ExtendIdentityHandler } from '@stackable-labs/sdk-extension-contracts'

// manifest.json:
//   {
//     "permissions": ["identity:extend"],
//     "identityClaims": ["loyalty_tier", "verified", "verified_by", "verified_at"]
//   }
const handleExtend = useCallback<ExtendIdentityHandler>((claims) => ({
  external_id: `custom_${claims.external_id}`,   // standard claim override (exempt)
  loyalty_tier: 'bronze',                           // custom — sync, known at login
  verified: false,                                  // custom — default; updated async post-verification
}), [])
useExtendIdentity(handleExtend)

// ── For post-login async push (e.g., after async verification completes): ─────
const capabilities = useCapabilities()
await capabilities.identity.extend({ verified: true })

// Consumer side — same or sibling extension reacts via identity:refresh
useIdentityEvent('refresh', (event) => {
  console.log('verified =>', event.data.state.user?.metadata?.verified)
})

```

## 6. Verify
- Confirm the permission is in manifest.json
- If data.fetch, confirm the domain is in allowedDomains
- For data/context/actions: confirm accessed via `useCapabilities()` hook
- For events: confirm using `useIdentityEvent`, `useMessagingEvent`, or `useActivityEvent` hooks
- For identity.extend: confirm using `useExtendIdentity` hook (login-time) and/or `capabilities.identity.extend(patch)` (imperative post-login)
