---
tags: [feature, image-recognition, findtext, pixel-search]
date: 2026-05-23
sources:
  - Services/FindTextService.cs
  - Services/ScriptCompilerService.cs
status: current
---

# Image Recognition 🔍

PowerX Keys includes a built-in visual matching system that lets macros find and click on screen elements by their appearance, not fixed coordinates. This uses the **FindText** algorithm (a port of Feiyue's AHK library).

## Overview

Three visual search modes are available as macro step types:

| Mode | What It Does |
|------|-------------|
| **ImageSearch** | Matches a captured bitmap region on screen |
| **PixelSearch** | Finds a single pixel of a specific color |
| **FindText** | Converts a bitmap to a binary pattern string for fast matching |

## FindText Engine

`FindTextService.cs` is a C# implementation that converts captured bitmaps into FindText-compatible pattern strings. The generated strings are embedded directly into compiled AHK scripts.

### Three Matching Modes

**1. Gray Mode (default)**
- Converts the image to grayscale
- Calculates dynamic threshold: `min + (max - min) / 2`
- Pixels below threshold → `1`, above → `0`
- Output format: `|<CapturedIcon>*{threshold}${width}.{base64}`

**2. Color Mode**
- Matches pixels against a specific hex color within a tolerance of ±10
- Useful for finding colored buttons or indicators
- Output format: `|<CapturedIcon>{hexColor}-{tolerance}${width}.{base64}`

**3. GrayDiff Mode (edge detection)**
- Compares each pixel's gray value against its 8 neighbors
- If any neighbor differs by more than 50, it's an edge → `1`
- Best for finding text or icons regardless of background color
- Output format: `|<CapturedIcon>**{diff}${width}.{base64}`

### Base64 Encoding

The `BitToFindTextBase64()` method converts binary strings to FindText's custom Base64:
- Character set: `0123456789+/ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz`
- Appends a `1` terminator + padding zeros to reach a multiple of 6 bits
- This allows AHK's regex `10*$` to strip the terminator cleanly

## Capture Workflow

1. User opens the **Capture Overlay** from the macro editor
2. Two-stage capture: first click anchors, second click defines the region
3. Smart Window Snapping: one-click capture of window bounds
4. Captured image is saved to `%LOCALAPPDATA%/PowerXKeys/Engine/Images/`
5. `FindTextService.GetFindTextCode()` generates the pattern string
6. Pattern is stored in `MacroStep.FindTextCode`

## Search Scopes

Macros can search within:
- **Full Screen**: Entire desktop
- **Custom Area**: User-defined rectangle
- **Window Area**: Static coordinates from capture
- **Window Live** (`WIN_LIVE:` prefix): Dynamic coordinates resolved at runtime via AHK `WinGetPos`

## Compilation

`ScriptCompilerService` generates AHK code that:
1. Calls `FindText()` with the stored pattern
2. If found: moves cursor to center of match, clicks
3. Supports "Click on Found Target" with bulletproof `MouseClick` logic
4. Auto-Detect mode: runs continuously in a background loop

## Key Files

- [FindTextService.cs](file:///c:/Users/Maaz/Documents/PowerX Keys/PowerX_Keys_V2_Rebuild/PowerX_Keys_V2/Services/FindTextService.cs)
- [ScriptCompilerService.cs](file:///c:/Users/Maaz/Documents/PowerX Keys/PowerX_Keys_V2_Rebuild/PowerX_Keys_V2/Services/ScriptCompilerService.cs)

## Related Pages

- [[macro-editor]]
- [[execution-pipeline]]
- [[macro-item]]
