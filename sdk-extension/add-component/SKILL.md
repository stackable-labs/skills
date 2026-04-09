---
name: add-component
description: "Add a UI component (ui.Card, ui.Button, ui.Tabs, etc.) to this extension. Use when the user wants to add a visual element."
---

# Add a UI Component

Add a UI component to this extension. Follow these steps:

## 1. Identify the component
All UI components use the `ui.*` namespace from `@stackable-labs/sdk-extension-react`.

**Available components by category:**
- **Layout:** Card, CardContent, CardHeader, Stack, Inline, Separator, ScrollArea
- **Text:** Text, Heading, Badge
- **Input:** Button, Input, Textarea, Select, SelectOption, Checkbox, Switch, Label, RadioGroup, RadioGroupItem
- **Feedback:** Skeleton, Tooltip, Progress, Alert
- **Navigation:** Tabs, TabsList, TabsTrigger, TabsContent, Link, Menu, MenuItem
- **Composite:** Collapsible, CollapsibleTrigger, CollapsibleContent, Avatar, Icon

## 2. Place the component
- Insert inside an existing `<Surface>` block — never outside a Surface
- Place in a logical position within the existing UI structure
- For compound components, include their required children:
  - **Card** → CardContent (and optionally CardHeader)
  - **Tabs** → TabsList + TabsTrigger(s) + TabsContent(s)
  - **Select** → SelectOption(s)
  - **Menu** → MenuItem(s)
  - **Collapsible** → CollapsibleTrigger + CollapsibleContent
  - **RadioGroup** → RadioGroupItem(s) + Label(s)

## 3. Import
Ensure `ui` is imported from `@stackable-labs/sdk-extension-react`. It usually already is.
No additional imports needed — all components are accessed via `ui.ComponentName`.

## 4. Constraints
- Do not remove or change any existing code — only add the new component
- Do not add components outside of a Surface
- Do not use raw HTML elements — only `ui.*` components are allowed in the sandbox
- Child-only tags (CardContent, CardHeader, SelectOption, TabsList, TabsTrigger, TabsContent,
  RadioGroupItem, CollapsibleTrigger, CollapsibleContent, MenuItem) must be nested inside
  their parent — never standalone
