---
name: add-surface
description: "Add a new surface (UI slot) to this Stackable extension. Use when the user wants to add a header, content, footer, or footer-links surface."
---

# Add a New Surface

Add a new surface to this extension. Follow these steps exactly:

## 1. Determine the target slot
Ask which target slot this surface is for. Valid targets:
- `slot.header` — appears at the top of the extension area
- `slot.content` — main content area
- `slot.footer` — bottom of the extension area
- `slot.footer-links` — footer link row

## 2. Create the surface file
Create a new file in `packages/extension/src/surfaces/` named after the slot
(e.g., `Header.tsx`, `Content.tsx`, `Footer.tsx`).

Use this template:
```tsx
import { Surface, useCapabilities, useContextData } from '@stackable-labs/sdk-extension-react'
import { ui } from '@stackable-labs/sdk-extension-react'

export const [SurfaceName] = () => {
  return (
    <Surface id="[target]">
      {/* Surface content here */}
    </Surface>
  )
}
```

If the surface needs state management, import and use the existing store from `../store`.

## 3. Register the surface in index.tsx
Import the new surface component and add it to the `createExtension` factory function
alongside existing surfaces.

## 4. Update manifest.json
Add the target to the `targets` array in `packages/extension/public/manifest.json`.
Also add any required permissions based on the target-permission mapping:
- `slot.header` → `context:read`
- `slot.content` → `context:read`, `data:query`, `actions:toast`, `actions:invoke`
- `slot.footer` → (none)
- `slot.footer-links` → (none)

Only add permissions that aren't already declared.

## 5. Verify
- Confirm the surface renders by checking the import chain: index.tsx → Surface component → Surface id matches manifest target
- Confirm manifest.json is valid JSON with no duplicate targets or permissions
