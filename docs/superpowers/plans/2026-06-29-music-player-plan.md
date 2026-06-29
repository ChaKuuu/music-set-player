# Music Set Player — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a single-file HTML music player that plays local audio file sets with user-selectable cover art.

**Architecture:** Single `music-player.html` with embedded CSS and JS. Five JS modules — Store (LocalStorage), Playlist (track array), AudioEngine (`<audio>` wrapper), CoverManager (image FileReader), UI (DOM bindings) — wired together via event-driven data flow. Zero dependencies, no build step.

**Tech Stack:** HTML5, CSS3, vanilla JavaScript, FileReader API, `<audio>` element, LocalStorage

**Files:**
- Create: `music-player.html` (all code lives here)

---

### Task 1: HTML Shell + CSS Foundation

**Files:**
- Create: `music-player.html`

- [ ] **Step 1: Write the HTML shell with all DOM elements and CSS theme**

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Music Set Player</title>
<style>
  *, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

  :root {
    --bg: #0d1117;
    --surface: #161b22;
    --accent: #7c3aed;
    --accent-hover: #6d28d9;
    --text: #e6edf3;
    --text-secondary: #8b949e;
    --radius-lg: 12px;
    --radius-md: 8px;
    --radius-sm: 6px;
    --transition: 200ms ease;
  }

  body {
    font-family: system-ui, -apple-system, sans-serif;
    background: var(--bg);
    color: var(--text);
    min-height: 100vh;
    display: flex;
    align-items: center;
    justify-content: center;
  }

  .player {
    width: 100%;
    max-width: 420px;
    padding: 32px 24px;
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 20px;
  }

  /* Cover area */
  .cover-area {
    width: 260px;
    height: 260px;
    border-radius: var(--radius-lg);
    background: var(--surface);
    display: flex;
    align-items: center;
    justify-content: center;
    cursor: pointer;
    overflow: hidden;
    position: relative;
    transition: opacity var(--transition), box-shadow var(--transition);
  }
  .cover-area:hover { box-shadow: 0 0 24px rgba(124,58,237,0.25); }
  .cover-area img {
    width: 100%; height: 100%;
    object-fit: cover;
    display: block;
  }
  .cover-placeholder {
    color: var(--text-secondary);
    font-size: 48px;
    pointer-events: none;
  }

  /* Info bar */
  .info-bar {
    text-align: center;
    width: 100%;
  }
  .set-name {
    font-size: 18px;
    font-weight: 600;
    margin-bottom: 4px;
  }
  .track-name {
    font-size: 13px;
    color: var(--text-secondary);
  }

  /* Progress */
  .progress-container {
    width: 100%;
    display: flex;
    align-items: center;
    gap: 10px;
  }
  .time { font-size: 12px; color: var(--text-secondary); min-width: 42px; font-variant-numeric: tabular-nums; }
  .time.current { text-align: right; }
  .time.total { text-align: left; }
  .progress-bar {
    flex: 1;
    -webkit-appearance: none;
    appearance: none;
    height: 4px;
    background: #30363d;
    border-radius: 2px;
    cursor: pointer;
  }
  .progress-bar::-webkit-slider-thumb {
    -webkit-appearance: none;
    width: 14px; height: 14px;
    background: var(--accent);
    border-radius: 50%;
    cursor: pointer;
  }
  .progress-bar::-moz-range-thumb {
    width: 14px; height: 14px;
    background: var(--accent);
    border-radius: 50%;
    border: none;
    cursor: pointer;
  }

  /* Controls */
  .controls-row {
    display: flex;
    align-items: center;
    gap: 16px;
  }
  .btn-play {
    width: 56px; height: 56px;
    border-radius: 50%;
    background: var(--accent);
    color: white;
    border: none;
    font-size: 22px;
    cursor: pointer;
    display: flex;
    align-items: center;
    justify-content: center;
    transition: background var(--transition), transform var(--transition);
  }
  .btn-play:hover { background: var(--accent-hover); transform: scale(1.05); }
  .btn-play:active { transform: scale(0.95); }
  .btn-play:disabled { opacity: 0.4; cursor: not-allowed; transform: none; }

  /* Volume */
  .volume-row {
    width: 100%;
    display: flex;
    align-items: center;
    gap: 8px;
    justify-content: center;
  }
  .volume-icon { font-size: 16px; color: var(--text-secondary); }
  .volume-slider {
    width: 120px;
    -webkit-appearance: none;
    appearance: none;
    height: 3px;
    background: #30363d;
    border-radius: 2px;
    cursor: pointer;
  }
  .volume-slider::-webkit-slider-thumb {
    -webkit-appearance: none;
    width: 12px; height: 12px;
    background: var(--accent);
    border-radius: 50%;
    cursor: pointer;
  }
  .volume-slider::-moz-range-thumb {
    width: 12px; height: 12px;
    background: var(--accent);
    border-radius: 50%;
    border: none;
    cursor: pointer;
  }

  /* Loop toggle */
  .loop-row { width: 100%; display: flex; justify-content: center; }
  .btn-loop {
    background: transparent;
    border: 1px solid #30363d;
    color: var(--text-secondary);
    padding: 6px 18px;
    border-radius: var(--radius-md);
    font-size: 13px;
    cursor: pointer;
    transition: all var(--transition);
  }
  .btn-loop:hover { border-color: var(--text-secondary); }
  .btn-loop.active {
    border-color: var(--accent);
    color: var(--accent);
    background: rgba(124,58,237,0.1);
  }

  /* Drop zone */
  .drop-zone {
    width: 100%;
    padding: 32px 16px;
    border: 2px dashed #30363d;
    border-radius: var(--radius-lg);
    text-align: center;
    color: var(--text-secondary);
    font-size: 14px;
    cursor: pointer;
    transition: border-color var(--transition), background var(--transition);
  }
  .drop-zone:hover, .drop-zone.drag-over {
    border-color: var(--accent);
    background: rgba(124,58,237,0.05);
  }
  .drop-zone.hidden { display: none; }

  /* Toast */
  .toast {
    position: fixed;
    bottom: 24px;
    left: 50%;
    transform: translateX(-50%);
    background: var(--surface);
    color: var(--text);
    padding: 10px 20px;
    border-radius: var(--radius-md);
    font-size: 13px;
    border: 1px solid #30363d;
    opacity: 0;
    pointer-events: none;
    transition: opacity 300ms ease;
    z-index: 100;
  }
  .toast.show { opacity: 1; }

  /* Hidden file input */
  .file-input-hidden { display: none; }
</style>
</head>
<body>

<div class="player">
  <!-- Cover -->
  <div class="cover-area" id="coverArea" title="点击或拖放图片更换封面">
    <span class="cover-placeholder" id="coverPlaceholder">🎵</span>
    <img id="coverImg" src="" alt="Cover" style="display:none">
  </div>
  <input type="file" class="file-input-hidden" id="coverFileInput" accept="image/*">

  <!-- Info -->
  <div class="info-bar">
    <div class="set-name" id="setName">拖放音频文件到下方</div>
    <div class="track-name" id="trackName">尚无曲目</div>
  </div>

  <!-- Progress -->
  <div class="progress-container">
    <span class="time current" id="timeCurrent">00:00</span>
    <input type="range" class="progress-bar" id="progressBar" min="0" max="100" value="0" disabled>
    <span class="time total" id="timeTotal">00:00</span>
  </div>

  <!-- Play/Pause -->
  <div class="controls-row">
    <button class="btn-play" id="btnPlay" disabled title="播放">▶</button>
  </div>

  <!-- Volume -->
  <div class="volume-row">
    <span class="volume-icon">🔊</span>
    <input type="range" class="volume-slider" id="volumeSlider" min="0" max="100" value="80">
  </div>

  <!-- Loop -->
  <div class="loop-row">
    <button class="btn-loop" id="btnLoop" title="循环模式">🔄 关闭循环</button>
  </div>

  <!-- Drop Zone -->
  <div class="drop-zone" id="dropZone" title="拖放或点击导入音频文件">
    📁 拖放音频文件到此处<br><small>或点击选择文件 (MP3, WAV, OGG, FLAC)</small>
  </div>
  <input type="file" class="file-input-hidden" id="audioFileInput" accept="audio/*" multiple>
</div>

<div class="toast" id="toast"></div>

<script>
// All JS goes here — filled in subsequent tasks
</script>

</body>
</html>
```

- [ ] **Step 2: Open the file and verify**

Open `music-player.html` in browser. Verify:
- Dark background, centered player card
- Cover placeholder shows 🎵
- Drop zone visible with dashed border
- Play button is disabled (gray)
- Volume slider at 80%
- Loop button shows "关闭循环"
- Progress slider is disabled

- [ ] **Step 3: Commit**

```bash
git add music-player.html
git commit -m "feat: add HTML shell and CSS foundation for music player"
```

---

### Task 2: Store Module (LocalStorage)

**Files:**
- Modify: `music-player.html`

- [ ] **Step 1: Add the Store module inside the `<script>` tag**

Find `<script>` and replace with:

```html
<script>
/* ============================================================
   Store — LocalStorage wrapper with graceful degradation
   ============================================================ */
const Store = (() => {
  let available = false;
  const PREFIX = 'music-player:';
  const DEFAULTS = { volume: 0.8, loop: false };

  try {
    const testKey = PREFIX + '__test';
    localStorage.setItem(testKey, '1');
    localStorage.removeItem(testKey);
    available = true;
  } catch (_) { /* No-op: LocalStorage unavailable */ }

  return {
    get(key) {
      if (!available) return DEFAULTS[key];
      try {
        const raw = localStorage.getItem(PREFIX + key);
        return raw !== null ? JSON.parse(raw) : DEFAULTS[key];
      } catch (_) { return DEFAULTS[key]; }
    },

    set(key, value) {
      if (!available) return;
      try { localStorage.setItem(PREFIX + key, JSON.stringify(value)); }
      catch (_) { /* Storage full or unavailable */ }
    }
  };
})();
</script>
```

- [ ] **Step 2: Verify in browser DevTools**

Open `music-player.html`, open Console:
```js
Store.get('volume')   // Should return 0.8
Store.get('loop')     // Should return false
Store.set('volume', 0.5)
Store.get('volume')   // Should return 0.5
// Check Application > Local Storage for key "music-player:volume"
```

- [ ] **Step 3: Commit**

```bash
git add music-player.html
git commit -m "feat: add Store module — LocalStorage with graceful degradation"
```

---

### Task 3: AudioEngine Module

**Files:**
- Modify: `music-player.html`

- [ ] **Step 1: Add AudioEngine module after Store**

```js
/* ============================================================
   AudioEngine — wraps <audio> element
   ============================================================ */
const AudioEngine = (() => {
  const audio = new Audio();
  const listeners = {};
  let currentFile = null;

  function on(name, fn) {
    if (!listeners[name]) listeners[name] = [];
    listeners[name].push(fn);
  }

  function emit(name, data) {
    (listeners[name] || []).forEach(fn => fn(data));
  }

  audio.addEventListener('timeupdate', () => {
    if (audio.duration && isFinite(audio.duration)) {
      emit('timeupdate', { currentTime: audio.currentTime, duration: audio.duration });
    }
  });

  audio.addEventListener('ended', () => emit('ended'));
  audio.addEventListener('loadedmetadata', () => emit('loadedmetadata', { duration: audio.duration }));
  audio.addEventListener('error', () => emit('error', { file: currentFile }));

  function load(file) {
    if (currentFile && audio.src && audio.src.startsWith('blob:')) {
      URL.revokeObjectURL(audio.src);
    }
    currentFile = file;
    audio.src = URL.createObjectURL(file);
    audio.load();
  }

  return {
    load,
    play()  { const p = audio.play(); if (p) p.catch(() => {}); },
    pause() { audio.pause(); },
    seek(seconds) { if (isFinite(seconds)) audio.currentTime = seconds; },
    setVolume(v)   { audio.volume = Math.max(0, Math.min(1, v)); },
    setLoop(bool)  { audio.loop = bool; },
    getCurrentTime() { return audio.currentTime; },
    getDuration()    { return audio.duration || 0; },
    isPaused()       { return audio.paused; },
    on,
    destroy() {
      audio.pause();
      if (audio.src && audio.src.startsWith('blob:')) URL.revokeObjectURL(audio.src);
    }
  };
})();
```

- [ ] **Step 2: Verify in browser DevTools**

```js
// Should not throw, AudioEngine is defined
typeof AudioEngine.load    // 'function'
typeof AudioEngine.play    // 'function'
typeof AudioEngine.seek    // 'function'
AudioEngine.setVolume(0.5)
AudioEngine.setLoop(true)
```

- [ ] **Step 3: Commit**

```bash
git add music-player.html
git commit -m "feat: add AudioEngine module — <audio> element wrapper"
```

---

### Task 4: Playlist Module

**Files:**
- Modify: `music-player.html`

- [ ] **Step 1: Add Playlist module after AudioEngine**

```js
/* ============================================================
   Playlist — manages the audio file queue
   ============================================================ */
const Playlist = (() => {
  let files = [];
  let index = -1;  // -1 means empty

  function add(newFiles) {
    const arr = Array.from(newFiles);
    // Filter to audio types only
    const audioFiles = arr.filter(f => f.type.startsWith('audio/') ||
      /\.(mp3|wav|ogg|flac|m4a|aac|wma|opus|webm)$/i.test(f.name));
    const skipped = arr.length - audioFiles.length;
    files.push(...audioFiles);
    if (index === -1 && files.length > 0) index = 0;
    return { added: audioFiles.length, skipped, total: files.length };
  }

  function remove(i) {
    if (i < 0 || i >= files.length) return null;
    const removed = files.splice(i, 1)[0];
    if (files.length === 0) { index = -1; }
    else if (index >= files.length) { index = files.length - 1; }
    else if (i < index) { index--; }
    return removed;
  }

  function current()  { return index >= 0 ? files[index] : null; }
  function next()     {
    if (files.length === 0) return null;
    index = (index + 1) % files.length;
    return files[index];
  }
  function getIndex() { return index; }
  function count()    { return files.length; }
  function all()      { return files.slice(); }

  return { add, remove, current, next, getIndex, count, all };
})();
```

- [ ] **Step 2: Verify in browser DevTools**

```js
// Create a mock File-like object to test
const blob = new Blob([''], { type: 'audio/mp3' });
const mockFile = new File([blob], 'test.mp3', { type: 'audio/mp3' });
const result = Playlist.add([mockFile]);
result.added   // 1
result.total   // 1
Playlist.count()  // 1
Playlist.current() // mockFile
Playlist.next()   // mockFile (wraps around)
Playlist.remove(0)
Playlist.count()  // 0
Playlist.current() // null
```

- [ ] **Step 3: Commit**

```bash
git add music-player.html
git commit -m "feat: add Playlist module — audio file queue management"
```

---

### Task 5: CoverManager Module

**Files:**
- Modify: `music-player.html`

- [ ] **Step 1: Add CoverManager module after Playlist**

```js
/* ============================================================
   CoverManager — handles cover image loading via FileReader
   ============================================================ */
const CoverManager = (() => {
  let currentDataURL = null;
  const listeners = {};

  function on(name, fn) {
    if (!listeners[name]) listeners[name] = [];
    listeners[name].push(fn);
  }

  function emit(name, data) {
    (listeners[name] || []).forEach(fn => fn(data));
  }

  function setFromFile(file) {
    if (!file || !file.type.startsWith('image/')) {
      emit('error', { message: '不是有效的图片文件' });
      return;
    }
    const reader = new FileReader();
    reader.onload = () => {
      currentDataURL = reader.result;
      emit('change', { dataURL: currentDataURL });
    };
    reader.onerror = () => {
      emit('error', { message: '图片读取失败' });
    };
    reader.readAsDataURL(file);
  }

  function getCurrent() { return currentDataURL; }

  return { setFromFile, getCurrent, on };
})();
```

- [ ] **Step 2: Verify in browser DevTools**

```js
typeof CoverManager.setFromFile  // 'function'
typeof CoverManager.getCurrent   // 'function'
CoverManager.getCurrent()        // null

// Can test with a real image file via drag-drop later
```

- [ ] **Step 3: Commit**

```bash
git add music-player.html
git commit -m "feat: add CoverManager module — cover image FileReader"
```

---

### Task 6: UI Wiring — Connect All Modules

**Files:**
- Modify: `music-player.html`

- [ ] **Step 1: Add the UI wiring code after CoverManager**

```js
/* ============================================================
   UI — DOM bindings, event handlers, module orchestration
   ============================================================ */
(function UI() {
  /* --- DOM refs --- */
  const coverArea  = document.getElementById('coverArea');
  const coverPlaceholder = document.getElementById('coverPlaceholder');
  const coverImg   = document.getElementById('coverImg');
  const coverFileInput = document.getElementById('coverFileInput');
  const setName    = document.getElementById('setName');
  const trackName  = document.getElementById('trackName');
  const progressBar = document.getElementById('progressBar');
  const timeCurrent = document.getElementById('timeCurrent');
  const timeTotal   = document.getElementById('timeTotal');
  const btnPlay     = document.getElementById('btnPlay');
  const volumeSlider = document.getElementById('volumeSlider');
  const btnLoop     = document.getElementById('btnLoop');
  const dropZone    = document.getElementById('dropZone');
  const audioFileInput = document.getElementById('audioFileInput');
  const toast       = document.getElementById('toast');

  let toastTimer = null;
  let isSeeking = false;

  /* --- Helpers --- */
  function fmtTime(seconds) {
    if (!isFinite(seconds) || seconds < 0) return '00:00';
    const m = Math.floor(seconds / 60);
    const s = Math.floor(seconds % 60);
    return String(m).padStart(2, '0') + ':' + String(s).padStart(2, '0');
  }

  function showToast(msg) {
    toast.textContent = msg;
    toast.classList.add('show');
    clearTimeout(toastTimer);
    toastTimer = setTimeout(() => toast.classList.remove('show'), 2500);
  }

  function updateTrackDisplay() {
    const file = Playlist.current();
    trackName.textContent = file
      ? `曲目 ${Playlist.getIndex() + 1} / ${Playlist.count()} — ${file.name}`
      : '尚无曲目';
    setName.textContent = Playlist.count() > 0 ? `音乐集 (${Playlist.count()} 首)` : '拖放音频文件到下方';
  }

  function updatePlayButton() {
    if (Playlist.count() === 0) {
      btnPlay.disabled = true;
      btnPlay.textContent = '▶';
      return;
    }
    btnPlay.disabled = false;
    btnPlay.textContent = AudioEngine.isPaused() ? '▶' : '⏸';
  }

  function updateLoopButton() {
    btnLoop.classList.toggle('active', AudioEngine.setLoop);
    // We track loop state separately — see below
  }

  /* --- Restore saved settings --- */
  const savedVolume = Store.get('volume');
  const savedLoop = Store.get('loop');
  AudioEngine.setVolume(savedVolume);
  volumeSlider.value = Math.round(savedVolume * 100);

  let loopActive = savedLoop;
  AudioEngine.setLoop(savedLoop);
  btnLoop.classList.toggle('active', loopActive);
  btnLoop.textContent = loopActive ? '🔄 循环中' : '🔄 关闭循环';

  /* --- Cover area events --- */
  coverArea.addEventListener('click', () => coverFileInput.click());

  coverArea.addEventListener('dragover', e => { e.preventDefault(); });
  coverArea.addEventListener('drop', e => {
    e.preventDefault();
    const file = e.dataTransfer.files[0];
    if (file) CoverManager.setFromFile(file);
  });

  coverFileInput.addEventListener('change', () => {
    const file = coverFileInput.files[0];
    if (file) CoverManager.setFromFile(file);
    coverFileInput.value = '';
  });

  CoverManager.on('change', ({ dataURL }) => {
    coverImg.src = dataURL;
    coverImg.style.display = 'block';
    coverPlaceholder.style.display = 'none';
  });

  CoverManager.on('error', ({ message }) => showToast(message));

  /* --- Audio import events --- */
  function importAudioFiles(fileList) {
    if (!fileList || fileList.length === 0) return;
    const result = Playlist.add(fileList);
    if (result.added === 0) {
      showToast('未识别到支持的音频格式 (MP3, WAV, OGG, FLAC 等)');
      return;
    }
    if (result.skipped > 0) {
      showToast(`已添加 ${result.added} 首，跳过 ${result.skipped} 个非音频文件`);
    }
    // Load first track if this was the first import
    if (result.added > 0 && result.total === result.added) {
      loadCurrentTrack();
    }
    updateTrackDisplay();
    updatePlayButton();
    dropZone.classList.toggle('hidden', Playlist.count() > 0);
  }

  function loadCurrentTrack() {
    const file = Playlist.current();
    if (!file) return;
    AudioEngine.load(file);
    updateTrackDisplay();
    updatePlayButton();
  }

  dropZone.addEventListener('click', () => audioFileInput.click());

  dropZone.addEventListener('dragover', e => {
    e.preventDefault();
    dropZone.classList.add('drag-over');
  });
  dropZone.addEventListener('dragleave', () => {
    dropZone.classList.remove('drag-over');
  });
  dropZone.addEventListener('drop', e => {
    e.preventDefault();
    dropZone.classList.remove('drag-over');
    importAudioFiles(e.dataTransfer.files);
  });

  audioFileInput.addEventListener('change', () => {
    importAudioFiles(audioFileInput.files);
    audioFileInput.value = '';
  });

  /* --- Play/Pause --- */
  btnPlay.addEventListener('click', () => {
    if (Playlist.count() === 0) return;
    if (AudioEngine.isPaused()) {
      AudioEngine.play();
    } else {
      AudioEngine.pause();
    }
    updatePlayButton();
  });

  /* --- Progress bar --- */
  progressBar.addEventListener('input', () => {
    isSeeking = true;
    const dur = AudioEngine.getDuration();
    if (dur > 0) {
      const seekTime = (progressBar.value / 100) * dur;
      timeCurrent.textContent = fmtTime(seekTime);
    }
  });

  progressBar.addEventListener('change', () => {
    const dur = AudioEngine.getDuration();
    if (dur > 0) {
      AudioEngine.seek((progressBar.value / 100) * dur);
    }
    isSeeking = false;
  });

  AudioEngine.on('timeupdate', ({ currentTime, duration }) => {
    if (!isSeeking && duration > 0) {
      progressBar.value = (currentTime / duration) * 100;
      timeCurrent.textContent = fmtTime(currentTime);
    }
  });

  AudioEngine.on('loadedmetadata', ({ duration }) => {
    progressBar.disabled = false;
    progressBar.value = 0;
    timeTotal.textContent = fmtTime(duration);
    timeCurrent.textContent = '00:00';
    updatePlayButton();
  });

  /* --- Track ended → next --- */
  AudioEngine.on('ended', () => {
    Playlist.next();
    loadCurrentTrack();
    AudioEngine.play();
  });

  AudioEngine.on('error', ({ file }) => {
    showToast(`播放出错: ${file ? file.name : '未知文件'}`);
    // Try next track
    Playlist.next();
    loadCurrentTrack();
    updatePlayButton();
  });

  /* --- Volume --- */
  volumeSlider.addEventListener('input', () => {
    const v = volumeSlider.value / 100;
    AudioEngine.setVolume(v);
    Store.set('volume', v);
  });

  /* --- Loop --- */
  btnLoop.addEventListener('click', () => {
    loopActive = !loopActive;
    AudioEngine.setLoop(loopActive);
    Store.set('loop', loopActive);
    btnLoop.classList.toggle('active', loopActive);
    btnLoop.textContent = loopActive ? '🔄 循环中' : '🔄 关闭循环';
  });

  /* --- Global drag-and-drop on the whole page --- */
  document.addEventListener('dragover', e => { e.preventDefault(); });
  document.addEventListener('drop', e => {
    e.preventDefault();
    // Don't import if drop was on cover area (handled separately)
    if (coverArea.contains(e.target)) return;
    const files = e.dataTransfer.files;
    if (files.length > 0) importAudioFiles(files);
  });
})();
```

- [ ] **Step 2: Open in browser and verify**

1. Drag 2-3 MP3 files onto the drop zone
   - Verify drop zone disappears after files loaded
   - Track name shows "曲目 1 / 3 — filename.mp3"
   - Play button becomes enabled (purple)
2. Click play
   - Button toggles to ⏸
   - Progress bar moves, time updates
   - When track ends, auto-advances to next
3. Drag an image onto the cover area
   - Placeholder disappears, image shows
4. Adjust volume slider — verify audio volume changes
5. Toggle loop button — verify state changes and persists after reload
6. Reload page — volume and loop state should restore

- [ ] **Step 3: Commit**

```bash
git add music-player.html
git commit -m "feat: wire UI — connect all modules with event handlers"
```

---

### Task 7: Toast System + Edge Case Handling

**Files:**
- Modify: `music-player.html`

- [ ] **Step 1: Verify toast is already integrated, add FileReader availability check**

At the very beginning of the `<script>` tag (before Store), add:

```js
/* ============================================================
   Environment check
   ============================================================ */
if (typeof FileReader === 'undefined') {
  document.body.innerHTML = `
    <div style="display:flex;align-items:center;justify-content:center;min-height:100vh;padding:24px;text-align:center;
      background:#0d1117;color:#e6edf3;font-family:system-ui,sans-serif;">
      <div>
        <p style="font-size:32px;margin-bottom:16px;">⚠️</p>
        <p style="font-size:18px;">您的浏览器不支持此播放器</p>
        <p style="color:#8b949e;margin-top:8px;">请使用 Chrome、Firefox、Edge 或 Safari 的最新版本</p>
      </div>
    </div>`;
  throw new Error('FileReader not available');
}
```

- [ ] **Step 2: Enhance Playlist.add to handle completely invalid imports**

Update `Playlist.add` to return structured feedback for empty imports. The existing `add` function already handles this (it returns `{added, skipped, total}`), but let's verify it works by adding a guard:

In the Playlist module, replace the `add` function with:

```js
function add(newFiles) {
  const arr = Array.from(newFiles);
  if (arr.length === 0) return { added: 0, skipped: 0, total: files.length };
  const audioFiles = arr.filter(f =>
    f.type.startsWith('audio/') ||
    /\.(mp3|wav|ogg|flac|m4a|aac|wma|opus|webm)$/i.test(f.name)
  );
  const skipped = arr.length - audioFiles.length;
  files.push(...audioFiles);
  if (index === -1 && files.length > 0) index = 0;
  return { added: audioFiles.length, skipped, total: files.length };
}
```

- [ ] **Step 3: Verify edge cases in browser**

Test each scenario from the spec:
1. **Unsupported format:** Drag a `.txt` file — toast shows "未识别到支持的音频格式"
2. **Empty playlist + play:** Reload page (no files), play button should be disabled
3. **Mixed import:** Drag 2 MP3 + 1 TXT — toast shows "已添加 2 首，跳过 1 个非音频文件"
4. **Network error on blob URL:** Already handled by AudioEngine error listener

- [ ] **Step 4: Commit**

```bash
git add music-player.html
git commit -m "feat: add environment check and edge case handling"
```

---

### Task 8: Final Polish — Visual Refinements

**Files:**
- Modify: `music-player.html`

- [ ] **Step 1: Add final visual polish to CSS**

Add these refinements inside the `<style>` block, after the existing `.toast.show` rule:

```css
  /* Active drag feedback for cover */
  .cover-area.drag-over { box-shadow: 0 0 32px rgba(124,58,237,0.4); }

  /* Smooth progress during seek */
  .progress-bar { transition: none; }
  .progress-bar::-webkit-slider-thumb { transition: transform 150ms ease; }
  .progress-bar::-webkit-slider-thumb:hover { transform: scale(1.3); }
  .progress-bar::-webkit-slider-thumb:active { transform: scale(0.9); }

  /* Volume thumb hover */
  .volume-slider::-webkit-slider-thumb { transition: transform 150ms ease; }
  .volume-slider::-webkit-slider-thumb:hover { transform: scale(1.3); }

  /* Fade in for cover image */
  .cover-area img { animation: fadeIn 300ms ease; }
  @keyframes fadeIn { from { opacity: 0; } to { opacity: 1; } }

  /* Responsive: shrink on very small screens */
  @media (max-width: 360px) {
    .player { padding: 20px 12px; }
    .cover-area { width: 200px; height: 200px; }
    .btn-play { width: 48px; height: 48px; font-size: 18px; }
  }
```

- [ ] **Step 2: Verify visual refinements**

Open in browser:
- Hover over progress bar thumb — should scale up
- Cover image fade-in when changing
- Shrink browser below 360px — layout adapts

- [ ] **Step 3: Commit**

```bash
git add music-player.html
git commit -m "feat: final visual polish — hover states, animations, responsive"
```

---

### Task 9: End-to-End Manual Verification

- [ ] **Step 1: Run through the full user flow**

1. Open `music-player.html` in Chrome
2. Verify: dark theme, cover placeholder 🎵, drop zone visible, play disabled
3. Drag 3 MP3 files onto drop zone
4. Verify: drop zone hidden, track info shows "曲目 1 / 3"
5. Click play ▶ — music starts, button changes to ⏸, progress bar updates
6. Let track 1 finish → auto-advances to track 2
7. Drag an image onto cover area → placeholder replaced by image
8. Click cover area → file picker opens → select another image → cover updates
9. Adjust volume → audio volume changes
10. Toggle loop → button shows "🔄 循环中" → track loops after end
11. Reload page → volume and loop state restored (cover and audio files are NOT restored — data is in blob URLs, per spec out-of-scope)
12. Drag a .txt file → toast shows warning
13. F5 with no files loaded → play button disabled
14. Resize window below 360px → layout shrinks gracefully

- [ ] **Step 2: Test in Firefox**

Repeat steps 1-14 in Firefox. All should work identically.

- [ ] **Step 3: Commit final version**

```bash
git add music-player.html
git commit -m "chore: end-to-end verification complete"
```

---

### Summary

| Task | What | Commits |
|------|------|---------|
| 1 | HTML shell + CSS | 1 |
| 2 | Store module | 1 |
| 3 | AudioEngine module | 1 |
| 4 | Playlist module | 1 |
| 5 | CoverManager module | 1 |
| 6 | UI wiring | 1 |
| 7 | Edge cases | 1 |
| 8 | Visual polish | 1 |
| 9 | E2E verification | 1 |

**Total:** 9 tasks, 9 commits, ~1 single HTML file
