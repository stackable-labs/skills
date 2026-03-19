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
createExtension(
  () => (
    <>
      <Header />
      <Content />
      <Footer />
    </>
  ),
  { extensionId: 'my-extension' },
)
```

## Surface component
Wraps content for a target slot. The `id` must match a target in `manifest.json`.
- `<Surface id="slot.content">...</Surface>`

## useCapabilities()
Returns the capabilities object for calling host-mediated APIs.
```tsx
const capabilities = useCapabilities()
// capabilities.data.query(payload)
// capabilities.data.fetch(url, init?)
// capabilities.context.read()
// capabilities.actions.toast(payload)
// capabilities.actions.invoke(action, payload?)
```

## useStore(store, selector?)
Subscribe to a shared store. Re-renders when the selected state changes.
```tsx
const viewState = useStore(appStore, (s) => s.viewState)
```

## useContextData()
Reads host-provided context. Returns `{ loading, customerId, customerEmail, ... }`.
```tsx
const { loading, customerId, customerEmail } = useContextData()
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
