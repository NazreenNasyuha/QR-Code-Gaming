# AGENTS.md — Repository Engineering & Agent Operating Guide

> **Project**: QR Cartridge Arcade — 6-Game Offline PWA Studio & Handheld  
> **Owner / Copyright**: © 2026 NazreenNasyuha — Licensed under the [MIT License](LICENSE)

This document defines the mandatory engineering standards, architectural invariants, security guardrails, and verification procedures for all AI coding agents and human contributors working in this repository.

---

## 1. Ownership, Copyright & Legal Requirements

- All original source code, documentation, and exported PWA templates in this repository are **Copyright (c) 2026 NazreenNasyuha** under the **MIT License** ([`LICENSE`](LICENSE)).
- **Header Mandate**: Every new source file (`.html`, `.js`, `.css`, `.ts`) MUST include a concise copyright header attributing `Copyright (c) 2026 NazreenNasyuha` and referencing the `MIT License`.
- Never remove, alter, or overwrite existing copyright notices or the `LICENSE` file.

---

## 2. Living Architecture Protocol (Mandatory)

1. **Read First**: Always read [`ARCHITECTURE.md`](ARCHITECTURE.md) completely before planning or making code modifications.
2. **Sync in the Same Task**: Whenever you add, rename, modify, or delete any file, cartridge, BIOS method, codec behavior, UI tab, or PWA export artifact, you **MUST** update [`ARCHITECTURE.md`](ARCHITECTURE.md) in the **same change** so the Application Summary, Folder Map, Mermaid diagrams, and Component Contracts stay 100% accurate.

---

## 3. Architectural Invariants (Do Not Break)

These invariants define the core contract of QR Cartridge Arcade. Any change that violates them must be rejected:

1. **1:1 G-BIOS Parity (`QR Gaming.html` ↔ `PWA_INDEX_HTML`)**:
   - The Studio handheld (`BIOS`) and the exported standalone PWA (`PWA_INDEX_HTML`) MUST expose identical `G` API semantics: `c`, `W` (`160`), `H` (`200`), `pal[0..15]`, `k`, `p`, `cls`, `r`, `t`, `snd`, `sc`, `ov`, `loop`.
   - Any addition, bug fix, or palette change to `BIOS` MUST be mirrored identically in `PWA_INDEX_HTML`.
2. **Display & Timing Determinism**:
   - Logical canvas resolution is fixed at `160×200` with crisp pixelated rendering (`image-rendering: pixelated`).
   - Fixed timestep accumulator runs at `30 FPS` (`1000 / 30` ms) with `acc` clamped (`<= 100` ms) and at most `2` simulation substeps per animation frame to prevent spiral-of-death freezes.
3. **16-Color Palette Index Stability**:
   - Never reorder or remove indices `0..15` in `PALETTE`, as all printed QR cartridges reference colors by integer index.
4. **QR Hash Wire Compatibility**:
   - Maintain backward compatibility with `#d.<base64url>` (`deflate-raw`) and `#r.<uri-encoded>` hash formats so previously printed physical QR cards continue to boot forever.
5. **QR Scannability & Byte-Budget Discipline**:
   - Built-in cartridges MUST stay well below QR Version 40 ECC-L capacity (`2,953 B`), targeting **Easy/Good Scan** (`< 1,400 B`) wherever possible and keeping complex multi-entity arcade titles under `1,800 B` total URL at ECC `L`.
   - QR canvases MUST render pure `#000000` modules on `#FFFFFF` with a minimum **4-module quiet zone**.
6. **Zero-Dependency Exported PWA**:
   - `PWA_INDEX_HTML`, `PWA_SW_JS`, and `PWA_MANIFEST_JSON` MUST contain zero external CDN links, zero external fonts, and zero hardcoded game logic so the shell works 100% offline after a single visit.
7. **Print Layout Integrity**:
   - `@media print` MUST hide all dark studio UI chrome and output a clean, high-contrast 2×3 white cartridge sheet with `break-inside: avoid` on `.gb-print-card`.

---

## 4. Security & Safety Guardrails (Strict Enforcement)

Because this application encodes and boots executable JavaScript from URL fragments, security discipline is non-negotiable:

1. **Zero DOM XSS**:
   - **NEVER** interpolate user-controlled or external strings (`baseUrl`, decoded cartridge source, error messages, `location.hash`, filenames) into `innerHTML` or `outerHTML`.
   - Always assign dynamic text via `.textContent` or build nodes with `document.createElement`.
2. **Mandatory Decompression Bomb Ceiling**:
   - Both Studio `decodeFromHash()` and exported PWA `dec()` MUST stream through `DecompressionStream('deflate-raw')` with a strict **`32,768` byte (32 KB)** ceiling (`MAX_DECODED_BYTES`) and cancel the reader immediately if exceeded. Never replace this with unbounded `new Response(ds.readable).text()`.
3. **No Silent Auto-Execution of Untrusted URL Hashes in Studio**:
   - When `QR Gaming.html` opens with an external `#d.` or `#r.` fragment in `window.location.hash`, it MUST load the decoded source into the `Custom Cart` editor and prompt the user to review before explicitly clicking **Apply & Re-Encode**.
4. **Strict Base URL Validation**:
   - `getCleanBaseUrl()` MUST only accept `https:` or `http:` URLs via `new URL(...)`, falling back safely on malformed input or dangerous schemes (`javascript:`, `data:`, `vbscript:`).
5. **Dedicated-Origin Deployment Policy**:
   - Document and enforce that the exported Offline PWA shell MUST only be hosted on a dedicated, static-only origin (e.g., an isolated GitHub Pages repo) with no sensitive cookies, auth sessions, or private storage on that origin.
6. **Zero Secrets & Clean Supply Chain**:
   - Never commit API keys, tokens, `.env` files, personal paths, or private certificates.
   - Keep `referrerpolicy="no-referrer"` on external Studio CDN tags and `crossorigin="anonymous"` on third-party scripts. If modularizing the Studio in the future, self-host `qrcode-generator` and compile Tailwind locally.

---

## 5. Coding Standards

### Studio Application (`QR Gaming.html`)
- Wrap all Studio logic in `'use strict'` inside an IIFE; never pollute `window` globals.
- Never use blocking browser dialogs (`alert`, `confirm`, `prompt`); use `showToast(msg, tone)` and inline status banners.
- Touch controls MUST use Pointer Events with `setPointerCapture` plus `pointerup`, `pointercancel`, `lostpointercapture`, and `window.blur` cleanup so virtual D-Pad/action buttons never stick.
- Prefer modern `navigator.clipboard.writeText` with graceful fallback for `file://` contexts.

### Game Cartridges (`CARTRIDGE_CATALOG`)
- Interact **exclusively** with the `G` BIOS API (`G.cls`, `G.r`, `G.t`, `G.k`, `G.p`, `G.snd`, `G.sc`, `G.ov`, `G.loop`, `G.pal`).
- Never access `document`, `window`, `localStorage`, `fetch`, or timers (`setInterval`/`setTimeout`) inside cartridge code — drive all animation and cooldowns from the `G.loop(frame)` callback.
- Ensure every cartridge resets cleanly when `BIOS.boot()` is called (no leaked state outside the cartridge scope).

---

## 6. Verification & Pre-Publish Checklist

Before committing or pushing changes to GitHub, run through every item below:

- [ ] **Syntax & Asset Validation**: Parse and syntax-check:
  - Main Studio `<script>` block
  - Exported `PWA_INDEX_HTML` inline `<script>` block
  - Exported `PWA_SW_JS` service worker source
  - Exported `PWA_MANIFEST_JSON` valid JSON
  - Every cartridge `code` entry in `CARTRIDGE_CATALOG` (compiles cleanly via `new Function('G', code)`)
- [ ] **Round-Trip & Telemetry Check**: Confirm all 6 built-in games boot cleanly from their `#d.` compressed hashes and fit within Easy/Good Scan byte budgets.
- [ ] **Security Audit**: Confirm zero new raw `innerHTML` interpolations with untrusted inputs, 32 KB decompression guard intact in both Studio & PWA, external hash review gate intact, and zero secrets/PII in `git diff`.
- [ ] **UI & Print Smoke Test**: Verify keyboard + touch D-Pad input, Tab 2 `2×3` print preview, and Tab 3 copy/download actions.
- [ ] **Docs & Git Hygiene**: Confirm `ARCHITECTURE.md` matches disk 100%, `LICENSE` and copyright headers credit `NazreenNasyuha`, and `git status` shows no ignored or temporary artifacts.

### Commit Convention
Use [Conventional Commits](https://www.conventionalcommits.org/):
- `feat(cart): add breakout cartridge within 850B QR budget`
- `fix(security): enforce 32KB streaming inflate limit in PWA shell`
- `docs(arch): sync ARCHITECTURE.md with updated BIOS palette`
- `chore(repo): add .gitignore, AGENTS.md, and MIT LICENSE`
