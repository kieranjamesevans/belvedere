# Layer Naming Vocabulary

All names should be output in camelCase.

---

## UI Component Canonical Names
These are the authoritative names for UI components. Aliases in parentheses should be normalized to the canonical name.

### Navigation/Structure
- accordion (collapse, disclosure, expandable)
- breadcrumbs
- carousel
- drawer (tray, flyout, sheet)
- dropdownMenu
- footer
- header
- navigation (nav, menu)
- tabs (tabbedInterface)
- treeView

### Actions
- button
- buttonGroup
- link
- dropdown

### Forms
- checkbox
- combobox (autocomplete)
- colorPicker
- dateInput
- datepicker (calendar)
- form
- label
- radioButton
- richTextEditor
- searchInput
- select
- slider (rangeInput)
- stepper (numberInput)
- textInput
- textarea
- toggle (switch)
- fileUpload (dropzone)

### Content/Display
- avatar
- card
- emptyState
- file
- image
- quote
- video
- heading
- icon
- separator (divider)

### Data/Layout
- list
- table
- pagination
- stack

### Utility
- tooltip
- popover
- skipLink
- visuallyHidden

### Feedback/Status
- alert (notification, banner, callout)
- badge (tag, chip)
- progressBar
- progressIndicator (timeline)
- skeleton (loadingState)
- spinner (loader)
- toast (snackbar)

---

## Structural/Layout Layer Names
These are valid names for non-component structural layers. Use these when a layer is scaffolding rather than a UI component.

### Containers
- outerContainer
- innerContainer
- container
- wrapper
- section
- row
- col
- stack
- group

### Page Regions
- header
- body
- footer
- sidebar
- panel
- content
- main

### Sub-structural
- overview
- type
- value
- item
- cell
- slot

### Visual/Asset
- background (bg)
- overlay
- scrim
- divider
- spacer
- icon
- image
- illustration
- logo
- thumbnail
- avatar

### Text
- label
- heading
- subheading
- caption
- description
- title
- text
- eyebrow

---

## Figma Auto-Generated Names to Flag as Dirty
These patterns indicate a layer name was never intentionally set and should be renamed:

- `Frame \d+` (e.g., "Frame 47", "Frame 14963")
- `Group \d+` (e.g., "Group 3")
- `Rectangle \d+` (e.g., "Rectangle 1")
- `Vector \d+`
- `Ellipse \d+`
- `Polygon \d+`
- `Star \d+`
- `Line \d+`
- `Path \d+`
- `image \d+` (lowercase, e.g., "image 1")
- `Text` (bare, with no other content)
- `Layer \d+`
- `Shape \d+`

---

## Naming Rules

1. **Apply chosen syntax consistently** — camelCase, kebab-case, snake_case, or Title Case as selected at run start
2. **Generic over qualified** — name the layer for what it *is*, not where it *lives*. The hierarchy already provides location context. Use `background` not `buttonBackground`, `list` not `productList`, `wrapper` not `navigationWrapper`, `track` not `carouselTrack`, `items` not `listItems`.
3. **Single-word always** — use the shortest generic term that describes the layer's role, full stop. Never compound for disambiguation. If two siblings are both `section`, that's fine — they're visually distinct on the canvas and the hierarchy tells the story. Repeating names across siblings is expected and correct.
4. **Never rename** component instances with meaningful existing names (e.g., `arrowUp`, `NM Logo`, `overlay`)
5. **Never rename** layers that already match a canonical or structural name from this vocabulary
6. **Preserve intent** — if a layer is named something clearly intentional but non-standard (e.g., `--p-gray-100`), leave it alone
7. **The hierarchy tells the "where" story** — a `background` inside a `button` inside a `header` is understood from its position. Its name only needs to say `background`.
