---
name: external-apis
description: "Direct HTTP requests via data.fetch, allowedDomains configuration, and API wrapper patterns. Use when connecting an extension to external APIs."
---

# External APIs

Extensions can make direct HTTP requests to external services using the `data.fetch`
capability. All requests are proxied through the host for security — the extension
never makes raw network calls from the sandbox.

## Setup

### 1. Add Permission

Add `data:fetch` to your manifest permissions:

```json
{
  "permissions": ["data:fetch"]
}
```

### 2. Configure Allowed Domains

Every domain your extension calls must be listed in `allowedDomains`:

```json
{
  "allowedDomains": ["api.example.com", "graphql.example.com"]
}
```

**Rules:**
- Exact hostnames only — no wildcards, no paths, no protocols
- The host rejects requests to unlisted domains
- Add each subdomain separately (e.g., `api.example.com` and `cdn.example.com`)

### 3. Make Requests

Access `data.fetch` through the `useCapabilities()` hook:

```tsx
const capabilities = useCapabilities()

const result = await capabilities.data.fetch('https://api.example.com/data', {
  method: 'GET',
  headers: { 'Authorization': 'Bearer token' },
})

if (!result.ok) throw new Error(`Request failed: ${result.status}`)
const data = result.data as MyType
```

## API Signature

```tsx
capabilities.data.fetch(url: string, init?: FetchRequestInit): Promise<FetchResponse>
```

**FetchRequestInit:**
| Field | Type | Default |
|-------|------|---------|
| `method` | `'GET' \| 'POST' \| 'PUT' \| 'PATCH' \| 'DELETE'` | `'GET'` |
| `headers` | `Record<string, string>` | `{}` |
| `body` | `unknown` | — |

**FetchResponse:**
| Field | Type | Description |
|-------|------|-------------|
| `status` | `number` | HTTP status code |
| `ok` | `boolean` | `true` if status is 2xx |
| `data` | `unknown` | Parsed response body |

## API Wrapper Pattern

Create a typed wrapper in `src/lib/api.ts` to keep components clean:

```tsx
import type { Capabilities } from '@stackable-labs/sdk-extension-react'

const BASE_URL = 'https://api.example.com'

export async function fetchCustomer(
  capabilities: Capabilities,
  customerId: string
): Promise<Customer> {
  const result = await capabilities.data.fetch(
    `${BASE_URL}/customers/${customerId}`
  )
  if (!result.ok) throw new Error(`Failed to fetch customer: ${result.status}`)
  return result.data as Customer
}

export async function updateCustomer(
  capabilities: Capabilities,
  customerId: string,
  data: Partial<Customer>
): Promise<Customer> {
  const result = await capabilities.data.fetch(
    `${BASE_URL}/customers/${customerId}`,
    {
      method: 'PATCH',
      headers: { 'Content-Type': 'application/json' },
      body: data,
    }
  )
  if (!result.ok) throw new Error(`Failed to update customer: ${result.status}`)
  return result.data as Customer
}
```

Then use it in components:

```tsx
const capabilities = useCapabilities()
const customer = await fetchCustomer(capabilities, customerId)
```

## data.fetch vs data.query

| | data.fetch | data.query |
|--|-----------|-----------|
| **Who handles the request** | Extension (via proxy) | Host application |
| **Permission** | `data:fetch` | `data:query` |
| **Domain config** | Required (`allowedDomains`) | Not needed |
| **Use when** | Calling external APIs directly | Host provides the API integration |

## Error Handling

Always check `result.ok` before accessing `result.data`:

```tsx
const result = await capabilities.data.fetch(url)
if (!result.ok) {
  capabilities.actions.toast({
    message: 'Request failed. Please try again.',
    type: 'error',
  })
  return
}
```
