# Atlas of Echoes

*A living chart of worlds in motion — where every mark upon the map breathes with the weight of those who wander it.*

Atlas of Echoes is a real-time, installable web map for tabletop RPG sessions. A Game Master's live world data (players, NPCs, enemies, and locations) is polled from a webhook and rendered on a hand-inked, dark-fantasy grid map that updates in place — no page reloads, no flicker.

---

## Features

- **Live-updating map** — polls a webhook every 5 seconds and merges changes into the existing DOM in place, so markers never flash, flicker, or jump during a refresh.
- **Entity markers** — players, NPCs, enemies, and locations each get a distinct marker style and color. When multiple entities share a tile, a **count bubble** (`+N`) shows how many are stacked there.
- **Health bars** — per-entity HP bar (max 20) with color tiers (healthy / wounded / critical), a shine "glint" animation when HP is overcharged above the max, and a flashing critical-light indicator when HP drops to 0 or below.
- **Filters** — toggle visibility of players, NPCs, enemies, and locations independently.
- **Search & camera controls** — search entities by name, pan/zoom the map, pinch-zoom and drag on mobile.
- **Room-based sessions** — players/GMs enter a Room ID to load that session's world data from the backend.
- **Progressive Web App (PWA)** — installable to a home screen or desktop, works offline for the app shell, and has its own icon set and manifest.
- **Ambient atmosphere** — drifting dust/ash, fog, and starfield effects for mood, all respecting a stable, non-distracting marker layer.

---

## File structure

All files are served from the **site root** — the manifest and service worker use root-relative paths (`/favicon.ico`, `/sw.js`, etc.), so this structure must be preserved on deploy:

```
/
├── index.html                   # entire app: markup, CSS, and JS in one file
├── sw.js                        # service worker (offline app-shell caching)
├── site.webmanifest             # PWA manifest (name, icons, theme colors)
├── favicon.ico
├── favicon-16x16.png
├── favicon-32x32.png
├── apple-touch-icon.png
├── android-chrome-192x192.png
└── android-chrome-512x512.png
```

There is no build step — it's a static site. Deploy the folder as-is to any static host (GitHub Pages, Netlify, Vercel, S3, etc.).

---

## Getting started

### Run locally

Because the service worker requires a secure context, don't just double-click `index.html`. Serve it instead:

```bash
# from the project folder
python3 -m http.server 8000
# then open http://localhost:8000
```

Any static file server works (`npx serve`, `live-server`, etc.) — `localhost` is treated as secure, so the PWA features work there too.

### Deploy

1. Upload all files in the structure above to your host's root.
2. Make sure the host serves over **HTTPS** — required for the service worker to register (localhost is the only exception).
3. Visit the site; the browser will offer an "Install" / "Add to Home Screen" prompt once the manifest and service worker are detected.

---

## Configuration

Key constants live near the top of the `<script>` block in `index.html`:

| Constant | Purpose | Default |
|---|---|---|
| `API_BASE` | Webhook URL the app polls for world data (Room ID is appended to it) | ngrok webhook URL |
| `UPDATE_MS` | Live-update polling interval, in milliseconds | `5000` |
| `GRID_SIZE` | Width/height of the map grid, in cells | `17` |
| `maxHp` (in `buildEntityCard`) | HP bar scale — health above this triggers the glint animation | `20` |

To point the app at a different backend, update `API_BASE`. The endpoint is expected to return a JSON array whose first element is an object describing the world's entities for the given Room ID.

### Entity data shape

Each entity is expected to include (not all fields are required):

```json
{
  "type": "player | npc | enemy | location",
  "name": "string",
  "x": 0,
  "y": 0,
  "health": 20,
  "class": "string",
  "location": "string",
  "icon": "emoji or short string"
}
```

---

## Customization

### Theme colors

All colors are CSS custom properties defined in `:root` at the top of `index.html`, grouped by purpose (parchment/ink base tones, entity colors, status colors). Notable ones:

```css
--col-player:    #4a7fac;  /* blue */
--col-npc:       #8a7048;  /* bronze */
--col-enemy:     #a83a3a;  /* crimson */
--col-location:  #9a8259;  /* weathered stone */
--legendary-gold: #d4a84b; /* reserved for rare/legendary accents only */
```

### Branding

App name, description, and theme color are set in three places that should stay in sync if you rebrand:

- `<title>` and the landing screen (`.logo-title`, `.logo-subtitle`) in `index.html`
- `site.webmanifest` (`name`, `short_name`, `description`, `theme_color`, `background_color`)
- `<meta name="theme-color">` and `apple-mobile-web-app-title` in `index.html`

### Icons

Regenerate icons at [favicon.io](https://favicon.io) (or similar) and drop the new files in at the same names/sizes to keep all the manifest/HTML references valid.

---

## Accessibility & motion

- Ambient/decorative animations (fog, dust, rune float) are separate from functional UI state, but the app does not yet respect `prefers-reduced-motion`. If this matters for your deployment, wrap decorative `animation` rules in a `@media (prefers-reduced-motion: reduce)` block.
- The critical-HP flash and count bubbles update via text/attribute changes only — no DOM teardown — to avoid flicker for screen magnifier and motion-sensitive users.

---

## Browser support

Built with modern, evergreen browsers in mind (Chrome, Edge, Firefox, Safari — desktop and mobile). PWA installability depends on the browser: Chromium-based browsers support full "Install app" prompts; Safari on iOS supports "Add to Home Screen" with slightly different manifest handling (hence the dedicated `apple-touch-icon` and `apple-mobile-web-app-*` meta tags).

---

## Known limitations

- Polling is a fixed 5-second interval with no backoff; repeated network failures will keep retrying at the same rate.
- Entity identity for in-place marker updates is currently name-based — two same-named entities at one location may not be distinguished. Prefer a stable `id` field from the backend if you extend the data model.
- No offline write support — the app can load a cached shell offline, but live world data always requires a network connection.

---

## License

Add your preferred license here.
