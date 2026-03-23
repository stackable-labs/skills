---
name: project-structure
description: "Extension project file structure, entry point, surfaces, store, and manifest configuration."
---

# Project Structure

A Stackable extension project is a monorepo with two packages: the extension itself
and a local preview host for development.

## Directory Layout

```
my-extension/
  packages/
    extension/                 # Your extension code
      public/
        manifest.json          # Extension manifest
      src/
        index.tsx              # Entry point
        store.ts               # Shared state (createStore)
        surfaces/              # Surface components
          Header.tsx           # slot.header
          Content.tsx          # slot.content
          Footer.tsx           # slot.footer
        components/            # Feature components
        lib/                   # API wrappers, utilities
    preview/                   # Local preview host — DO NOT MODIFY
  package.json                 # Workspace root
  tsconfig.json
```

## Key Files

### manifest.json

The extension manifest declares what the extension needs from the host:

```json
{
  "name": "My Extension",
  "version": "0.1.0",
  "targets": ["slot.header", "slot.content", "slot.footer"],
  "permissions": ["context:read", "data:fetch"],
  "allowedDomains": ["api.example.com"]
}
```

| Field | Purpose |
|-------|---------|
| `name` | Display name shown to users |
| `version` | Semver version string |
| `targets` | Surface slots the extension renders into |
| `permissions` | Capabilities the extension uses |
| `allowedDomains` | Hostnames for data.fetch requests (exact match, no wildcards) |

### index.tsx — Entry Point

The entry point bootstraps the extension runtime:

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

createExtension(<Extension />)
```

`createExtension` handles:
- Sandboxed iframe communication with the host
- Capability injection (makes `useCapabilities()` work)
- Surface registration (maps `<Surface id="...">` to host layout slots)

**Do not modify `index.html`** — the extension always bootstraps through `createExtension`.

### store.ts — Shared State

The store provides cross-surface state management:

```tsx
import { createStore } from '@stackable-labs/sdk-extension-react'

type View = { type: 'list' } | { type: 'detail'; id: string }

interface AppState {
  view: View
  navigate: (view: View) => void
}

export const store = createStore<AppState>((set) => ({
  view: { type: 'list' },
  navigate: (view) => set({ view }),
}))
```

All surfaces import from the same store instance. Use `useStore(store, selector)`
to read state with minimal re-renders.

### Surface Files

Each surface is a React component in `src/surfaces/`:

```tsx
import { Surface, ui } from '@stackable-labs/sdk-extension-react'

export const Content = () => (
  <Surface id="slot.content">
    <ui.Card>
      <ui.CardContent>
        <ui.Text>Extension content</ui.Text>
      </ui.CardContent>
    </ui.Card>
  </Surface>
)
```

The `id` prop must match a target in `manifest.json`.

### Preview Host

The `packages/preview/` directory contains a local preview host that simulates
the production environment. **Do not modify this directory** — it is pre-configured
and managed by the SDK tooling.

## Import Conventions

All SDK imports come from two packages:

```tsx
// Runtime: hooks, components, utilities
import { ui, Surface, createExtension, useCapabilities, createStore, useStore, useContextData }
  from '@stackable-labs/sdk-extension-react'

// Types and constants (dev-time only)
import type { ExtensionManifest } from '@stackable-labs/sdk-extension-contracts'
```

Components use the `ui.*` namespace — do not import components directly.
