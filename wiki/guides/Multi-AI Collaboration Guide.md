# 🤖 Multi-AI Collaboration & Brainstorming Guide (Master SOP)

> **For All AI Agents (Antigravity, Kiro, etc.)**: This document defines the mandatory multi-agent collaboration, feature brainstorming, prompt generation, and noise filtering protocol for PowerX Keys. Read this document whenever asked to brainstorm features or create prompts for other agents!

---

## 💡 System Concept & Roles

The goal is to get diverse, high-quality architectural viewpoints on any topic or feature by collaborating with multiple AI models (Antigravity, Kiro, Claude, DeepSeek, Gemini, etc.).

### Roles:
- **Manager Agent (Primary AI)**: Gathers technical context from codebase, writes the structured prompt file, synthesizes agent responses, filters out over-engineered noise, and presents a clean list to the user.
- **Specialist Agents (Peer AIs)**: Read the prompt file, analyze calmly and deeply, and record their feedback and creative ideas directly into their designated section inside the prompt file.

---

## 🔄 The 5-Step Brainstorming & Prompt Workflow

### Step 1: Create the Technical Prompt File
When the user asks to brainstorm or generate a prompt for a feature (e.g. Image Block, Pixel Block, UIElement Capture, Window Block):
1. Search the codebase for the feature's exact code implementation (`ScriptCompilerService.cs`, ViewModels, Services).
2. Create a new prompt file in `Obsidian Vault/prompts/<feature_name>_brainstorm_prompt.md`.

The prompt file **MUST** contain:
- **Feature Overview**: What the feature does and why it exists in PowerX Keys.
- **Technical Implementation**: How it compiles/executes in C#, WPF, and AutoHotkey v2 (include actual code snippets).
- **Properties & Scope System**: Full list of model properties and configuration settings.
- **Pain Points & Known Limitations**: Clear list of current flaws (e.g., dynamic titles, DPI scaling, false matches).
- **Seed Ideas**: A few initial ideas to prime agent thinking.
- **Agent Response Sections**: Blank Markdown headers (`### Agent 1`, `### Agent 2`, `### Agent 3`).

---

### Step 2: Multi-Agent Review & Writing
1. The user forwards/copy-pastes the prompt file to other agents (Kiro, Antigravity, etc.).
2. Each specialist agent reads the technical context and writes their assessment and suggestions **directly into their designated section** (`### Agent 1`, `### Agent 2`, etc.) inside the prompt file.

---

### Step 3: Synthesis & Filtering (Removing Noise)
The Manager Agent reads all agent responses from the prompt file and performs **strict filtering**:

- ❌ **Filter Out (Nonsense / Over-Engineered)**:
  - Complex Machine Learning / ONNX models / OpenCV dependencies.
  - Unnecessary UI bloat (e.g. 12-factor scoring engines, DevTools inspectors, complex dropdowns).
  - Impractical or fragile suggestions (e.g. full UIA tree hashing, title bar screenshot comparisons).

- ✅ **Keep Only**:
  - Simple, lightweight, high-ROI improvements.
  - Ideas that fix real pain points without adding heavy dependencies.
  - Smart, user-centric concepts (e.g. "Fix It Now" runtime repair, mini-cluster matching, HSL hue locking).

---

### Step 4: Present Filtered List to User
Present the clean, filtered ideas to the user in chat using plain English, bullet points, and emojis. 😊🚀
Group logically by category (e.g., Matching Reliability, Capture UX, New Actions).

---

### Step 5: User Selection & Backlog Logging
The user reviews the filtered list, provides feedback/doubts, and chooses which ideas to shortlist.

The Manager Agent logs the shortlisted items into `Obsidian Vault/ideas/ideas.md` under **Section 5 (Recent Session Backlog & Design Ideas)** with explicit notes:
- Add user-requested notes (e.g. *"Discuss pros/cons before implementing"*, *"Has user doubts — evaluate further"*).
- **Golden Rule**: Never implement shortlisted ideas immediately — always wait for the user to explicitly request implementation in a future task.

---

## 📁 Active Prompt Files Registry (`Obsidian Vault/prompts/`)

| File | Feature / Topic | Status |
|---|---|---|
| [`uielement_block_brainstorm_prompt.md`](file:///c:/Users/Maaz/Documents/New%20folder/Obsidian%20Vault/prompts/uielement_block_brainstorm_prompt.md) | UIElement Block & UIA | Completed |
| [`image_block_brainstorm_prompt.md`](file:///c:/Users/Maaz/Documents/New%20folder/Obsidian%20Vault/prompts/image_block_brainstorm_prompt.md) | Image Block & 3-Tier Cascade | Completed |
| [`pixel_block_brainstorm_prompt.md`](file:///c:/Users/Maaz/Documents/New%20folder/Obsidian%20Vault/prompts/pixel_block_brainstorm_prompt.md) | Pixel Block & Color Matching | Completed |
| [`window_block_brainstorm_prompt.md`](file:///c:/Users/Maaz/Documents/New%20folder/Obsidian%20Vault/prompts/window_block_brainstorm_prompt.md) | Window Block & Activation Cascade | Completed |
| `dynamic_repair_brainstorm_prompt.md` | "Fix It Now" Dynamic Runtime Repair | Pending creation when requested |

---

## 🤖 Instructions for Future Session Agents

When a user in any future session says:
- *"Create a prompt for [feature] to ask other agents"*
- *"Get ideas from other agents for [feature]"*
- *"Brainstorm [feature]"*

**DO THIS IMMEDIATELY**:
1. Read this master guide (`Obsidian Vault/wiki/guides/Multi-AI Collaboration Guide.md`).
2. Search the codebase for the feature's exact code path (`ScriptCompilerService.cs`, ViewModels, Services).
3. Generate the structured prompt file in `Obsidian Vault/prompts/`.
4. Follow the 5-step execution workflow strictly.
