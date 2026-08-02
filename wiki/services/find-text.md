---
tags: [service, image-search, pattern-matching, findtext, ahk]
date: 2026-05-23
sources:
  - Services/FindTextService.cs
  - Scripts/FindText.ahk
status: complete
---

# Find Text (Image Search)

A dual-layer image search system consisting of a **C# encoder** (`FindTextService`) and an **AHK pattern matcher** (`FindText.ahk`). Based on [FeiYue's FindText v10.2](https://www.autohotkey.com/boards/viewtopic.php?f=83&t=116471) — a high-performance screen-to-text pattern recognition library.

## Purpose

- Captures screen regions and converts them into searchable text patterns
- Enables the "Fast Engine" mode for `ImageSearch` macro steps
- Three encoding modes: Gray Threshold, Color Match, Gray Difference (edge detection)
- Used by [[script-compiler]] to generate AHK `FindText()` calls

## Architecture

```mermaid
graph LR
    A["Screen Capture"] --> B["FindTextService (C#)"]
    B -->|Base64 pattern string| C["ScriptCompilerService"]
    C --> D["FindText.ahk (AHK)"]
    D --> E["Screen Match Result"]
```

## C# Encoder — FindTextService

### Encoding Modes

| Mode | Format | Description |
|------|--------|-------------|
| **Gray** (default) | `*threshold$w.base64` | Dynamic threshold: `(maxGray + minGray) / 2`. Pixel is "1" if gray ≤ threshold |
| **Color** | `hexColor-tolerance$w.base64` | Matches pixels within ±10 RGB of target color |
| **GrayDiff** | `**diff$w.base64` | Edge detection — pixel is "1" if any neighbor differs by > `diff` in gray value |

### Gray Calculation

Uses weighted formula matching AHK's internal logic:
```
gray = (R * 38 + G * 75 + B * 15) >> 7
```

### Base64 Encoding

FeiYue's custom Base64 character set (NOT standard Base64):
```
0123456789+/ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz
```

- Bits are packed MSB-first into 6-bit chunks
- Terminator: appends `1` followed by `0`s to reach multiple of 6
- AHK regex `10*$` strips the terminator on decode

## AHK Matcher — FindText.ahk

The `FindText()` function (3663 lines) searches the screen for patterns:

### Search Modes

| Mode | ID | Description |
|------|-----|-------------|
| Color | 1 | Match specific colors with tolerance |
| Gray Threshold | 2 | Binary threshold on grayscale |
| Gray Difference | 3 | Edge detection via neighbor comparison |
| Color Position | 4 | Multi-colored verification code recognition |
| Multi/Shape/Pic | 5 | FindMultiColor, FindShape, FindPic |

### Machine Code Acceleration

The core search uses **compiled C code** embedded as base64 machine code (x86 + x64):
- `PicFind()` function compiled with `gcc -O2`
- Loaded via `DllCall()` directly from memory
- Parameters include: bitmap data, stride, search bounds, pattern data, error tolerance

### Search Directions

9 directional scanning strategies:
1. Left→Right, Top→Bottom
2. Right→Left, Top→Bottom
3. Left→Right, Bottom→Top
4. Right→Left, Bottom→Top
5. Top→Bottom, Left→Right
6. Bottom→Top, Left→Right
7. Top→Bottom, Right→Left
8. Bottom→Top, Right→Left
9. **Center outward** (spiral)

### Error Tolerance

- `err1` — fault tolerance for text/foreground pixels (0.1 = 10%)
- `err0` — fault tolerance for background pixels
- Negative values enable left/right dilation for misaligned text

### Wait Mode

Built-in polling with `OutputX := "wait"`:
- `wait` / `wait1` — wait for image to **appear**
- `wait0` — wait for image to **disappear**
- Configurable timeout + stable time

## Key Methods

### FindTextService (C#)

| Method | Description |
|--------|-------------|
| `GetFindTextCode(Bitmap, mode, hexColor)` | Converts bitmap to FindText pattern string |
| `BitToFindTextBase64(bits)` | Encodes binary string to FeiYue's custom Base64 |

### FindText.ahk

| Method | Description |
|--------|-------------|
| `FindText()` | Main search function — returns array of match objects `{x, y, w, h, id}` |
| `PicFind()` | Machine code accelerated pattern matching |
| `GetBitsFromScreen()` | Captures screen bitmap for searching |
| `JoinText()` | Combines multiple patterns for sequential text matching |

## Key Files

- [FindTextService.cs](file:///c:/Users/Maaz/Documents/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX_Keys_V2/Services/FindTextService.cs) — 140 lines
- [FindText.ahk](file:///c:/Users/Maaz/Documents/PowerX%20Keys/PowerX_Keys_V2_Rebuild/PowerX_Keys_V2/Scripts/FindText.ahk) — 3663 lines, 130KB

## Related Pages

- [[script-compiler]]
- [[macro-execution]]
- [[ui-element-capture]]
