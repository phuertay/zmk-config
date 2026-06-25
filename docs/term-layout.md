# Hands Down Term layout (dacman56)

**Authoritative source:** [`docs/reference/kbdterm.c`](reference/kbdterm.c) — Windows keyboard layout tables extracted from `kbtrm001.dll`. All Term character mappings and adaptive reasoning in this repo must agree with that file.

## How firmware + host fit together

1. **ZMK** sends standard HID keycodes (the `L`, `E`, `W`, … keys in devicetree — QWERTY usage IDs).
2. **Windows Term layout** (`kbdterm.c`) maps each virtual key to the character that appears on screen.
3. **Firmware adaptives** (`config/adaptive.dtsi`) add morphs on top of raw key output. Adaptive triggers always key off the **HID keycode last sent**, not the Term glyph.

When reading or editing adaptives, think in **keys sent** (Term typing sequence), then check the expected **glyph** in `kbdterm.c`.

## Term virtual key → character (base layer)

From `vk_to_wchar5` in `kbdterm.c` (Base column):

| HID key sent | Term output |
|--------------|-------------|
| `L` | `l` |
| `E` | `e` |
| `W` | `w` |
| `I` | `i` |
| `N` | `n` |
| `G` | `g` |
| `M` | `m` |
| `J` | `j` |
| `SPACE` | space (Shift+Space → `_`) |

Full tables live in `kbdterm.c` (`scancode_to_vk`, `vk_to_wchar5`).

## dacman56 default layer → HID keys sent

Relevant bindings on `default_layer_hd` (see `config/dacman56.keymap`):

| Keycap area | Binding | HID sent (default tap) |
|-------------|---------|------------------------|
| Top row, 2nd alpha | `&ak_W` | `W` |
| Top row, 3rd alpha | `&ak_E` | `E` |
| Top row, 8th alpha | `&ak_I` | `I` |
| Home row, ring | `&hms RALT G` | `G` |
| Home row, middle-right | `&kp L` | `L` |
| Home row, index-right | `&hms_m_m` | `M` |
| Right thumb | `&ak_SPACE` | `SPACE` (with roll delays) |
| Left thumb | `&hmss SPACE GRAVE` | plain `SPACE` / hold `GRAVE` |

Top-row scan codes in `kbdterm.c` (physical row above home): `J` `G` `M` `F` `W` `K` …  
Home-row alpha scans: `N` `D` `P` … `A` `E` `I` `H` … `L` `C` `B` …

## Example: `ing` is typed as `lew`

Term **key sequence** for the ending **`-ing`** (keys the firmware sends, in order):

```
Keys sent:  L  →  E  →  W  →  SPACE
            │     │     │
            │     │     └─ &ak_W: akt_w_e_ng (prior E) → BSPC N G  →  "ng"
            │     └─ &ak_E: default → E
            └─ &kp L: default → L
```

| Step | HID sent | Keymap binding | Firmware morph |
|------|----------|----------------|----------------|
| 1 | `L` | `&kp L` | — |
| 2 | `E` | `&ak_E` | — |
| 3 | `W` | `&ak_W` | `akt_w_e_ng` if prior `E` |
| 4 | `SPACE` | `&ak_SPACE` | delayed per `akt_lew_*` |

**Adaptive triggers** must use these HID codes (`<L>`, `<E>`, `<W>`), not glyphs like `<I>`.

`kbdterm.c` base mapping for those VKs is `l`, `e`, `w`; `akt_w_e_ng` replaces the `e` with `ng` on step 3. Confirm `-ing` output on hardware with Term layout active.

Glyph reference from `kbdterm.c` (if you send a single key with no morph):

| Glyph | HID key |
|-------|---------|
| `i` | `I` |
| `l` | `L` |
| `n` | `N` |
| `g` | `G` |

## `ak_SPACE` (right thumb only)

Left thumb `&hmss SPACE GRAVE` stays plain space. Only `&kp SPACE` → `&ak_SPACE` on default layers.

| Trigger | Prior key sent | Window | Delay before space |
|---------|----------------|--------|---------------------|
| `akt_lew_l` | `L` | 80 ms | 40 ms (`m_space_delayed_roll`) |
| `akt_lew_e` | `E` | 80 ms | 40 ms |
| `akt_lew_w` | `W` | 80 ms | 25 ms (`m_space_delayed`) |

Tunables in `config/macros.dtsi`: `MACRO_WAIT_SPACE_DELAY`, `MACRO_WAIT_SPACE_DELAY_ROLL`, `MACRO_WAIT_SPACE_PRIOR`, `MACRO_WAIT_SPACE_ROLL`.

Space is **always sent** after the delay — never swallowed.

## `akt_w_e_ng` (Term `ew` → `ng`)

On `ak_W`, when the prior key was `E` within 200 ms:

```dts
bindings = <&kp BSPC &kp N &kp G>;
```

This is the firmware morph for the last two keys of the `lew` roll. It runs before `ak_SPACE` handles the thumb.

## Updating this doc

1. Re-export or diff against `kbtrm001.dll` → update `docs/reference/kbdterm.c`.
2. Re-check dacman56 bindings in `config/dacman56.keymap`.
3. Re-check adaptive triggers in `config/adaptive.dtsi` use **keys sent**, not expected glyphs.
