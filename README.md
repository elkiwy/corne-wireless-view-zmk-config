# Corne Wireless — ZMK Config

Personal ZMK firmware for a wireless **Corne (CRKBD)** split keyboard: **nice!nano v2** controllers + **nice!view** displays, with a custom "kiwy hacker" display image and mouse emulation.

Firmware is built in the cloud by **GitHub Actions** — no local toolchain needed.

---

## Update the firmware (quick guide)

1. **Edit the mapping** in `config/corne.keymap` (keys/layers) or `config/corne.conf` (settings).
2. **Commit & push** — this triggers the build automatically:
   ```sh
   git add config/
   git commit -m "describe your change"
   git push
   ```
3. **Download the built firmware** once the Actions run finishes:
   ```sh
   RUN=$(gh run list --limit 1 --json databaseId -q '.[0].databaseId')
   gh run watch "$RUN" --exit-status        # waits until the build is done
   gh run download "$RUN" -D ~/Downloads/corne-firmware
   ```
   (Or just open the repo on GitHub → **Actions** → latest run → download the `firmware` artifact.)
4. **Flash** (see below).

> Changing the **display image** is different — that lives in the `elkiwy/zmk` fork, not here. See `CLAUDE.md`.

### Flashing
Double-tap the **reset** button on a half → it shows up as a USB drive named **`NICENANO`** → drag the matching `.uf2` onto it. It reboots itself.

- **Keymap or config change:** only the **left** half needs reflashing.
- **Bigger changes (ZMK version, Bluetooth):** reflash **both** halves.

You'll find these files in the downloaded artifact:
| File | Flash to |
|---|---|
| `corne_left …uf2` | Left half |
| `corne_right …uf2` | Right half |
| `settings_reset …uf2` | Recovery only (see Troubleshooting) |

---

## Keymap cheatsheet

4 layers. **Auto-shift** is on the letters/numbers: **tap = normal, hold (~250 ms) = shifted** (so hold `a` → `A`).

Layers are reached with the **inner thumb keys**:
- Hold **left inner thumb** → **Nums/Arrows** layer
- Hold **right inner thumb** → **Symbols** layer

### Layer 0 — NORMAL (default)
```
 TAB  │  Q   W   E   R   T   ║   Y   U   I   O   P   │ BSPC
 ESC  │  A   S   D   F   G   ║   H   J   K   L   ;   │  '
 SHFT │  Z   X   C   V   B   ║   N   M   ,   .   /   │ CTRL
                  GUI │ NUMS │ SPACE  ║  ENTER │ SYMB │ ALT
```
*(letters + `; ' , . /` are auto-shift: hold for the shifted symbol/capital)*

### Layer 1 — GAME (toggle on/off from the Nums layer)
Same as NORMAL but **without auto-shift** (so holding a key repeats it, as games expect).
```
 TAB  │  Q   W   E   R   T   ║   Y   U   I   O   P   │ BSPC
 ESC  │  A   S   D   F   G   ║   H   J   K   L   ;   │  '
 SHFT │  Z   X   C   V   B   ║   N   M   ,   .   /   │ CTRL
                  GUI │ NUMS │ SPACE  ║  ENTER │ SYMB │ ALT
```

### Layer 2 — NUMS / ARROWS (hold left inner thumb)
```
 TAB  │  1   2   3   4   5   ║   6   7   8   9   0   │ BSPC
BTCLR │ BT0 BT1 BT2 BT3 BT4  ║  ←   ↓   ↑   →    ·   │  ·
 SHFT │  ·   ·   ·   ·   ·   ║   ·   ·   ·   ·   ·   │ GAME⇄
                  GUI │ ──── │ SPACE  ║  ENTER │  ·   │ ALT
```
- **Numbers** are auto-shift (hold `1` → `!`, etc.).
- **Bluetooth:** `BTCLR` clears the current pairing; `BT0`–`BT4` switch between 5 paired hosts.
- **Arrows:** ← ↓ ↑ → on the right hand.
- **GAME⇄** (bottom-right): toggles the Game layer on/off.

### Layer 3 — SYMBOLS + MOUSE (hold right inner thumb)
```
 TAB  │  !   @   #   $   %   ║   ^   &   *   (   )   │ BSPC
 CTRL │ RCLK ↑  LCLK SCRL↑ · ║   -   =   [   ]   \   │  `
 SHFT │  ←   ↓   →  SCRL↓  · ║   _   +   {   }   |   │  ~
                  GUI │ ──── │ SPACE  ║  ENTER │  ·   │ ALT
```
**Mouse cluster (left hand):**
- **Move pointer:** `S`=↑, `X`=↓, `Z`=←, `C`=→ (an inverted-T like arrow keys)
- **Click:** `D`=left click, `A`=right click
- **Scroll:** `F`=scroll up, `V`=scroll down

Pointer speed is 2× the ZMK default (tunable via `ZMK_POINTING_DEFAULT_MOVE_VAL` in `config/corne.keymap`).

---

## Troubleshooting

**Keyboard shows as connected but nothing types** (common after a firmware update — stale Bluetooth bond):
1. Forget the keyboard in your computer's Bluetooth settings.
2. On the keyboard, hold the **left inner thumb** (Nums layer) and press the **ESC-position key** (left of `A`) — that's `BT_CLR`, it clears the current pairing.
3. Pair again from the computer.

**Still broken / the two halves won't talk to each other** — full reset:
1. Flash `settings_reset …uf2` to **both** halves (double-tap reset, drag the file).
2. Flash `corne_left …uf2` and `corne_right …uf2` back to their halves.
3. Press both reset buttons at about the same time so the halves re-pair.
4. Forget + re-pair on the computer.

> Note: while the reset firmware is on the halves, the keyboard intentionally has Bluetooth **off** and won't appear in any Bluetooth list until the normal firmware is flashed back.

---

For deeper details (architecture, the custom-art fork, modern-ZMK gotchas), see **`CLAUDE.md`**.
