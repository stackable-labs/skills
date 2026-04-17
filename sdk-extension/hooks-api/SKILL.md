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
// capabilities.extend.identity(payload) — enrich identity claims (prefer useExtendIdentity hook)
```

## useStore(store, selector?)
Subscribe to a shared store. Re-renders when the selected state changes.
```tsx
const viewState = useStore(appStore, (s) => s.viewState)
```

## useContextData()
Reads host-provided context including extension settings. Returns `{ loading, customerId, customerEmail, settings, ... }`.
```tsx
const { loading, customerId, customerEmail, settings } = useContextData()
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
Subscribe to identity events pushed from the host. Requires `events:identity` permission and matching entries in manifest `events` array.
- `eventType: 'login' | 'logout' | 'refresh' | 'expired'`
- `handler: (event: IdentityEvent) => void`
- `IdentityEvent: { eventName: IdentityEventType, data: { state: IdentityState, timestamp: string } }`
- `IdentityState: { authenticated: boolean, user: UserIdentity | null, expiresAt?: string }`

```tsx
useIdentityEvent('login', (event) => {
  console.log('User logged in:', event.data.state.user?.email)
})
useIdentityEvent('logout', () => {
  console.log('User logged out')
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
Subscribe to host activity events. Requires `events:activity` permission and matching entries in manifest `events` array.
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
Register a handler to enrich identity JWT claims before signing. Requires `extend:identity` permission.
- `handler: ExtendIdentityHandler` — `(claims: IdentityBaseClaims) => Record<string, unknown> | Promise<Record<string, unknown>>`
- `IdentityBaseClaims: { external_id: string, email?: string, name?: string, [key: string]: unknown }`

```tsx
useExtendIdentity((claims) => ({
  external_id: `custom_${claims.external_id}`,
  loyalty_tier: 'gold',
}))
```

With `useCallback` (for memoized handlers):
```tsx
import { useCallback } from 'react'
import { useExtendIdentity } from '@stackable-labs/sdk-extension-react'
import type { ExtendIdentityHandler } from '@stackable-labs/sdk-extension-contracts'

const handleExtend = useCallback<ExtendIdentityHandler>((claims) => ({
  external_id: `custom_${claims.external_id}`,
  loyalty_tier: 'gold',
}), [])
useExtendIdentity(handleExtend)
```

## Identity via context.read()
Identity state is available in the `context.read()` response as an `identity` field. Requires `context:read` permission (no separate identity permission needed).
```tsx
const context = await capabilities.context.read()
// context.identity — { authenticated, user, expiresAt? }
```
