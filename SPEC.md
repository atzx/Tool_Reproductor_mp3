# SPEC.md — Tool Player mp3

## Overview
Single-page web application for audio playback built with vanilla HTML, CSS, and JavaScript. No external dependencies. Runs entirely in the browser using the Web Audio API (`<audio>` element with AudioContext for visualization).

---

## 1. Header & Theme Toggle

### 1.1 Header Bar
- **Element:** `.header` div, flexbox with `justify-content: space-between`.
- Contains app title on the left and theme toggle button on the right.
- Title: `<h1>Tool Player mp3</h1>`.

### 1.2 Theme Toggle
- **Button:** `.theme-toggle` (circular, bordered, id: `themeToggle`).
- **Default state:** Moon icon `🌙` — dark mode active.
- **On click:** `toggleTheme()` toggles `.light-mode` class on `<body>`, swaps icon (`🌙` ↔ `☀️`).
- Triggers toast notification: "Modo claro" / "Modo oscuro".
- Keyboard shortcut: `T` key.
- State persisted to localStorage.

### 1.3 Theming System (CSS Custom Properties)
All themed colors controlled via CSS variables on `:root` (dark default). `body.light-mode` overrides:

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

---

## 2. File Loading

### 2.1 File Input
- **Element:** `<input type="file" id="fileInput" accept="audio/*" multiple title="Seleccionar archivos de audio">`
- Filters by `file.type.startsWith('audio/')`.
- Input reset after selection via `e.target.value = ''`.

### 2.2 Drag & Drop
- **Drop zone:** `.playlist` outer div.
- Visual feedback: `.dragover` class (green border + tint).
- Passes `e.dataTransfer.files` to `handleFiles()`.

---

## 3. Track Model

```js
{ name: string, url: string, plays: number }
```
- `name`: original file name.
- `url`: blob URL via `URL.createObjectURL(file)`. Revoked on track removal or playlist clear.
- `plays`: running count of times the track has been played.

---

## 4. Playlist

### 4.1 Structure
- **Outer container:** `.playlist` (handles drag/drop)
- **Inner container:** `#playlistItems` (holds track DOM nodes)
- **Header:** `.playlist-header` with:
  - Track counter (`#trackCount`)
  - Export button (`#exportBtn`) — downloads JSON with names + play counts
  - Sort button (`#sortBtn`) — toggles A-Z / Z-A sorting
  - Clear button (`#clearPlaylistBtn`) — removes all tracks, revokes all blob URLs, resets player
  - Search input (`#searchInput`) — filters visible tracks in real-time

### 4.2 Track Items
- Each `.playlist-item` contains:
  - Name `<span>` — the file name
  - Play count `<span class="play-count">` — number of times played
  - Delete button — removes the track, handles index adjustments, revokes blob URL
- Playing track gets `.playing` class (green bg).
- Clicking a track calls `playTrack()`.
- Max height 400px with custom scrollbar.

### 4.3 Sorting
- `toggleSortOrder()` sorts `tracks` array by `name.localeCompare()`, toggles `sortAscending`.
- Rebuilds all playlist DOM via `rebuildPlaylistDOM()`.
- Maintains current track index during sort.
- Shows toast: "Ordenado A-Z" / "Ordenado Z-A".

### 4.4 Search / Filter
- `filterPlaylist()` hides items whose name doesn't match the query.
- Uses `.hidden` class (`display: none`).

### 4.5 Clear
- `clearPlaylist()`:
  - Revokes all blob URLs
  - Empties `tracks` array
  - Resets audio player, vinyl, equalizer
  - Clears search input and cancels sleep timer

---

## 5. Playback Controls

### 5.1 Play / Pause
- **Button:** `#playBtn` — `▶` / `⏸`.
- State tracked via `play` and `pause` events on `<audio>`.
- Play button gets `.playing-indicator` class (pulse animation).
- `togglePlay()` calls `audio.play()` / `audio.pause()`.

### 5.2 Next / Previous
- **Next** (`⏭`): random if shuffle on, else wrapped increment.
- **Previous** (`⏮`): if `currentTime > 2` restart track, else wrapped decrement.

### 5.3 Progress Bar
- Custom styled `<input type="range">` with:
  - Default `min=0`, `max=100` (percentage-based)
  - Custom track/thumb via `::-webkit-slider-*` / `::-moz-range-*`
  - Thumb scales on hover
  - `#progressBar` has larger thumb (18px)
- Value set on `timeupdate`: `(currentTime / duration) * 100`.
- Seek on `input` event: `(value / 100) * duration`.
- Progress tooltip: on `mousemove`, shows time at cursor position via positioned overlay.

### 5.4 Time Display
- `#currentTime` and `#duration` shown as `M:SS`.
- `.time-info` with hover opacity transition.

### 5.5 Volume Control
- `#volumeControl`: slider 0-1, step 0.1.
- Width 80px in controls row.
- Changes persisted to localStorage.

### 5.6 Speed Control
- `.speed-control` with 4 buttons: 0.5x, 1x, 1.5x, 2x.
- Active speed highlighted with `.active-speed` class.
- Sets `audioPlayer.playbackRate`.
- Persisted to localStorage.

---

## 6. Playback Modes

### 6.1 Shuffle
- **Button:** `#shuffleBtn` (`🔀`), toggle with `S` key.
- Wiggle animation on activation (`@keyframes shuffleWiggle`).
- Toast + localStorage.

### 6.2 Repeat
- **Button:** `#repeatBtn` (`🔁` / `🔂`), toggle with `R` key.
- Cycles: `none` → `all` → `one` → `none`.
- Toast with mode label + localStorage.

---

## 7. End-of-Track Logic
1. `repeatMode === 'one'`: replay.
2. `repeatMode === 'all'` or `isShuffle`: `nextTrack()`.
3. Otherwise: next only if not last track.

---

## 8. Sleep Timer
- **Button:** `#sleepBtn` (`⏱`), opens `#sleepOptions` panel.
- Options: 5min, 10min, 30min, 1h, Cancel.
- `setSleepTimer(minutes)`: starts `setInterval` countdown.
- `cancelSleepTimer()`: clears interval, removes active-mode.
- Countdown shown in `#sleepCountdown` span.
- Auto-pauses playback on expiry via `audioPlayer.pause()`.
- Active indicator on sleep button.

---

## 9. Vinyl Record Animation
- `.vinyl` CSS disc with radial gradient grooves and center label.
- `.spinning` class: `@keyframes spinVinyl` (2s linear infinite).
- `.paused` class: `animation-play-state: paused`.
- Start/stop synced with `play`/`pause` events.
- Light mode variant with adjusted colors.
- Shine overlay via `::before` pseudo-element.

---

## 10. Equalizer Visualization
- **Canvas:** `#equalizerCanvas` (300×40px), rendered via `CanvasRenderingContext2D`.
- **Audio chain:** `audio element → MediaElementSource → AnalyserNode → GainNode → destination`.
- AnalyserNode: `fftSize: 128`, `frequencyBinCount: 64`.
- Bars drawn via `getByteFrequencyData()`, green-to-purple gradient based on intensity.
- Animation loop via `requestAnimationFrame()`, cleanly cancelled on pause.
- AudioContext created lazily on first play.
- Light mode uses green-tone bars.

---

## 11. Fade In
- `GainNode` inserted into audio chain for volume ramps.
- On `play` event: `setValueAtTime(0)` → `linearRampToValueAtTime(1, 300ms)`.
- Handles AudioContext suspend/resume.

---

## 12. Media Session API
- Sets `navigator.mediaSession.metadata` (title) on play.
- Action handlers: play, pause, previoustrack, nexttrack.
- Enables lock screen / notification controls on mobile and desktop.

---

## 13. Picture-in-Picture (PiP)
- **Button:** `🖼` in mode buttons.
- `togglePiP()`: enters/exits PiP mode via `audioPlayer.requestPictureInPicture()`.
- Guards for `readyState >= 2`.
- Toast on enter/leave PiP events.

---

## 14. Toast Notifications
- Fixed container `#toastContainer` (top-right).
- Toasts auto-dismiss after 3s with slide-in/slide-out animation.
- Types: `.success` (green border), `.info` (blue border), `.error` (red border).
- Usage: file add, theme toggle, shuffle/repeat change, sleep timer, export, delete.

---

## 15. LocalStorage Persistence
- **Key:** `mp3player_state`.
- **Saved fields:** volume, isShuffle, repeatMode, speed, theme.
- `saveState()` called on every preference change.
- `loadState()` restores on page load.
- Graceful error handling.

---

## 16. Keyboard Shortcuts

| Key              | Action                       |
|------------------|------------------------------|
| `Space`          | Toggle play/pause            |
| `ArrowRight`     | Seek forward 5s              |
| `ArrowLeft`      | Seek backward 5s             |
| `ArrowUp`        | Volume +0.1                  |
| `ArrowDown`      | Volume -0.1                  |
| `N`              | Next track                   |
| `P`              | Previous track               |
| `S`              | Toggle shuffle               |
| `R`              | Toggle repeat mode           |
| `T`              | Toggle theme                 |

---

## 17. UI / Styling

### 17.1 Layout
- Body: flexbox row with wrapping, centered, `align-items: flex-start`, no gap.
- Header: `width: 100%`, spans full width above both columns.
- Player container: `flex: 0 0 50%`, `max-width: 50%`, `box-sizing: border-box` — left half of viewport.
- Playlist: `flex: 0 0 50%`, `max-width: 50%`, `box-sizing: border-box` — right half of viewport.
- On narrow viewports (`<720px`): both stack to 100% width vertically.

### 17.2 Micro-interactions
- Buttons: `scale(0.93)` on `:active`, 0.1s transition.
- Play button: pulse glow `@keyframes pulseBtn` when playing.
- Playlist items: `padding-left` shift on hover, `scale(0.98)` on active.
- Range thumbs: `scale(1.2)` on hover.
- All themed elements: 0.3s color transition.

### 17.3 Custom Scrollbar
- Webkit pseudo-elements: 8px width, transparent track, themed thumb.
- `.playlist` max-height: `calc(100vh - 100px)`, `overflow-y: auto`.

### 17.4 Responsive Design
- `@media (max-width: 720px)`: Layout stacks vertically when viewport is too narrow.
  - Both columns become 100% width, stacked vertically
  - Reduced padding and font sizes
  - Vinyl shrinks to 100px
  - Volume slider spans full width
  - Playlist max-height reduced to 300px

### 17.5 Export
- Downloads `playlist_YYYY-MM-DD.json` with track names and play counts.

---

## 18. Known Limitations

- **Blob URLs:** Revoked on individual track removal or playlist clear, but accumulated blobs during a session may persist if tracks are never explicitly removed.
- **No AudioContext without interaction:** AudioContext created lazily on first play (browser policy).
- **No ID3 parsing:** Track metadata (artist, album, cover art) not extracted from files.
- **No remote URLs:** Designed for local file playback only.
- **Reordering:** Drag-and-drop track reorder not implemented.
