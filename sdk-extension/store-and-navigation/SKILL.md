---
name: store-and-navigation
description: "Cross-surface state management with createStore, useStore selectors, and store-based navigation patterns. Use when managing shared state or view navigation."
---

# Store & Navigation

Extensions use `createStore` for cross-surface state management. The store is shared
across all surfaces (header, content, footer), enabling coordinated UI updates from
a single state source.

## Creating a Store

Define your store in `src/store.ts`:

```tsx
import { createStore } from '@stackable-labs/sdk-extension-react'

type ViewState = { type: 'list' } | { type: 'detail'; id: string }

interface AppState {
  viewState: ViewState
}

export const appStore = createStore<AppState>({
  viewState: { type: 'list' },
})
```

## Reading State in Surfaces

Use `useStore` with a selector to subscribe to specific slices of state:

```tsx
import { useStore } from '@stackable-labs/sdk-extension-react'
import { appStore } from '../store'

export const Content = () => {
  const viewState = useStore(appStore, (s) => s.viewState)

  if (viewState.type === 'list') {
    return <ListView onSelect={(id) => appStore.set({ viewState: { type: 'detail', id } })} />
  }
  return <DetailView id={viewState.id} onBack={() => appStore.set({ viewState: { type: 'list' } })} />
}
```

## Store-Based Navigation

Use discriminated unions for view state instead of string constants or URL routing.
The store replaces traditional routing — there are no URLs inside the sandbox.

### View State Pattern

```tsx
// Define all possible views as a discriminated union
type ViewState =
  | { type: 'list' }
  | { type: 'detail'; id: string }
  | { type: 'edit'; id: string }
  | { type: 'create' }

// TypeScript narrows the type based on the discriminant
switch (viewState.type) {
  case 'list':    return <ListView />
  case 'detail':  return <DetailView id={viewState.id} />
  case 'edit':    return <EditView id={viewState.id} />
  case 'create':  return <CreateView />
}
```

### Cross-Surface Coordination

The header can show context based on what the content surface displays:

```tsx
// Header surface — shows back button when on detail view
export const Header = () => {
  const viewState = useStore(appStore, (s) => s.viewState)

  return (
    <Surface id="slot.header">
      <ui.Inline>
        {viewState.type !== 'list' && (
          <ui.Button
            variant="ghost"
            onClick={() => appStore.set({ viewState: { type: 'list' } })}
          >
            Back
          </ui.Button>
        )}
        <ui.Heading level={3}>
          {viewState.type === 'list' ? 'All Items' : 'Item Detail'}
        </ui.Heading>
      </ui.Inline>
    </Surface>
  )
}
```

## Selector Performance

Use narrow selectors to minimize re-renders. Each `useStore` call only
triggers a re-render when its selected value changes:

```tsx
// Good — component only re-renders when viewState changes
const viewState = useStore(appStore, (s) => s.viewState)

// Bad — component re-renders on ANY state change
const state = useStore(appStore, (s) => s)
```

## Best Practices

- **One store per extension** — define in `src/store.ts`, import everywhere
- **Discriminated unions for views** — TypeScript can narrow the type and catch missing cases
- **Narrow selectors** — select only what the component needs
- **Use `appStore.set()`** — update state directly via `appStore.set({ viewState: ... })` from any component
- **No URL routing** — the extension runs in a sandboxed iframe with no URL bar
