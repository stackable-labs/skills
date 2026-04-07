---
name: permissions
description: "Permission strings, capability-to-permission mapping, and target conventions. Use when configuring manifest.json or debugging permission errors."
---

# Permissions

Declare permissions in `manifest.json` before using the corresponding capability.

## Available Permissions (9)

- `context:read`
- `data:query`
- `data:fetch`
- `actions:toast`
- `actions:invoke`
- `events:identity`
- `events:messaging`
- `events:activity`
- `extend:identity`

## Capability → Permission Mapping

| Capability | Required Permission |
|-----------|-------------------|
| `context.read` | `context:read` |
| `data.query` | `data:query` |
| `data.fetch` | `data:fetch` |
| `actions.toast` | `actions:toast` |
| `actions.invoke` | `actions:invoke` |
| `extend.identity` | `extend:identity` |

## Target → Permission Conventions

Common permission sets for each surface target:

| Target | Typical Permissions |
|--------|-------------------|
| `slot.header` | `context:read` |
| `slot.content` | `context:read`, `data:query`, `actions:toast`, `actions:invoke` |
| `slot.footer` | (none) |
| `slot.footer-links` | (none) |

Only declare permissions your extension actually uses.
