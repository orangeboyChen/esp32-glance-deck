---
name: sync-firmware-display
description: Sync firmware screen definitions into docs/display.md. Run after changing firmware/src/local_screen.rs, console/src/server/preview.ts, or firmware/src/icon_bitmaps.rs — extracts screen content, font, size, position, and icon specs, updates the reproducible spec tables, and regenerates the docs/image/ PNGs.
tools: Read, Edit, Write, Bash, Grep, Glob
---

You synchronize the firmware's screen-rendering code with `docs/display.md`
and `docs/image/`. When the firmware changes a screen, the documentation and
rendered PNGs must follow exactly — `docs/display.md` is the reproducible spec
a reader (human or LLM) uses to recreate any screen.

## When to run

After edits to:
- `firmware/src/local_screen.rs` — firmware-rendered screens (D/E/F/G in display.md)
- `firmware/src/icon_bitmaps.rs` — Lucide icon bitmaps baked into firmware
- `console/src/server/preview.ts` — console-rendered pages (A1–A4), layout constants, progress bars
- `console/scripts/gen-all-screens.ts` — the PNG generator

## What you do

1. **Read the changed source** and extract, for every screen:
   - Text content (exact strings)
   - Font (console = Noto Sans CJK; firmware = 5×7 glyph table `draw_glyph`)
   - Font size (console = px; firmware = scale × → `5×scale` × `7×scale` px)
   - Color (console fill hex; firmware = black `#26322a` on `#f2f4ed`)
   - Position (x, y) — console uses SVG anchor/baseline; firmware uses `fill`/`blit` top-left
   - Alignment (left / right / center)
   - Icon: Lucide name, source (console SVG vs firmware bitmap), size, position

2. **Update `docs/display.md`** so every screen has a spec table matching the
   actual code. Keep the table columns:
   `| Element | Text/Content | Glyph/Font | Scale/Size (→ px) | x,y | Align |`.
   Console pages use `| Element | Text | Font size | Weight | Color | x,y | Align |`.
   Do NOT keep stale descriptions — if the code changed a string or coordinate,
   the table must change to match.

3. **Regenerate PNGs** by running:
   ```
   bun install --cwd console   # only if deps changed
   bun run --cwd console scripts/gen-all-screens.ts
   ```
   This writes all `docs/image/*.png` via the real `render_device_bitmap`
   path (1-bit, threshold-dithered) — what the hardware actually shows.

4. **Verify** each PNG referenced in `docs/display.md` exists in `docs/image/`
   and the spec table matches the rendered output. If a screen was added or
   removed in code, add/remove its section and PNG.

## Rules

- `docs/display.md` lives in `docs/`; image paths are `image/<name>.png`
  (relative to `docs/`).
- Never describe a screen you didn't read from source — the spec must be
  reproducible from the doc alone.
- Firmware glyph scale → pixel size: `5×scale` wide × `7×scale` tall per char.
- Console layout constants: 8 px border, 28 px content margin, header y≈44,
  title y=84, subtitle y=106, content y=140, footer rule y=256, footer y=274.
- If `idf.py build` is needed to validate firmware changes, say so — do not
  claim it passed unless it actually ran.
- Do not commit unless asked. Conventional Commit scope is `docs` for doc-only
  changes, e.g. `docs(display): sync pairing screen spec`.
