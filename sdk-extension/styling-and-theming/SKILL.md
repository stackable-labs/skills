---
name: styling-and-theming
description: "Host theme inheritance, className usage, layout components, and CSS constraints. Use when styling extension UI or working with the host theme."
---

# Styling & Theming

Extensions inherit the host application's theme automatically. The SDK component library
(`ui.*` namespace) renders inside the host's styling context, so colors, fonts, and
spacing match the host UI without any configuration.

## Host Theme Inheritance

The `ui.*` components automatically use the host's design tokens:
- **Colors** — text, backgrounds, borders adapt to the host's color scheme
- **Typography** — font family, sizes, and weights match the host
- **Spacing** — padding and margins follow the host's spacing scale
- **Dark mode** — components respond to the host's light/dark mode setting

No CSS or theme configuration is needed in the extension.

## The className Prop

Most `ui.*` components accept a `className` prop for layout adjustments:

```tsx
<ui.Card className="mt-4">
  <ui.CardContent className="p-6">
    <ui.Text className="text-center">Centered text</ui.Text>
  </ui.CardContent>
</ui.Card>
```

**Use className for:**
- Layout (margin, padding, flexbox, grid)
- Positioning (relative, absolute)
- Sizing (width, height, max-width)
- Spacing between elements

**Avoid overriding with className:**
- Colors and backgrounds (breaks theme consistency)
- Font families and sizes (breaks typography scale)
- Border styles (use component variants instead)

## Layout Components

Use the built-in layout components instead of raw CSS:

| Component | Purpose |
|-----------|---------|
| `<ui.Stack>` | Vertical layout with consistent spacing |
| `<ui.Inline>` | Horizontal layout with consistent spacing |
| `<ui.Separator>` | Visual divider between sections |
| `<ui.ScrollArea>` | Scrollable container for overflow content |

```tsx
<ui.Stack className="gap-4">
  <ui.Heading level={3}>Section Title</ui.Heading>
  <ui.Text>Section content</ui.Text>
  <ui.Separator />
  <ui.Inline className="gap-2">
    <ui.Button variant="primary">Save</ui.Button>
    <ui.Button variant="ghost">Cancel</ui.Button>
  </ui.Inline>
</ui.Stack>
```

## Component Variants

Use component props (not CSS) to control visual style:

```tsx
// Buttons — variant controls appearance
<ui.Button variant="primary">Primary action</ui.Button>
<ui.Button variant="ghost">Secondary action</ui.Button>

// Badges — variant + hue + tone for semantic colors
<ui.Badge variant="default" hue="blue" tone="strong">Active</ui.Badge>
<ui.Badge variant="default" hue="red" tone="subtle">Error</ui.Badge>

// Text — tone for semantic meaning
<ui.Text tone="subdued">Helper text</ui.Text>
<ui.Text tone="critical">Error message</ui.Text>
```

## CSS Constraints

Extensions run in a sandboxed iframe. These constraints apply:

- **No global CSS** — styles don't leak between extensions or into the host
- **No `document` access** — cannot inject stylesheets or modify the DOM directly
- **No `window.location`** — cannot read or modify the URL
- **Component-only styling** — use `className` on `ui.*` components, not raw HTML elements
- **Bundle size limit** — keep CSS dependencies minimal (under 500KB total bundle)

## Responsive Design

Extensions render in a constrained viewport (the host's sidebar or panel). Design for
narrow widths:

```tsx
// Use ScrollArea for content that may overflow
<ui.ScrollArea className="h-[400px]">
  <ui.Stack className="gap-3">
    {items.map((item) => (
      <ItemCard key={item.id} item={item} />
    ))}
  </ui.Stack>
</ui.ScrollArea>
```

- Assume a **narrow viewport** (~300-400px wide)
- Use `<ui.ScrollArea>` for long lists or content
- Avoid horizontal scrolling — stack elements vertically
- Test at the minimum panel width the host supports
