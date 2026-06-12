# Vanilla ZMK migration

**Default branch:** `main` (renamed from `vanilla`) · **Archive:** `adaptive-legacy` (renamed from `Adaptive`, legacy `phuertay/zmk` fork)

This config builds on **[zmkfirmware/zmk](https://github.com/zmkfirmware/zmk)** with **[urob/zmk-adaptive-key](https://github.com/urob/zmk-adaptive-key)** and **[petercpark/zmk-hid-io-plover-hid](https://github.com/petercpark/zmk-hid-io-plover-hid)**. User-facing docs live in **`README.md`**.

## Status

| Item | State |
|---|---|
| Implementation | Done ([#17](https://github.com/phuertay/zmk-config/pull/17)) |
| CI | Green — `dacman56_left`, `dacman56_right`, `settings_reset` |
| Hardware validation | Pending |
| Default branch | `main` (2026-06) — planning PR [#16](https://github.com/phuertay/zmk-config/pull/16) superseded by [#17](https://github.com/phuertay/zmk-config/pull/17)–[#22](https://github.com/phuertay/zmk-config/pull/22) |

## Current layer stack (10 layers)

| Idx | Define | Layer |
|---:|---|---|
| 0 | `DEFAULT_HD` | Primary default (numpad layer-taps, steno macro) |
| 1 | `STENO` | Plover HID steno |
| 2 | `NUM_HD_ULTRA` | Ultra numpad |
| 3 | `NUM_HD_STENO_MODS` | Conditional: steno + ultra numpad |
| 4 | `NAV_OVL` | Nav overlay (RC/LA arrows when held) |
| 5 | `DEFAULT_HD_2` | Alt default (plain Tab/R, no F18 signals) |
| 6 | `SPFN_HD` | SpaceFn |
| 7 | `MOUSE` | Mouse keys |
| 8 | `FUNC` | Settings + macros (both thumb stickies) |
| 9 | `FUNC_STENO_MODS` | Conditional: steno + func |

**Removed from fork (30 → 10 layers):** 13 dead `adaptive_*` overlays, legacy numpad variants, left numpad chain (`NUM_L_HD` / `RCNUM_L` / `FN_L`), `FN`, `DEFAULT_HD` duplicates.

## Pinning upstream SHAs

When CI breaks without local changes, pin in **`config/west.yml`**:

1. [Actions](https://github.com/phuertay/zmk-config/actions) → green run → **West Update** log
2. Ctrl+F `=== updating zmk (zmk):` (and urob / petercpark anchors)
3. Copy full SHA from GitHub commit URL into commented `revision:` lines

Verified **2026-06-06:** ZMK `773dec58…`, urob `e5b335a7…`, petercpark `d8954574…` — see `west.yml` header.

## Hardware checklist

- [ ] All 14 `&ak_*` adaptive keys + hold-tap morphs
- [ ] Ultra numpad + steno numpad mods
- [ ] `DEFAULT_HD_2` via `&mo` from numpad; plain Tab/R typing
- [ ] **`NAV_OVL`** — hold `A` homerow; numpad `.` hold; steno+numpad `I` hold
- [ ] **`FUNC`** — sticky thumbs (see README); merged settings/macros layer
- [ ] Steno / Plover (`+-` / `^-`)
- [ ] Combos on default layers

## History (fork → vanilla)

| Before | After |
|---|---|
| `phuertay/zmk` + antecedent-morph `&ad_*` | `zmkfirmware/zmk` + urob `&ak_*` |
| `CONFIG_ZMK_PLOVER_HID` | `CONFIG_ZMK_HID_IO_PLOVER_HID` + `&plv` |
| `m_excel_go_to` on left numpad (`NUM_L_HD`) | Removed with left numpad layer |
| Separate `FUNC_HD` / `FUNC_MACR` stickies | Single **`FUNC`** layer, both thumbs |

Archived planning detail (layer audit, schema mapping) is in git history before this file was trimmed (2026-06).
