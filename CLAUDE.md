# OrbitSmith — Project Instructions for Claude Code

## Project Overview
OrbitSmith is a free, open-source space situational awareness toolkit at orbitsmith.net.
Owner: Seita Iizuka (@iizukaseita)

## File Structure
- `index.html` — Landing page (i18n: EN/JA/ZH/ES, canvas starfield, orbital rings)
- `tracker.html` — 3D debris tracker (Three.js, 30,000+ objects, CDM conjunctions)
- `imagery.html` — Satellite imagery viewer (Leaflet.js, 21 layers, NASA/ESA/ESRI)
- `designer.html` — **NEW: Mission Designer** (see docs/designer-spec.md for full spec)
- `about.html` — About page
- `contact.html` — Contact page
- `worker.js` — Cloudflare Worker (deployed separately via dashboard, NOT via git)

## Design System (MUST follow)
- Background: `#000`
- Headings font: `Orbitron` (Google Fonts)
- Body font: `IBM Plex Mono` + `Noto Sans JP`
- Accent color: `#00e5ff` (cyan)
- Card bg: `rgba(4,8,16,0.7)`, border: `rgba(255,255,255,0.04)`, radius: `12px`
- Nav height: `52px`, fixed top, blur backdrop
- Status: Go `#00e676`, Warn `#ffc107`, No-Go `#ff5252`
- All pages are single HTML files (CSS+JS inline, external deps via CDN only)

## Deploy Workflow
- Git push to `iizukaseita/orbitsmith` → Cloudflare Pages auto-deploy
- **IMPORTANT: Always preview locally first before git push**
  1. Create/edit files in ~/Desktop/Coding/Orbitsmith/
  2. User opens in browser (file:///) to review
  3. Iterate until approved
  4. THEN: git add → commit → push

## Coding Rules
- Single HTML file per page (no build tools, no bundlers)
- External libs via CDN only (Three.js, Leaflet.js, Chart.js, Google Fonts)
- i18n: `data-i18n` attributes + language switch buttons (same pattern as index.html)
- Version naming: v8c is current, designer.html starts at v9a
- Worker changes: NEVER push via git. Manual deploy via Cloudflare dashboard.
- Find/replace edits must use EXACT strings — no high-level descriptions

## Current Task: Mission Designer (designer.html)
Full specification: **Read `docs/designer-spec.md` before starting any implementation.**
Key points:
- 5-step wizard: Mission Select → Config → Analysis → Launch → Summary
- 7 mission templates (Optical EO, SAR, IoT, AIS, Tech Demo, Science, Comms)
- 5 budget modules: Power, Link, Orbit Decay, Mass, Thermal
- Summary page uses Terminal typewriter animation
- All calculations client-side JavaScript (no backend needed)
- Help tooltips on all technical terms (data in i18n-ready JSON)
- Mockup files for reference: docs/designer-mockup-v2.html, docs/satellite-hologram.html, docs/summary-terminal.html
