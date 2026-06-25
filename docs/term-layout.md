# Hands Down Term layout (dacman56)

**Authoritative source:** [`docs/reference/kbdterm.c`](reference/kbdterm.c) — Windows keyboard layout tables from `kbtrm001.dll`.

## Labels → layout → output

Everything goes through this pipeline:

```
Keycap label pressed  →  Term layout (kbdterm.c)  →  character on screen
       (E, W, D, …)         maps label → glyph            (m, g, n, …)
```

- **Labels** = QWERTY keycap names / HID keycodes the firmware sends (`&kp D`, `&kp W`, …).
- **Layout** = `kbdterm.c` on the host (plus Windows scan→VK if applicable).
- **Output** = the glyph you see.

**Never read morph bindings as output letters.** Read them as **labels** and look up what Term makes of each:

| Morph binding | Label | Term output |
|---------------|-------|-------------|
| `&kp D` | D | `n` |
| `&kp W` | W | `g` |
| `&kp E` | E | `m` |
| `&kp L` | L | `i` |

So `akt_w_e` with `BSPC` `D` `W` is **not** “type d and w” — it is “send labels **D** and **W**, which Term outputs as **`ng`**” (after backspace).

Adaptives and morphs are specified in **labels**. Triggers use **labels** of keys pressed (`<E>`, `<W>`, …). The layout converts labels to output.

## The one rule

**Think in keycap labels you press and send**, not in the characters that appear on screen.

| Wrong | Right |
|-------|-------|
| “`&kp D` types `d`.” | “Label **D** → Term outputs **`n`**.” |
| “`akt_w_e` sends dw.” | “`akt_w_e` sends labels **D** **W** → output **`ng`**.” |
| “To type `test`, press `T` `E` `S` `T`.” | “To type `test`, press `` ` `` `K` `S` `` ` ``.” |

## How to find labels for a word

For each output character you want:

1. Find which **label** produces it in the keycap → output table below (from `kbdterm.c`).
2. Write those **labels in order** — that is the typing sequence.
3. If a firmware **morph** already maps a label roll to the right output (e.g. `ew` → `ng`), use the morph’s label trigger, not the output letters.

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

## Term: label → output (base layer)

From `kbdterm.c` (`scancode_to_vk` → `vk_to_wchar5`, Base column):

| Label | Output | Label | Output | Label | Output |
|-------|--------|-------|--------|-------|--------|
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

**Shift+Space** → `_`.

## Worked examples

### `test` → `` `KS` ``

| Output | Label |
|--------|-------|
| t | `` ` `` |
| e | K |
| s | S |
| t | `` ` `` |

### `soup` → `S,M,G`

| Output | Label |
|--------|-------|
| s | S |
| o | , |
| u | M |
| p | G |

### `ng` → `ew` (adaptive morph)

| Step | Label | Firmware | Output |
|------|-------|----------|--------|
| 1 | E | `&ak_E` | `m` |
| 2 | W | `&ak_W` + **`akt_w_e`** | **`ng`** |

Roll: **`E` → `W`**. Morph on `ak_W` when prior label is `E`:

```dts
akt_w_e {
    trigger-keys = <E>;
    bindings = <&kp BSPC &kp D &kp W>;  /* labels D,W → ng */
};
```

### `ing` → `ldw` (no morph)

| Output | Label | Binding |
|--------|-------|---------|
| i | L | `&kp L` |
| n | D | `&kp D` |
| g | W | `&ak_W` |

**`L` → `D` → `W`**. Each label maps through Term directly.

## Firmware layers

| Layer | Role |
|-------|------|
| **Term (`kbdterm.c`)** | label → output on the host |
| **ZMK keymap** | physical position → binding → label sent |
| **Adaptives** | morphs and delayed space; **labels in, labels for triggers** |

### dacman56 bindings (default layer)

| Label | Binding |
|-------|---------|
| L | `&kp L` |
| D | `&kp D` |
| E | `&ak_E` |
| W | `&ak_W` |
| right thumb | `&ak_SPACE` |

## `ak_SPACE` (right thumb)

Delays space during rolls so morphs can finish. Triggers on **labels** last pressed:

| Trigger | Prior label | Delay |
|---------|-------------|-------|
| `akt_ldw_l` / `akt_ldw_d` | `L` / `D` | 40 ms |
| `akt_ldw_w` / `akt_ew_w` | `W` | 25 ms |
| `akt_ew_e` | `E` | 40 ms |

Tunables: `MACRO_WAIT_SPACE_*` in `config/macros.dtsi`. Space is always sent — never swallowed.

## Updating

1. Refresh [`docs/reference/kbdterm.c`](reference/kbdterm.c).
2. Rebuild label → output table.
3. Specify morphs with **labels**; verify output via the table.
4. Match `trigger-keys` to **labels pressed**, not glyphs.
