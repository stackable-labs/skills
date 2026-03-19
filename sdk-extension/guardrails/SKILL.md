---
name: guardrails
description: "SDK constraints and rules that must always be followed: sandbox restrictions, component usage, manifest requirements, entry point rules. Use as a checklist when writing or reviewing extension code."
---

# Guardrails

These rules must always be followed when writing extension code.

## Sandbox
- Never access `document` or `window.location` directly — the extension runs in a sandboxed iframe
- Never modify files in `packages/preview/` — the preview host is pre-configured

## Components
- Use the `ui.*` namespace for components (`<ui.Card>`, `<ui.Button>`) — don't import components directly
- Only use the attributes listed in the component reference — the host rejects unknown attributes
- Use `<ui.ScrollArea>` for content that may overflow

## Manifest
- Always declare permissions in `manifest.json` before using capabilities
- Don't use `data.fetch` without adding the domain to `allowedDomains` in manifest
- `allowedDomains` must be exact hostnames — no wildcards, no paths

## Entry Point
- Don't modify `index.html` — the extension entry point is always `src/index.tsx` via `createExtension`

## State Management
- Use discriminated unions for view state types, not string constants
- Always handle loading states when using capabilities or context data

## Performance
- Keep bundle size under 500KB
- Use the API wrapper pattern — don't call `data.fetch` inline in components
- Use narrow selectors with `useStore(store, selector)` to minimize re-renders
