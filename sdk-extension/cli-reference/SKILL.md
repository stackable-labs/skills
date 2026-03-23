---
name: cli-reference
description: "CLI commands for creating, developing, validating, and deploying Stackable extensions."
---

# CLI Reference

The Stackable CLI provides commands to create, develop, validate, and deploy extensions.

## create

Scaffold a new extension project:

```bash
pnpm --config.dlx-cache-max-age=0 dlx @stackable-labs/create-extension <project-name>
```

**Arguments:**

| Argument | Description |
|----------|-------------|
| `project-name` | Directory name for the new project |

**Options:**

| Flag | Description |
|------|-------------|
| `--template <flavor>` | Template flavor (see below) |
| `--app-id <id>` | Skip App selection |
| `--extension-port <port>` | Extension dev server port (default: 6543) |
| `--preview-port <port>` | Preview dev server port |
| `--skip-install` | Skip package manager install |
| `--skip-git` | Skip git initialization |

**Template flavors:**

| Flavor | Description |
|--------|-------------|
| `minimal` | Bare minimum — single surface, hello-world component |
| `starter` | Common patterns — store, api helpers, menu (default) |
| `kitchen-sink` | Everything — every component, capability, surface, and hook |

**Example:**
```bash
pnpm --config.dlx-cache-max-age=0 dlx @stackable-labs/create-extension order-lookup
cd order-lookup
pnpm install
```

## dev

Start dev servers with a public tunnel for live preview:

```bash
pnpm --config.dlx-cache-max-age=0 dlx @stackable-labs/cli-app-extension@latest dev
```

**Options:**

| Flag | Description |
|------|-------------|
| `--dir <path>` | Project root (default: cwd) |
| `--extension-port <port>` | Override extension port |
| `--preview-port <port>` | Override preview port |
| `--no-tunnel` | Skip tunnel, just run vite dev |

**What it does:**
1. Reads `.env.stackable` for cached App/Extension context (prompts if missing)
2. Creates Cloudflare tunnels for the extension dev server
3. Starts Vite dev servers with hot reload
4. Displays a `_stackable_dev` query param to append to your host app URL

### Host App Override

The CLI outputs a query param like:

```
?_stackable_dev=ext-123%3Ahttps%3A%2F%2Fabc.trycloudflare.com
```

Append this to your deployed host app URL to load your local extension instead
of the production bundle. The override is browser-session only — no DB changes,
no shared state. Each developer gets isolated overrides.

## validate

Check your extension for common errors before deploying:

```bash
pnpm --config.dlx-cache-max-age=0 dlx @stackable-labs/cli-app-extension@latest validate
```

**What it checks:**

| Check | Description |
|-------|-------------|
| Manifest structure | Valid JSON, required fields present |
| Permission usage | Capabilities used in code have matching permissions in manifest |
| Surface targets | Each `<Surface id="...">` has a matching target in manifest |
| Import validation | Uses `ui.*` namespace, no direct component imports |
| Domain configuration | `data.fetch` calls match `allowedDomains` entries |
| Sandbox compliance | No `document` or `window.location` access |

**Exit codes:**
- `0` — all checks pass
- `1` — errors found (must fix before deploying)

## deploy

Package and deploy the extension:

```bash
pnpm --config.dlx-cache-max-age=0 dlx @stackable-labs/cli-app-extension@latest deploy
```

**What it does:**
1. Runs validation checks (same as validate)
2. Builds the extension for production
3. Packages the build output
4. Uploads to the Stackable extension registry

**Prerequisites:**
- All validation checks must pass
- You must be authenticated (follow the prompts if not)

## scaffold

Scaffold a local project from an existing extension:

```bash
pnpm --config.dlx-cache-max-age=0 dlx @stackable-labs/cli-app-extension@latest scaffold [extensionId]
```

**Options:**

| Flag | Description |
|------|-------------|
| `--app-id <id>` | App ID (auto-resolved from extensionId if omitted) |
| `--project-id <id>` | Studio project ID (fetches files/manifest from Studio) |
| `--skip-install` | Skip package manager install |
| `--skip-git` | Skip git initialization |

## update

Update an existing extension:

```bash
pnpm --config.dlx-cache-max-age=0 dlx @stackable-labs/cli-app-extension@latest update [extensionId]
```

**Options:**

| Flag | Description |
|------|-------------|
| `--app-id <id>` | App ID (auto-resolved from extensionId if omitted) |
| `--name <name>` | New extension name |
| `--targets <targets>` | Comma-separated target slots |
| `--bundle-url <url>` | New bundle URL |
| `--enabled <bool>` | Enable/disable extension |
| `--dir <path>` | Project root (default: cwd) |
