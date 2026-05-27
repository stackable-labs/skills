---
name: hooks-api
description: "Extension hooks and API reference: createExtension, Surface, useCapabilities, createStore, useContextData. Use when writing extension runtime code."
---

# Hooks & API Reference

All imports from `@stackable-labs/sdk-extension-react`.

## createExtension(factory, options?)
Bootstrap the extension runtime. Call once in `src/index.tsx`.
- `factory: () => React.ReactElement` — render function returning all surfaces
- `options?: { extensionId?: string }` — extension identifier

```tsx
import { createExtension } from '@stackable-labs/sdk-extension-react'
import { Header } from './surfaces/Header'
import { Content } from './surfaces/Content'
import { Footer } from './surfaces/Footer'

const Extension = () => (
  <>
    <Header />
    <Content />
    <Footer />
  </>
)

// NOTE: extensionId is optional — used when connected to a registered extension
createExtension(() => <Extension />, { extensionId: 'my-extension' })
```

## Surface component
Wraps content for a target slot. The `id` must match a target in `manifest.json`.
- `<Surface id="slot.content">...</Surface>`

## useCapabilities()
Returns the capabilities object for calling host-mediated APIs.
```tsx
const capabilities = useCapabilities()
// capabilities.context.read()          — includes identity state in response
// capabilities.data.query(payload)
// capabilities.data.fetch(url, init?)
// capabilities.actions.toast(payload)
// capabilities.actions.invoke(action, payload?) — actions: newConversation, setConversationTags, setConversationFields, open, close, show, hide
// capabilities.identity.extend(patch) — push enrichment claims to user.metadata + JWT custom_claims (imperative; for handler-style at login, use useExtendIdentity hook). Each patch key MUST be declared in manifest.identityClaims or the host filter drops it.
```

## useStore(store, selector?)
Subscribe to a shared store. Re-renders when the selected state changes.
```tsx
const viewState = useStore(appStore, (s) => s.viewState)
```

## useContextData()
Reads host-provided context including extension settings. Returns `{ loading, customerId, customerEmail, messaging, settings, ... }`. `messaging.conversationId` is the active Messaging conversation ID (or `null` until one exists).
```tsx
const { loading, customerId, customerEmail, messaging, settings } = useContextData()
const conversationId = messaging?.conversationId
```

## useSettings()
Convenience hook for reading extension settings. Returns non-secret settings scoped to this extension on this instance.
```tsx
const settings = useSettings()
const apiBaseUrl = settings.baseUrl as string
```

## useSurfaceContext()
Returns host-provided context specific to the current surface slot.
```tsx
const surfaceContext = useSurfaceContext()
```

## useExtension()
Returns extension-level context.
```tsx
const { extensionId } = useExtension()
```

## createStore(initialState)
Create a shared store for cross-surface state coordination.
```tsx
const appStore = createStore<AppState>({ viewState: { type: 'menu' } })
```

### Store\<T\> interface
- `get(): T` — read current state
- `set(partial: Partial<T>): void` — merge partial state update
- `subscribe(listener: (state: T) => void): () => void` — subscribe, returns unsubscribe fn

## useIdentityEvent(eventType, handler)
Subscribe to identity events pushed from the host via the framework. Requires `events:identity` permission and matching entries in manifest `events` array.
- `eventType: 'login' | 'logout' | 'refresh' | 'expired'`
- `handler: (event: IdentityEvent) => void`
- `IdentityEvent: { eventName: IdentityEventType, data: { state: IdentityState, timestamp: string } }`
- `IdentityState: { authenticated: boolean, user: UserIdentity | null, expiresAt?: string }`

```tsx
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

## useMessagingEvent(eventType, handler)
Subscribe to messaging events (e.g. postback button clicks) pushed from the host widget. Requires `events:messaging` permission and matching entries in manifest `events` array.
- `eventType: 'postback' | 'postback:<actionName>'`
- `handler: MessagingEventHandler` — `(event: MessagingEvent) => void`
- `MessagingPostbackEvent: { eventName: 'postback', data: { actionName: string, conversationId: string, timestamp: string } }`
- `'postback'` receives ALL postback events (requires elevated marketplace review)
- `'postback:<actionName>'` receives only events matching the specific actionName

```tsx
useMessagingEvent('postback:Buy Now', (event) => {
  console.log('Postback:', event.data.actionName, event.data.conversationId)
})
```

With `useCallback` (for memoized handlers):
```tsx
import { useCallback } from 'react'
import { useMessagingEvent } from '@stackable-labs/sdk-extension-react'
import type { MessagingEventHandler } from '@stackable-labs/sdk-extension-contracts'

const handlePostback = useCallback<MessagingEventHandler>((event) => {
  console.log('Postback:', event.data.actionName, event.data.conversationId)
}, [])
useMessagingEvent('postback:Buy Now', handlePostback)
```

## useActivityEvent(eventType, handler)
Subscribe to activity events pushed from the host via the framework. Requires `events:activity` permission and matching entries in manifest `events` array.
- `eventType: 'click' | 'page_view' | 'form_submit' | 'product_view' | 'add_to_cart' | 'purchase' | 'search' | '*'` (domain-stripped)
- `handler: ActivityEventHandler` — `(event: ActivityEvent) => void`
- `ActivityEvent: { eventName: string, data: Record<string, unknown> }`
- `'*'` receives ALL activity events

```tsx
useActivityEvent('product_view', (event) => {
  console.log('Activity:', event.eventName, event.data)
})
```

## useEvent(eventType, handler)
Generic cross-domain event hook (should not be used unless absolutely required). Subscribe to any event using fully-qualified event types.
- `eventType: EventType` — fully-qualified (e.g., `'activity:product_view'`, `'identity:login'`, `'messaging:postback'`)
- Domain wildcard (e.g., `'activity'`) receives all events in that domain
- `handler: (event: BaseEvent) => void`

```tsx
useEvent('activity', (event) => {
  console.log('Activity:', event.data)
})
```

## useExtendIdentity(handler)
Register a handler to enrich identity JWT claims before signing. Requires `identity:extend` permission.
- `handler: ExtendIdentityHandler` — `(claims: IdentityBaseClaims) => Record<string, unknown> | Promise<Record<string, unknown>>`
- `IdentityBaseClaims: { external_id: string, email?: string, name?: string, [key: string]: unknown }`

```tsx
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

With `useCallback` (for memoized handlers):
```tsx
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
```

## Identity via context.read()
Identity state is available in the `context.read()` response as an `identity` field. Requires `context:read` permission (no separate identity permission needed).
```tsx
const context = await capabilities.context.read()
// context.identity — { authenticated, user, expiresAt? }
```
