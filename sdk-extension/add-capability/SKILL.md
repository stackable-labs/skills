---
name: add-capability
description: "Wire up a new capability (data.fetch, data.query, context.read, actions.toast, actions.invoke) in this extension. Use when adding a new host-mediated API."
---

# Add a Capability

Wire up a new capability in this extension. Follow these steps exactly:

## 1. Determine the capability
Ask which capability to add. Valid capabilities:
- `data.query` — host-mediated data requests (action name + params → host returns data)
- `data.fetch` — direct HTTP requests from the sandbox (requires allowedDomains)
- `context.read` — read host-provided context (customerId, customerEmail, etc.)
- `actions.toast` — show toast notifications (success, error, info, warning)
- `actions.invoke` — invoke host actions (e.g., open new conversation)

## 2. Add permission to manifest.json
Add the corresponding permission to `packages/extension/public/manifest.json`:
- `data.query` → `"data:query"`
- `data.fetch` → `"data:fetch"`
- `context.read` → `"context:read"`
- `actions.toast` → `"actions:toast"`
- `actions.invoke` → `"actions:invoke"`

Only add if not already declared.

## 3. If data.fetch — add allowedDomains
Ask for the API domain(s) and add them to the `allowedDomains` array in manifest.json.

## 4. If data.fetch — create API wrapper
Create `packages/extension/src/lib/api.ts` with the API wrapper pattern:
```tsx
type FetchFn = (url: string, init?: FetchRequestInit) => Promise<FetchResponse>

export function createApi(fetch: FetchFn) {
  return {
    async getData(): Promise<DataType> {
      const result = await fetch('https://api.example.com/data', { method: 'GET' })
      if (!result.ok) throw new Error(`Request failed: ${result.status}`)
      return result.data as DataType
    },
  }
}
```

## 5. Use the capability in a surface
Access via the `useCapabilities()` hook:
```tsx
const capabilities = useCapabilities()
// data.query: capabilities.data.query({ action: 'getItems', ... })
// data.fetch: const api = createApi(capabilities.data.fetch)
// context.read: capabilities.context.read()  (or use useContextData() hook)
// actions.toast: capabilities.actions.toast({ message: 'Done!', type: 'success' })
// actions.invoke: capabilities.actions.invoke('actionName', { ... })
```

## 6. Verify
- Confirm the permission is in manifest.json
- If data.fetch, confirm the domain is in allowedDomains
- Confirm the capability is accessed via useCapabilities(), not imported directly
