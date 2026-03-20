---
name: suggestions
description: "Contextual suggestion patterns for recommending next steps based on current extension code, capabilities, and surfaces. Use when deciding what to suggest after making changes."
---

# Next Steps — Suggestion Patterns

Use these patterns to analyze the current extension and suggest 2-3 specific, actionable next steps.
Suggestions should reference actual data, components, and capabilities in the extension — never generic.

## When to suggest

- After making code changes (`updateCode`, `updateManifest`)
- After adding a surface, capability, or component
- Do NOT suggest after answering a question or explaining code

## Suggestion format

- **Label:** 2-5 word actionable verb phrase (e.g., "Show order total", "Add loading state")
- **Prompt:** Specific enough that clicking it produces a useful result. Reference actual data or code in the extension (e.g., "Add the shipping address from the data.fetch response below the order total")

## Pattern categories

### Code-aware — analyze what's in the code vs. what's possible

| Signal | Suggestion direction |
|--------|---------------------|
| Data fields fetched but not rendered | Suggest rendering unused fields (e.g., "Show order total", "Display customer email") |
| Component rendered without styling refinement | Suggest layout improvements (e.g., "Add spacing to header", "Style the card layout") |
| Surface exists but is sparse (few child components) | Suggest adding content (e.g., "Add status badge to header", "Include action buttons") |
| No error/loading handling for async operations | Suggest robustness (e.g., "Add loading state", "Handle fetch errors") |
| Hardcoded values that could come from context | Suggest dynamic data (e.g., "Use customer name from context") |

### Capability-aware — suggest capabilities the extension doesn't use yet

| Current state | Suggestion direction |
|---------------|---------------------|
| No capabilities wired | Suggest one relevant to the extension's apparent purpose |
| Has `data.fetch` but no `context.read` | "Wire up customer context" |
| Has `data.query` but no `actions.toast` | "Add success notifications" |
| Uses `context.read` data but doesn't pass it to `data.fetch` | "Use context in API call" |
| Has `data.fetch` but no error handling | "Handle API errors gracefully" |

### Surface-aware — suggest additional UI surfaces

| Current surfaces | Suggestion direction |
|-----------------|---------------------|
| Only `content` | "Add a header surface" or "Add footer actions" |
| Only `header` | "Add a content surface" |
| Has content but no `footer-links` | "Add quick action links" |
| Has header + content but no footer | "Add footer with action buttons" |

### Platform-aware — suggest based on extension target

| Target platform | Suggestion direction |
|----------------|---------------------|
| Commerce (Shopify, BigCommerce) | Order data, product info, customer purchase history |
| Support (Zendesk, Freshdesk) | Ticket context, conversation data, agent tools |
| General | Configuration, settings, external API integrations |

## Quality rules

- Never suggest something already present in the code
- Prefer suggestions that build on what the user just did (e.g., after adding a surface, suggest populating it)
- Mix suggestion types: one "enhance what's there", one "add something new", one "improve quality" (loading/error states, styling)
- Labels must be short enough to fit in a chip button (2-5 words)
- Prompts should be one sentence that clearly describes the change

## Examples by context

| After this action | Label | Prompt |
|---|---|---|
| Added `data.fetch` to an orders API | "Show order total" | "Add the order total from the data.fetch response to the content surface" |
| Added a header surface | "Add status badge" | "Add a status badge component to the header showing the current order status" |
| Wired up `context.read` | "Use in API call" | "Pass the customerId from context.read as a parameter in the data.fetch call" |
| Styled a component | "Add loading state" | "Add a loading skeleton that displays while the data.fetch call is in progress" |
| Added `data.query` | "Show results in list" | "Render the query results as a list of cards in the content surface" |
| Added a button | "Wire up action" | "Connect the button click to an actions.invoke call" |
| Added error handling | "Add success toast" | "Show a success toast notification when the operation completes" |
