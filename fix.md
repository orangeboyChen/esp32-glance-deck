# display.md review — fixes to apply

11 issues from the screen-design review. This file lists each problem, the
current state, and the proposed change to `docs/display.md` (and, where noted,
the rendering that the spec would then require). **No code changes here** —
this is the plan; the spec tables in `docs/display.md` get updated next.

---

## 1. `GLANCE DECK` brand text is unnecessary

**Current**: Every console page (A1–A4) shows `GLANCE DECK` (12 px, bold) at
(66, 44), pushed right of the icon.

**Problem**: Pointless branding on a 400×300 personal display. Wastes the
prime top-left slot and forces the icon to share the header row. The device
already knows what it is; the user doesn't need a constant brand banner.

**Fix**: Remove the `Brand` row from A1's table and the A2–A4 shared layout.
Move the icon to the title row (or drop it to the title baseline). Reclaim the
top area: title moves up to y≈44, content starts higher. Update the layout
constants block (header y≈44 → title y=44).

---

## 2. No Chinese / CJK content shown

**Current**: All sample pages are English-only (`Usage`, `Alert`, `Monthly
quota low`, `Resets`=`Tomorrow`). The doc says "Noto Sans CJK" but never
demonstrates CJK.

**Problem**: The whole point of bundling Noto Sans SC is Chinese display. A
reader/LLM has no spec example of how CJK renders (font subset, line breaks,
mixing CJK + Latin values). The device targets a CJK-capable product.

**Fix**: Add a CJK sample — either convert one A-page to Chinese (e.g. A1
title `用量`, subtitle `订阅`, lines `今日剩余`/`本周已用`/`重置时间`) or add an
A3-variant Home page with CJK (`日历`=`2 事件`, `温度`=`24°C`). Document the
font subset behaviour: Noto Sans SC subset covers GB2312 + Latin; mixed CJK +
Latin in one line uses the same family; CJK chars are ~1em wide vs Latin ~0.5em
so line layout differs.

---

## 3. Border around the screen — necessary?

**Current**: Every screen has a border: console pages `rect(8,8,384,284)` 4 px
stroke; firmware screens `rectangle(12,12,376,276)` 2 px.

**Problem**: A 4 px black border eats ~12px of the perimeter and visually
shrinks the 400×300 area. On a reflective LCD the bezel already frames the
screen — a software border is redundant and darkens the edges. Not clear it
earns its pixels.

**Fix**: Decide and document. Recommendation: **drop the border on console
pages** (rely on the physical bezel), keep a 1 px or no border on firmware
screens. If kept, reduce to 1 px / `#9ba89f`. Update every spec table's
`Border` row + the layout-constants block. Note: removing the border changes
the safe content margin (currently 28 px includes the border inset; without
it, margin can stay 28 px from the panel edge).

---

## 4. A6 Offline retention should show the *actual last page*, not a re-drawn stub

**Current**: A6 "Offline retention" shows a hand-drawn `USAGE` + 3 rows +
`OFFLINE` banner, rendered by firmware `local_screen.rs` with 5×7 glyphs. But
the whole point of offline retention is "keep the last verified frame" — the
device flushes the **cached bitmap**, not a re-rendered text screen.

**Problem**: The spec contradicts the behavior. When offline, firmware does
NOT redraw a Usage page with 5×7 glyphs — it keeps the last console-rasterized
bitmap (A1-style, with Noto Sans CJK, icons, progress bars) verbatim. The
`OFFLINE` banner is the only addition. The current table describes a screen
that doesn't exist in firmware.

**Fix**: Change A6's spec to: "the last verified console page bitmap is
retained as-is; only an `OFFLINE` status indicator is overlaid." The PNG
should show a real A1 Usage page (not the firmware 5×7 stub) with an
`OFFLINE` tag in the status slot. Update the table to reference A1's layout
and describe only the offline overlay (e.g. status text `Offline` replaces
`Online` in the header, or a small `OFFLINE` badge bottom-right). This likely
means A6's PNG must be generated from `render_device_bitmap` (like A1) plus an
overlay, not from the firmware path.

---

## 5. Page indicator dots count is dynamic, not always 4; last dot is always Settings

**Current**: A5 says "up to 10 dots" but the A6/B1/C PNGs all show 4 dots,
hardcoded. The spec never says the count comes from the backend, nor that the
last dot is fixed to the Settings page.

**Problem**: The number of dots = number of enabled pages for this device,
configured in the console (1–10). It is **not** a constant 4. Also, by
convention the **last dot is always the System/Settings page** — a fixed
anchor so the user can always reach settings. The current spec + PNGs mislead.

**Fix**: Rewrite A5 to state: dot count = console-configured enabled page
count (1–10); active dot = current page; **last dot is always the System
page** (page_id `system`), pinned regardless of how many content pages precede
it. Update A6/B1/C sample tables to use a realistic count (e.g. 5 dots:
usage, alerts, home, environment, system) with the last being system. Note
the pinning rule in the layout-constants block.

---

## 6. D1 pairing digits are too large — `ENTER CODE IN CONSOLE` touches the border

**Current**: D1 uses 5×7 digit glyphs at **scale 8** (40×56 px each), six
digits at y=112. The caption `ENTER CODE IN CONSOLE` at scale 3 sits at y=210.
From the PNG, the digits fill the vertical space and the caption is pushed
near the bottom border — visually cramped and touching the frame.

**Problem**: Scale 8 is oversized for six digits on a 300 px tall screen
(56 px digits + 56 px icon + caption + margins = overflow). The caption
collides with the border.

**Fix**: Reduce digit scale to **6** (30×42 px) or **5** (25×35 px). Re-layout
D1 vertically: icon (56×56) at y=40, digits at y=110 (scale 6), caption at
y=180 (scale 3). Keep 6 digits centered with pitch `5×scale + scale`. Update
the D1 table's Scale and x,y columns. Re-render PNG to confirm the caption
clears the bottom border (y+height < 288).

---

## 7. D2 never labels what `GD12AB34EF` is — no SSID, no "password" label

**Current**: D2 shows `WIFI SETUP` then `GD12AB34EF` (scale 3). The doc says
"the 10-char per-start WPA2 password" in prose, but the **screen itself** has
no label — a user staring at `GD12AB34EF` has no idea it's the Wi-Fi password,
and there's no SSID shown either.

**Problem**: The screen is unusable as documentation of itself. The device
starts a SoftAP; the user needs both the **SSID** (network name to join) and
the **password** (to authenticate). Showing a bare string with no label is
ambiguous.

**Fix**: Add labels to D2:
- `SSID` label + the SSID value (e.g. `GlanceDeck-AB12`)
- `PASSWORD` label + `GD12AB34EF`
Re-layout: icon (64×64) at y=40, `WIFI SETUP` title at y=120, `SSID: ...` at
y=160, `PASSWORD: ...` at y=200. Update the D2 table. Note in prose: SSID is
derived from device id; password is per-start random (10 chars, A–Z0–9). Both
shown only on this screen, never in MQTT.

---

## 8. E4a "CHECKING UPDATE" should have a loading icon

**Current**: E4a shows only the text `CHECKING UPDATE` (scale 3, centered).

**Problem**: A check-in-progress state with no visual indicator of activity
looks static/frozen. A loading spinner or animated icon signals "working".
Even on a 1-bit reflective LCD (no animation during refresh), a spinner glyph
that advances on each frame refresh conveys progress.

**Fix**: Add a Lucide `loader` or `loader-circle` icon above/below the text,
centered. Specify: icon 48×48 px, centered x, y=80; text `CHECKING UPDATE`
scale 3 at y=160. Note the icon rotates 90° per frame refresh (4-frame cycle)
to suggest activity — document the animation cadence (one step per
~500 ms refresh, not smooth). Update the E4a table. If animation is rejected
on power grounds, use a static `hourglass` icon instead and say so.

---

## 9. E4b "UPDATE READY" must show version + size

**Current**: E4b shows `UPDATE READY` + `LONG PRESS TO APPLY` / `SHORT TO
CANCEL`. No version, no size.

**Problem**: The user is asked to apply an update with zero information about
*what* they're installing. A firmware update screen without version + size is
a non-decision. The control plane already has `version` and `image_sha256`
(see mqtt-protocol.md `ota/check/state`).

**Fix**: Add to E4b:
- `VERSION` = e.g. `0.2.0` (from the release manifest)
- `SIZE` = e.g. `1.2 MB` (from `image_bytes` or content-length)
Re-layout: `UPDATE READY` title scale 3 at y=90, `VERSION 0.2.0` scale 2 at
y=130, `SIZE 1.2 MB` scale 2 at y=155, `LONG PRESS TO APPLY` y=195, `SHORT TO
CANCEL` y=220. Update the E4b table. Note the values come from the
`ota/check/state` payload (`version`, `image_sha256`/size).

---

## 10. E4d "UPDATE CHECK FAILED" must show the failure reason

**Current**: E4d shows only `UPDATE CHECK FAILED`.

**Problem**: A failure with no reason is unactionable. Per mqtt-protocol.md,
`ota/check/state` returns `status: 'failed'` with an `error_message`; the
control plane's `consume_ota_check` sends `failed` + `error_message` (e.g.
`no_compatible_release`, network timeout, signature invalid). The screen
should surface a short reason.

**Fix**: Add a reason line to E4d:
- `UPDATE CHECK FAILED` (title, scale 3, y=120)
- `<reason>` (scale 2, y=160) — e.g. `NO COMPATIBLE RELEASE`, `NETWORK
  TIMEOUT`, `SIGNATURE INVALID`
Note: reason is bounded to ≤16 uppercase chars (firmware glyph limit) and
mapped from the control plane's `error_message`. Update the E4d table. If the
reason exceeds the limit, truncate with `...` and document that.

---

## 11. F OTA status missing icon + download percentage

**Current**: F shows `SYSTEM` / `OTA: DOWNLOADING` / `KEEP POWER CONNECTED` /
`DOWNLOAD -> VERIFY -> RESTART`. No icon, no progress percentage.

**Problem**: An OTA download screen without a download icon or percentage
gives no sense of progress. The device knows bytes downloaded vs total (from
`stream_image` `content-length` + `total`); showing it reassures the user the
update is moving.

**Fix**: Add to F:
- A Lucide `download` or `arrow-down-to-line` icon (48×48, centered, y=60)
- A download progress line: `42%` or `1.2 / 2.8 MB` (scale 3, y=120), updated
  as bytes stream in
- Optionally a thin progress bar (reuse A1's bar spec, 344×8, y=140)
Re-layout: icon y=50, `SYSTEM` title y=95 (or drop title — icon implies OTA),
phase `DOWNLOADING` y=125, progress `42%` y=155, bar y=170, reminder
`KEEP POWER CONNECTED` y=210, flow line y=240. Update the F table. Note:
percentage = `round(downloaded_bytes / image_bytes × 100)`; only shown during
`Downloading` phase, hidden during `Verifying`/`Restarting`.

---

## Summary of cross-cutting changes

- **Layout constants**: drop brand row (1); re-balance header; decide border
  policy (3).
- **CJK**: add at least one CJK sample page (2).
- **A6**: re-spec as "retained A1 bitmap + offline overlay", not firmware stub
  (4).
- **A5/dots**: dynamic count from backend, last dot = System (5); update A6/
  B1/C samples accordingly.
- **D1**: shrink digits to scale 5–6, re-space (6).
- **D2**: add SSID + PASSWORD labels (7).
- **E4a**: add loading icon (8).
- **E4b**: add version + size (9).
- **E4d**: add failure reason (10).
- **F**: add download icon + percentage/progress (11).

After this list is approved, update `docs/display.md` spec tables and
regenerate `docs/image/` PNGs via `console/scripts/gen-all-screens.ts` (which
itself will need the new fields — but that's a code change, deferred).
