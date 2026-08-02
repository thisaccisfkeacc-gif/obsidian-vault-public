# Block Audit: Keyboard (Deep — Cross-Path)

**Date:** 2026-07-24
**Scope:** C# in-process, AHK compiled, AHK preview (SingleStep)

---

## Summary

Keyboard block handles single keys, combos (Ctrl+C), and Hold/Release states. Compilation uses `{Blind}` mode for single keys and modifier prefix notation for combos.

---

## BUGS

### 1. C# single-key Press broken for special keys
- **Severity:** High
- **Location:** MacroExecutionService.cs:1381-1384
- **Problem:** `SafeSendKeys(step.Value)` passes raw AHK key names (e.g., `"Enter"`, `"Space"`) to .NET `SendKeys.SendWait()`. But .NET expects `"{ENTER}"`, `"{SPACE}"` notation. `SendKeys.SendWait("Enter")` types the literal characters "E-n-t-e-a" instead of pressing Enter. Only affects non-combo, non-Hold/Release single special keys in C# path.

### 2. AHK Sandbox mode doesn't handle combo Hold/Release
- **Severity:** Medium
- **Location:** ScriptCompilerService.cs:4885-4898
- **Problem:** Sandbox keyboard compiler (used for Custom Keystrokes hotkey tests) wraps entire `Value` string for Hold/Release without splitting combos. Generates invalid AHK syntax. Main compile path correctly uses `ConvertComboHoldReleaseToAhk`.

### 3. AHK Sandbox missing `{Blind}` for single keys
- **Severity:** Low
- **Location:** ScriptCompilerService.cs:4879-4881
- **Problem:** Sandbox uses `Send("{key}")` without `{Blind}`. Main path uses `Send("{Blind}" . "{key}")`. Without `{Blind}`, AHK may interfere with modifier state.

---

## DEAD CODE

1. `IsKeyCombo` property — hardcoded to `false`. XAML DataTriggers for it never fire.
2. `OnPropertyChanged(nameof(IsKeyCombo))` in Value setter — wasted notification.
3. XAML DataTriggers for `IsKeyCombo` — dead markup.

---

## REDUNDANCIES

1. Sandbox keyboard compilation duplicates main compile logic with different behavior. Should reuse main path.
2. `NumpadEnter` extended-key detection duplicated — string check redundant with VK code lookup.

---

## MISSING FEATURES

| Feature | C# | AHK Full | AHK Preview |
|---|---|---|---|
| Single key press | ⚠️ (broken for special) | ✅ | ❌ (uses C# path) |
| Combo press | ✅ | ✅ | ❌ (uses C# path) |
| Hold/Release | ✅ | ✅ | ❌ (uses C# path) |
| `{Blind}` mode | N/A | ✅ | N/A |
| Sandbox combo Hold/Release | N/A | ❌ | N/A |

---

## INCONSISTENCIES

1. `{Blind}` usage — AHK Full: yes. C#: no (uses `keybd_event`). AHK Sandbox: no. Three behaviors for same key press.
2. Combo parsing — AHK Full uses `SplitCombo`. C# splits on `" + "`. AHK Sandbox doesn't split.
3. Release order — Both C# and AHK release in reverse order. Consistent.
4. Single letter case — Recording lowercases. C# lowercases. AHK doesn't. Minor inconsistency.
5. NumpadEnter — C# treats as extended Enter (same VK). AHK treats as separate key name.

---

## VERIFIED OK

- Tri-state cycle (Press/Hold/Release) — correct across all paths
- Recording captures — consistent key names
- Combo modifier ordering — consistent
- DisplayValue formatting — correct
- IsValid — correctly rejects "None" sentinel
