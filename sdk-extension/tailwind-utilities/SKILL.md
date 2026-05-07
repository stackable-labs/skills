---
name: tailwind-utilities
description: "Tailwind utility classes pre-emitted in the embedded widget bundle (margin, padding, sizing, color, layout, typography, state variants). Use when picking className values for extension UI — anything outside this list will silently no-op at runtime."
---

# Supported Tailwind Utilities

Stackable extensions render inside the embedded widget's Shadow Root via the platform and ship **zero custom CSS by design** — they rely entirely on the platforms's bundled stylesheet for their visual styling. To keep that bundle small while still covering the common needs of extension UI, the platform pre-emits a curated subset of Tailwind v4 utility classes via a safelist. The classes listed below are guaranteed to be available in any extension that uses them.

Total: **790 utility classes** across **38 families**.

## Using utilities

Apply them via the `className` prop on `ui.*` components:

```tsx
<ui.Card className="mt-4 p-6">
  <ui.Stack className="gap-3">
    <ui.Text className="text-lg font-semibold uppercase tracking-wide">Title</ui.Text>
    <ui.Text className="text-zinc-600">Body copy</ui.Text>
  </ui.Stack>
</ui.Card>
```

Always prefer:
- Component variants (`<ui.Button variant="primary">`) over color overrides
- Layout components (`<ui.Stack>`, `<ui.Inline>`) over manual flexbox class chains
- The semantic color tokens (`text-foreground`, `bg-background`) which adapt to the host theme

## Safelist limits

The safelist is intentionally a fixed list, **NOT the full Tailwind catalog** to provide UI consistency and reduce bundle size. Anything beyond what is listed below will be missing from the platform stylesheet at runtime — the class will appear in your DOM but the corresponding CSS rule will not exist, and the style will silently no-op. In particular:

- **Arbitrary values are NOT supported** in general (e.g. `w-[123px]`, `bg-[#1a1a1a]`, `text-[15px]`). The single exception is `aspect-[9/16]` (portrait video). For any other arbitrary value, use the named utility closest to your target.
- **Off-list color shades** (e.g. `bg-cyan-500`, `text-rose-500`) are not pre-emitted. The supported color set below is curated for visual consistency with the host theme.
- **Off-list ladder values** (e.g. `mt-32`, `text-4xl`, `w-128`) are not pre-emitted. The ladders below cover sizes appropriate for the embedded widget's narrow viewport.

If you need something that's not on this list, file an issue with your use case — the safelist is curated, not frozen.

## Utility families

### Margin (88)

`m-0`, `m-0.5`, `m-1`, `m-1.5`, `m-2`, `m-2.5`, `m-3`, `m-3.5`, `m-4`, `m-5`, `m-6`, `m-7`, `m-8`, `m-9`, `m-10`, `m-11`, `m-12`, `m-14`, `m-16`, `mb-0`, `mb-1`, `mb-2`, `mb-3`, `mb-4`, `mb-5`, `mb-6`, `mb-8`, `mb-10`, `mb-12`, `mb-16`, `ml-0`, `ml-1`, `ml-2`, `ml-3`, `ml-4`, `ml-5`, `ml-6`, `ml-8`, `ml-10`, `ml-12`, `ml-16`, `ml-auto`, `mr-0`, `mr-1`, `mr-2`, `mr-3`, `mr-4`, `mr-5`, `mr-6`, `mr-8`, `mr-10`, `mr-12`, `mr-16`, `mr-auto`, `mt-0`, `mt-1`, `mt-2`, `mt-3`, `mt-4`, `mt-5`, `mt-6`, `mt-8`, `mt-10`, `mt-12`, `mt-16`, `mx-0`, `mx-1`, `mx-2`, `mx-3`, `mx-4`, `mx-5`, `mx-6`, `mx-8`, `mx-10`, `mx-12`, `mx-16`, `mx-auto`, `my-0`, `my-1`, `my-2`, `my-3`, `my-4`, `my-5`, `my-6`, `my-8`, `my-10`, `my-12`, `my-16`

### Padding (85)

`p-0`, `p-0.5`, `p-1`, `p-1.5`, `p-2`, `p-2.5`, `p-3`, `p-3.5`, `p-4`, `p-5`, `p-6`, `p-7`, `p-8`, `p-9`, `p-10`, `p-11`, `p-12`, `p-14`, `p-16`, `pb-0`, `pb-1`, `pb-2`, `pb-3`, `pb-4`, `pb-5`, `pb-6`, `pb-8`, `pb-10`, `pb-12`, `pb-16`, `pl-0`, `pl-1`, `pl-2`, `pl-3`, `pl-4`, `pl-5`, `pl-6`, `pl-8`, `pl-10`, `pl-12`, `pl-16`, `pr-0`, `pr-1`, `pr-2`, `pr-3`, `pr-4`, `pr-5`, `pr-6`, `pr-8`, `pr-10`, `pr-12`, `pr-16`, `pt-0`, `pt-1`, `pt-2`, `pt-3`, `pt-4`, `pt-5`, `pt-6`, `pt-8`, `pt-10`, `pt-12`, `pt-16`, `px-0`, `px-1`, `px-2`, `px-3`, `px-4`, `px-5`, `px-6`, `px-8`, `px-10`, `px-12`, `px-16`, `py-0`, `py-1`, `py-2`, `py-3`, `py-4`, `py-5`, `py-6`, `py-8`, `py-10`, `py-12`, `py-16`

### Gap (26)

`gap-0`, `gap-1`, `gap-2`, `gap-3`, `gap-4`, `gap-5`, `gap-6`, `gap-8`, `gap-10`, `gap-12`, `gap-x-0`, `gap-x-1`, `gap-x-2`, `gap-x-3`, `gap-x-4`, `gap-x-5`, `gap-x-6`, `gap-x-8`, `gap-y-0`, `gap-y-1`, `gap-y-2`, `gap-y-3`, `gap-y-4`, `gap-y-5`, `gap-y-6`, `gap-y-8`

### Width / Height (46)

`h-0`, `h-1`, `h-1/2`, `h-1/3`, `h-1/4`, `h-2`, `h-2/3`, `h-3`, `h-3/4`, `h-4`, `h-5`, `h-6`, `h-8`, `h-10`, `h-12`, `h-16`, `h-20`, `h-24`, `h-32`, `h-48`, `h-auto`, `h-full`, `h-screen`, `w-0`, `w-1`, `w-1/2`, `w-1/3`, `w-1/4`, `w-2`, `w-2/3`, `w-3`, `w-3/4`, `w-4`, `w-5`, `w-6`, `w-8`, `w-10`, `w-12`, `w-16`, `w-20`, `w-24`, `w-32`, `w-48`, `w-auto`, `w-full`, `w-screen`

### Sizing constraints (17)

`max-w-2xl`, `max-w-3xl`, `max-w-4xl`, `max-w-5xl`, `max-w-fit`, `max-w-full`, `max-w-lg`, `max-w-md`, `max-w-none`, `max-w-sm`, `max-w-xl`, `max-w-xs`, `min-w-0`, `min-w-fit`, `min-w-full`, `min-w-max`, `min-w-min`

### Aspect ratio (4)

`aspect-[9/16]`, `aspect-auto`, `aspect-square`, `aspect-video`

### Display (7)

`block`, `flex`, `grid`, `hidden`, `inline`, `inline-block`, `inline-flex`

### Flex (10)

`flex-1`, `flex-auto`, `flex-col`, `flex-col-reverse`, `flex-initial`, `flex-none`, `flex-nowrap`, `flex-row`, `flex-row-reverse`, `flex-wrap`

### Justify content (6)

`justify-around`, `justify-between`, `justify-center`, `justify-end`, `justify-evenly`, `justify-start`

### Align items (5)

`items-baseline`, `items-center`, `items-end`, `items-start`, `items-stretch`

### Align self (6)

`self-auto`, `self-baseline`, `self-center`, `self-end`, `self-start`, `self-stretch`

### Position (5)

`absolute`, `fixed`, `relative`, `static`, `sticky`

### Z-index (7)

`z-0`, `z-10`, `z-20`, `z-30`, `z-40`, `z-50`, `z-auto`

### Overflow (10)

`overflow-auto`, `overflow-hidden`, `overflow-scroll`, `overflow-visible`, `overflow-x-auto`, `overflow-x-hidden`, `overflow-x-scroll`, `overflow-y-auto`, `overflow-y-hidden`, `overflow-y-scroll`

### Text size (6)

`text-2xl`, `text-base`, `text-lg`, `text-sm`, `text-xl`, `text-xs`

### Font weight (6)

`font-bold`, `font-extrabold`, `font-light`, `font-medium`, `font-normal`, `font-semibold`

### Text transform (4)

`capitalize`, `lowercase`, `normal-case`, `uppercase`

### Letter spacing (tracking) (5)

`tracking-normal`, `tracking-tight`, `tracking-wide`, `tracking-wider`, `tracking-widest`

### Line height (leading) (6)

`leading-loose`, `leading-none`, `leading-normal`, `leading-relaxed`, `leading-snug`, `leading-tight`

### Text color (61)

`text-amber-500`, `text-amber-600`, `text-amber-700`, `text-blue-500`, `text-blue-600`, `text-blue-700`, `text-emerald-500`, `text-emerald-600`, `text-emerald-700`, `text-gray-50`, `text-gray-100`, `text-gray-200`, `text-gray-300`, `text-gray-400`, `text-gray-500`, `text-gray-600`, `text-gray-700`, `text-gray-800`, `text-gray-900`, `text-green-500`, `text-green-600`, `text-green-700`, `text-indigo-500`, `text-indigo-600`, `text-indigo-700`, `text-neutral-50`, `text-neutral-100`, `text-neutral-200`, `text-neutral-300`, `text-neutral-400`, `text-neutral-500`, `text-neutral-600`, `text-neutral-700`, `text-neutral-800`, `text-neutral-900`, `text-red-500`, `text-red-600`, `text-red-700`, `text-sky-500`, `text-sky-600`, `text-sky-700`, `text-slate-50`, `text-slate-100`, `text-slate-200`, `text-slate-300`, `text-slate-400`, `text-slate-500`, `text-slate-600`, `text-slate-700`, `text-slate-800`, `text-slate-900`, `text-zinc-50`, `text-zinc-100`, `text-zinc-200`, `text-zinc-300`, `text-zinc-400`, `text-zinc-500`, `text-zinc-600`, `text-zinc-700`, `text-zinc-800`, `text-zinc-900`

### Background color (60)

`bg-amber-50`, `bg-amber-100`, `bg-amber-500`, `bg-amber-600`, `bg-blue-50`, `bg-blue-100`, `bg-blue-500`, `bg-blue-600`, `bg-emerald-50`, `bg-emerald-100`, `bg-emerald-500`, `bg-emerald-600`, `bg-gray-50`, `bg-gray-100`, `bg-gray-200`, `bg-gray-300`, `bg-gray-400`, `bg-gray-500`, `bg-gray-600`, `bg-gray-700`, `bg-gray-800`, `bg-gray-900`, `bg-green-50`, `bg-green-100`, `bg-green-500`, `bg-green-600`, `bg-neutral-50`, `bg-neutral-100`, `bg-neutral-200`, `bg-neutral-300`, `bg-neutral-400`, `bg-neutral-500`, `bg-neutral-600`, `bg-neutral-700`, `bg-neutral-800`, `bg-neutral-900`, `bg-red-50`, `bg-red-100`, `bg-red-500`, `bg-red-600`, `bg-slate-50`, `bg-slate-100`, `bg-slate-200`, `bg-slate-300`, `bg-slate-400`, `bg-slate-500`, `bg-slate-600`, `bg-slate-700`, `bg-slate-800`, `bg-slate-900`, `bg-zinc-50`, `bg-zinc-100`, `bg-zinc-200`, `bg-zinc-300`, `bg-zinc-400`, `bg-zinc-500`, `bg-zinc-600`, `bg-zinc-700`, `bg-zinc-800`, `bg-zinc-900`

### Border color (13)

`border-gray-100`, `border-gray-200`, `border-gray-300`, `border-gray-400`, `border-slate-100`, `border-slate-200`, `border-slate-300`, `border-slate-400`, `border-transparent`, `border-zinc-100`, `border-zinc-200`, `border-zinc-300`, `border-zinc-400`

### Border width (11)

`border`, `border-0`, `border-2`, `border-4`, `border-8`, `border-b`, `border-l`, `border-r`, `border-t`, `border-x`, `border-y`

### Border radius (8)

`rounded-2xl`, `rounded-3xl`, `rounded-full`, `rounded-lg`, `rounded-md`, `rounded-none`, `rounded-sm`, `rounded-xl`

### Opacity (15)

`opacity-0`, `opacity-5`, `opacity-10`, `opacity-20`, `opacity-25`, `opacity-30`, `opacity-40`, `opacity-50`, `opacity-60`, `opacity-70`, `opacity-75`, `opacity-80`, `opacity-90`, `opacity-95`, `opacity-100`

### Box shadow (7)

`shadow-2xl`, `shadow-inner`, `shadow-lg`, `shadow-md`, `shadow-none`, `shadow-sm`, `shadow-xl`

### Cursor (7)

`cursor-default`, `cursor-help`, `cursor-move`, `cursor-not-allowed`, `cursor-pointer`, `cursor-text`, `cursor-wait`

### Transition (18)

`duration-75`, `duration-100`, `duration-150`, `duration-200`, `duration-300`, `duration-500`, `duration-700`, `duration-1000`, `ease-in`, `ease-in-out`, `ease-linear`, `ease-out`, `transition`, `transition-all`, `transition-colors`, `transition-none`, `transition-opacity`, `transition-transform`

## State variants

### Focus (3)

`focus:outline-none`, `focus:ring-2`, `focus:ring-offset-2`

### Disabled (2)

`disabled:cursor-not-allowed`, `disabled:opacity-50`

### Hover — opacity (5)

`hover:opacity-50`, `hover:opacity-75`, `hover:opacity-80`, `hover:opacity-90`, `hover:opacity-100`

### Hover — text color (41)

`hover:text-amber-500`, `hover:text-amber-600`, `hover:text-amber-700`, `hover:text-blue-500`, `hover:text-blue-600`, `hover:text-blue-700`, `hover:text-emerald-500`, `hover:text-emerald-600`, `hover:text-emerald-700`, `hover:text-gray-500`, `hover:text-gray-600`, `hover:text-gray-700`, `hover:text-gray-800`, `hover:text-gray-900`, `hover:text-green-500`, `hover:text-green-600`, `hover:text-green-700`, `hover:text-indigo-500`, `hover:text-indigo-600`, `hover:text-indigo-700`, `hover:text-neutral-500`, `hover:text-neutral-600`, `hover:text-neutral-700`, `hover:text-neutral-800`, `hover:text-neutral-900`, `hover:text-red-500`, `hover:text-red-600`, `hover:text-red-700`, `hover:text-sky-500`, `hover:text-sky-600`, `hover:text-sky-700`, `hover:text-slate-500`, `hover:text-slate-600`, `hover:text-slate-700`, `hover:text-slate-800`, `hover:text-slate-900`, `hover:text-zinc-500`, `hover:text-zinc-600`, `hover:text-zinc-700`, `hover:text-zinc-800`, `hover:text-zinc-900`

### Hover — background color (32)

`hover:bg-amber-50`, `hover:bg-amber-100`, `hover:bg-amber-500`, `hover:bg-amber-600`, `hover:bg-blue-50`, `hover:bg-blue-100`, `hover:bg-blue-500`, `hover:bg-blue-600`, `hover:bg-emerald-50`, `hover:bg-emerald-100`, `hover:bg-emerald-500`, `hover:bg-emerald-600`, `hover:bg-gray-50`, `hover:bg-gray-100`, `hover:bg-gray-200`, `hover:bg-green-50`, `hover:bg-green-100`, `hover:bg-green-500`, `hover:bg-green-600`, `hover:bg-neutral-50`, `hover:bg-neutral-100`, `hover:bg-neutral-200`, `hover:bg-red-50`, `hover:bg-red-100`, `hover:bg-red-500`, `hover:bg-red-600`, `hover:bg-slate-50`, `hover:bg-slate-100`, `hover:bg-slate-200`, `hover:bg-zinc-50`, `hover:bg-zinc-100`, `hover:bg-zinc-200`

## Theme variants

### Dark — text color (61)

`dark:text-amber-500`, `dark:text-amber-600`, `dark:text-amber-700`, `dark:text-blue-500`, `dark:text-blue-600`, `dark:text-blue-700`, `dark:text-emerald-500`, `dark:text-emerald-600`, `dark:text-emerald-700`, `dark:text-gray-50`, `dark:text-gray-100`, `dark:text-gray-200`, `dark:text-gray-300`, `dark:text-gray-400`, `dark:text-gray-500`, `dark:text-gray-600`, `dark:text-gray-700`, `dark:text-gray-800`, `dark:text-gray-900`, `dark:text-green-500`, `dark:text-green-600`, `dark:text-green-700`, `dark:text-indigo-500`, `dark:text-indigo-600`, `dark:text-indigo-700`, `dark:text-neutral-50`, `dark:text-neutral-100`, `dark:text-neutral-200`, `dark:text-neutral-300`, `dark:text-neutral-400`, `dark:text-neutral-500`, `dark:text-neutral-600`, `dark:text-neutral-700`, `dark:text-neutral-800`, `dark:text-neutral-900`, `dark:text-red-500`, `dark:text-red-600`, `dark:text-red-700`, `dark:text-sky-500`, `dark:text-sky-600`, `dark:text-sky-700`, `dark:text-slate-50`, `dark:text-slate-100`, `dark:text-slate-200`, `dark:text-slate-300`, `dark:text-slate-400`, `dark:text-slate-500`, `dark:text-slate-600`, `dark:text-slate-700`, `dark:text-slate-800`, `dark:text-slate-900`, `dark:text-zinc-50`, `dark:text-zinc-100`, `dark:text-zinc-200`, `dark:text-zinc-300`, `dark:text-zinc-400`, `dark:text-zinc-500`, `dark:text-zinc-600`, `dark:text-zinc-700`, `dark:text-zinc-800`, `dark:text-zinc-900`

### Dark — background color (60)

`dark:bg-amber-50`, `dark:bg-amber-100`, `dark:bg-amber-500`, `dark:bg-amber-600`, `dark:bg-blue-50`, `dark:bg-blue-100`, `dark:bg-blue-500`, `dark:bg-blue-600`, `dark:bg-emerald-50`, `dark:bg-emerald-100`, `dark:bg-emerald-500`, `dark:bg-emerald-600`, `dark:bg-gray-50`, `dark:bg-gray-100`, `dark:bg-gray-200`, `dark:bg-gray-300`, `dark:bg-gray-400`, `dark:bg-gray-500`, `dark:bg-gray-600`, `dark:bg-gray-700`, `dark:bg-gray-800`, `dark:bg-gray-900`, `dark:bg-green-50`, `dark:bg-green-100`, `dark:bg-green-500`, `dark:bg-green-600`, `dark:bg-neutral-50`, `dark:bg-neutral-100`, `dark:bg-neutral-200`, `dark:bg-neutral-300`, `dark:bg-neutral-400`, `dark:bg-neutral-500`, `dark:bg-neutral-600`, `dark:bg-neutral-700`, `dark:bg-neutral-800`, `dark:bg-neutral-900`, `dark:bg-red-50`, `dark:bg-red-100`, `dark:bg-red-500`, `dark:bg-red-600`, `dark:bg-slate-50`, `dark:bg-slate-100`, `dark:bg-slate-200`, `dark:bg-slate-300`, `dark:bg-slate-400`, `dark:bg-slate-500`, `dark:bg-slate-600`, `dark:bg-slate-700`, `dark:bg-slate-800`, `dark:bg-slate-900`, `dark:bg-zinc-50`, `dark:bg-zinc-100`, `dark:bg-zinc-200`, `dark:bg-zinc-300`, `dark:bg-zinc-400`, `dark:bg-zinc-500`, `dark:bg-zinc-600`, `dark:bg-zinc-700`, `dark:bg-zinc-800`, `dark:bg-zinc-900`

### Dark — border color (13)

`dark:border-gray-600`, `dark:border-gray-700`, `dark:border-gray-800`, `dark:border-gray-900`, `dark:border-slate-600`, `dark:border-slate-700`, `dark:border-slate-800`, `dark:border-slate-900`, `dark:border-transparent`, `dark:border-zinc-600`, `dark:border-zinc-700`, `dark:border-zinc-800`, `dark:border-zinc-900`

### Dark — hover text (15)

`dark:hover:text-gray-50`, `dark:hover:text-gray-100`, `dark:hover:text-gray-200`, `dark:hover:text-gray-300`, `dark:hover:text-gray-400`, `dark:hover:text-slate-50`, `dark:hover:text-slate-100`, `dark:hover:text-slate-200`, `dark:hover:text-slate-300`, `dark:hover:text-slate-400`, `dark:hover:text-zinc-50`, `dark:hover:text-zinc-100`, `dark:hover:text-zinc-200`, `dark:hover:text-zinc-300`, `dark:hover:text-zinc-400`

### Dark — hover background (9)

`dark:hover:bg-gray-700`, `dark:hover:bg-gray-800`, `dark:hover:bg-gray-900`, `dark:hover:bg-slate-700`, `dark:hover:bg-slate-800`, `dark:hover:bg-slate-900`, `dark:hover:bg-zinc-700`, `dark:hover:bg-zinc-800`, `dark:hover:bg-zinc-900`
