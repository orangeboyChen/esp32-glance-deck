# Display review decisions

This is the implementation checklist for the current display contract. It
replaces the older review notes that described superseded borders, icons, and
stub pages.

1. **A1 Usage header** — use a single header row: the usage icon is optically
   vertically centered with the title; no brand banner consumes the row.
2. **Alert page** — remove the dedicated Alert page/icon from the editor. Alert
   rules remain in the control plane; a user may still publish any normal page.
3. **Page indicator** — use centered 8 px circles, derived from the enabled
   page count. The active circle is filled, `system` is last, and the overlay
   is restored away after two seconds.
4. **A6 offline** — retain the exact last verified A-series bitmap. Do not
   redraw a local substitute replacement or show a spinner.
5. **B1 boot** — show `CONNECTING` while Wi-Fi/MQTT starts. A cached page is
   an offline-retention fallback, not a boot-status display.
6. **D1 pairing** — state its purpose (`PAIR DEVICE` / `ENTER CODE IN
   CONSOLE`) and use the same embedded Noto Sans SC family as A-series pages;
   no oversized separate digit style.
7. **D2 Wi-Fi setup** — show labelled SSID and password. Limit the SSID value
   column and ellipsize it with `...`; preserve the full generated password.
8. **E1/E4** — center maintenance and update-check content deliberately,
   including success and error states.
9. **F OTA** — do not draw a fake download icon. Put the percentage on a
   separate row above the A1-style meter; refresh at bounded progress changes
   so text never overlaps the bar or causes per-chunk flashing.
10. **Firmware structure** — keep local rendering in `local_screen/canvas.rs`,
    `provisioning.rs`, `maintenance.rs`, and `ota.rs`. Runtime/network/OTA
    protocol code must not contain pixel drawing loops.
    HTTPS download, signature verification, and OTA partition writes run in a
    dedicated FreeRTOS-backed worker; the UI task receives bounded status
    messages and remains the sole display owner.
