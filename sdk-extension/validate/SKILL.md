---
name: validate
description: "Validate this extension for common errors before deploying: manifest, permissions, surfaces, imports. Use before publishing or when debugging issues."
---

# Validate Extension

Check this extension for common errors before deploying. Run through each check
and report all issues found.

## 1. Manifest validation
Read `packages/extension/public/manifest.json` and verify:
- [ ] Valid JSON
- [ ] `name` is a non-empty string
- [ ] `version` follows semver (e.g., "1.0.0")
- [ ] `targets` is a non-empty array of valid target strings (slot.header, slot.content, slot.footer, slot.footer-links)
- [ ] `permissions` contains only valid permission strings (context:read, data:query, data:fetch, actions:toast, actions:invoke, events:identity, events:messaging, events:activity, extend:identity)
- [ ] `allowedDomains` is an array (can be empty if data:fetch is not used)
- [ ] If `data:fetch` permission is declared, `allowedDomains` is not empty

## 2. Permission-to-usage matching
Scan all `.tsx` files in `packages/extension/src/` for capability usage:
- `capabilities.data.query` or `data.query` → needs `data:query` permission
- `capabilities.data.fetch` or `data.fetch` → needs `data:fetch` permission
- `capabilities.context.read` or `useContextData` → needs `context:read` permission
- `capabilities.actions.toast` → needs `actions:toast` permission
- `capabilities.actions.invoke` → needs `actions:invoke` permission
- `useExtendIdentity` → needs `extend:identity` permission
- `useIdentityEvent` → needs `events:identity` permission (also check manifest `events` array has matching entries)
- `useMessagingEvent` → needs `events:messaging` permission (also check manifest `events` array has matching entries)
- `useActivityEvent` → needs `events:activity` permission (also check manifest `events` array has matching entries)

Report:
- **Missing permissions:** capabilities used in code but not declared in manifest
- **Unused permissions:** permissions declared in manifest but not used in code

## 3. Surface-to-target matching
- Each `.tsx` file with a `<Surface id="...">` should have a matching target in manifest.json
- Each target in manifest.json should have a corresponding Surface component

## 4. Import validation
Verify that:
- [ ] UI components are imported via `ui.*` namespace from `@stackable-labs/sdk-extension-react`
- [ ] No direct imports of `document`, `window.location`, or other browser globals
- [ ] No modifications to `packages/preview/` directory
- [ ] Entry point is `src/index.tsx` using `createExtension`

## 5. Summary
Print a summary: total issues found, categorized by severity (error vs warning).
Errors must be fixed before deploying. Warnings are recommendations.
