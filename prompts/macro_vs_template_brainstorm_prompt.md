# Macro vs Template Architecture — Multi-Agent Brainstorm Prompt

**Created:** July 27, 2026  
**Purpose:** Design a clear architectural distinction between full **Saved Macros** and reusable **Saved Templates** (block logic snippets)

---

## 🎯 Feature Overview

PowerX Keys currently has a **Saved Macros** system where users save complete macros for reuse. However, users also want to save **partial block logic** (like a custom If/Else construct, a repeating pattern, or a commonly used sequence) without it being a full macro.

**The problem:**
- Saved Macros = complete, executable macros with triggers
- Saved Templates = reusable block snippets (no trigger, just logic chunks)

**Goal:** Create a clear separation and update the "Call Macro" block to support both.

---

## 🛠️ Current Implementation

### Saved Macros (Existing)
- Stored in `macros.json` or similar config
- Each macro has:
  - Unique ID
  - Name
  - Trigger (hotkey, schedule, etc.)
  - Full step collection
  - Scope settings
- "Call Macro" block executes a saved macro by ID

### What's Missing
- No way to save a **block snippet** (partial logic)
- No distinction between "full macro" and "template"
- "Call Macro" block only calls full macros, not templates

---

## 💡 Use Cases for Templates

### Example 1: Common If/Else Pattern
User creates a complex If/Else block that:
1. Checks if image exists
2. If yes → clicks it
3. If no → scrolls down and retries

They want to reuse this pattern across multiple macros without recreating it each time.

### Example 2: Login Sequence
User has a standard login flow (type username → tab → type password → enter). They want to insert this into different macros as a template.

### Example 3: Error Handling Wrapper
User creates a "Try/Catch" style If/Else block that:
1. Tries an action
2. If fails → sends notification and continues

They want to wrap other blocks inside this template.

---

## ❌ Current Problems & Flaws

### Flaw 1: No Template Storage
Only full macros can be saved. Partial logic must be copy-pasted manually.

### Flaw 2: "Call Macro" is Limited
The "Call Macro" block can only call full macros with triggers. It can't insert template blocks inline.

### Flaw 3: No Visual Distinction
If we add templates, how do users distinguish them from macros in the UI? Same list? Separate tabs?

### Flaw 4: Template Parameterization
If a template has variable parts (like "click at X, Y"), how do users customize X and Y when inserting the template?

### Flaw 5: Nesting Complexity
Can a template contain another template? Can a macro call a template? Can a template call a macro? What are the rules?

---

## 💡 Seed Ideas (Initial Thoughts)

### Idea A: Separate Storage, Separate UI
- Macros stored in `macros.json`
- Templates stored in `templates.json`
- UI has two tabs: "Macros" and "Templates"
- "Call Macro" block gets a sibling: "Insert Template" block

**Pros:** Clean separation, easy to understand  
**Cons:** More UI, two systems to maintain

### Idea B: Unified Storage with Type Flag
- Both stored in same JSON
- Each item has a `Type` field: "Macro" or "Template"
- UI shows both in one list with different icons
- "Call/Insert" block adapts based on type

**Pros:** Simpler storage, one system  
**Cons:** Mixed list could be confusing

### Idea C: Templates as Macro Fragments
- Templates are just macros without triggers
- When inserting a template, the steps are copied inline (not referenced)
- No "Call Template" — just copy-paste logic

**Pros:** No new architecture, simple  
**Cons:** No live updates if template changes, duplicates logic

### Idea D: Parameterized Templates
- Templates support "slots" (variables like `{{X}}`, `{{Y}}`)
- When inserting, user fills in the slots
- Template logic is stored once, instances reference it

**Pros:** Powerful, DRY (Don't Repeat Yourself)  
**Cons:** Complex to implement, steeper learning curve

---

## 🤔 Open Questions for Agents

1. Should templates be stored separately or unified with macros?
2. When a template is inserted, should it be a reference (live update) or a copy (static)?
3. Should templates support parameterization (variables)?
4. How should the "Call Macro" block evolve to support templates?
5. What's the right UI for browsing and inserting templates?
6. Should templates be allowed to call other templates (nesting)?
7. How do we handle template deletion when macros are using it?

---

## ✍️ Agent Response Sections

*Instructions for agents: Write your analysis and suggestions directly in your designated section below. Be specific and practical.*

---

### Agent 1: Antigravity

#### 1. Unified Storage with Type Flag (`IsTemplate`)
* **Database**: Store both Macros and Templates in the existing `macros.db` (or JSON). Add an `IsTemplate` boolean column (or `Type` enum: `Macro` | `Template`).
* **Why**: Avoids creating a duplicate database/file storage system, reuses all existing C# models, repositories, and SQLite schema migrations.

#### 2. Static Copy Insertion ("Expand Inline")
* **Behavior**: When a user inserts a Template into a Macro, it creates a **static copy** of the blocks directly inside the macro.
* **Why**: Referencing templates dynamically creates fragile dependencies (e.g. deleting or modifying a template breaks 5 macros without warning). Static copying keeps every macro self-contained and safe.

#### 3. UX & Block Editor Evolution
* **Sidebar / Drawer**: Add a "Templates" tab in the Block Palette/Drawer.
* **Action**: Dragging a Template into the editor canvas expands it into its individual blocks instantly.
* **"Call Macro" Block**: Remains strictly for calling full executable Macros (as subroutines/functions). Templates are inserted as block snippets, not called via engine routines.

#### 4. Parameterization (No Over-Engineering)
* Rely on standard Macro Variables (e.g., `{Var1}`) rather than building a complex slot/template parameter engine. Keep it light, fast, and simple for users.

---

### Agent 2: [Agent Name]

You can paste this directly under **Agent 2**.

---

## **Agent 2: GPT**

### **1. Separate Concepts, Unified Backend**

I would **store Macros and Templates in the same database/model**, but treat them as **different concepts in the UI**.

Internally:

```text
AutomationItem

├── Macro
└── Template
```

Shared properties:

* Name
* Description
* Folder
* Tags
* Created
* Modified
* Blocks

Macro-only:

* Triggers
* Schedule
* Hotkeys
* Enabled

Template-only:

* Category
* Parameters
* Preview Image

One backend.

Two user experiences.

---

### **2. Templates Should Support Two Modes**

Don't force users to choose one architecture.

Support both.

### Expand Inline (Default)

```text
Template

↓

Copied

↓

Independent Blocks
```

Perfect for beginners.

Editing the template later does NOT affect existing macros.

---

### Linked Template (Advanced)

```text
Macro

↓

Template Reference

↓

Template
```

Updating the template updates every linked instance.

PowerX Keys should clearly display:

```text
Linked

Version 3

Update Available
```

Users choose per insertion.

---

### **3. Template Versioning**

Every template has versions.

```text
Login Template

v1

v2

v3
```

If Macro A uses v2,

and the template becomes v3,

show:

> Update available.

The user decides whether to migrate.

This avoids breaking existing automations.

---

### **4. Parameters Should Feel Like Components**

Instead of variables like

```text
{{X}}
{{Button}}
{{Delay}}
```

build a friendly parameter editor.

Example:

```text
Username

________

Password

________

Retry Count

3
```

When saving a template,

PowerX Keys automatically detects editable values and asks:

> Make this configurable?

This feels much more approachable than placeholder syntax.

---

### **5. Template Categories**

Templates should behave like reusable building blocks.

Examples:

```text
Browser

Files

Windows

Login

Error Handling

Loops

OCR

Image Search

Clipboard

Custom
```

Users browse by purpose rather than filename.

---

### **6. Template Preview**

Every template automatically generates a miniature flow preview.

Example:

```text
IF

↓

Image

↓

Click

↓

ELSE

↓

Scroll

↓

Repeat
```

Users immediately understand what the template does before inserting it.

---

### **7. Dependency Inspector**

If a linked template is used,

PowerX Keys shows:

```text
Used By

14 Macros
```

Before deleting:

```text
Delete?

Used by

14 macros

Convert references

Expand inline

Cancel
```

This prevents accidental breakage.

---

### **8. Smart Template Suggestions**

While building a macro,

PowerX Keys recognizes common patterns.

Example:

```text
Image

↓

Click

↓

Wait

↓

Retry
```

Suggestion:

> Save this as a reusable template?

The software helps users build their own library naturally.

---

### **9. Nested Templates**

Allow nesting,

but with limits.

Example:

```text
Maximum Depth

3
```

Also detect circular references.

```text
Template A

↓

Template B

↓

Template A

❌
```

Prevent recursion automatically.

---

### **10. Template Marketplace Architecture**

Design templates so they're portable.

Each template stores:

* Metadata
* Author
* Version
* Description
* Tags
* Required PowerX Version

Eventually users can import/export templates like plugins.

This future-proofs the system.

---

### **11. Template Designer Mode**

When editing a template,

show a slightly different editor.

Additional tools:

* Parameter Manager
* Preview Data
* Documentation
* Example Usage
* Version History

Templates become first-class assets rather than just copied blocks.

---

### **12. Macro vs Template Philosophy**

The distinction should be extremely clear:

**Macro**

> A complete automation that can execute independently.

**Template**

> A reusable piece of automation logic designed to be inserted or referenced inside other automations.

This mental model is simple, scalable, and avoids confusing users.

---

### **13. Radical Rethink: Automation Assets**

Instead of treating Macros and Templates as completely separate systems, introduce the concept of **Automation Assets**.

Everything in PowerX Keys becomes an asset:

* Macro
* Template
* Trigger
* Variable Set
* OCR Dictionary
* Image Collection
* Function
* Window Profile

The editor simply works with assets.

This provides a consistent architecture today while leaving room for future reusable components without redesigning the storage model or UI.


---

### Agent 3: [Agent Name]

### Agent 2: Grok

#### 1. Unified Storage with Clear Type Separation
Store both in the same underlying storage (`macros.db` / existing JSON structure) but with an explicit `Type` field (`Macro` | `Template`).

- Reuses every existing model, repository, serialization path, and migration.
- Avoids dual-file / dual-schema maintenance.
- UI can still present them as completely separate experiences (see UX below).

This is the lowest-risk, highest-reuse approach.

#### 2. Static Copy on Insert (Expand Inline) — Primary Mode
When a user inserts a Template into a Macro:
- The blocks are **copied** into the macro at that moment.
- The resulting macro becomes fully self-contained.
- No live reference, no runtime dependency on the template.

**Why this is the right default:**
- Deleting or editing a template never silently breaks existing macros.
- Debugging is trivial (what you see in the editor is exactly what runs).
- Matches how most users mentally treat “snippets”.

Optional advanced mode (can be added later): “Insert as Linked Template” that stores a reference. Keep it disabled or behind an “Advanced” toggle so the common path stays simple and safe.

#### 3. Parameterization — Keep It Lightweight
Do **not** invent a new slot/template-parameter system.

- Templates simply use the existing macro variable system (`{Username}`, `{Password}`, `{TargetX}`, etc.).
- When the template is expanded, any variables it contains remain as-is.
- The user can then set or override those variables in the parent macro’s variable list or via Set Variable blocks.

This gives 80 % of the value of parameterization with zero new engine complexity.

#### 4. Evolution of the “Call Macro” Block
Keep “Call Macro” strictly for full executable Macros (subroutines that have their own trigger context, scope, and return behavior).

Introduce a separate **“Insert Template”** action in the Block Palette (or a mode on the same block that switches based on type).  
- Choosing a Template → expands the blocks inline.  
- Choosing a Macro → inserts a Call Macro step that executes it as a subroutine.

This keeps the mental model clean:  
**Call** = run something else  
**Insert** = paste reusable logic into the current macro

#### 5. UI Recommendations
- In the main library / sidebar: two clear tabs or segmented control — **Macros** | **Templates**.
- Different icons (macro = play/rocket, template = puzzle piece or brackets).
- Templates appear in the Block Palette under a “Templates” category so users can drag them directly onto the canvas.
- On drag/drop or double-click: expand immediately and select the newly inserted blocks so the user can move or edit them.
- Right-click on a selection of blocks in any macro → “Save as Template…” (prompts for name + optional description).

#### 6. Nesting Rules (Simple & Safe)
- A Template **may** contain other Templates (they are expanded at insert time).
- A Template **may not** contain a Call Macro step that would create a circular reference at runtime (detect and block at save time).
- A Macro may freely Call other Macros and may contain expanded Templates.
- No runtime “template calling template” indirection — everything is flattened on insert.

This prevents the classic “template dependency hell” while still allowing composition.

#### 7. Deletion & Update Handling
- Deleting a Template is always safe because macros hold static copies.
- Optionally show a one-time warning: “This template was used in X macros (as a static copy). Deleting it will not affect them.”
- Provide a “Update from Template” command on a selection of blocks that were originally inserted from a template (store a hidden `SourceTemplateId` + version hash if desired). This is a nice-to-have, not required for v1.

#### Summary Recommendation (High-ROI Path)
1. Add `Type` field → unified storage.  
2. Templates are expanded as static copies.  
3. Reuse existing variables for parameterization.  
4. Keep “Call Macro” pure; add “Insert Template” (or dual-mode block).  
5. Separate tabs + Block Palette category for discovery.  
6. Flat expansion on insert (no live references, limited nesting).  

This delivers the requested capability with minimal new architecture, zero risk of cascading breakage, and a clear mental model for users.
---

### Agent 4: Claude

#### 1. Storage: Unified Model, Differentiated Schema
Same database, same model base class. The `Type` discriminator (`Macro` vs `Template`) is the clean middle ground — avoids dual-file complexity (Idea A's con) while keeping separation clearer than a single boolean flag.

Key schema insight: **Templates and Macros share Blocks but diverge in metadata.** A Template doesn't need Triggers, Schedules, or Scope — but it gains `Category`, `Parameters` (optional schema), and `Preview` artifacts. Use a single table with nullable columns rather than separate tables; it keeps queries simple and migrations minimal.

#### 2. Insert Behavior: Static Copy with Optional Source Tracking
Default behavior: **Expand Inline** (static copy). This is the safest, most intuitive default for 90% of users.

However, store a lightweight `SourceTemplateId` + `SourceTemplateVersion` on the inserted block group (hidden metadata, not visible in the editor). This enables a future "Update from Template" command without the complexity of live references. It's a **forward-compatibility trick** — zero cost now, unlocks linked behavior later if needed.

#### 3. Parameterization: Auto-Detect, Optional Override
No new variable syntax. When saving a block selection as a template, scan the blocks for existing macro variables (`{Var1}`, `{Username}`, etc.) and prompt:
> "These variables were found in your selection. Name them for easier reuse?"

This auto-discovers parameters without requiring the user to learn a slot system. If no variables exist, the template is saved as-is (simple). If variables exist, they become named parameters in the template UI.

#### 4. Block Palette Integration (No New Block Type)
Don't add a new "Insert Template" block. Instead:
- Templates appear as **collapsible items in the Block Palette sidebar** under a "Templates" header (with a filter/search).
- **Drag a template onto the canvas** → expands inline (static copy).
- **Double-click a template** → shows a parameter fill-in dialog (if parameters exist), then expands.
- The existing **"Call Macro" block** stays unchanged — it calls full Macros only. Templates aren't callable at runtime; they're design-time composition tools.

This keeps the mental model crisp: Macros are runtime units, Templates are design-time snippets.

#### 5. Nesting: Allowed at Design Time, Flattened on Insert
A template can contain blocks that were themselves expanded from other templates. This happens naturally during authoring — no special nesting engine needed. At insert time, we flatten recursively so the resulting macro has no template dependencies.

- **Circular detection**: When saving a selection as a template, check if the selection references itself via Call Macro. Block at save time with a clear message.
- **No runtime nesting**: Templates are not interpreted at runtime; they don't exist after insertion. This sidesteps the entire recursion/dependency problem.

#### 6. Deletion: Safe by Default, Informative Always
Since templates produce static copies, deletion is always safe. But show a lightweight dialog:
> "This template was used in 12 macros (last use: 3 days ago). Delete anyway?"
> [Delete] [Cancel]

The usage count comes from a lightweight index (scan `SourceTemplateId` metadata on block groups). No complex reference tracking needed.

#### 7. What I'd Cut for v1 (Over-Engineering Detected)
The following ideas from other agents are **too heavy for v1**:
- **Template Marketplace / Import-Export** (GPT §10) — future concern, not core
- **Template Designer Mode** (GPT §11) — nice, but the regular block editor suffices
- **Automation Assets unification** (GPT §13) — philosophically clean but massive refactor
- **Template Versioning with migration prompts** (GPT §3) — unnecessary since we use static copies
- **Linked Template mode** (GPT §2, Grok §2) — adds reference complexity; defer to post-v1

#### 8. Summary: The v1 MVP

| Decision | Choice |
|---|---|
| Storage | Unified model, `Type` discriminator, single table |
| Insert | Static copy (Expand Inline) |
| Source tracking | Hidden `SourceTemplateId` + version (for future use) |
| Parameters | Auto-detect existing variables; no new syntax |
| Block Palette | Templates category, drag-to-expand |
| Call Macro | Unchanged — macros only |
| Nesting | Allowed, flattened on insert, circular detection |
| Deletion | Safe, with usage count info dialog |
| Cut for v1 | Marketplace, Designer Mode, Linked mode, Versioning, Automation Assets |

This ships the feature in weeks, not months, with a clear upgrade path to advanced features later.

---

## 📋 Summary for User

After all agents respond, the Manager Agent will:
1. Filter out over-engineered suggestions (complex DSL, heavy abstractions)
2. Keep simple, high-ROI ideas
3. Present clean list to user
4. Log shortlisted items to `Obsidian Vault/ideas/ideas.md`

