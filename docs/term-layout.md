# Hands Down Term layout (dacman56)

**Authoritative source:** [`docs/reference/kbdterm.c`](reference/kbdterm.c) — Windows keyboard layout tables from `kbtrm001.dll`.

## The one rule

**Think in QWERTY keycap labels** (what is printed on the keys you press), **not** in the characters that appear on screen.

Term remaps each physical key position (scan code) to a different output character. The label on the cap and the glyph typed are usually **different**.

| Wrong | Right |
|-------|-------|
| “To type `test`, press `T` `E` `S` `T`.” | “To type `test`, press `` ` `` `K` `S` `` ` ``.” |
| “To type `i`, press `I`.” | “To type `i`, press `L`.” |
| “HID `T` always means `t`.” | “The `` ` `` cap (scan `0x29`) means `t`.” |

## How to find the keys for a word

For each character you want on screen:

1. Look up which **scan code** produces it in `kbdterm.c` (`scancode_to_vk` → `vk_to_wchar5`, Base column).
2. Convert that scan code to the **US QWERTY keycap label** at that position (table below).
3. Write down the **keycap labels in order** — that is your typing sequence.

Firmware adaptives (see below) may change what the host receives for some rolls; they still key off **which key was pressed last** (the HID / keycap side), not the glyph.

## US QWERTY scan code → keycap label

Standard AT set 1 positions used by `kbdterm.c`:

| Scan | Keycap | Scan | Keycap | Scan | Keycap |
|------|--------|------|--------|------|--------|
| `0x10` | Q | `0x1E` | A | `0x2C` | Z |
| `0x11` | W | `0x1F` | S | `0x2D` | X |
| `0x12` | E | `0x20` | D | `0x2E` | C |
| `0x13` | R | `0x21` | F | `0x2F` | V |
| `0x14` | T | `0x22` | G | `0x30` | B |
| `0x15` | Y | `0x23` | H | `0x31` | N |
| `0x16` | U | `0x24` | J | `0x32` | M |
| `0x17` | I | `0x25` | K | `0x33` | , |
| `0x18` | O | `0x26` | L | `0x29` | `` ` `` |
| `0x19` | P | `0x27` | ; | | |

## Term: keycap label → output (base layer)

Derived from `kbdterm.c` (scan → VK → character). Alpha keys only — see `kbdterm.c` for punctuation, AltGr, dead keys.

| Press cap | Output | Press cap | Output | Press cap | Output |
|-----------|--------|-----------|--------|-----------|--------|
| Q | j | A | r | Z | x |
| W | g | S | s | X | z |
| E | m | D | n | C | l |
| R | f | F | d | V | b |
| T | w | G | p | | |
| Y | k | H | h | | |
| U | y | J | a | | |
| I | c | K | e | | |
| O | u | L | **i** | | |
| P | o | ; | h | | |
| `` ` `` | **t** | M | u | | |
| , | **o** | | | | |

**Shift+Space** → `_` (see `VK_SPACE` in `kbdterm.c`).

## Worked examples

### `test` → `` `KS` ``

| Output | Scan | Press cap |
|--------|------|-----------|
| t | `0x29` | `` ` `` |
| e | `0x25` | K |
| s | `0x1F` | S |
| t | `0x29` | `` ` `` |

### `soup` → `S,M,G`

| Output | Scan | Press cap |
|--------|------|-----------|
| s | `0x1F` | S |
| o | `0x33` | , |
| u | `0x32` | M |
| p | `0x22` | G |

(comma is the **`,`** keycap, not the letter `C`.)

### `ing` → `lew` (+ firmware)

| Output | Press cap | Notes |
|--------|-----------|--------|
| i | L | scan `0x26` → `I` → `i` |
| n | (from roll) | `akt_w_e_ng` on `ak_W` after `E` |
| g | (from roll) | same morph: `BSPC` `N` `G` |
| space | right thumb | `&ak_SPACE` with `akt_lew_*` delays |

Keycap sequence: **`L` → `E` → `W` → space** (written **`lew`**).

```
Caps:     L  →  E  →  W  →  SPACE
          │     │     │
          │     │     └─ &ak_W + akt_w_e_ng (prior E) → ng
          │     └─ &ak_E
          └─ &kp L
```

## Firmware + host

| Layer | Role |
|-------|------|
| **Term (`kbdterm.c`)** | Maps key **positions** (scan codes) to glyphs on the host. |
| **ZMK keymap** | Each physical position has a binding (`&kp L`, `&ak_W`, …) that sends an HID keycode. |
| **Adaptives (`adaptive.dtsi`)** | Morphs and delayed space; triggers use **HID keycodes of keys pressed** (`<L>`, `<E>`, `<W>`), matching keycap names when bindings are plain `&kp *`. |

When editing adaptives, ask: **which keycap label did I just press?** — not “what letter appeared?”

### dacman56 bindings (default layer, main rolls)

| Keycap | Binding | Default HID sent |
|--------|---------|------------------|
| L | `&kp L` | `L` |
| E | `&ak_E` | `E` |
| W | `&ak_W` | `W` |
| S | `&ak_S` | `S` |
| G | `&hms RALT G` | `G` |
| M | `&hms_m_m` | `M` |
| `` ` `` | `&hmss SPACE GRAVE` (hold) | `GRAVE` |
| right thumb | `&ak_SPACE` | `SPACE` (delayed after `L`/`E`/`W`) |

If a binding sends the same HID code as the keycap label, adaptive `trigger-keys` use that letter (e.g. `<L>` for the `L` key).

## `ak_SPACE` (right thumb)

| Trigger | Prior key pressed | Window | Delay |
|---------|-------------------|--------|-------|
| `akt_lew_l` | `L` | 80 ms | 40 ms |
| `akt_lew_e` | `E` | 80 ms | 40 ms |
| `akt_lew_w` | `W` | 80 ms | 25 ms |

Tunables in `config/macros.dtsi`: `MACRO_WAIT_SPACE_DELAY`, `MACRO_WAIT_SPACE_DELAY_ROLL`, `MACRO_WAIT_SPACE_PRIOR`, `MACRO_WAIT_SPACE_ROLL`.

Space is always sent after the delay — never swallowed.

## `akt_w_e_ng`

On `ak_W`, prior `E` within 200 ms:

```dts
bindings = <&kp BSPC &kp N &kp G>;
```

Term `ew` roll step of `lew` → `ng`.

## Updating

1. Refresh [`docs/reference/kbdterm.c`](reference/kbdterm.c) from `kbtrm001.dll`.
2. Rebuild the keycap → output table from `scancode_to_vk` + `vk_to_wchar5`.
3. Re-check `config/adaptive.dtsi` triggers against **keycap / HID keys pressed**.
4. Re-check `config/dacman56.keymap` bindings.
