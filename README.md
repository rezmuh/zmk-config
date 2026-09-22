# zmk-config

ZMK firmware configuration for my split ergonomic keyboards, sharing a single
Colemak-DH keymap with home-row mods.

## Keyboards

| Keyboard | Keys | Shield | Board | Firmware artifacts |
|---|---|---|---|---|
| Ferris/Sweep | 34 (3×5 + 2 thumbs per side) | `cradio_left` / `cradio_right` | nice!nano v2 | `cradio_left`, `cradio_right` |
| Chocofi | 36 (3×5 + 3 thumbs per side) | `corne_left` / `corne_right` + `five_column_transform` | nice!nano v2 | `chocofi_left`, `chocofi_right` |

Both run on nice!nano v2 over BLE, left half as central, with sleep enabled,
max TX power, and 1ms/5ms press/release debounce. The `settings_reset`
artifact works on both boards (it's board-only, no shield).

## How the config works

```
config/cradio.keymap ─┐   (thin shim: 4 thumb bindings per layer)
                       ├─► config/common.dtsi   (all layers, behaviors, combos)
config/corne.keymap  ──┘   (thin shim: 6 thumb bindings per layer)
```

Key positions **0–29 (the 3×5 rows) are identical on both shields** — only the
thumb row differs. So `common.dtsi` holds the entire keymap, and each shim
just defines three macros with that keyboard's thumb bindings before including
it:

```c
#define T_BASE  &lt NAV TAB &kp SPC    &kp RET &lt SYM BSPC      // cradio: 4 thumbs
#define T_BASE  &kp ESC &lt NAV TAB &kp SPC  &kp RET &lt SYM BSPC &kp DEL  // corne: 6 thumbs
#include "common.dtsi"
```

The Chocofi's two extra keys are the **outermost thumb keys (positions 30 and
35)**; the main four thumbs sit on 31/32/33/34, the positions equivalent to
the Sweep's 30–33.

> **Naming gotcha:** ZMK picks config files by *shield* name, so the Chocofi's
> files are `corne.conf` / `corne_left.conf` / `corne_right.conf` — a
> `chocofi.conf` would be silently ignored. Kconfig has no include mechanism,
> so `cradio.conf` and `corne.conf` must be kept in sync manually (only the
> keyboard name differs).

## Keymap

Base layout is **Colemak-DH** with home-row mods via a custom hold-tap
behavior (`am`: 150ms tapping term, tap-preferred, retro-tap, 125ms
quick-tap).

### BASE

```
╭─────┬─────┬─────┬─────┬─────╮   ╭─────┬─────┬─────┬─────┬─────╮
│  Q  │  W  │  F  │  P  │  B  │   │  J  │  L  │  U  │  Y  │  ;  │
│ A ◆ │ R ⎇ │ S ⎈ │ T ⇧ │  G  │   │  M  │ N ⇧ │ E ⎈ │ I ⎇ │ O ◆ │
│  Z  │  X  │  C  │  D  │  V  │   │  K  │  H  │  ,  │  .  │  /  │
╰─────┴─────┼─────┼─────┤     ├─────┼─────┼─────┴─────┴─────╯
            │ TAB │ SPC │     │ RET │ BSPC│
            │ NAV │     │     │     │ SYM │
            ╰─────┴─────╯     ╰─────┴─────╯
```

- Home-row mods (hold): `◆` GUI, `⎇` Alt, `⎈` Ctrl, `⇧` Shift —
  GACS order, mirrored: left hand `GUI ALT CTL SFT` (left variants), right
  hand `SFT CTL ALT GUI` (right variants). The NAV and SYM layers keep the
  same mod order on their home rows.
- `NAV` / `SYM` thumbs are layer-taps (`&lt`): **hold** for the layer, **tap**
  for the key (TAB / BSPC). 150ms tapping term, matching the home-row mods.
- On the Chocofi, the outermost thumbs are plain `ESC` (left) and `DEL`
  (right); the Sweep doesn't have those keys.

### NAV (hold left thumb)

```
╭─────┬─────┬─────┬─────┬─────╮   ╭─────┬─────┬─────┬─────┬─────╮
│ BT1 │ BT2 │  ⏮  │  ⏯  │  ⏭  │   │ VOL-│ MUTE│ VOL+│ DEL │ BSPC│
│ GUI │ ALT │ CTL │ SFT │ ESC │   │     │  ←  │  ↓  │  ↑  │  →  │
│     │     │     │S-TAB│ TAB │   │     │     │ PGDN│ PGUP│     │
╰─────┴─────┼─────┼─────┤     ├─────┼─────┼─────┴─────┴─────╯
            │     │     │     │     │     │
            ╰─────┴─────╯     ╰─────┴─────╯
```

### SYM (hold right thumb)

```
╭─────┬─────┬─────┬─────┬─────╮   ╭─────┬─────┬─────┬─────┬─────╮
│     │     │     │     │     │   │  \  │  7  │  8  │  9  │  0  │
│ GUI │ ALT │ CTL │ SFT │  [  │   │  ]  │  4  │  5  │  6  │  -  │
│     │     │     │     │  `  │   │  '  │  1  │  2  │  3  │  =  │
╰─────┴─────┼─────┼─────┤     ├─────┼─────┼─────┴─────┴─────╯
            │     │     │     │     │     │
            ╰─────┴─────╯     ╰─────┴─────╯
```

Right hand becomes a numpad; left hand holds brackets and quotes.

### Bluetooth profiles

ZMK supports up to 5 BLE profiles — one paired device per profile. `BT1` /
`BT2` on the NAV layer (`&bt BT_SEL 0` / `&bt BT_SEL 1`) switch instantly
between device #1 and device #2.

**Pairing:** select a profile, then pair the device from its Bluetooth
settings; repeat for the other profile. Switching applies on the left
(central) half only — the right half follows automatically.

**Clearing:** the `BT_CLR` combo clears only the *active* profile — select
the broken profile first, then clear, then re-pair.

### Combos

| Combo | Keys | Layer | Action |
|---|---|---|---|
| Caps Word | positions 4+5 (top inner corners, `B`+`J`) | BASE | `&caps_word` |
| BT Clear | positions 20+29 (bottom outer corners) | NAV | `&bt BT_CLR` — clears the current BLE profile |
| ESC | both left thumbs | BASE | Sweep only — the Chocofi has a dedicated ESC thumb |
| DEL | both right thumbs | BASE | Sweep only — the Chocofi has a dedicated DEL thumb |

## Build & flash

Firmware builds via GitHub Actions on every push
(`zmkfirmware/zmk/.github/workflows/build-user-config.yml@main`); download the
artifacts from the latest run.

1. Enter the bootloader on one half — double-tap the reset button, or (on
   boards without a button, like the Sweep) short the RST pads to GND twice.
   The half mounts as a USB drive.
2. Copy the matching `.uf2` (`cradio_left` / `chocofi_left` for the left half,
   etc.) onto the drive. It reboots automatically.
3. Repeat for the other half. Flash `settings_reset` first (then re-flash the
   real firmware) when switching keyboards or clearing stale pairing data.
4. Pair: the **left** half is the BLE central — pair it with the host; the
   right half pairs to the left automatically. Use the `BT_CLR` combo
   (NAV layer, both bottom-outer corner keys) to clear a profile.

## Customizing

- **Change keys, layers, behaviors, or combos** → edit `config/common.dtsi`.
  One edit applies to both keyboards on the next build.
- **Change thumb keys for one keyboard** → edit the `T_*` macros in that
  keyboard's shim (`cradio.keymap` / `corne.keymap`).
- **Change radio/power/debounce settings** → edit both `config/cradio.conf`
  and `config/corne.conf` (keep them in sync).

## Adding another keyboard

1. Add `config/<shield>.keymap` defining `T_BASE` / `T_NAV` / `T_SYM` with the
   right number of thumb bindings, then `#include "common.dtsi"`. Positions
   0–29 must match the 3×5 layout above for the shared keymap to make sense.
2. Add `config/<shield>.conf` (copy from an existing one) plus
   `<shield>_left.conf` / `<shield>_right.conf` for split boards.
3. Add the board/shield entries to `build.yaml`.
4. Push and download the artifacts from the Actions run.
