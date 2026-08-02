# 🚀 T-Drive — Master Project Context & Specification Guide

> **Note for AI Agents**: Read this document completely to understand T-Drive's architecture, backend Telegram storage engine, frontend design system, completed features, and technical constraints before making any changes.

---

## 1. Project Overview & Tech Stack
**T-Drive** is a modern, high-performance web-based desktop cloud storage application built on top of Telegram's unlimited cloud infrastructure.

* **Backend**: Node.js | Express 5 | Telegram MTProto (`telegram` / GramJS SDK) | Multer | Archiver | bcryptjs
* **Frontend**: HTML5 | Vanilla CSS3 (Custom Design System tokens) | Vanilla ES6+ JavaScript
* **Protocols**: REST API | WebDAV (`/webdav` endpoint for native Windows `Z:\` Drive mounting)
* **Data Store**: Telegram Private Channel (Message IDs + Custom Metadata Captions)

---

## 2. Directory Structure & Key Files
```
TDrive/
├── server.js               ← Main Express server, WebDAV engine, GramJS Telegram client, chunking & API routes
├── package.json            ← Dependencies (express, telegram, multer, archiver, bcryptjs, cors, dotenv)
├── session.txt             ← StringSession storage for Telegram authentication
└── public/
    ├── index.html          ← Web app structure, modals, topbar, sidebar, command palette
    ├── style.css           ← Sober Slate Dark (#0F172A) design system, glassmorphism tokens, grid & list layouts
    └── app.js              ← Core frontend logic, preview lightbox, selection bar, keyboard shortcuts, API calls
```

---

## 3. Backend Engine Architecture & Telegram Storage Protocol

### A. Channel Resolution & Authentication
* Telegram connection is managed via GramJS (`TelegramClient` + `StringSession`).
* Storage channel entity is cached in `tgState.entity`.
* Server endpoints require session authentication (`requireAuth` middleware).

### B. Virtual Folder Engine (`#TDRIVE_FOLDER:`)
* Folders are lightweight virtual objects stored as Telegram text messages with caption `#TDRIVE_FOLDER:{"isFolder":true,"name":"...","parentPath":"..."}`.
* Folder tree navigation is computed dynamically via `parentPath` string matching.

### C. Large File Auto-Chunking Engine (`#TDRIVE_CHUNK:`)
* Files **<= 1.9GB** are uploaded directly as native Telegram documents with 4 parallel upload workers (`workers: 4`).
* Files **> 1.9GB** are automatically split into ~1.9GB chunk parts with metadata caption `#TDRIVE_CHUNK:{"fileId":"...","part":0,"totalParts":...}` and uploaded in parallel streams.
* On download/stream, chunks are re-assembled on the fly into a continuous stream.

### D. Bulk `.ZIP` Download Engine (`GET /api/download-bulk?ids=1,2,3`)
* Uses `archiver` streaming to pack multiple selected files/photos into a downloadable `.zip` archive on the fly with **0 RAM overhead**.

### E. WebDAV & Windows `Z:\` Virtual Drive (`/webdav`)
* Express WebDAV route supports `PROPFIND`, `GET`, `HEAD`, `OPTIONS`.
* 1-Click `net use Z: http://localhost:3000/webdav /persistent:yes` integration mounts T-Drive as a native Windows Drive letter (`Z:\`) in File Explorer.

---

## 4. Frontend UI/UX Architecture & Design System

### A. Design System Tokens (`public/style.css`)
* **Background**: Deep Slate (`#08090E` / `#0F172A`)
* **Accents**: Indigo (`--accent`), Cyan (`--cyan`), Amber (`--amber`), Red (`--red`)
* **Aesthetics**: Glassmorphism cards, sober typography, flat subtle box-shadows (`0 4px 16px rgba(0,0,0,0.35)`).

### B. View Modes & Interactive Layouts
* **Grid View**: Responsive card grid with hover highlights.
* **List View**: Ultra-compact row layout with clickable column headers (`Name`, `Size`, `Date` sorting) and a dedicated checkbox column on the far left.

### C. Lightbox Preview & Media Controls
* Supports Images, Videos, Audio, PDFs, and fallback documents.
* **Arrow Key Navigation (`←` / `→`)**: Instantly flips between media files inside current directory.
* **Spacebar Quick-Look**: Toggles full-screen preview on selected file.
* **Video Subtitles & Speed**: Load external `.srt` or `.vtt` subtitle files, with playback speed selector (`0.5x` to `2.0x`).

### D. Multi-Selection & Floating Action Bar
* Checkboxes on file cards enable multi-selection.
* Floating bottom selection bar provides `Select All`, `Bulk Delete`, and `Download as .ZIP`.

### E. Custom Modals & Command Palette (`Ctrl+K`)
* **New Folder Modal** (`#folderModal`)
* **Rename File Modal** (`#renameModal`)
* **Delete Confirmation Modal** (`#deleteModal`)
* **Share Link Modal** (`#shareModal`) with expiration options (`24 Hours`, `7 Days`, `Never`)
* **Folder PIN Lock Modal** (`#pinModal`) with 4-digit PIN verification
* **Command Palette** (`Ctrl+K`) for quick keyboard shortcuts

---

## 5. Completed Feature History

### Batch 1 (Completed)
- [x] Arrow Key Lightbox Navigation (`←` / `→`) & Spacebar Quick-Look
- [x] Multi-File Checkbox Selection & Floating `.ZIP` Download Bar
- [x] Drag-and-Drop File Moving into Folders
- [x] Parallel Multi-Worker Upload Speed Indicator
- [x] Quick Category Filter Pills (`All`, `Photos & Videos`, `Documents`, `Archives`)

### Batch 2 (Completed)
- [x] Starred / Favorite Files System (`#nav-starred` + 3-dots menu option)
- [x] Private Folder PIN Lock (`#pinModal` + 4-digit bcrypt verification)
- [x] Soft-Delete Trash Bin (`#nav-trash` + 1-click `Restore` / `Permanent Delete`)
- [x] Advanced Date & File Size Range Search Filters
- [x] Expiring Public Share Links Modal (`#shareModal`)

### Batch 3 (Completed)
- [x] Video Subtitle Support (`.srt` / `.vtt`) & Playback Speed Controls (`0.5x` - `2.0x`)
- [x] Windows Virtual Drive Mount (`Z:\` Drive via WebDAV)

---

## 6. Product Backlog (Future Features)
1. 📥 **Request File Upload Link (Guest Upload Drop Box)** *(Highest Priority Backlog)*
2. 🖼️ **Image EXIF Metadata Viewer & Image Editor**
3. 🏷️ **Multi-Tag System & Tag Manager**
4. 📜 **File Notes & Sticky Annotations**
5. 📂 **Folder Templates & Reusable Structures**
6. 📊 **Storage Analytics & Distribution Dashboard**
7. ⚡ **Command-Line Interface (`tdrive` CLI)**

---

## 7. Developer Rules & Verification Guidelines
1. **Communication**: Keep responses concise, friendly, and in English. Avoid corporate jargon.
2. **UI/UX Standard**: Keep designs professional, sleek, and sober. Avoid flashy over-the-top animations.
3. **Verification**: Always run `node -c server.js` to verify JavaScript syntax before finalizing changes.
4. **Fixing**: Fix minor bugs silently and state results clearly.

---

## 8. Core Agent Guidelines & Research Protocol
1. 🌐 **Cross-Check with Open-Source Projects**: Before implementing complex features or guessing solutions, inspect established open-source cloud storage projects (e.g. Telegram WebDAV, Telegram cloud storage engines, open-source file managers) to leverage proven patterns.
2. 🔍 **Web & Documentation Research First**: If a bug, API signature, or technical requirement is ambiguous or unknown, perform active web research and documentation lookups instead of making blind assumptions or trial-and-error edits.
3. 🛑 **Zero-Guessing Rule**: Never infer variable names, API schemas, or file paths without inspecting the authoritative source files or empirical logs first.
