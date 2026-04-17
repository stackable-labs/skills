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

## data.fetch — HTTP Requests to External APIs
Make HTTP requests to external APIs. Domain must be in `allowedDomains` in manifest. Requests are proxied through the Stackable platform server.
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

### Secret injection via `{{settings.xxx}}` placeholders
For API keys and tokens stored as `secret: true` fields in `settingsSchema`, use template placeholders in header values. The proxy resolves them server-side — **the real secret never enters extension code**.

```tsx
const result = await capabilities.data.fetch('https://api.example.com/orders', {
  method: 'GET',
  headers: {
    'X-API-Key': '{{settings.apiKey}}',
    'Authorization': 'Bearer {{settings.token}}',
  },
})
```

- Placeholders are only allowed in **header values** (not URLs, header names, or body)
- For `required: true` secret fields, the proxy returns 400 if the value is not configured
- For optional secret fields, the entire header is omitted if the value is not configured
- Declare secret fields in your `manifest.json` `settingsSchema` with `"secret": true`

## context.read — Read Host Context
Read host-provided context (customer ID, email, extension settings, etc.).
- **Permission required:** `context:read`
- **Usage:** `capabilities.context.read(): Promise<ContextData>`
- **ContextData shape:** `{ customerId?: string, customerEmail?: string, settings?: Record<string, unknown>, [key: string]: unknown }`
- **Convenience hooks:**
  - `useContextData()` returns `ContextData & { loading: boolean }`
  - `useSettings()` returns `Record<string, unknown>` — shorthand for `contextData.settings ?? {}`

```tsx
// Read all context (customer + settings)
const { loading, customerId, customerEmail, settings } = useContextData()

// Read only extension settings (convenience)
const settings = useSettings()
const apiBaseUrl = settings.baseUrl as string

// Alternative: use the capability directly
const context = await capabilities.context.read()
```

### Extension settings in context
Non-secret settings declared in `settingsSchema` are automatically available via `contextData.settings`. Values are scoped to the calling extension on the current instance — an extension never sees other extensions' settings.

- **Secret fields are never included** in context — use `{{settings.xxx}}` placeholders in `data.fetch` headers instead
- Settings propagate on **page load** — changes made in the admin dashboard take effect on the next page reload, not mid-session
- No new permission needed — `context:read` is the only gate

## actions.toast — Show Toast Notifications
Display a toast notification in the host UI.
- **Permission required:** `actions:toast`
- **Usage:** `capabilities.actions.toast(payload: ToastPayload): Promise<void>`
- **ToastPayload:** `{ message: string, type?: 'success'|'error'|'info'|'warning', duration?: number }`

```tsx
capabilities.actions.toast({ message: 'Saved!', type: 'success' })
```

## actions.invoke — Invoke Host Actions
Trigger host-defined actions (e.g., open a new conversation, set conversation tags/fields).
- **Permission required:** `actions:invoke`
- **Usage:** `capabilities.actions.invoke<T>(action: string, payload?: Record<string, unknown>): Promise<T>`
- **Available actions:**
  - `'newConversation'` — start a new Zendesk conversation (optionally with tags/fields)
  - `'setConversationTags'` — set tags on the current/next conversation
  - `'setConversationFields'` — set custom fields on the current/next conversation
  - `'open'` / `'close'` / `'show'` / `'hide'` — control the Zendesk messenger widget

```tsx
// New conversation with tags and fields
await capabilities.actions.invoke('newConversation', {
  tags: ['stackable', 'order-lookup'],
  fields: [{ id: 'stackable_action', value: 'order_status' }],
  metadata: { orderId: '12345' },
})

// Standalone: set tags on current/next conversation
await capabilities.actions.invoke('setConversationTags', ['escalated', 'order-issue'])

// Standalone: set custom fields
await capabilities.actions.invoke('setConversationFields', [
  { id: 'order_status', value: 'shipped' },
])
```

**Zendesk constraints:** Tags max 20, auto-lowercased/sanitized. Fields require `web_widget_conversation_ticket_metadata` feature flag. Both `conversationTags` and `conversationFields` **replace** on each call (not additive).

## events:identity — Identity Event Subscription
Subscribe to real-time identity events (login, logout, refresh, expired) pushed from the host.
- **Permission required:** `events:identity`
- **Manifest events array:** Declare specific events to listen for (e.g. `["identity:login", "identity:logout"]`)
- **Hook:** `useIdentityEvent(eventType, handler)`
- **Event types:** `'login' | 'logout' | 'refresh' | 'expired'`

```json
{
  "permissions": ["events:identity"],
  "events": ["identity:login", "identity:logout"]
}
```

```tsx
import { useIdentityEvent } from '@stackable-labs/sdk-extension-react'

useIdentityEvent('login', (event) => {
  console.log('User logged in:', event.data.state.user?.email)
})
useIdentityEvent('logout', () => {
  console.log('User logged out')
})
```

**Note:** Identity state is also available via `context.read()` → `identity` field (requires `context:read`, no separate permission needed).

## events:messaging — Messaging Event Subscription
Subscribe to messaging events (e.g. postback button clicks) pushed from the host widget.
- **Permission required:** `events:messaging`
- **Manifest events array:** Declare specific events to listen for (e.g. `["messaging:postback:Buy Now"]`) or `"messaging:postback"` for all postbacks (requires elevated marketplace review)
- **Hook:** `useMessagingEvent(eventType, handler)` — `MessagingEventHandler` type exported for use with `useCallback`
- **Event types:** `'postback'` (all postbacks) or `'postback:<actionName>'` (specific postback)
- **Important:** Only `postback`-type buttons fire this event. The Zendesk bot builder's "Present options" creates `reply`-type buttons (no event). Use the Sunshine Conversations API with `{ "type": "postback", "text": "Button Label", "payload": "..." }` actions to create postback buttons.
- **actionName caveat:** The `actionName` in the event is the button's display **text** (e.g. `"Add to cart"`), NOT the postback `payload` string. The payload is not exposed by the Zendesk Web Widget. Design manifest `events` entries to match button text: `"messaging:postback:Add to cart"`.

```json
{
  "permissions": ["events:messaging"],
  "events": ["messaging:postback:Add to cart", "messaging:postback:Check order"]
}
```

```tsx
import { useMessagingEvent } from '@stackable-labs/sdk-extension-react'

useMessagingEvent('postback:Buy Now', (event) => {
  console.log('Postback:', event.data.actionName, event.data.conversationId)
})
```

## events:activity — Activity Event Subscription
Subscribe to host activity events (e.g. page views, clicks, purchases) pushed from the host application.
- **Permission required:** `events:activity`
- **Manifest events array:** Declare specific events to listen for (e.g. `["activity:product_view", "activity:add_to_cart"]`) — manifest uses fully-qualified strings
- **Hook:** `useActivityEvent(eventType, handler)` — `ActivityEventHandler` type exported for use with `useCallback`
- **Event types (domain-stripped):** `'click' | 'page_view' | 'form_submit' | 'product_view' | 'add_to_cart' | 'purchase' | 'search' | '*'`
- **Well-known event names:**

| Event | Example payload fields |
|---|---|
| `page_view` | `{ url, title, referrer }` |
| `click` | `{ elementId, elementText, url }` |
| `product_view` | `{ productId, productName, price }` |
| `add_to_cart` | `{ productId, quantity, price }` |
| `purchase` | `{ orderId, total, currency, items }` |
| `search` | `{ query, resultCount }` |
| `form_submit` | `{ formId, formName, fields }` |

- `'*'` receives ALL activity events

```json
{
  "permissions": ["events:activity"],
  "events": ["activity:product_view", "activity:add_to_cart"]
}
```

```tsx
import { useActivityEvent } from '@stackable-labs/sdk-extension-react'

useActivityEvent('product_view', (event) => {
  console.log('Activity:', event.eventName, event.data)
})
```

**Generic alternative:** `useEvent('activity:product_view', handler)` — a cross-domain hook that accepts fully-qualified event types. Domain wildcard (e.g., `'activity'`) receives all events in that domain.

## extend:identity — Identity Claim Enrichment
Enrich identity JWT claims before signing. The host sends base claims to your extension, and you return additional claims to merge into the token.
- **Permission required:** `extend:identity`
- **Hook:** `useExtendIdentity(handler)` — `ExtendIdentityHandler` type exported for use with `useCallback`
- **Handler signature:** `(claims: IdentityBaseClaims) => Record<string, unknown> | Promise<Record<string, unknown>>`
- **IdentityBaseClaims:** `{ external_id: string, email?: string, name?: string, [key: string]: unknown }`

```json
{
  "permissions": ["extend:identity"]
}
```

```tsx
import { useExtendIdentity } from '@stackable-labs/sdk-extension-react'

useExtendIdentity((claims) => ({
  external_id: `custom_${claims.external_id}`,
  loyalty_tier: 'gold',
}))
```
