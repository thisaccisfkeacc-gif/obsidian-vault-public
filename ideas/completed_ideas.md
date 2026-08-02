# ✅ Completed Ideas & Tasks

- [x] **Macro vs Template Architecture** *(Completed 2026-08-02)*
  - [x] **Storage**: Unified model, single table, `Type` discriminator (`Macro` | `Template`) — nullable columns for Template-only fields
  - [x] **Insert Behavior**: Static copy (Expand Inline) — blocks are copied into the macro at insert time
  - [x] **Source Tracking**: Store hidden `SourceTemplateId` + `SourceTemplateVersion` on inserted block groups
  - [x] **Parameterization**: Auto-detect existing macro variables on save
  - [x] **Block Palette**: Templates appear as a collapsible category in the sidebar
  - [x] **Call Macro**: Calls full Macros only; Templates are design-time composition tools
  - [x] **Nesting**: Flattened on insert; circular reference detection at save time
  - [x] **Deletion**: Safe (static copies) with usage count info
