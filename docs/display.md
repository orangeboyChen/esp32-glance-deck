# Display — every screen on the hardware

This document enumerates **every** screen the Waveshare ESP32-S3-RLCD-4.2
panel can show. Each screen has a rendered PNG (in [image/](image/))
plus a text description of its content, text/characters, font, and layout so a
reader or LLM can reproduce it without inspecting the image.

- **Panel**: 400 × 300 px, 1-bit, MSB-first, reflective ST7305. A set bit is
  black. Every frame is exactly 15,000 bytes (`400 × 300 / 8`), format
  `mono1-msb`.
- **Two render paths**:
  1. **Console-rendered** (A-pages): the control plane rasterizes a
     `Display_document` to a verified `mono1-msb` frame via
     `render_device_bitmap` ([console/src/server/preview.ts](console/src/server/preview.ts)).
     Text = Noto Sans SC subset (Latin + CJK, GB2312), hard-thresholded to
     1-bit. Icons = Lucide SVG paths, 32×32 px.
  2. **Firmware-rendered** (B/D/E/F/G): drawn by `firmware/src/local_screen/`
     from the same Noto Sans SC source font as A-pages. The firmware embeds a
     pre-rasterized bounded ASCII subset for offline operation; it does not
     parse SVG or load a font file at runtime.

## Layout constants

- **Border**: none. The physical bezel frames both console and firmware pages.
- **No brand text**: there is no `GLANCE DECK` banner — the device does not
  self-brand.
- **Content margin**: 28 px (console). Icon + title share the header row:
  icon (28, 22), title baseline y=52. Subtitle y=74, content from y=110,
  footer rule y=266, footer text y=282.
- **CJK**: Noto Sans SC subset covers GB2312 + Latin. CJK glyphs are ~1em
  wide vs Latin ~0.5em; lines mix CJK labels with Latin/CJK values. Line wrap
  is by the console rasterizer, not the device.
- **Page indicator dots**: count = console-configured enabled page count
  (1–10), **dynamic from the backend**, not a constant. The **last dot is
  always the System page** (`page_id=system`), pinned regardless of how many
  content pages precede it. Centered at y=278, 8 px circular dots, 14 px apart.

## Icon reference

| Screen | Icon | Lucide name | Source | Size |
|---|---|---|---|---|
| A1 Usage | trend | `trending-up` | console SVG | 32×32 |
| A3 Home | home | `house` | console SVG | 32×32 |
| A4 System | monitor | `monitor` | console SVG | 32×32 |

Console icons are embedded in `render_icon`; local device states use the shared
title, text, and progress geometry instead of decorative icons.

---


## A. Normal pages (console-rendered, verified)

All A-pages are rasterized by the console (`render_device_bitmap`,
[console/src/server/preview.ts](console/src/server/preview.ts)) from a
`Display_document` into a 1-bit `mono1-msb` frame. Text uses Noto Sans SC
(GB2312 + Latin subset), hard-thresholded to black/white. Icons are Lucide SVG
paths scaled to 32×32 px. **No software border** (physical bezel frames the
screen), **no brand text**. Layout: 28 px content margin, icon (28,22) +
title y=52, subtitle y=74, content from y=110, footer rule y=266, footer
text y=282.

### A1. Usage

![Usage page](image/usage.png)

Reproducible spec — every element with font, size, color (fill), anchor (x,y),
and alignment. Font: Noto Sans SC. Colors: `#26322a` (dark, "black"),
`#627168` (mid, muted), `#9ba89f` (rule). Background `#f2f4ed`. Coordinates are
SVG baseline/anchor points on a 400×300 canvas, **no border**.

| Element | Text | Font size | Weight | Color | x,y (anchor) | Align |
|---|---|---|---|---|---|---|
| Icon | Lucide `trending-up` | 32×32 px | — | `#26322a` stroke, 2 px | (28, 22) top-left | — |
| Title | `Usage` | 26 | 400 | `#26322a` | (70, 52) | left, vertically centered with icon |
| Subtitle | `Codex subscription` | 12 | 400 | `#627168` | (28, 74) | left |
| Bar 1 label | `Day 10%` | 13 | 400 | `#627168` | (28, 110) | left |
| Bar 1 value | `79 / 776 USD` | 13 | 700 | `#26322a` | (372, 110) | right |
| Bar 1 track | rect | 344×8, rx 4 | — | `none` stroke `#26322a` 2 px | (28, 122) | — |
| Bar 1 fill | rect | 34×4, rx 2 | — | `#26322a` | (30, 124) | — |
| Bar 2 | `Week 30%` / `618 / 2054 USD` | 13 | — | as above | label (28,144), value (372,144), bar (28,156) | — |
| Bar 3 | `Month 25%` / `2068 / 8216 USD` | 13 | — | as above | label (28,178), value (372,178), bar (28,190) | — |
| Line | `Resets` / `Tomorrow` | 13 / 16 | 400 / 700 | `#627168` / `#26322a` | (28, 228) / (372, 228) | left / right |
| Footer rule | line | — | — | `#9ba89f` | (28,266)→(372,266) | — |
| Footer text | `IMMUTABLE DISPLAY RELEASE` | 10 | 400 | `#627168` | (28, 282) | left |

Progress bar fill width = `round(344 × percent / 100) - 4`, clamped ≥ 0.
Sample values reflect aggregated usage limits (micro-USD ÷ 1e6): Day 79/776,
Week 618/2054, Month 2068/8216.

**CJK variant**: the same page renders in Chinese when the console document
uses CJK — title `用量`, subtitle `订阅`, bars `今日`/`本周`/`本月`,
`重置时间`=`明天`. Noto Sans SC subset covers these; CJK glyphs are ~1em
wide so the layout engine may drop one bar row if titles overflow. The device
receives the already-rasterized bitmap; it does not know the language.


### A2–A4. Home, System, and custom pages

These share the A1 layout (icon + title/subtitle/lines/footer, no border, no
brand). Only the icon, texts, and line rows differ. Font: Noto Sans SC. Colors
as in A1 (`#26322a` dark, `#627168` muted). Line label = 13 px `#627168` left
at x=28; line value = 16 px bold `#26322a` right at x=372. First line y=110,
then 24 px row pitch.

| Screen | Image | Icon (Lucide, 32×32 at 28,22) | Title | Subtitle | Lines (label = value) |
|---|---|---|---|---|---|
| A2 Home | ![Home](image/home.png) | `house` | `Home` | `Good morning` | `Calendar`=`2 events`, `Temperature`=`24 C`, `Humidity`=`48%` |
| A3 System | ![System](image/system.png) | `monitor` | `System` | `Last verified page retained` | `Wi-Fi`=`Reconnect`, `Last update`=`09:30`, `Power`=`Battery N/A`, `Firmware`=`0.1.0` |

Alerting remains a control-plane automation feature, not a special page type
or page-editor icon. Any ordinary page may present alert content.

### A5. Page indicator overlay (2 s, after any page change)

![Page indicator overlay](image/page-indicator.png)

| Element | Spec |
|---|---|
| Dot count | **dynamic** = console-configured enabled page count for this device (1–10), fetched from the backend — not a constant |
| Last dot | **always the System page** (`page_id=system`), pinned as the rightmost dot regardless of how many content pages precede it |
| Active dot | filled 8 px circle, `#26322a` |
| Inactive dot | outlined 8 px circle, `#26322a`, fill none |
| Position | centered horizontally at y=278; spacing 14 px center-to-center |
| Duration | 2 s after any page change, then hidden |
| Disabled | critical-battery (<10%) |
| Restore | after 2 s firmware flushes the original verified cached frame |

### A6. Offline retention

![Offline retention](image/offline.png)

When offline, the **last verified console page bitmap is retained verbatim** —
the device flushes the cached 1-bit frame (an A1/A2/A3/A4 page with Noto Sans
SC text, icons, progress bars), **not** a re-drawn text screen. The panel is
never blanked, no spinner. Cached pages remain locally switchable; an uncached
requested target waits for connectivity without replacing the visible frame.

There is no local text overlay: the offline page remains precisely the same
verified A-series bitmap that was last flushed.

| Element | Source | Note |
|---|---|---|
| Page body | retained A1 console bitmap | the exact last flushed frame, not redrawn |
| Dots | A5 indicator, dynamic count, last=system | active = current cached page |


---

## B. Boot / runtime

### B1. Boot — connecting

![Boot — connecting](image/boot.png)

On boot the device first shows the local `CONNECTING` screen while Wi-Fi and
the control plane initialize. Once connected it validates and flushes the
active page. Cached pages remain an offline-retention fallback, not an
ambiguous boot-progress screen.

| Element | Source | Note |
|---|---|---|
| Title | `CONNECTING` | common local title row |
| Subtitle | `WIFI AND CONTROL PLANE` | common local subtitle row |
| Body | `PLEASE WAIT` | centered local text |

### B2. Boot — no release yet → pairing (see D1)

A newly enrolled device with no cached release falls through to enrollment and
shows the D1 pairing screen.

---

## C. Local navigation (short KEY)

![Local navigation](image/navigation.png)

Trigger: short KEY press advances to the next enabled cached page, wrapping
around. Firmware flushes the verified target frame, overlays the A5 dots for
2 s, then reports the confirmed page to MQTT only after the flush succeeds.

| Element | Source | Note |
|---|---|---|
| Page body | the newly-selected cached page bitmap | A1/A2/A3/A4-style, not redrawn |
| Dots | A5 indicator, dynamic count, last=system | active = newly selected page |

---

## D. Enrollment & provisioning (firmware-rendered)

All D/E/G screens use the common firmware `local_screen` canvas and the same
Noto Sans SC typeface as A-pages, pre-rasterized into a bounded embedded ASCII
subset. No SVG parsing, runtime font loading, icon bitmap, or software border
is used.

### D1. Pairing code

![Pairing screen](image/pairing.png)

The pairing page states its purpose in the common local title/subtitle rows:
`PAIR DEVICE` / `ENTER CODE IN CONSOLE`. Its short-lived code is generated
locally and never sent to MQTT. It uses the same Noto Sans SC typeface as the
console and other local pages, at the large embedded size.

| Element | Text/Content | Glyph | Scale (→ px) | x,y (top-left) | Align |
|---|---|---|---|---|---|
| Title | `PAIR DEVICE` | Noto Sans SC | 26 px | (28, 26) | left |
| Subtitle | `ENTER CODE IN CONSOLE` | Noto Sans SC | 14 px | (28, 70) | left |
| Code | six digits | Noto Sans SC | 42 px | centered, y=120 | center |
| Hint | `OPEN CONSOLE TO CONTINUE` | Noto Sans SC | 14 px | centered, y=184 | center |

Each digit uses its Noto Sans SC advance width in the embedded 42 px raster;
the full six-digit value is centered without a custom digit pitch, separate
icon, or oversized digit style.

### D2. Wi-Fi setup credentials

![Wi-Fi setup screen](image/wifi.png)

Drawn by firmware. Trigger: station reconnect fails or first boot with no
Wi-Fi — device starts a WPA2 SoftAP and shows its SSID and password. Both are
labeled on screen so the user knows which is which. Per-start, never in MQTT,
never in normal display docs.

| Element | Text/Content | Glyph | Scale (→ px) | x,y | Align |
|---|---|---|---|---|---|
| Title | `WIFI SETUP` | Noto Sans SC | 26 px | (28, 26) | left |
| Subtitle | `JOIN THIS NETWORK` | Noto Sans SC | 14 px | (28, 70) | left |
| SSID label/value | `SSID` / `GlanceDeck-AB12` | Noto Sans SC | 14 px | y=112, label x=28 value x=92 | left |
| Password label | `PASSWORD` | Noto Sans SC | 14 px | (28, 164) | left |
| Password value | `GD12AB34EF` (10 chars, A–Z0–9) | Noto Sans SC | 26 px | (28, 188) | left |

SSID is derived from the device MAC (e.g. `GlanceDeck-<4 hex>`); password is
per-start random. The SSID value column is capped at 280 px and overlong names
are ellipsized with `...`; the password is never truncated. Both appear only
on this screen.

---

## E. Maintenance flow (firmware-rendered, ≤16 uppercase chars)

Long KEY on a normal page enters maintenance. Short KEY cancels → last normal
page. All maintenance and update-check content uses the shared local header,
row, and centered text layout; no screen is offset or decorated with an icon.

### E1. Maintenance overview (1st long press)

![Maintenance screen](image/maintenance.png)

| Element | Text/Content | Glyph | Scale (→ px) | x,y | Align |
|---|---|---|---|---|---|
| Title | `MAINTENANCE` | Noto Sans SC | 26 px | centered, y=72 | center |
| Subtitle | `SHORT: CHECK UPDATE` | Noto Sans SC | 14 px | centered, y=116 | center |
| Action | `LONG: WIFI SETUP` | Noto Sans SC | 18 px | centered, y=154 | center |

### E2. Wi-Fi setup confirmation (2nd long press)

![Wi-Fi setup confirmation](image/maintenance-hold.png)

| Element | Text | Glyph | Scale (→ px) | x,y | Align |
|---|---|---|---|---|---|
| Title | `WIFI SETUP` | Noto Sans SC | 26 px | (28,26) | left |
| Hint 1 | `LONG AGAIN TO START` | Noto Sans SC | 18 px | centered, y=128 | center |
| Hint 2 | `SHORT TO CANCEL` | Noto Sans SC | 14 px | centered, y=172 | center |

### E3. Starting Wi-Fi setup (3rd long press → restart)

![Starting Wi-Fi setup](image/maintenance-starting.png)

| Element | Text | Glyph | Scale (→ px) | x,y | Align |
|---|---|---|---|---|---|
| Title | `WIFI SETUP` | Noto Sans SC | 26 px | (28,26) | left |
| Subtitle | `STARTING ACCESS POINT` | Noto Sans SC | 14 px | (28,70) | left |
| Body | `RESTARTING` | Noto Sans SC | 18 px | centered, y=144 | center |

Device then restarts into the SoftAP (D2).

### E4. Local OTA check sub-states (short press from maintenance)

**E4a — immediately after the check request:**

![Checking update](image/ota-checking.png)

| Element | Text/Content | Glyph | Scale (→ px) | x,y | Align |
|---|---|---|---|---|---|
| Title | `SYSTEM UPDATE` | Noto Sans SC | 26 px | centered, y=62 | center |
| Subtitle | `CHECKING FOR RELEASE` | Noto Sans SC | 14 px | centered, y=110 | center |
| Text | `CHECKING` | Noto Sans SC | 26 px | centered, y=146 | center |

There is no spinner animation: repeatedly updating a reflective panel wastes
power and makes the background flicker.

**E4b — control plane returned a candidate (not yet applied):**

![Update ready](image/ota-ready.png)

| Element | Text/Content | Glyph | Scale (→ px) | x,y | Align |
|---|---|---|---|---|---|
| Title | `SYSTEM UPDATE` | Noto Sans SC | 26 px | centered, y=54 | center |
| Subtitle | `UPDATE READY` | Noto Sans SC | 14 px | centered, y=102 | center |
| Version | `VERSION 0.2.0` | Noto Sans SC | 14 px | centered, y=130 | center |
| Hint 1 | `LONG TO APPLY` | Noto Sans SC | 18 px | centered, y=178 | center |
| Hint 2 | `SHORT TO CANCEL` | Noto Sans SC | 14 px | centered, y=218 | center |

Version comes from the `ota/check/state` payload. A separate long press
confirms; a short press cancels back to the last normal page. The signed image
size is learned from the HTTPS response only when the download begins.

**E4c — no newer release:**

![Up to date](image/ota-uptodate.png)

| Element | Text | Glyph | Scale (→ px) | x,y | Align |
|---|---|---|---|---|---|
| Text | `UP TO DATE` | Noto Sans SC | 26 px | centered, y=142 | center |

**E4d — control plane failed:**

![Update check failed](image/ota-failed.png)

| Element | Text/Content | Glyph | Scale (→ px) | x,y | Align |
|---|---|---|---|---|---|
| Title | `SYSTEM UPDATE` / `CHECK FAILED` | Noto Sans SC | 26 / 14 px | centered, y=54 / y=102 | center |
| Reason | `<reason>` (≤16 chars) | Noto Sans SC | 14 px | centered, y=170 | center |

The reason is mapped from the control plane's `error_message` and bounded to
≤16 uppercase ASCII chars (firmware glyph limit). Examples: `NO COMPATIBLE
RELEASE`, `NETWORK TIMEOUT`, `SIGNATURE INVALID`. If the message exceeds the
limit it is truncated with `...`.

---

## F. OTA status (remote + locally confirmed, shared phases)

![OTA status](image/ota-status.png)

Drawn by firmware. Phases: `Downloading` → `Verifying` → `Restarting`, then
`Checking`, `Complete`, `Rolled back`, or a concrete failure. OTA is refused
below 30% battery unless external power (`power_unsafe_for_ota`). The device
uses a status layout rather than a hand-drawn download icon.

| Element | Text/Content | Glyph | Scale (→ px) | x,y | Align |
|---|---|---|---|---|---|
| Title | `SYSTEM UPDATE` | Noto Sans SC | 26 px | (28,26) | left |
| Phase | `DOWNLOADING` | Noto Sans SC | 14 px | (28,70) | left |
| Progress | `PROGRESS` / `42%` | Noto Sans SC | 14 px | row y=116, left/right | — |
| Progress bar track | rect 344×8 | — | — | (28,142), stroke 2 px | — |
| Progress bar fill | rect `round(344×42/100)-4`×4 | — | — | (30,144) | — |
| Reminder | `KEEP POWER CONNECTED` | Noto Sans SC | 14 px | centered, y=194 | center |
| Flow | `DOWNLOAD VERIFY RESTART` | Noto Sans SC | 14 px | centered, y=240 | center |

Progress percentage = `round(downloaded_bytes / image_bytes × 100)`, shown only
during `Downloading`; hidden during `Verifying`/`Restarting` (those phases show
only the phase name). It is refreshed at 10% boundaries and 100%, avoiding
per-chunk background flashing. The HTTPS download, manifest verification, and
OTA-partition writer run in a dedicated FreeRTOS-backed Rust worker; the UI task
is the sole owner of the display and receives bounded OTA state messages.

---

## G. Error frame

![Error frame](image/error.png)

Drawn by firmware. Never a bare `Error` — always paired with the specific
corrective action and, where available, a short reason. Audio is
supplementary; color is never the only cue (monochrome panel).

| Element | Text/Content | Glyph | Scale (→ px) | x,y | Align |
|---|---|---|---|---|---|
| Title | `<failure>` (≤16 chars) | Noto Sans SC | 26 px | centered, y=90 | center |
| Action | `<corrective action>` (≤16 chars) | Noto Sans SC | 18 px | centered, y=142 | center |
| Reason | `<reason>` (≤16 chars, optional) | Noto Sans SC | 18 px | centered, y=184 | center |

Sample: title `WIFI CONN FAILED`, action `REOPEN SETUP`, reason `AUTH FAILED`.
All strings bounded to ≤16 uppercase ASCII chars; longer messages truncate
with `...`.

---

## State transition map

```
                       short KEY / show_page
   Normal page  ─────────────────────────────────►  verified target
   (cached)                                              │
       ▲                                                 │ 2 s dots
       │                                                 ▼
       │                                            normal page
       │ long KEY
       ▼
   MAINTENANCE ──long──► HOLD 3X ──long──► STARTING ──restart──► (D2 Wi-Fi setup)
       │                      │
       │ short                │ short
       │                      │
       ▼                      ▼
   CHECKING UPDATE       last normal page
       │
       ├──► UPDATE READY ──long──► OTA phases (F)
       ├──► UP TO DATE
       └──► UPDATE CHECK FAILED
```

---

## What the hardware never shows

```
- indefinite spinner / blank panel while offline
- color-only status cue
- arbitrary prose on pairing / Wi-Fi / maintenance frames
- credentials in MQTT state, non-serial logs, or the System page
- invented battery percentage (Battery unavailable until carrier installed)
```

---

## Sources

```
firmware/src/local_screen/           pairing_code_frame / maintenance_frame / wifi_setup_frame
                                     / error_frame / embedded Noto Sans SC glyph subset
firmware/src/main.rs                 boot flush, maintenance long-press sequence, OTA-check states
firmware/src/display.rs              DisplayRelease / DisplayPage validation, 400x300 / 15000B / mono1-msb
firmware/src/rlcd.rs                 RlcdRenderer::flush_frame, show_pairing_code
firmware/docs/display.md             ASCII mockups (companion file)
firmware/docs/hardware-interaction.md   KEY gestures, power states, OTA thresholds
firmware/docs/display-assets.md      CJK font and release contract
```
