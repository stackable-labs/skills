---
name: surfaces
description: "Surface types, lifecycle, layout slots, and multi-surface composition. Use when adding or configuring surfaces in an extension."
---

# Surfaces

Surfaces are the UI slots where your extension renders content inside the host application.
Each surface maps to a specific layout position and is declared as a React component using
the `<Surface>` wrapper from `@stackable-labs/sdk-extension-react`.

## Surface Slots

| Slot | Target String | Typical Use |
|------|--------------|-------------|
| Header | `slot.header` | Compact status bar, quick actions, summary info |
| Content | `slot.content` | Main extension UI — forms, data display, workflows |
| Footer | `slot.footer` | Actions, save buttons, contextual controls |
| Footer Links | `slot.footer-links` | Navigation links, external references |

## Declaring a Surface

Each surface is a React component wrapped in `<Surface>`:

```tsx
import { Surface, ui } from '@stackable-labs/sdk-extension-react'

export function Content() {
  return (
    <Surface id="slot.content">
      <ui.Card>
        <ui.CardContent>
          <ui.Text>Extension content</ui.Text>
        </ui.CardContent>
      </ui.Card>
    </Surface>
  )
}
```

The `id` prop must match a target declared in your `manifest.json`:
```json
{
  "targets": ["slot.content"]
}
```

## Registering Surfaces

Surfaces are composed in the entry point (`src/index.tsx`) via `createExtension`:

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

`createExtension` bootstraps the extension runtime — it handles the sandboxed iframe
communication, capability injection, and surface registration with the host.

## Surface Lifecycle

1. **Mount** — Host creates an iframe for the extension and loads the entry point
2. **Register** — `createExtension` scans the rendered tree for `<Surface>` components
3. **Render** — Each `<Surface>` renders its children into the matching host slot
4. **Update** — Surfaces re-render when props, state, or context data changes
5. **Unmount** — Host removes the extension iframe when no longer needed

## Multi-Surface State

Surfaces share state via `createStore` (see Store & Navigation). Each surface can read
and write to the same store, enabling coordinated behavior across layout slots:

```tsx
import { useStore } from '@stackable-labs/sdk-extension-react'
import { appStore } from '../store'

export const Header = () => {
  const viewState = useStore(appStore, (s) => s.viewState)
  return (
    <Surface id="slot.header">
      <ui.Text>Current view: {viewState.type}</ui.Text>
    </Surface>
  )
}
```

## Best Practices

- **One surface per file** — keep surfaces in `src/surfaces/` with clear names (Header.tsx, Content.tsx)
- **Minimal surface components** — surfaces should compose feature components from `src/components/`
- **Always declare targets** — every `<Surface id="...">` must have a matching target in manifest.json
- **Use ScrollArea** — wrap content surfaces in `<ui.ScrollArea>` for overflow handling
