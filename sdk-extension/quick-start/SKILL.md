---
name: quick-start
description: "Step-by-step guide to create, develop, and deploy your first Stackable extension."
---

# Quick Start

Create, develop, and deploy your first Stackable extension in minutes.

## Prerequisites

- **Node.js** 22 or later
- **pnpm** (recommended) or npm

## 1. Create a New Extension

Scaffold a new extension project:

```bash
pnpm --config.dlx-cache-max-age=0 dlx @stackable-labs/create-extension my-extension
cd my-extension
pnpm install
```

This creates a project with:
- A working extension with header, content, and footer surfaces
- A local preview host for development
- TypeScript configuration
- A manifest with default permissions and targets

## 2. Preview Your Extension

Run the Stackable CLI dev server from your project directory:

```bash
pnpm --config.dlx-cache-max-age=0 dlx @stackable-labs/cli-app-extension@latest dev
```

> **Note:** This uses `pnpm dlx` to run the Stackable CLI without installing it
> globally. The `--config.dlx-cache-max-age=0` flag ensures you always get the
> latest version.

The dev command:
1. Starts a local Vite dev server for your extension with hot reload
2. Creates a public Cloudflare tunnel to your local server
3. Displays a **query parameter** you can use to preview your extension

### Testing against your host app

The CLI outputs a query param like:

```
?_stackable_dev=ext-123%3Ahttps%3A%2F%2Fabc.trycloudflare.com
```

Copy this and **append it to your host application URL** to load your local extension
instead of the production bundle. For example:

```
https://your-app.com/dashboard?_stackable_dev=ext-123%3Ahttps%3A%2F%2Fabc.trycloudflare.com
```

This override is **browser-session only** — no database changes, no shared state.
Each developer gets isolated overrides. Changes you make locally appear immediately
in the host app via hot reload.

Use `--no-tunnel` if you only want to run the local Vite dev server without a tunnel.

## 3. Explore the Project

```
my-extension/
  packages/
    extension/
      public/
        manifest.json       # Targets, permissions, allowed domains
      src/
        index.tsx           # Entry point — createExtension bootstraps the runtime
        store.ts            # Shared state across surfaces
        surfaces/           # One component per surface slot
          Header.tsx
          Content.tsx
          Footer.tsx
        components/         # Feature components used by surfaces
        lib/                # API wrappers and utilities
    preview/                # Local preview host — DO NOT MODIFY
```

## 4. Make Your First Change

Open `packages/extension/src/surfaces/Content.tsx` and modify the content:

```tsx
import { Surface, ui } from '@stackable-labs/sdk-extension-react'

export const Content = () => (
  <Surface id="slot.content">
    <ui.Card>
      <ui.CardContent>
        <ui.Heading level={3}>Hello, Stackable!</ui.Heading>
        <ui.Text>My first extension is working.</ui.Text>
      </ui.CardContent>
    </ui.Card>
  </Surface>
)
```

Save the file — the preview updates automatically.

## 5. Add a Capability

To read host context data (customer info, etc.):

1. Add the permission to `manifest.json`:
```json
{
  "permissions": ["context:read"]
}
```

2. Use it in a surface:
```tsx
import { Surface, useContextData, ui } from '@stackable-labs/sdk-extension-react'

export const Content = () => {
  const { loading, customerId } = useContextData()

  if (loading) return <Surface id="slot.content"><ui.Skeleton /></Surface>

  return (
    <Surface id="slot.content">
      <ui.Card>
        <ui.CardContent>
          <ui.Text>Customer: {customerId}</ui.Text>
        </ui.CardContent>
      </ui.Card>
    </Surface>
  )
}
```

## 6. Validate & Deploy

Before deploying, validate your extension:

```bash
pnpm --config.dlx-cache-max-age=0 dlx @stackable-labs/cli-app-extension@latest validate
```

This checks manifest structure, permission usage, surface targets, and import patterns.

When ready, deploy:

```bash
pnpm --config.dlx-cache-max-age=0 dlx @stackable-labs/cli-app-extension@latest deploy
```

## Next Steps

- **[Components](/docs/reference/components)** — browse the full UI component catalog
- **[Capabilities](/docs/reference/capabilities)** — learn about data.query, data.fetch, and more
- **[Surfaces](/docs/reference/surfaces)** — understand surface types and layout slots
- **[Patterns](/docs/reference/patterns)** — see common extension code patterns
