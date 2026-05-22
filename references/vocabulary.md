# Layer Naming Vocabulary

All names should be output in camelCase.

---

## UI Component Canonical Names
These are the authoritative names for UI components. Aliases in parentheses should be normalized to the canonical name.

### Navigation/Structure
- accordion (collapse, disclosure, expandable, arrowToggle, collapsible, collapsibleSections, details, expander)
- breadcrumbs (breadcrumbTrail)
- carousel (contentSlider)
- drawer (tray, flyout, sheet)
- dropdownMenu (selectMenu)
- footer
- header
- hero (jumbotron, promoBanner)
- modal (dialog, popup, modalWindow)
- navigation (nav, menu, topNav)
- tabs (tabbedInterface, tabset, contentTabs, sectionTabs)
- treeView

### Actions
- button (cta)
- buttonGroup (toolbar)
- link (anchor, hyperlink)
- segmentedControl (toggleButtonGroup)

### Forms
- checkbox
- combobox (autocomplete, autosuggest)
- colorPicker
- dateInput
- datepicker (calendar, datetimePicker)
- fieldset
- form (formPanel)
- label (formLabel)
- radioButton (radio, radioGroup)
- rating
- richTextEditor (rte, wysiwygEditor)
- searchInput (search)
- select (dropdown, selectInput)
- slider (rangeInput)
- stepper (nudger, quantity, counter, numberInput)
- textInput (textbox, textBox)
- textarea
- toggle (switch, lightswitch, toggleButton)
- fileUpload (dropzone, fileInput, fileUploader)

### Content/Display
- avatar (portrait, profileImage, profilePhoto, userAvatar, userImage)
- card (tile, contentTile)
- emptyState
- file (attachment, download)
- image (picture)
- quote (pullQuote, blockQuote)
- video (videoPlayer)
- heading
- icon
- separator (divider, horizontalRule, verticalRule)

### Data/Layout
- list (listView)
- table
- pagination (pageControls, pageNav, pager)
- stack

### Utility
- tooltip (toggletip)
- popover
- skipLink
- visuallyHidden (screenreaderOnly)

### Feedback/Status
- alert (notification, banner, callout, feedback, message)
- badge (tag, chip, label, statusBadge, statusChip, statusPill, tagPill)
- progressBar (progress)
- progressIndicator (timeline, meter, progressTracker, steps, stepperProgress)
- skeleton (loadingState, skeletonLoader)
- spinner (loader, loading)
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
