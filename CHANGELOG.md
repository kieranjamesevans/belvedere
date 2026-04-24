# Changelog

## v0.1.0 — Initial release

First working version of Belvedere, developed across multiple Figma files.

### Features
- Group-to-frame conversion with correct absolute position handling
- Dirty name detection and rename (auto-generated patterns + copy suffixes)
- AI semantic normalization pass for non-canonical custom names
- Redundant single-child frame detection with user approval before removal
- Screen frame standardization (`PascalCase-Role-Breakpoint`)
- Support for camelCase, PascalCase, kebab-case, snake_case, Title Case
- Belvedere intro widget on every run

### Naming rules established
- Name what it is, not where it lives
- Single word always — never compound for disambiguation
- Siblings can share names; hierarchy provides context
- Display text content is not a structural layer name
- Locked layers are assessed for naming, but flagged as requiring unlock
- Slash-path component names are canonical and exempt from renaming

### Known constraints
- Requires Figma MCP with editor seat
- Scan and apply must happen in the same plugin call (node references go stale between calls)
- Multi-page traversal uses `setCurrentPageAsync` — one page at a time
- Restoring a file version reassigns node IDs — always rescan after a restore
