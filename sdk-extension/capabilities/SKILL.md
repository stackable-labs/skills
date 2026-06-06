---
name: capabilities
description: "Extension capabilities: data.query, data.fetch, context.read, actions.toast, actions.invoke. Use when wiring up platform-mediated APIs or direct HTTP requests."
---

# Capabilities

Access capabilities via the `useCapabilities()` hook:
```tsx
const capabilities = useCapabilities()
```

## data.query — Platform-Mediated Requests
The Stackable platform handles the API call. Extension sends an action name + params, the platform returns data.
- **Permission required:** `data:query`
- **Usage:** `capabilities.data.query<T>(payload: ApiRequest): Promise<T>`
- **ApiRequest shape:** `{ action: string; [key: string]: unknown }`
- **When to use:** When the platform handles the API integration

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

> See [Instance Settings](./instance-settings) for the full schema-declaration + storage-mode story, including which field types accept `secret: true`.

## context.read — Read Platform Context
Read framework-provided context (customer ID, email, messaging conversation, extension settings, etc.).
- **Permission required:** `context:read`
- **Usage:** `capabilities.context.read(): Promise<ContextData>`
- **ContextData shape:** `{ customerId?: string, customerEmail?: string, messaging?: { conversationId?: string | null, appId?: string | null }, settings?: Record<string, unknown>, [key: string]: unknown }`
- **Convenience hooks:**
  - `useContextData()` returns `ContextData & { loading: boolean }`
  - `useSettings()` returns `Record<string, unknown>` — shorthand for `contextData.settings ?? {}`

```tsx
// Read all context (customer + messaging + settings)
const { loading, customerId, customerEmail, messaging, settings } = useContextData()
const conversationId = messaging?.conversationId

// Read only extension settings (convenience)
const settings = useSettings()
const apiBaseUrl = settings.baseUrl as string

// Alternative: use the capability directly
const context = await capabilities.context.read()
```

### Messaging context
`messaging.conversationId` is the active Messaging conversation ID, or `null` until the widget has an open conversation. Use this when you need the ID for an API call from a non-event surface. If you're already reacting to a postback button click, prefer reading `event.data.conversationId` from `useMessagingEvent` — it doesn't require `context:read`.

### Extension settings in context
Non-secret settings declared in `settingsSchema` are automatically available via `contextData.settings`. Values are scoped to the calling extension on the current instance — an extension never sees other extensions' settings.

- **Secret fields are never included** in context — use `{{settings.xxx}}` placeholders in `data.fetch` headers instead
- Settings propagate on **page load** — changes made in the admin dashboard take effect on the next page reload, not mid-session
- No new permission needed — `context:read` is the only gate

## actions.toast — Show Toast Notifications
Display a toast notification in the framework widget's UI.
- **Permission required:** `actions:toast`
- **Usage:** `capabilities.actions.toast(payload: ToastPayload): Promise<void>`
- **ToastPayload:** `{ message: string, type?: 'success'|'error'|'info'|'warning', duration?: number }`

```tsx
capabilities.actions.toast({ message: 'Saved!', type: 'success' })
```

## actions.invoke — Invoke Platform Actions
Trigger framework-defined actions (e.g., open a new conversation, set conversation tags/fields).
- **Permission required:** `actions:invoke`
- **Usage:** `capabilities.actions.invoke<T>(action: string, payload?: Record<string, unknown>): Promise<T>`
- **Available actions:**
  - `'newConversation'` — start a new Messaging conversation (optionally with tags/fields)
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
Subscribe to real-time identity events (login, logout, refresh, expired) pushed from the host via the framework.
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
Subscribe to activity events (e.g. page views, clicks, purchases) pushed from the host via the framework.
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

## identity.extend — Identity Claim Enrichment
Enrich identity JWT claims and `identityState.user.metadata` so the current user's signed token AND any sibling extension can react to them. Two complementary APIs:

1. **`useExtendIdentity(handler)`** — synchronous hook that fires ONCE at initial login.
2. **`capabilities.identity.extend(patch)`** — imperative call that fires at any time after login (post-verification, post-checkout, any user-triggered async flow). Re-signs the JWT, updates `user.metadata`, and broadcasts `identity:refresh`.

Both share the **`identity:extend`** permission and the **`manifest.identityClaims`** declaration gate.

### Manifest contract

```json
{
  "permissions": ["identity:extend"],
  "identityClaims": ["loyalty_tier", "verified", "verified_by", "verified_at"]
}
```

- **Standard JWT claims (`external_id`, `email`, `name`) are exempt** — they're part of the signing contract and may be overridden without declaration.
- **Custom keys MUST be declared in `identityClaims`** or the host filter drops them with a `console.warn`.
- **Reserved JWT/Zendesk keys** (`iss`, `sub`, `aud`, `exp`, `nbf`, `iat`, `jti`, `scope`, `email_verified`, `user_fields`) **cannot** appear in `identityClaims` — the Lambda sanitizer always wins on collision.
- **Key format:** `/^[a-z_][a-z0-9_]{0,63}$/` (lowercase identifier, ≤64 chars).
- **Maximum 20 entries.**

### Initial-login enrichment (handler-style)

- **Hook:** `useExtendIdentity(handler)` — `ExtendIdentityHandler` type exported for use with `useCallback`
- **Handler signature:** `(claims: IdentityBaseClaims) => Record<string, unknown> | Promise<Record<string, unknown>>`
- **IdentityBaseClaims:** `{ external_id: string, email?: string, name?: string, [key: string]: unknown }`

```tsx
import { useExtendIdentity } from '@stackable-labs/sdk-extension-react'

// manifest.json:
//   {
//     "permissions": ["identity:extend"],
//     "identityClaims": ["loyalty_tier", "verified", "verified_by", "verified_at"]
//   }
// Standard JWT claims (external_id, email, name) are exempt from declaration.
// Custom claims must be in manifest.identityClaims or they're dropped with a warn.
//
// Fires ONCE at initial login — return what's known synchronously. For post-login
// async updates (e.g., after verification completes via a webhook or polling),
// use capabilities.identity.extend(patch) — see the 'identity.extend' capability.
useExtendIdentity((claims) => ({
  external_id: `custom_${claims.external_id}`,   // standard claim override (exempt)
  loyalty_tier: 'bronze',                           // custom — sync, known at login
  verified: false,                                  // custom — default; updated async post-verification
}))
```

### Async post-login push (imperative)

```tsx
const capabilities = useCapabilities()

// Anytime after login — webhook callback, user action, async verification, etc.
await capabilities.identity.extend({
  verified: true,
  verified_by: 'xyzProvider',
  verified_at: new Date().toISOString(),
})
// → host filters against manifest.identityClaims
// → user.metadata updated
// → JWT re-signed, pushed to Zendesk loginUser
// → identity:refresh broadcast to all extensions with events:identity
```

Per-extension calls are serialized to prevent concurrent `loginUser` callbacks from orphaning each other.

### Consuming enriched state from another extension

Any extension with `events:identity` permission can react to enrichment updates. Same minimal pattern as the `events:identity` section above — `'login'` covers the initial enriched state, `'refresh'` covers post-login pushes:

```tsx
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

For a snapshot read instead of event-driven reaction (auto re-renders when context changes):

```tsx
const { identity } = useContextData()
const verified = Boolean(identity?.user?.metadata?.verified)
```

### Install-time enforcement

Two enabled extensions on the same instance **MUST NOT** declare overlapping `identityClaims` keys. The runtime merge is order-dependent (`Object.assign` across per-extension contributions), so one extension's value would silently overwrite the other's. The marketplace install API blocks the install with a 409 conflict, surfacing the specific overlapping key + conflicting extension name in the admin install dialog. Coordinate keys with downstream extensions or namespace them (e.g. `<vendor>_loyalty_tier`).

### Bundle-scan findings at upload

The publisher-side bundle scan validates your declaration at submission time:

| Finding | Severity | Triggers when |
| --- | --- | --- |
| `identityClaims_missing` | warning | `identity:extend` declared, `identityClaims` empty (custom claims would be dropped) |
| `identityClaims_no_permission` | warning | `identityClaims` declared, `identity:extend` permission missing |
| `identityClaims_invalid_key` | error | Key fails the format regex |
| `identityClaims_reserved_key` | error | Collides with a reserved JWT / Zendesk claim |
| `identityClaims_standard_key` | warning | Redundant — standard claims (`external_id`, `email`, `name`) are exempt |
| `identityClaims_too_many` | error | More than 20 entries |

## messaging.send — Send Messages to Conversations

Post a message into the **active conversation** bound to the current Instance. The host attributes each message to the extension via the per-Instance author label set by the admin (see "Author label" below). Two complementary APIs:

1. **`useMessaging()`** — React hook returning tuple `[send, { loading, error, data }]` — lets callers rename `send` per-instance when used multiple times in one component. Preferred for UI components; narrows the error surface to actionable codes only.
2. **`messagingSendCapability(payload)`** / **`capabilities.messaging.send(payload)`** — imperative; useful outside React render. Exposes the full wire-level error taxonomy.

### Manifest contract

```json
{
  "permissions": ["messaging:send"]
}
```

- **`messaging:send` permission** — without it the host gate rejects the call before any network request.

### Author label (host-resolved, per-Instance)

Extensions do **not** set the author label. The host resolves it from:
1. `instance.config.settings.messagingDisplayName` — admin sets via the dashboard Instance form (visible only after OAuth completes; gated on `messagingAppId` being populated)
2. **Fallback**: `extension.manifest.name` — used when admin hasn't set a label

This prevents impersonation (extension A can't author as extension B) and gives admins per-deployment branding control.

### Runtime requirements

1. **Instance connected to a messaging provider** — bearer token stored, ready to post
2. **An active conversation** when `send()` is called (else host-handled `no_conversation` — `send()` returns `null`)
3. **Instance not disconnected** from the messaging provider (else host-handled `reauth_required` — admin reconnect surfaced in dashboard)

### Payload — discriminated by `kind`

Four kinds:

| kind | Required fields | Optional |
|---|---|---|
| `'text'` | `body: string` | `actions?: MessageItemAction[]`, `metadata?`, `htmlText?`, `markdownText?`, `disableUserInput?` |
| `'image'` | `url: string`, `altText: string` | `body?`, `actions?`, `metadata?` |
| `'file'` | `url: string`, `altText: string` | `body?`, `metadata?` (no `actions` — file kind doesn't accept item-level actions) |
| `'carousel'` | `items: [MessageItem, ...MessageItem[]]` (1–10 items, each with `title`, `description?`, `imageUrl?`, `actions?`) | `displaySettings?: { imageAspectRatio?: 'square' \| 'horizontal' }` |

**Action types** (per-item or per-message): `reply` (quick-reply button — emits postback to `useMessagingEvent`), `link` (opens URL), `postback` (custom payload to your handler), `locationRequest` (asks user for location).

### Hook usage

```tsx
import { useMessaging, useContextData } from '@stackable-labs/sdk-extension-react'

const { messaging } = useContextData()
const [send, { loading, error }] = useMessaging()

// Proactive gate: skip the call when there's no conversation — avoids the
// host-handled no_conversation log + null return.
const canSend = !!messaging?.conversationId

const onApprove = async () => {
  try {
    await send({
      kind: 'text',
      body: 'Order approved ✓',
      actions: [{ type: 'reply', label: 'Got it', payload: 'ACK' }],
    })
  } catch {
    // error holds the typed SendMessageActionableErrorCode
  }
}

if (error === 'rate_limited') {
  // Render a "slow down" notice
}
```

### Imperative usage

```tsx
const capabilities = useCapabilities()
const result = await capabilities.messaging.send({ kind: 'text', body: 'Hello' })
// result is either { messageId, receivedAt } OR { error: SendMessageErrorCode }
```

### Typed error codes — split by who acts

The wire taxonomy has 6 codes. The hook (`useMessaging`) narrows them into two groups:

**Actionable (extension catches + renders UI)** — `send()` throws and `state.error` is populated:

| Code | When |
| --- | --- |
| `invalid_message` | Payload failed validation (e.g. empty `body` for text, carousel >10 items, list kind sent) |
| `rate_limited` | Upstream rate limit hit — back off + retry |
| `upstream_error` | Provider returned 5xx — transient; retry with backoff |

**Host-handled (extension ignores)** — `send()` resolves to `null`, `state.error` stays `null`, SDK logs a breadcrumb:

| Code | What the framework does |
| --- | --- |
| `no_conversation` | `console.info` — pre-empt via `useContextData().messaging?.conversationId` |
| `reauth_required` | `console.warn` — admin sees a "Reconnect" CTA in the dashboard (server flips `messagingDisconnected: true`) |
| `forbidden` | `console.warn` — should not reach in production with correct manifest |

The imperative `messagingSendCapability` path returns the full taxonomy without the actionable/host-handled split — useful when you need every code (e.g. dynamic dispatch).

### Defense-in-depth gates

Three gates fire in series, surfacing the same `forbidden` error if any blocks:

1. **Embeddable host gate** (`CapabilityRPCHandler`) — checks `sandbox.manifest.permissions.includes('messaging:send')` before the call leaves the sandbox.
2. **Backend handler gate** — checks `claims.permissions?.includes('messaging:send')` on the proxy-token JWT.
3. **Server-side instance check** — `instance.config.messagingDisconnected` short-circuits with `reauth_required` before any upstream call.

### Receiving replies — pair with `events:messaging`

`reply` and `postback` action clicks fire on extensions with the `events:messaging` permission via `useMessagingEvent` — see the `events:messaging` section above.

> **Note:** the postback `payload` field is currently not surfaced by the Web Widget (only the button text `actionName`). Until that gap is closed, design action labels to be self-describing or pair sends with a follow-up `data.query` lookup.
