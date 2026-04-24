# belvedere

Belvedere
"Life is more than mere survival, and we just might live the good life yet."
Belvedere is a Claude skill for cleaning up Figma files. It normalizes layer names, converts groups to frames, collapses redundant nesting, and applies a canonical naming vocabulary — consistently, across an entire file, without you having to do it layer by layer.

**What it does**

Converts groups to frames — automatically, with correct position handling
Renames dirty layers — auto-generated names like Frame 47, Group 12, Rectangle Copy 3
Normalizes custom names — catches non-canonical names like video group, Rich Text Icons, Blog and proposes correct alternatives via an AI semantic pass
Removes redundant frames — single-child wrapper frames with no visual properties, with child promotion (you approve before anything is removed)
Standardizes screen frames — top-level frames follow PascalCase-Role-Breakpoint (e.g. Cart-Customer-D, CheckoutReview-SalesRep-M)
Supports multiple naming conventions — camelCase, PascalCase, kebab-case, snake_case, Title Case

**
Requirements**

Claude with a Pro, Team, or Enterprise plan
Figma MCP connected to Claude with an editor seat on the file you want to clean


**Installation**

Download belvedere.skill from the latest release
In Claude, open Settings → Skills
Upload belvedere.skill
Belvedere is now available in any conversation


**Usage**
Trigger Belvedere by saying:
belvedere https://www.figma.com/design/your-file-url
Or use any of these phrases: clean up layers in [URL], rename layers in [URL], fix layer names in [URL]
Belvedere will ask you which naming syntax to use, then walk through the file page by page — showing you previews before applying anything.

**Naming standard**
Belvedere uses a canonical vocabulary built around two principles:
Name what it is, not where it lives. A rectangle inside a button inside a header is named background — not headerButtonBackground. The hierarchy already tells the location story.
Single word always. section, container, list, wrapper, track, icons. Never compound names for disambiguation — siblings can share names, and that's fine.
CategoryExamplesStructurecontainer, wrapper, section, header, body, footerLayoutrow, col, stack, list, items, trackVisualbackground, overlay, divider, image, icon, iconsTextlabel, heading, caption, description, titleInteractivebutton, input, dropdown, toggle, checkbox, stepperFeedbacktoast, alert, badge, spinner, skeleton
Screen frame convention
Top-level screen frames follow PascalCase-Role-Breakpoint — Cart-Customer-D, CheckoutReview-SalesRep-M, InvoicePayment-Buyer-T. No project prefixes, no numbered prefixes, no copy suffixes. Breakpoints: D desktop, T tablet, M mobile.

**Developing the standard**
Belvedere is designed to learn. When you encounter a naming edge case that isn't handled well, the right move is to define the rule and update references/vocabulary.md and SKILL.md. The skill is the standard — keep them in sync.

**Credits**
Built by Kieran
