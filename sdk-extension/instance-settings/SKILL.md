---
name: instance-settings
description: "Instance Settings: declaring settingsSchema in your extension manifest, the install-time admin form, and reading regular values via useSettings() vs secure values via {{settings.x}} placeholders in data.fetch. Use when adding per-Instance configuration to an extension."
---

# Instance Settings

Instance Settings are the per-Instance configuration values your extension needs in
order to do its job — API keys, account identifiers, environment toggles, feature
flags. You declare them once in your extension's `manifest.json`; whoever installs
your extension fills them in from the admin dashboard at install time; your code
reads the values at runtime.

There are two flavors:

- **Regular** values live in plaintext and are readable from your extension code.
  Use them for non-sensitive configuration the UI needs to branch on.
- **Secure** values are encrypted at rest and **never** leave the server. Use them
  for credentials. Your code references them as a placeholder; the proxy
  substitutes the real value server-side when it makes the outbound request.

## 1. Declare your settings in `manifest.json`

Add a `settingsSchema` array to `packages/extension/public/manifest.json`. Each
entry describes one input on the install-time admin form:

```json
{
  "name": "My Extension",
  "version": "1.0.0",
  "targets": ["slot.content"],
  "permissions": ["context:read", "data:fetch"],
  "allowedDomains": ["api.example.com"],
  "settingsSchema": [
    {
      "identifier": "apiKey",
      "label": "API Key",
      "type": "text",
      "secret": true,
      "required": true,
      "description": "Your account API key. Stored encrypted; injected server-side into outbound request headers."
    },
    {
      "identifier": "environmentType",
      "label": "Environment",
      "type": "select",
      "options": [
        { "label": "Production", "value": "prod" },
        { "label": "Sandbox", "value": "sandbox" }
      ]
    }
  ]
}
```

That's the entire authoring step. The next time someone installs (or re-syncs)
your extension, the admin dashboard renders a form from this schema on the
Instance settings page. Regular fields show inline values; secret fields show a
masked input that accepts a value once and displays `••••` after save (the
cleartext value is never returned by the API).

### Field types

| Type | Renders as | Accepts `secret: true` |
|------|------------|--------------------------|
| `text` | Single-line text input | Yes |
| `textarea` | Multi-line text input | Yes |
| `email` | Email input with format validation | Yes |
| `number` | Numeric input (`min` / `max` / `step`) | No |
| `select` | Dropdown (use `allowMultiple` for multi-select) | No |
| `radio` | Radio button group | No |
| `toggle` | On/off switch | No |
| `tags` | Tag list / multi-string entry | No |

The `secret: true` flag is only valid on `text`, `textarea`, `email` types.

All fields share these optional properties: `required`, `placeholder`,
`description` (help text shown below the input), and `dependsOn` (conditional
visibility — show this field only when another field has a matching value).

## 2. The install-time admin form

When someone installs your extension into one of their Instances, the admin
dashboard reads your `settingsSchema` and generates a form. Each field becomes
an input; `required` fields block save; `description` text becomes inline help;
`dependsOn` rules show or hide fields based on other field values.

Each Instance gets its own copy of these values, so the same extension can run
side-by-side on multiple Instances with completely different credentials and
configuration. The installer can edit values later from the same Instance
settings page — your code always sees the current values.

## 3. Read regular values: `useSettings()` (or `useContextData()`)

Regular (non-secret) values are exposed to your extension code via the
`useSettings()` hook, keyed by each field's `identifier`. This requires the
`context:read` permission.

```tsx
import { Surface, ui, useSettings } from '@stackable-labs/sdk-extension-react'

export const Content = () => {
  const settings = useSettings()
  const environmentType = (settings.environmentType as string) || 'prod'

  return (
    <Surface id="slot.content">
      <ui.Card>
        <ui.CardContent>
          <ui.Text>Looking up orders…</ui.Text>
          {environmentType === 'sandbox' && (
            <ui.Text className="text-xs opacity-50">Sandbox Mode</ui.Text>
          )}
        </ui.CardContent>
      </ui.Card>
    </Surface>
  )
}
```

If you need settings alongside other context data (customer, locale, etc.), they
also come back from `useContextData()`:

```tsx
import { useContextData } from '@stackable-labs/sdk-extension-react'

const { loading, settings, customerId } = useContextData()
```

> **Note:** Secret fields are **never** present in `useSettings()` or on
> `ctx.settings`. Reading `settings.apiKey` for a `secret: true` field returns
> `undefined`, even when a value is configured. If you find yourself reaching
> for a secret value in extension code, you almost certainly want the
> `data.fetch` placeholder pattern below instead.

## 4. Use secure values: `{{settings.<identifier>}}` in `data.fetch`

Secure values are decrypted only by the proxy Lambda that handles `data.fetch` —
at substitution time, per-request, then discarded. Your code references them as
template placeholders in **header values**:

```tsx
import { useCapabilities, useSettings } from '@stackable-labs/sdk-extension-react'

const capabilities = useCapabilities()
const settings = useSettings()
const environmentType = (settings.environmentType as string) || 'prod'

const result = await capabilities.data.fetch('https://api.example.com/orders', {
  method: 'GET',
  headers: {
    // environmentType is a regular value — interpolate it normally
    'X-Environment': environmentType,
    // apiKey is secret — the proxy substitutes it server-side; the cleartext
    // value never enters this code path
    'X-Api-Key': '{{settings.apiKey}}',
  },
})
if (!result.ok) throw new Error(`Request failed: ${result.status}`)
```

Placeholders are only resolved in **header values** — not URLs, not header
names, not the request body. For `required: true` secret fields the proxy
returns `400` when the value is not configured; for optional secret fields
the entire header is omitted.

## Regular vs secure at a glance

| | Regular | Secure (`secret: true`) |
|--|---------|--------------------------|
| **Stored** | Plaintext | Encrypted with a per-Instance derived key |
| **Readable from extension code?** | Yes — `useSettings().<identifier>` | **Never** — secrets are not serialized to the browser or sandbox |
| **How to use the value** | Read at render time and use freely | Reference as a `{{settings.<identifier>}}` placeholder in `data.fetch` headers — the proxy substitutes server-side |
| **Returned by admin API after save?** | Yes (the value is shown back in the form) | No (the form shows `••••`; the cleartext value is unrecoverable) |

## When in doubt, mark it `secret: true`

The cost is negligible — one encryption per write, one decryption per outbound
request. The blast radius of an accidental log line, screenshot, or sandbox
exfiltration drops to zero. Secrets on one Instance also can't be used to
decrypt secrets on any other Instance, even when both Instances run the same
extension — encryption keys are derived per-Instance.

If a value is sensitive at all (API keys, OAuth tokens, signing secrets, webhook
shared secrets, personal access tokens), declare the field `secret: true` and
reach for it via the `data.fetch` placeholder pattern. Save `useSettings()` for
the values your UI legitimately needs to branch on.
