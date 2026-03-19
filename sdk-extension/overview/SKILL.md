---
name: overview
description: "Stackable Labs Extension SDK overview, packages, import conventions, and project structure. Use when starting a new extension, asking about the SDK, or needing architectural context."
---

# Stackable Labs Extension Development

## SDK Packages
This extension is built on the following npm packages. When generating code,
read the installed source in node_modules to ensure type-accurate imports:
- `@stackable-labs/sdk-extension-react` — hooks, createExtension, Surface, UI components
- `@stackable-labs/sdk-extension-contracts` — TypeScript types, constants, manifest schema

## Import Convention
Components are accessed via the `ui.*` namespace:
```tsx
import { ui, Surface, createExtension, useCapabilities } from '@stackable-labs/sdk-extension-react'
// Usage: <ui.Card>, <ui.Button>, <ui.Text>, etc.
// Do NOT import components directly (e.g., import { Card } will not work)
```

## Project Structure
```
packages/extension/
  public/
    manifest.json          # Extension manifest (targets, permissions, allowedDomains)
  src/
    index.tsx              # Entry point — createExtension bootstraps the runtime
    store.ts               # Shared store for cross-surface state (createStore)
    surfaces/              # One file per surface (Header.tsx, Content.tsx, Footer.tsx)
    components/            # Feature components rendered within surfaces
    lib/                   # API wrappers and utilities
packages/preview/          # Local preview host — DO NOT MODIFY
```

> Keep it minimal — the generation script handles most content automatically.

## Key Requirements

### Fetching Data
- No direct API calls using standard `fetch` allowed (blocked for security)
- Any external API calls must be made using the `data.fetch` capability
- Domain for any API call needs to be safelisted via `allowedDomains` in `manifest.json`

## Troubleshooting

### "Permission denied" errors
- Verify the capability's permission is declared in `manifest.json`
- Check that the permission string matches exactly (e.g., `data:fetch`, not `data.fetch`)

### data.fetch returns network errors
- Confirm the domain is in `allowedDomains` in `manifest.json`
- Only exact hostnames are allowed — no wildcards, no paths
- Check that the URL uses HTTPS

### Surface not rendering
- Confirm the `<Surface id="...">` matches a target in `manifest.json`
- Confirm the surface component is imported and rendered in `index.tsx`
- Check the browser console for errors

### Context data is undefined
- Ensure `context:read` permission is declared
- Always check the `loading` flag before using context values
- Context is only available when the host provides it (e.g., when viewing a ticket)

## Best Practices

### Performance
- Keep bundle size under 500KB
- Use `useStore(store, selector)` with narrow selectors to minimize re-renders
- Avoid fetching data in multiple surfaces — fetch once and share via store

### Code Organization
- One file per surface in `src/surfaces/`
- Shared state in `src/store.ts`
- API wrappers in `src/lib/`
- Feature components in `src/components/`
