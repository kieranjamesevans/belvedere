---
name: belvedere
description: Belvedere cleans up and normalizes Figma layer names and structure across a file or page. Renames dirty/auto-generated layers using a canonical vocabulary, and identifies redundant wrapper frames for removal with automatic child promotion. Trigger this skill when the user says "belvedere", "clean up layers", "rename layers", "fix layer names", "normalize layers", "clean up structure", "remove redundant frames", or "clean up [Figma file URL]". Also trigger when the user shares a Figma URL and asks for layer hygiene, naming cleanup, structural cleanup, or design file organization. Always use this skill for any request involving bulk layer renaming or structural cleanup in Figma.
---

# Belvedere

Cleans up Figma layer names and redundant nesting. Normalizes names to a chosen convention using a canonical vocabulary, converts groups to frames, removes unnecessary wrapper frames, and runs an AI semantic pass to catch non-canonical custom names.

---

## Trigger phrases
- "belvedere"
- "clean up layers in [URL]"
- "rename layers in [URL]"
- "fix layer names in [URL]"
- "clean up [Figma file URL]"

---

## Before You Start

Read `references/vocabulary.md` — it contains canonical component names, structural layer names, dirty name patterns, and naming rules.

---

## Workflow

### Step 0 — Announce Belvedere

Display this exact widget using the show_widget tool before anything else:

```html
<div style="display:flex;align-items:center;gap:20px;padding:1rem 0;">
  <div style="position:relative;width:80px;height:80px;flex-shrink:0;">
    <svg viewBox="0 0 80 80" xmlns="http://www.w3.org/2000/svg" style="width:80px;height:80px;">
      <circle cx="40" cy="40" r="39" fill="#8B1A1A" />
      <circle cx="40" cy="40" r="39" fill="url(#vignette)" />
      <defs>
        <radialGradient id="vignette" cx="40%" cy="35%" r="65%" gradientUnits="userSpaceOnUse">
          <stop offset="0%" stop-color="#C0392B" stop-opacity="0.6"/>
          <stop offset="100%" stop-color="#4A0000" stop-opacity="0"/>
        </radialGradient>
      </defs>
      <text x="40" y="52" text-anchor="middle" font-family="Georgia, serif" font-size="36" font-weight="400" fill="rgba(255,255,255,0.92)" letter-spacing="-1">B</text>
      <circle cx="40" cy="40" r="39" fill="none" stroke="rgba(255,255,255,0.15)" stroke-width="1"/>
    </svg>
  </div>
  <div style="max-width:340px;">
    <p style="font-size:15px;font-weight:500;color:var(--color-text-primary);margin:0 0 4px;letter-spacing:0.01em;">Belvedere has arrived.</p>
    <p style="font-size:13px;color:var(--color-text-secondary);margin:0;line-height:1.5;">Life is more than mere survival, and we just might live the good life yet.</p>
  </div>
</div>
```

---

### Step 1 — Ask for naming syntax

Ask the user which naming convention to use before doing anything else:

> "What naming syntax should I use for renamed layers?"
> - camelCase (e.g. `outerContainer`)
> - PascalCase (e.g. `OuterContainer`)
> - kebab-case (e.g. `outer-container`)
> - snake_case (e.g. `outer_container`)
> - Title Case (e.g. `Outer Container`)

Store the chosen syntax and apply it consistently throughout the run. All vocabulary examples in `references/vocabulary.md` are in camelCase — convert them to the chosen syntax before use.

---

### Step 2 — Parse the file URL

Extract `fileKey` from the Figma URL:
- URL format: `https://www.figma.com/design/:fileKey/:fileName?...`
- If a `node-id` param is present, scope to that page
- If no node-id, use `figma.currentPage`

---

### Step 3 — Get the layer tree

**Critical:** The Figma Plugin API only keeps the active page's nodes in memory. Multi-page traversal in a single `use_figma` call causes stale node errors.

- Always call `await figma.setCurrentPageAsync(page)` before working on each page
- Process one page at a time
- Skip resource/component pages (pages named "Components", "Graveyard", "Cover", separator pages with blank names or `---`)
- Scan and apply changes in the same plugin call — never scan in one call and apply in another, as node references go stale between calls

---

### Step 4 — Convert groups to frames (automatic)

Convert all editable GROUP nodes to FRAME nodes before any renaming. This is lossless. No user confirmation needed.

**Implementation (proven in testing):**

```javascript
async function convertGroupToFrame(group) {
  const parent = group.parent;
  if (!parent) return null;
  const index = parent.children.indexOf(group);

  // Capture absolute positions BEFORE any moves
  const childData = [...group.children].map(child => ({
    id: child.id,
    absX: child.absoluteTransform[0][2],
    absY: child.absoluteTransform[1][2],
  }));

  const frame = figma.createFrame();
  frame.name = group.name || 'frame';
  frame.resize(group.width, group.height);
  frame.x = group.x;
  frame.y = group.y;
  frame.fills = [];
  frame.clipsContent = false;

  // Insert frame first, then move children
  parent.insertChild(index, frame);

  for (const { id, absX, absY } of childData) {
    const child = figma.getNodeById(id);
    if (!child) continue;
    frame.appendChild(child);
    // Recalculate position relative to new frame origin
    child.x = absX - frame.absoluteTransform[0][2];
    child.y = absY - frame.absoluteTransform[1][2];
  }

  try { if (group.children.length === 0) group.remove(); } catch(e) {}
  return frame.name;
}
```

**Rules:**
- Sort groups deepest-first before processing to avoid parent/child conflicts
- Skip any group inside `INSTANCE`, `COMPONENT`, or `COMPONENT_SET` — check all ancestors, not just the direct parent
- Use `figma.getNodeById(id)` (sync) for stability during batch operations
- After conversion, converted frames feed into the rename pass like any other frame

---

### Step 5 — Collect all layers for assessment

Scan every non-exempt node. The goal is to assess everything — not just auto-generated names. A user-given name is not automatically correct.

**Exempt from assessment (skip entirely):**
- Slash-path component names (e.g. `product/card/desktop`, `icons/chevron-left`)
- Design token strings (`--p-gray-100` style)
- Layers already exactly matching a canonical vocabulary term

**Not exempt — always assess:**
- Auto-generated dirty names (`Frame 3`, `Group 12`, `Rectangle`, etc.)
- Copy suffix names (`quantity field copy`, `product title copy 3`)
- Custom names that may not be canonical (`video group`, `Rich Text Icons`, `Blog`, `Recent`)
- Locked layers — locking is edit protection, not naming approval. Flag bad names but note they require unlocking before applying

**Dirty name patterns:**
- `Frame \d+`, `Group \d+`, `Rectangle \d+`, `Vector \d+`, `Ellipse \d+`
- `Polygon \d+`, `Star \d+`, `Line \d+`, `Path \d+`
- `image \d+`, `Text` (bare), `Layer \d+`, `Shape \d+`
- `/^(.+?)\s+copy(\s+\d+)?$/i` — copy suffix pattern

**Never traverse into:**
- `INSTANCE`, `COMPONENT`, or `COMPONENT_SET` subtrees
- `Path` or `Polygon` nodes inside `BOOLEAN_OPERATION` parents

---

### Step 5b — AI semantic normalization

Send all non-exempt, non-canonical layer names in a single batch to Claude via the Anthropic API for normalization:

```javascript
const response = await fetch("https://api.anthropic.com/v1/messages", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({
    model: "claude-sonnet-4-20250514",
    max_tokens: 1000,
    messages: [{
      role: "user",
      content: `You are a Figma layer naming assistant. Propose a canonical replacement for each layer name using this vocabulary reference:\n\n${vocabularyReference}\n\nRules:\n- Name what the layer IS, not where it lives\n- Single word always — never compound for disambiguation\n- Apply [chosen syntax] consistently\n- Display text labels (e.g. "Recently Purchased", "Price", "Item") are content, not structure — return unchanged\n- Slash-path names are canonical — return unchanged\n- If unsure, return unchanged\n\nRespond ONLY with a JSON object. No preamble, no markdown.\n\nLayer names: ${JSON.stringify(namesToAssess)}`
    }]
  })
});
const data = await response.json();
const nameMap = JSON.parse(data.content.find(b => b.type === 'text')?.text || '{}');
```

If the API call fails, continue without it — the dirty name pass still runs. Note in the preview that semantic normalization was skipped.

---

### Step 6 — Identify redundant frames

A frame is redundant if it meets **all** of:
- Exactly one child
- No fill, stroke, or background
- No auto-layout padding (or auto-layout not enabled)
- Not a component or instance
- Not a clipping mask (`clipsContent` is false)

**Note:** Any frame can be flagged, regardless of whether its name is dirty or intentional.

**Never flag:**
- The root/page level frame
- Frames inside component sets

---

### Step 7 — Show previews and get confirmation

Present two tables before applying anything:

**Rename preview:**
```
Layers to rename: N

Layer ID  | Current Name      | Proposed Name
----------|-------------------|---------------
0:123     | Rectangle         | background
0:456     | video group       | container
```

**Structure preview:**
```
Redundant frames to remove: N
(Child promoted to parent on removal)

Layer ID  | Frame Name  | Child        | Parent
----------|-------------|--------------|--------
0:789     | Frame 14    | wrapper      | section
```

Ask: "Should I apply these changes? You can tell me to skip specific IDs."

---

### Step 8 — Apply changes

**Critical:** Scan and apply in the same plugin call. Node references go stale between separate calls.

Apply renames:
```javascript
const node = figma.getNodeById(id);
if (node) node.name = newName;
```

Remove redundant frames (deepest-first to avoid orphaning):
```javascript
async function removeRedundantFrame(frame) {
  const parent = frame.parent;
  const child = frame.children[0];
  const index = parent.children.indexOf(frame);

  const childAbsX = child.absoluteTransform[0][2];
  const childAbsY = child.absoluteTransform[1][2];

  parent.insertChild(index, child);
  child.x = childAbsX - parent.absoluteTransform[0][2];
  child.y = childAbsY - parent.absoluteTransform[1][2];

  try { frame.remove(); } catch(e) {}
}
```

---

### Step 9 — Confirm

Report:
- Groups converted to frames: N
- Layers renamed: N
- Redundant frames removed: N
- Anything skipped and why

---

## Naming Rules

1. **Apply chosen syntax consistently** — camelCase, PascalCase, kebab-case, snake_case, or Title Case
2. **Name what it IS, not where it lives** — `background` not `buttonBackground`, `list` not `productList`
3. **Single word always** — never compound for disambiguation. Siblings can share names. The hierarchy provides context.
4. **Cross-reference vocabulary** — before finalising any name, check `references/vocabulary.md`. Aliases resolve to canonical terms.
5. **Display text is not a layer name** — text content like "Recently Purchased" or "Price" inside a text node is content, not structure. Leave it.
6. **Preserve design tokens** — any layer with a `--` prefix is intentional, leave it.

---

## Screen frame convention

Top-level screen frames follow this pattern: `PascalCase-Role-Breakpoint`

- No project prefix (drop `JB_`, `NM_`, etc.)
- No numbered prefixes (`001`, `002`)
- No copy suffixes — collapse duplicates to the base name
- Breakpoints: `D` (desktop), `T` (tablet), `M` (mobile)
- Roles: `Customer`, `SalesRep`, `Admin`, `Buyer`, `Payer` etc.
- Separators: dashes throughout, no underscores

Examples: `Cart-Customer-D`, `CheckoutReview-SalesRep-M`, `InvoicePayment-Buyer-T`

---

## Known edge cases

**Position drift on group conversion** — always capture `child.absoluteTransform` before `frame.appendChild(child)`. The coordinate space changes on reparenting and must be recalculated.

**Stale node references** — scan and apply in the same `use_figma` call. Restoring a file version reassigns node IDs — rebuild the rename list from a fresh scan after any restore.

**Redundant frame chains** — process deepest-first. A chain of 3 single-child frames collapses correctly when removed in depth order.

**Read-only files** — stop after Step 7 (preview only). Inform the user they need edit access to apply.

**Auto-layout frames** — flag only if padding is definitively 0 and there is no fill. A zero-padding wrapper may still be participating in layout spacing.
