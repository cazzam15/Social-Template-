# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Social Template Studio** — a single-file browser app for generating and downloading social media graphics. No build step, no dependencies, no framework. Everything runs in the browser via the Canvas API.

Live site: **https://cazzam15.github.io/Social-Template-/**
GitHub repo: **https://github.com/cazzam15/Social-Template-**

## Files

| File | Purpose |
|------|---------|
| `index.html` | The app — canonical source of truth for GitHub Pages |
| `social-template-studio.html` | Local working copy (kept in sync with index.html) |
| `manifest.json` | PWA manifest (name, icons, theme color, start_url) |
| `sw.js` | Service worker — caches app shell for offline use |
| `icons/icon-192.png` | PWA home screen icon (192×192) |
| `icons/icon-512.png` | PWA splash/store icon (512×512) |

**Always edit `social-template-studio.html` first, then copy it to `index.html` before committing.**

```powershell
Copy-Item social-template-studio.html index.html -Force
```

## Deploying

```powershell
git add .
git commit -m "your message"
git push
```

GitHub Pages auto-deploys from `main` branch root. Allow ~60 seconds for changes to go live.

## Architecture

Everything lives in one HTML file — inline CSS, inline JS, no external assets.

**Rendering pipeline:**
- `vals()` reads all form inputs into a plain object
- `render(ctx, w, h, v)` draws the full image onto any canvas context at native resolution using only the Canvas 2D API
- `draw()` calls `render()` scaled into the preview canvas
- The download button creates an offscreen canvas at native template resolution and calls `render()` again — so preview and export are pixel-identical

**Templates** (`TEMPLATES` array) define platform presets (width, height, aspect ratio, icon). Switching template calls `resizeCvs()` then `draw()`.

**Mobile layout** uses CSS `@media (max-width: 767px)` to hide all three panels and show one at a time via `.tab-active` class. `initMobileLayout()` / `applyActiveTab()` manage which panel is visible. On desktop the standard 3-column CSS grid applies with no JS involvement.

**Canvas sizing** (`resizeCvs()`): on mobile uses `window.innerWidth - 32` as max width; on desktop subtracts the two sidebar widths (260px + 310px + 64px gutter).

## PWA Notes

- `manifest.json` `start_url` and `scope` are set to `/Social-Template-/` (the GitHub Pages subpath) — do not change these without also updating `sw.js` asset paths
- Service worker cache key is `social-studio-v1` — bump this string when deploying breaking changes so old caches are evicted
- Icons were generated with `System.Drawing` in PowerShell; regenerate if the brand colour changes (`#702888`)
