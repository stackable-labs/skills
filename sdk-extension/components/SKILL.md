---
name: components
description: "UI component catalog with allowed attributes per tag and available icon names. Use when building extension UI, looking up component attributes, or asking about available components."
---

# UI Components

Access all components via the `ui.*` namespace:
```tsx
import { ui } from '@stackable-labs/sdk-extension-react'
```

**39 available components** — only use the attributes listed below for each component.

## Layout

### `<ui.Card>` (`ui-card`)
Allowed attributes: `className`, `onClick`

### `<ui.CardContent>` (`ui-card-content`)
Allowed attributes: `className`

### `<ui.CardHeader>` (`ui-card-header`)
Allowed attributes: `className`

### `<ui.Inline>` (`ui-inline`)
Allowed attributes: `gap`, `className`

### `<ui.ScrollArea>` (`ui-scroll-area`)
Allowed attributes: `className`

### `<ui.Separator>` (`ui-separator`)
Allowed attributes: `className`

### `<ui.Stack>` (`ui-stack`)
Allowed attributes: `gap`, `direction`, `className`

## Text

### `<ui.Badge>` (`ui-badge`)
Allowed attributes: `variant`, `hue`, `tone`, `className`

### `<ui.Heading>` (`ui-heading`)
Allowed attributes: `level`, `className`

### `<ui.Text>` (`ui-text`)
Allowed attributes: `className`, `tone`

## Input

### `<ui.Button>` (`ui-button`)
Allowed attributes: `variant`, `size`, `disabled`, `onClick`, `type`, `className`, `title`

### `<ui.Checkbox>` (`ui-checkbox`)
Allowed attributes: `checked`, `onChange`, `disabled`, `className`, `id`

### `<ui.Input>` (`ui-input`)
Allowed attributes: `type`, `placeholder`, `value`, `onChange`, `disabled`, `className`, `id`

### `<ui.Label>` (`ui-label`)
Allowed attributes: `htmlFor`, `className`

### `<ui.RadioGroup>` (`ui-radio-group`)
Allowed attributes: `value`, `defaultValue`, `onChange`, `className`

### `<ui.RadioGroupItem>` (`ui-radio-group-item`)
Allowed attributes: `value`, `disabled`, `className`, `id`

### `<ui.Select>` (`ui-select`)
Allowed attributes: `value`, `defaultValue`, `onChange`, `placeholder`, `disabled`, `className`

### `<ui.SelectOption>` (`ui-select-option`)
Allowed attributes: `value`, `disabled`, `className`

### `<ui.Switch>` (`ui-switch`)
Allowed attributes: `checked`, `onChange`, `disabled`, `size`, `className`, `id`

### `<ui.Textarea>` (`ui-textarea`)
Allowed attributes: `placeholder`, `value`, `onChange`, `disabled`, `rows`, `className`, `id`

## Navigation

### `<ui.Link>` (`ui-link`)
Allowed attributes: `href`, `target`, `rel`, `className`

### `<ui.Menu>` (`ui-menu`)
Allowed attributes: `title`, `className`

### `<ui.MenuItem>` (`ui-menu-item`)
Allowed attributes: `icon`, `label`, `description`, `onClick`, `className`

### `<ui.Tabs>` (`ui-tabs`)
Allowed attributes: `defaultValue`, `className`

### `<ui.TabsContent>` (`ui-tabs-content`)
Allowed attributes: `value`, `className`

### `<ui.TabsList>` (`ui-tabs-list`)
Allowed attributes: `className`

### `<ui.TabsTrigger>` (`ui-tabs-trigger`)
Allowed attributes: `value`, `className`

## Feedback

### `<ui.Alert>` (`ui-alert`)
Allowed attributes: `variant`, `title`, `className`

### `<ui.Progress>` (`ui-progress`)
Allowed attributes: `value`, `className`

### `<ui.Skeleton>` (`ui-skeleton`)
Allowed attributes: `width`, `height`, `className`

### `<ui.Tooltip>` (`ui-tooltip`)
Allowed attributes: `content`, `className`

## Composite

### `<ui.Avatar>` (`ui-avatar`)
Allowed attributes: `src`, `alt`, `className`

### `<ui.Collapsible>` (`ui-collapsible`)
Allowed attributes: `defaultOpen`, `className`

### `<ui.CollapsibleContent>` (`ui-collapsible-content`)
Allowed attributes: `className`

### `<ui.CollapsibleTrigger>` (`ui-collapsible-trigger`)
Allowed attributes: `className`

### `<ui.Icon>` (`ui-icon`)
Allowed attributes: `name`, `size`, `className`

### `<ui.Image>` (`ui-image`)
Allowed attributes: `src`, `alt`, `loading`, `width`, `height`, `className`

### `<ui.QRCode>` (`ui-qr-code`)
Allowed attributes: `value`, `size`, `variant`, `level`, `alt`, `className`

### `<ui.Video>` (`ui-video`)
Allowed attributes: `videoId`, `autoPlay`, `showControls`, `allowTracking`, `shape`, `title`, `className`

## Available Icons (31)

Use with `<ui.Icon name="icon-name" />`. Valid icon names:
`arrow-left`, `calendar`, `check-circle-2`, `chevron-left`, `chevron-right`, `clock`, `credit-card`, `external-link`, `help-circle`, `info`, `loader-2`, `mail`, `map-pin`, `message-circle`, `message-square`, `package`, `phone`, `search`, `shopping-bag`, `sparkles`, `truck`, `user`, `video`, `x-circle`, `alert-circle`, `book-open`, `qr-code`, `image`, `image-off`, `circle-play`, `tv-minimal-play`
