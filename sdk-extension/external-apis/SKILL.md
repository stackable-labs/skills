---
name: external-apis
description: "Direct HTTP requests via data.fetch — allowed domains (including wildcard subdomains like *.myshopify.com), allowedDomains configuration, API wrapper patterns, and data.fetch vs data.query. Use when connecting an extension to external APIs or configuring allowedDomains."
---

# External APIs

Extensions can make direct HTTP requests to external services using the `data.fetch`
capability. All requests are proxied through the framework for security — the extension
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
  "allowedDomains": ["api.example.com", "graphql.example.com", "*.myshopify.com"]
}
```

**Rules:**
- Exact hostnames or `*.<suffix>` wildcards (subdomain match) — no paths, no protocols
- Wildcards take the form `*.<suffix>` and must use a multi-label suffix (e.g., `*.example.com`, not `*.com`)
- The framework will reject requests to unlisted domains
- Add each subdomain separately when listing exact hosts (e.g., `api.example.com` and `cdn.example.com`)

#### Wildcard Domains

**Prefer exact hostnames where possible.** Use `*.example.com` only when you
need to match any subdomain — for example, tenant-per-subdomain platforms
(Shopify shops, Salesforce subdomains, Zendesk subdomains, multi-region API hosts) where
you don't know the full set of hostnames at authoring time or will differ by instance.

```json
{
  "allowedDomains": ["*.myshopify.com"]
}
```

With this entry, the framework allows requests to `acme.myshopify.com`,
`widgets.myshopify.com`, and any subdomain at any depth (e.g.,
`shop.eu.myshopify.com`). A request to `evil.com` is still rejected.

**Apex is separate.** `*.myshopify.com` does **not** match `myshopify.com`
itself — this matches CORS, TLS-certificate, and DNS-wildcard convention. To
allow both the apex and any subdomain, list both:

```json
{
  "allowedDomains": ["myshopify.com", "*.myshopify.com"]
}
```

**Worked example:**

```json
{
  "allowedDomains": [
    "*.myshopify.com",
    "myshopify.com",
    "api.example.com",
    "*.staging.example.com"
  ]
}
```

This allows any customer shop subdomain, the Shopify apex, an exact API host,
and any subdomain (at any depth) under `staging.example.com`.

**What's rejected and where.** Wildcards must use a multi-label suffix
(e.g., `*.example.com`, not `*.com`). Two layers enforce this:

- *Format-level* (instant feedback in the CLI, MCP, and Studio AI panel) —
  `*` alone, multiple wildcards, mid-string wildcards (`api-*.example.com`),
  single-label suffixes (`*.com`, `*.localhost`, `*.io`), protocols
  (`https://...`), paths, ports, and IP literals.
- *Server-level* (at submission) — TLD patterns such as `*.co.uk`,
  `*.com.au`, and `*.github.io` may pass the format check but will be
  rejected at submission, since they would otherwise allow any registrant
  under a shared registry.

**Local development.** Exact `localhost` and IPv4 literals (`127.0.0.1`) are
still valid for exact-host entries. Dev/staging extension modes bypass the
domain check entirely, so wildcards aren't needed for local iteration — they
matter at submission and in production.

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
| **Who handles the request** | Extension (via proxy) | Platform |
| **Permission** | `data:fetch` | `data:query` |
| **Domain config** | Required (`allowedDomains`) | Not needed |
| **Use when** | Calling external APIs directly | Platform provides the API integration |

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
