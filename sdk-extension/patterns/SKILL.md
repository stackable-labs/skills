---
name: patterns
description: "Code patterns extracted from the reference extension: entry point, store navigation, surface composition, API wrapper. Use when implementing common extension patterns."
---

# Code Patterns

## Entry Point

```tsx
// src/index.tsx
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

createExtension(() => <Extension />, { extensionId: '__EXTENSION_ID__' })
```

## Store-Based Navigation

```tsx
// src/store.ts
import { createStore } from '@stackable-labs/sdk-extension-react'

export type ViewState = { type: 'menu' }

export interface AppState {
  viewState: ViewState
}

export const appStore = createStore<AppState>({
  viewState: { type: 'menu' },
})
```

## Content Surface with Loading State

```tsx
// src/surfaces/Content.tsx
import { ui, useStore, useContextData, Surface } from '@stackable-labs/sdk-extension-react'
import { appStore } from '../store'

export function Content() {
  const viewState = useStore(appStore, (s) => s.viewState)
  const { loading } = useContextData()
  // Non-secret settings from settingsSchema (add settingsSchema to manifest.json to use). Ex:
  // const settings = useSettings()
  // const apiEndpoint = settings.apiEndpoint as string

  if (loading) {
    return (
      <Surface id="slot.content">
        <ui.Stack direction="column" gap="2" className="animate-pulse">
          <ui.Card className="h-24" />
          <ui.Card className="h-32" />
        </ui.Stack>
      </Surface>
    )
  }

  return (
    <Surface id="slot.content">
      {viewState.type === 'menu' && (
        <ui.Menu>
          {/* Add ui.MenuItem entries here */}
        </ui.Menu>
      )}
    </Surface>
  )
}
```

## Surface Composition

```tsx
// src/surfaces/Footer.tsx
import { ui, Surface } from '@stackable-labs/sdk-extension-react'

export function Footer() {
  return (
    <>
      <Surface id="slot.footer">
        <ui.Text className="text-xs">Powered by My Extension</ui.Text>
      </Surface>
      <Surface id="slot.footer-links">
        <ui.FooterLink href="https://example.com">My Extension</ui.FooterLink>
      </Surface>
    </>
  )
}
```

## Simple Surface

```tsx
// src/surfaces/Header.tsx
import { ui, Surface } from '@stackable-labs/sdk-extension-react'

export function Header() {
  return (
    <Surface id="slot.header">
      <ui.Text>Header content goes here</ui.Text>
    </Surface>
  )
}
```

## API Wrapper (data.query + data.fetch)

```tsx
// src/lib/api.ts
/**
 * API wrapper patterns — choose one based on your integration model.
 *
 * data.query  → host-mediated: the host handles the API call, extension sends
 *               an action name + params, host returns data.
 *               Permission: "data:query"
 *
 * data.fetch  → direct HTTP: the extension calls external APIs through the
 *               platform proxy. Domains must be in allowedDomains in manifest.
 *               Permission: "data:fetch"
 *
 * Usage in a surface:
 *   const capabilities = useCapabilities()
 *   const api = createApi(capabilities.data.query)
 *   // or: const api = createFetchApi(capabilities.data.fetch)
 */

import type { ApiRequest, FetchRequestInit, FetchResponse } from '@stackable-labs/sdk-extension-contracts'

type QueryFn = <T = unknown>(payload: ApiRequest) => Promise<T>
type FetchFn = (url: string, init?: FetchRequestInit) => Promise<FetchResponse>

// ── data.query wrapper ──────────────────────────────────────────────────────

export function createApi(query: QueryFn) {
  return {
    async getItems(): Promise<unknown[]> {
      return query<unknown[]>({ action: 'getItems' })
    },

    async getItem(itemId: string): Promise<unknown> {
      return query<unknown>({ action: 'getItem', itemId })
    },
  }
}

// ── data.fetch wrapper ──────────────────────────────────────────────────────
//
// For API keys and secrets, use {{settings.xxx}} placeholders in headers.
// The proxy resolves them server-side — the real secret never enters extension code.
// Declare secret fields in manifest.json settingsSchema with "secret": true.

const API_BASE_URL = import.meta.env.VITE_API_BASE_URL as string

export function createFetchApi(fetch: FetchFn) {
  return {
    async getItems(): Promise<unknown[]> {
      const result = await fetch(`${API_BASE_URL}/items`, {
        method: 'GET',
        headers: {
          'X-API-Key': '{{settings.apiKey}}',
        },
      })
      if (!result.ok) throw new Error(`getItems failed: ${result.status}`)
      return result.data as unknown[]
    },

    async getItem(itemId: string): Promise<unknown> {
      const result = await fetch(`${API_BASE_URL}/items/${itemId}`, {
        method: 'GET',
        headers: {
          'X-API-Key': '{{settings.apiKey}}',
        },
      })
      if (!result.ok) throw new Error(`getItem failed: ${result.status}`)
      return result.data as unknown
    },
  }
}
```
