---
name: add-capability
description: "Wire up a new capability (data.fetch, data.query, context.read, actions.toast, actions.invoke) in this extension. Use when adding a new host-mediated API."
---

# Add a Capability

Wire up a new capability in this extension. Follow these steps exactly:

## 1. Determine the capability
Ask which capability to add. Valid capabilities:
- `data.query` — host-mediated data requests (action name + params → host returns data)
- `data.fetch` — direct HTTP requests from the sandbox (requires allowedDomains)
- `context.read` — read host-provided context (customerId, customerEmail, etc.)
- `actions.toast` — show toast notifications (success, error, info, warning)
- `actions.invoke` — invoke host actions (e.g., open new conversation)
- `extend:identity` — enrich identity JWT claims before signing
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
- `extend:identity` → `"extend:identity"`
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

### For events and extend — ALWAYS use dedicated hooks (INSTEAD of useCapabilities direct):
**IMPORTANT:** Event and extend capabilities have their own React hooks. Never use `capabilities.events.*` or `capabilities.extend.*` — those do not exist.

```tsx
// events:identity — use useIdentityEvent hook
import { useIdentityEvent } from '@stackable-labs/sdk-extension-react'

useIdentityEvent('login', (event) => {
  console.log('User logged in:', event.data.state.user?.email)
})
useIdentityEvent('logout', () => {
  console.log('User logged out')
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
// extend:identity — use useExtendIdentity hook
import { useExtendIdentity } from '@stackable-labs/sdk-extension-react'

useExtendIdentity((claims) => ({
  external_id: `custom_${claims.external_id}`,
  loyalty_tier: 'gold',
}))
```

## 6. Verify
- Confirm the permission is in manifest.json
- If data.fetch, confirm the domain is in allowedDomains
- For data/context/actions: confirm accessed via `useCapabilities()` hook
- For events: confirm using `useIdentityEvent`, `useMessagingEvent`, or `useActivityEvent` hooks
- For extend: confirm using `useExtendIdentity` hook
