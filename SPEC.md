# SPEC.md — Reproductor de Música Mejorado

## Overview
Single-page web application for audio playback built with vanilla HTML, CSS, and JavaScript. No external dependencies. Runs entirely in the browser using the Web Audio API (`<audio>` element).

---

## 1. Header & Theme Toggle

### 1.1 Header Bar
- **Element:** `.header` div, flexbox with `justify-content: space-between`.
- Contains app title on the left and theme toggle button on the right.
- Title: `<h1>Tool Player mp3</h1>`.
- Separated from content below by a bottom border.

### 1.2 Theme Toggle
- **Button:** `.theme-toggle` (circular, bordered, id: `themeToggle`).
- **Default state:** Moon icon `🌙` — dark mode active.
- **On click:** `toggleTheme()` toggles `.light-mode` class on `<body>`, and swaps the button icon:
  - `🌙` (dark mode) → `☀️` (light mode)
  - `☀️` (light mode) → `🌙` (dark mode)
- Button has `aria-label="Cambiar modo"` for accessibility.
- Smooth transitions via CSS `transition` on background-color and border-color (0.3s).

### 1.3 Theming System (CSS Custom Properties)
All themed colors are controlled via CSS variables defined on `:root`. The `body.light-mode` selector overrides them:

| Variable               | Dark Mode     | Light Mode    |
|------------------------|---------------|---------------|
| `--bg-color`           | `#1a1a2e`     | `#f0f0f0`     |
| `--container-bg`       | `#16213e`     | `#ffffff`     |
| `--text-color`         | `#e0e0e0`     | `#333333`     |
| `--playlist-item-bg`   | `#0f3460`     | `#ffffff`     |
| `--playlist-item-hover`| `#1a1a2e`     | `#f0f0f0`     |
| `--dropzone-border`    | `#444`        | `#ccc`        |
| `--header-border`      | `#333`        | `#ddd`        |
| `--primary`            | `#4CAF50`     | `#4CAF50`     |
| `--primary-hover`      | `#45a049`     | `#45a049`     |
| `--active`             | `#2196F3`     | `#2196F3`     |

- Elements consume these variables and update instantly on toggle with no repaint flicker.

---

## 2. File Loading

### 2.1 File Input
- **Element:** `<input type="file" id="fileInput" accept="audio/*" multiple>`
- Accepts any file with an `audio/*` MIME type.
- Multiple file selection supported.
- On change: filters out non-audio files via `file.type.startsWith('audio/')`, stores valid files as track objects, appends items to the playlist, auto-plays the first track if no track is currently playing.
- Input is reset after each selection (`e.target.value = ''`) so the same files can be re-selected.

### 2.2 Drag & Drop
- **Drop zone:** `.playlist` div.
- Visual feedback: `dragover` event adds a `.dragover` class (green border + green tint background). Removed on `dragleave` and `drop`.
- On drop: extracts `e.dataTransfer.files` and passes them to `handleFiles()` (same pipeline as file input).

---

## 3. Track Model

Each track is stored as an object:
```js
{ name: string, url: string }
```
- `name`: the original file name.
- `url`: a blob URL created via `URL.createObjectURL(file)`. Object URLs are never revoked — this is a known limitation (blobs persist for session lifetime).

---

## 4. Playlist

### 3.1 Visual
- Rendered inside `.playlist` div.
- Each track is a `.playlist-item` div showing the file name.
- Clicking a `.playlist-item` calls `playTrack()` with the corresponding index.
- The currently playing track gets the `.playing` class (green background, white text).

### 3.2 Behavior
- Playlist items are appended in the order files are loaded.
- No reordering, sorting, or removal is supported.

---

## 5. Playback Controls

### 4.1 Play / Pause
- **Button:** `▶` / `⏸` toggles between play and pause.
- **Toggle function** checks `audioPlayer.paused` and switches the button icon and calls `audio.play()` / `audio.pause()`.

### 4.2 Next Track
- **Button:** `⏭`
- If shuffle is active: picks a random index via `Math.floor(Math.random() * tracks.length)`.
- If shuffle is off: increments index by 1, wrapping to 0 at the end (`(currentTrackIndex + 1) % tracks.length`).
- Calls `playTrack(newIndex)`.

### 4.3 Previous Track
- **Button:** `⏮`
- If `audio.currentTime > 2`: rewinds to 0 (restart current track).
- Otherwise: decrements index by 1, wrapping to the last track (`(currentTrackIndex - 1 + tracks.length) % tracks.length`).

### 4.4 Progress Bar
- **Element:** `<input type="range" id="progressBar">`
- Range: `0` to `100` (percentage).
- `timeupdate` event updates the bar: `(currentTime / duration) * 100`.
- User `input` event seeks: `(value / 100) * duration`.

### 4.5 Time Display
- **Current time** (`#currentTime`) and **duration** (`#duration`) shown as `M:SS`.
- Duration is set on `loadedmetadata` event.
- Current time is updated on every `timeupdate` event.

### 4.6 Volume Control
- **Element:** `<input type="range" id="volumeControl" min="0" max="1" step="0.1" value="1">`
- Directly maps to `audioPlayer.volume`.

---

## 6. Playback Modes

### 5.1 Shuffle (`isShuffle`)
- **Toggle button:** `🔀` (id: `shuffleBtn`).
- When active: `nextTrack()` picks a random track (including possibly the same track).
- Active state visually indicated by `.active-mode` class (blue background).
- Keyboard shortcut: `S` key.

### 5.2 Repeat (`repeatMode`)
- **Toggle button:** `🔁` / `🔂` (id: `repeatBtn`).
- Cycles through three states: `none` → `all` → `one` → `none`...
- `none`: no repeat. On end, stops if last track.
- `all`: repeats entire playlist. On end, calls `nextTrack()` (wraps to start).
- `one`: repeats current track. On end, calls `audio.play()` (restart).
- Active state (any mode !== `none`) shown via `.active-mode` class.
- Button icon changes: `🔁` for none/all, `🔂` for one.
- Keyboard shortcut: `R` key.

---

## 7. End-of-Track Logic

Handled in the `ended` event listener:
1. If `repeatMode === 'one'`: re-play current track.
2. If `repeatMode === 'all'` or `isShuffle === true`: call `nextTrack()`.
3. Otherwise (repeat 'none', no shuffle): call `nextTrack()` only if `currentTrackIndex < tracks.length - 1`. If it's the last track, playback stops.

---

## 8. Keyboard Shortcuts

| Key              | Action                       |
|------------------|------------------------------|
| `Space`          | Toggle play/pause            |
| `ArrowRight`     | Seek forward 5 seconds       |
| `ArrowLeft`      | Seek backward 5 seconds      |
| `ArrowUp`        | Increase volume by 0.1       |
| `ArrowDown`      | Decrease volume by 0.1       |
| `N`              | Next track                   |
| `P`              | Previous track               |
| `S`              | Toggle shuffle               |
| `R`              | Toggle repeat mode           |

- All shortcuts are case-insensitive (comparison via `e.key.toLowerCase()`).
- Only `Space` calls `e.preventDefault()` (to prevent page scroll).

---

## 9. UI / Styling

### 8.1 Layout
- Flexbox, centered layout with `min-height: 100vh`.
- Two main containers: `.player-container` (controls) and `.playlist` (track list), each 300px wide.

### 9.2 Color Scheme (Themed via CSS Custom Properties)
All colors are defined as CSS variables on `:root` (dark mode default). `body.light-mode` overrides them for light mode. See [Section 1.3](#13-theming-system-css-custom-properties) for the full variable table.
- Primary button: `#4CAF50` (green), darkens to `#45a049` on hover.
- Active mode indicator: `#2196F3` (blue).
- All themed elements transition smoothly on toggle (0.3s `transition`).

### 9.3 Drop Zone
- Dashed border using `var(--dropzone-border)`.
- `.dragover`: green border `var(--primary)` + translucent green background.

### 9.4 Playing Track
- `.playing` class: green background (`var(--primary)`), white text.

### 9.5 Header
- Flexbox bar at the top (340px wide) with title on the left and theme toggle button on the right.
- Bottom border uses `var(--header-border)`.
- Theme toggle button: 40×40px circle with border matching `var(--text-color)`.

---

## 10. Known Limitations

- **Memory:** Blob URLs are never revoked (`URL.revokeObjectURL` is never called). For long sessions with many files, memory usage grows unbounded.
- **Playlist:** No remove, reorder, or clear functionality. No persistence (lost on page refresh).
- **Error handling:** No error handling for unsupported codecs, missing metadata, or failed file reads. No fallback display when no tracks are loaded beyond the placeholder text.
- **Accessibility:** Buttons use emoji icons without `aria-label` attributes. No `role` or `tabindex` management.
- **Responsive:** Fixed-width containers (300px) — does not adapt well to small viewports.
- **CORS/Streaming:** Relies on local file blob URLs. Not designed for streaming remote URLs.
