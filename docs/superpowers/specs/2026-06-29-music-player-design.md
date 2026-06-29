# Music Set Player — Design Spec

**Date:** 2026-06-29  
**Status:** Approved

---

## 1. Overview

A single-file web music player for local audio file sets with user-selectable cover art. Open the HTML file in any browser and start playing — no server, no build step, no dependencies.

## 2. Requirements

- **Audio source:** Multiple local audio files imported as one set, played in sequence
- **Cover art:** User clicks cover area to select an image; also accepts drag-and-drop
- **Controls:** Play/pause, seekable progress bar, volume slider, loop toggle
- **Persistence:** Remember volume and loop state via LocalStorage
- **Visual:** Dark purple theme, vertical layout, cover-first
- **Tech:** Single HTML file, vanilla JS, zero dependencies

## 3. Architecture

```
music-player.html
├── <style>          — All CSS, embedded
├── <body>           — DOM structure
│   ├── .cover-area  — Clickable/droppable cover image zone
│   ├── .info-bar    — Set name + current track name
│   ├── .progress    — Seekable progress bar
│   ├── .controls    — Play/pause button + time display
│   ├── .volume      — Volume slider
│   ├── .loop-toggle — Loop on/off
│   └── .drop-zone   — Audio file import area
└── <script>         — All JS, embedded
    ├── AudioEngine  — <audio> element wrapper
    ├── Playlist     — Array of file objects, index pointer
    ├── CoverManager — Cover image set/replace via FileReader
    ├── UI           — DOM bindings, event handlers
    └── Store        — LocalStorage get/set for settings
```

## 4. Data Flow

```
User drops files → Playlist builds array → AudioEngine loads track[0]
User clicks play → AudioEngine.play() → UI updates time/progress
Track ends → Playlist.next() → AudioEngine.load(next) → auto-play
User clicks cover → CoverManager.pick() → FileReader → <img> src update
Volume/loop change → Store.save() → AudioEngine.apply()
Page load → Store.load() → AudioEngine.apply() → restore state
```

## 5. Components

### 5.1 AudioEngine
- Wraps a single `<audio>` element
- API: `load(file)`, `play()`, `pause()`, `seek(seconds)`, `setVolume(0-1)`, `setLoop(bool)`
- Emits: `timeupdate`, `ended`, `loadedmetadata`

### 5.2 Playlist
- `files: File[]` — imported audio files in order
- `index: number` — current track position
- API: `add(files)`, `remove(index)`, `current()`, `next()`, `prev()`, `reorder()`
- On `ended`: auto-increment index, load next track, play

### 5.3 CoverManager
- `currentCover: dataURL | null`
- API: `setFromFile(file)` — FileReader → dataURL → set on `<img>`
- Accepts click (file input) and drop (File API)

### 5.4 Store
- Keys: `volume`, `loop`
- `load()` returns defaults if nothing stored
- `save(key, value)` on every change

### 5.5 UI
- Progress bar: range input styled as slider, syncs with `audio.currentTime`
- Time display: `MM:SS / MM:SS` format
- Play/pause: single button, toggles icon
- Volume: range input
- Loop: toggle button with active state
- Drop zone: visible when playlist is empty, hidden when tracks loaded

## 6. Error Handling

| Scenario | Behavior |
|----------|----------|
| Unsupported audio format | Skip file, show toast notification |
| File read error | Show error on cover/audio import |
| Empty playlist + play | Button disabled, no-op |
| Browser lacks FileReader | Show fallback message |
| LocalStorage unavailable | Graceful degradation, no persistence |

## 7. Visual Spec

- Background: `#0d1117`
- Accent: `#7c3aed` (purple)
- Surface: `#161b22`
- Text primary: `#e6edf3`
- Text secondary: `#8b949e`
- Border radius: 12px (cover), 8px (buttons), 6px (inputs)
- Font: system-ui, sans-serif
- Layout: max-width 420px, centered, vertical stack
- Transitions: 200ms ease on buttons, hover states

## 8. Browser Compatibility

- Chrome 90+, Firefox 90+, Edge 90+, Safari 15+
- No IE11 support

## 9. Out of Scope

- No playlist persistence across sessions (LocalStorage size limits for file data)
- No audio visualizer / spectrum
- No metadata extraction (ID3 tags, etc.)
- No mobile PWA / offline service worker
- No keyboard shortcuts in v1

## 10. Future Extension Points

- Keyboard shortcuts (space for play/pause, arrows for seek)
- Drag-to-reorder playlist
- Multiple playlists / tabs
- Export/import playlist as JSON
