---
name: capabilities
description: "Extension capabilities: data.query, data.fetch, context.read, actions.toast, actions.invoke. Use when wiring up host-mediated APIs or direct HTTP requests."
---

# Capabilities

Access capabilities via the `useCapabilities()` hook:
```tsx
const capabilities = useCapabilities()
```

## data.query — Host-Mediated Requests
The host handles the API call. Extension sends an action name + params, host returns data.
- **Permission required:** `data:query`
- **Usage:** `capabilities.data.query<T>(payload: ApiRequest): Promise<T>`
- **ApiRequest shape:** `{ action: string; [key: string]: unknown }`
- **When to use:** When the host application handles the API integration

```tsx
const result = await capabilities.data.query<Customer>({
  action: 'getCustomer',
  customerId: '123',
})
```

## data.fetch — Direct HTTP from Sandbox
Extension makes HTTP requests directly. Domain must be in `allowedDomains` in manifest.
- **Permission required:** `data:fetch`
- **Usage:** `capabilities.data.fetch(url: string, init?: FetchRequestInit): Promise<FetchResponse>`
- **FetchRequestInit:** `{ method?: 'GET'|'POST'|'PUT'|'PATCH'|'DELETE', headers?: Record<string,string>, body?: unknown }`
- **FetchResponse:** `{ status: number, ok: boolean, data: unknown }`
- **When to use:** When the extension calls external APIs directly

```tsx
const result = await capabilities.data.fetch('https://api.example.com/data', {
  method: 'GET',
  headers: { 'Authorization': 'Bearer token' },
})
if (!result.ok) throw new Error(`Request failed: ${result.status}`)
const data = result.data as MyType
```

## context.read — Read Host Context
Read host-provided context (customer ID, email, etc.).
- **Permission required:** `context:read`
- **Usage:** `capabilities.context.read(): Promise<ContextData>`
- **ContextData shape:** `{ customerId?: string, customerEmail?: string, [key: string]: unknown }`
- **Convenience hook:** `useContextData()` returns `ContextData & { loading: boolean }`

```tsx
// Preferred: use the hook
const { loading, customerId, customerEmail } = useContextData()

// Alternative: use the capability directly
const context = await capabilities.context.read()
```

## actions.toast — Show Toast Notifications
Display a toast notification in the host UI.
- **Permission required:** `actions:toast`
- **Usage:** `capabilities.actions.toast(payload: ToastPayload): Promise<void>`
- **ToastPayload:** `{ message: string, type?: 'success'|'error'|'info'|'warning', duration?: number }`

```tsx
capabilities.actions.toast({ message: 'Saved!', type: 'success' })
```

## actions.invoke — Invoke Host Actions
Trigger host-defined actions (e.g., open a new conversation).
- **Permission required:** `actions:invoke`
- **Usage:** `capabilities.actions.invoke<T>(action: string, payload?: Record<string, unknown>): Promise<T>`

```tsx
await capabilities.actions.invoke('newConversation', { subject: 'Follow-up' })
```

## events:identity — Identity Event Subscription
Subscribe to real-time identity events (login, logout, refresh, expired) pushed from the host.
- **Permission required:** `events:identity`
- **Manifest events array:** Declare specific events to listen for (e.g. `["identity.login", "identity.logout"]`)
- **Hook:** `useIdentityEvent(eventType, handler)`
- **Event types:** `'identity.login' | 'identity.logout' | 'identity.refresh' | 'identity.expired'`

```json
{
  "permissions": ["events:identity"],
  "events": ["identity.login", "identity.logout"]
}
```

```tsx
import { useIdentityEvent } from '@stackable-labs/sdk-extension-react'

useIdentityEvent('identity.login', (event) => {
  console.log('User logged in:', event.state.user?.email)
})
```

**Note:** Identity state is also available via `context.read()` → `identity` field (requires `context:read`, no separate permission needed).

## extend:identity — Identity Claim Enrichment
Enrich identity JWT claims before signing. The host sends base claims to your extension, and you return additional claims to merge into the token.
- **Permission required:** `extend:identity`
- **Hook:** `useIdentityExtend(handler)`
- **Handler signature:** `(claims: IdentityBaseClaims) => Record<string, unknown> | Promise<Record<string, unknown>>`
- **IdentityBaseClaims:** `{ external_id: string, email?: string, name?: string, [key: string]: unknown }`

```json
{
  "permissions": ["extend:identity"]
}
```

```tsx
import { useIdentityExtend } from '@stackable-labs/sdk-extension-react'

useIdentityExtend((claims) => ({
  external_id: `shopify_${claims.external_id}`,
  loyalty_tier: 'gold',
}))
```
