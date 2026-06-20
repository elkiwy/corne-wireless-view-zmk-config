# CLAUDE.md

Guidance for working on this repo. Read this before changing the keymap, config, or build setup.

## What this is

A **ZMK config repo** for a wireless **Corne (CRKBD) split keyboard**:
- Controllers: **nice!nano v2** (one per half)
- Displays: **nice!view** (via `nice_view_adapter`)
- Left half = **central** (talks to the host over USB/BT). Right half = **peripheral** (shows the custom art).

Firmware is compiled by **GitHub Actions** (`.github/workflows/build.yml`), not locally — the local Zephyr SDK on this machine is gone (see below).

## Repo layout

| File | Purpose |
|---|---|
| `config/corne.keymap` | The keymap — layers, behaviors, mouse keys. **Source of truth for key mappings.** |
| `config/corne.conf` | Kconfig flags (sleep, BT power, debounce, battery %, pointing/mouse). |
| `config/west.yml` | Pins which ZMK source to build against (see "Custom art" below). |
| `build.yaml` | GitHub Actions build matrix (which board + shields to build). |

The `customArt*`/`script-output/`/`test-images/` files are leftover art-generation scratch from 2023 — not used by the build, not committed. Ignore them.

## Critical: where the custom art lives

The custom **"kiwy hacker" nice!view art is NOT in upstream ZMK.** It lives in a personal fork:
- Fork: **`elkiwy/zmk`**, branch **`update-2026`**
- Files: `app/boards/shields/nice_view/widgets/art.c` (image data + descriptor) and `peripheral_status.c` (uses it on the peripheral half).

`config/west.yml` therefore points the `zmk` project at remote `elkiwy`, revision `update-2026` — **not** `zmkfirmware/main`. If you repoint west.yml at upstream, you lose the custom art.

The local clone at `~/Documents/zmk` is where that fork is checked out. Branch `backup-oct2023-art` there snapshots the original 2023 firmware state.

## Critical: modern ZMK conventions (Zephyr Hardware Model v2)

This config was migrated from 2023-era ZMK to current ZMK. Things that changed and bite you:

- **Board name in `build.yaml` is `nice_nano//zmk`** — NOT `nice_nano_v2`. Modern ZMK uses board revisions + a required `zmk` variant. `nice_nano//zmk` = nice!nano v2 (revision 2.0.0, single SoC so the SoC slot is empty → `//`).
- **Mouse/pointing** uses the official API: `#include <dt-bindings/zmk/pointing.h>`, `CONFIG_ZMK_POINTING=y` in `corne.conf`, behaviors `&mmv` (move), `&msc` (scroll), `&mkp` (click). The old `mouse.h`/`CONFIG_ZMK_MOUSE` API is dead — don't use it.
- **nice!view art** uses LVGL 9: descriptor field `.header.cf = LV_COLOR_FORMAT_I1`, and `lv_image_set_src()` (not the LVGL 8 `LV_IMG_CF_INDEXED_1BIT` / `lv_img_set_src()`). The raw 1-bit bitmap data (8-byte palette + rows) is identical between LVGL 8 and 9; only the descriptor wrapper differs.

## Mouse move speed

Controlled by `ZMK_POINTING_DEFAULT_MOVE_VAL` (upstream default `600`). It's `#define`d in `corne.keymap` **before** `#include <dt-bindings/zmk/pointing.h>` (the header uses `#ifndef`). Currently `1200` (2× default). Scroll speed is `ZMK_POINTING_DEFAULT_SCRL_VAL` (default `10`), overridable the same way.

## How to build (the only supported path)

Local builds DON'T work here (Zephyr SDK 0.15.0 directory was deleted). Everything goes through GitHub Actions:

```sh
# 1. edit config/corne.keymap or config/corne.conf
# 2. commit + push
git add config/ build.yaml
git commit -m "..."
git push                        # triggers .github/workflows/build.yml

# 3. wait for the build, then download the firmware artifact
RUN=$(gh run list --limit 1 --json databaseId -q '.[0].databaseId')
gh run watch "$RUN" --exit-status
gh run download "$RUN" -D ~/Downloads/corne-firmware
```

Artifact contains: `corne_left ...uf2`, `corne_right ...uf2`, and `settings_reset ...uf2`.

### Changing the art (requires touching the fork)

Editing art means committing to `elkiwy/zmk@update-2026` and pushing it, then re-running the config build. The keymap/conf live here; the art lives in the fork.

## Flashing

Double-tap reset on a half → it mounts as `NICENANO` → drag the matching `.uf2`.
- **Keymap/conf change only:** reflash the **left (central)** half only.
- **ZMK version / split / BT change:** reflash **both** halves.

## Recovery: "connected but not typing" (common after firmware changes)

Stale host bond. Quick fix: forget the keyboard in the host's Bluetooth settings, invoke `&bt BT_CLR` on the keyboard (NUMARROW layer, ESC-position key), re-pair.

Full reset if that fails: flash `settings_reset ...uf2` to **both** halves, then flash the normal firmware back to both, press both reset buttons together to re-pair the halves, then re-pair on the host. (settings_reset firmware has BT disabled on purpose — the keyboard won't appear in BT lists until normal firmware is reflashed.)

## Gotchas

- Build target string must be `nice_nano//zmk`; plain `nice_nano` resolves to the base Zephyr board and fails with "board is not set up for ZMK".
- `gh run view --log` sometimes returns empty in this environment; use the API instead: `gh api repos/elkiwy/corne-wireless-view-zmk-config/actions/jobs/<JOB_ID>/logs`.
- The keymap has 4 layers; layer indices are `#define`d (NORMAL=0, GAME=1, NUMARROW=2, SYMBOLS=3). Keep the defines and the `keymap {}` node order in sync.
