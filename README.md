# QR Cartridge Arcade — 6-Game Offline PWA Studio & Handheld

> **Copyright (c) 2026 NazreenNasyuha** — Licensed under the [MIT License](LICENSE)

A zero-backend retro micro-console studio and offline PWA handheld where **the entire playable game lives inside each QR code URL hash (`#d.<base64url>`)**.

---

## Live URLs (GitHub Pages)

Once GitHub Pages is enabled on `main` (`/` root):

- **Offline QR Pocket Console (QR scan target)**: `https://nazreennasyuha.github.io/QR-Code-Gaming/`
- **6-Game QR Cartridge Studio & Print Sheet**: `https://nazreennasyuha.github.io/QR-Code-Gaming/QR%20Gaming.html`

### Enable GitHub Pages (1-Time Setup)
1. Open **Settings → Pages** in this repository.
2. Under **Build and deployment → Source**, choose **Deploy from a branch**.
3. Select Branch **`main`** and Folder **`/ (root)`**, then click **Save**.
4. Wait ~1 minute, open the Studio URL, print the **2×3 Cartridge Sheet**, and scan any card on your phone — after one scan, `sw.js` caches the console shell so all printed QR cartridges launch in Airplane Mode.

---

## Built-In Cartridges (16-Color `160×200` @ `30 FPS` G-BIOS)

| Cartridge | Highlights | Controls |
| --- | --- | --- |
| **SNAKE** | 16×18 grid, queued inputs, timed bonus berry, turbo | D-Pad Steer • Hold `A` Turbo |
| **PONG** | Portrait court, dash/power-hit, sets to 7 | `◀▶` Paddle • `A` Dash |
| **TETRIS** | 10×20 board, 7 pieces, wall-kick, ghost piece | `◀▶` Move • `▼` Soft • `A`/`▲` Rotate • `B` Drop |
| **SPACE INVADERS** | 5×8 alien fleet, destructible shields, waves | `◀▶` Move • `A` Fire Laser |
| **COLUMNS** | 6×13 gem well, match-3+, cascading chains | `◀▶` Move • `▼` Soft • `A`/`▲` Cycle • `B` Drop |
| **BOMBERMAN** | 11×11 destructible arena, chain blasts, stages | D-Pad Move • `A` Drop Bomb |
| **CUSTOM CART** | Live minified editor with instant QR re-encoding | Write your own `G`-API game |

---

## Security & Architecture Documentation

- **[ARCHITECTURE.md](ARCHITECTURE.md)** — Full system architecture, Mermaid diagrams, `G`-BIOS API spec, and security threat model.
- **[AGENTS.md](AGENTS.md)** — Repository engineering standards, invariants, and safety guardrails.
- **[LICENSE](LICENSE)** — MIT License (`Copyright (c) 2026 NazreenNasyuha`).
