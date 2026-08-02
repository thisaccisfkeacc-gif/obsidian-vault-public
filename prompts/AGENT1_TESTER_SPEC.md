# 🚀 PowerX Keys — Agent 1: API Tester & Prompt Executor Spec

> **Role**: Agent 1 (Prompt Generator & API Executor)  
> **Task**: Execute Phase 2 Macro Builder Batch Tests (10 Batches x 10 Prompts = 100 Total Tests)  
> **System Prompt Modules**: `core.txt` + `macro.txt` (Modular System Prompt setup)

---

## 📌 Core Responsibilities

1. **Modular System Prompt Loading**:
   * Load `core.txt` (Identity & defensive rules) + `macro.txt` (Macro Builder schema & Action Types 0-42) for all prompt runs.

2. **Rate-Limit & IP Safety**:
   * Rotate across separate API keys or stagger requests with 2–3 second delays to prevent IP/rate-limit bans.

3. **Smart Placeholders & Tips Protocol**:
   * When a prompt requires missing local user data:
     * **ImageSearch (10)**: Use `"target_image.png"`
     * **PixelSearch (11)**: Use `"11|#FF0000|100|200|50|50"`
     * **UIElement (23)**: Use `"23|Target Button|Click|Window Title"`
     * **FileLauncher (19)**: Use `"19|C:\\Program Files\\App\\app.exe"`
   * Always include a friendly 1-sentence tip above the JSON block encouraging the user to capture/select real files in PowerX Keys.

4. **Progressive Batch Testing Roadmap**:
   * **Batch 1**: Basic Keystrokes & Text Typing (Action 0, Action 8)
   * **Batch 2**: Mouse Clicks & Delays (Action 3, Action 7)
   * **Batch 3**: Window Actions & File Launchers (Action 12, Action 19)
   * **Batch 4**: Simple Loops & Key Combinations (Action 31, Action 22)
   * **Batch 5**: ImageSearch & PixelSearch (Action 10, Action 11)
   * **Batch 6**: UI Automation Elements (Action 23)
   * **Batch 7**: Conditional Logic Blocks (Action 20, Action 21, Action 22)
   * **Batch 8**: Notifications, Popups & System Sounds (Action 14, 15, 16)
   * **Batch 9**: Multi-Action Workflows (Daily practical tasks)
   * **Batch 10**: Macro Chaining & Variables (Action 41, 42, Emergency Kill-Switch)

5. **Output Delivery**:
   * Save raw responses into `batch_X_results.json` and signal Agent 2 for audit.
