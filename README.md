# phuertay/zmk-config — dacman56

ZMK user config for the [dacman56](https://github.com/phuertay/zmk-keyboard-dacman56) split keyboard (nice!nano v1).

The **`vanilla`** branch builds against upstream **[zmkfirmware/zmk](https://github.com/zmkfirmware/zmk)** with external modules for adaptive keys and Plover HID. The **`Adaptive`** branch still uses the legacy **`phuertay/zmk`** fork until hardware validation is complete.

---

## Stack (`vanilla` branch)

| Component | Source |
|---|---|
| ZMK firmware | [zmkfirmware/zmk](https://github.com/zmkfirmware/zmk) `@main` |
| Adaptive keys | [urob/zmk-adaptive-key](https://github.com/urob/zmk-adaptive-key) |
| Plover / steno HID | [petercpark/zmk-hid-io-plover-hid](https://github.com/petercpark/zmk-hid-io-plover-hid) |
| Keyboard shield | [phuertay/zmk-keyboard-dacman56](https://github.com/phuertay/zmk-keyboard-dacman56) |

CI uses the official workflow:

```yaml
# .github/workflows/build.yml
uses: zmkfirmware/zmk/.github/workflows/build-user-config.yml@main
```

West manifest and SHA pinning instructions live in **`config/west.yml`** (header comment + `LAST KNOWN GOOD` lines per project).

---

## Kconfig (`config/dacman56.conf`)

Plover HID (second USB HID interface):

```
CONFIG_USB_HID_DEVICE_COUNT=2
CONFIG_ZMK_HID_IO=y
CONFIG_ZMK_HID_IO_PLOVER_HID=y
```

Adaptive key timing (urob module):

```
CONFIG_ZMK_ADAPTIVE_KEY_WAIT_MS=7
CONFIG_ZMK_ADAPTIVE_KEY_TAP_MS=2
```

`CONFIG_ZMK_BEHAVIOR_ADAPTIVE_KEY` is enabled automatically from devicetree when adaptive-key behaviors are present — do not set it in the user config.

---

## Layer map (10 layers)

| Index | Define | Layer name |
|---:|---|---|
| 0 | `DEFAULT_HD` | `default_layer_hd` — primary (numpad layer-taps, steno macro) |
| 1 | `STENO` | `steno` |
| 2 | `NUM_HD_ULTRA` | `numeric_layer_ultra_KP_N` |
| 3 | `NUM_HD_STENO_MODS` | conditional: steno + ultra numpad |
| 4 | `NAV_OVL` | `nav_overlay_layer` — RC/LA nav cluster overlay |
| 5 | `DEFAULT_HD_2` | `default_layer_hd_2` — plain Tab/R alt default |
| 6 | `SPFN_HD` | `spacefn_layer_hd` |
| 7 | `MOUSE` | `mouse_layer` |
| 8 | `FUNC` | `function_layer` — settings + macros (merged) |
| 9 | `FUNC_STENO_MODS` | conditional: steno + func |

### `DEFAULT_HD` vs `DEFAULT_HD_2`

Both share the adaptive home row (`&ak_*`). **`DEFAULT_HD_2`** drops numpad F18 signaling for plain typing:

| | `DEFAULT_HD` | `DEFAULT_HD_2` |
|---|---|---|
| Tab | `&lt_s NUM_HD_ULTRA TAB` | `&kp TAB` |
| R / F | numpad layer-taps | `&kp R`, `&lt NUM_HD_ULTRA F` |
| RBKT thumb | numpad `to_tap_s` | `&hm TILDE RBKT` |
| Outer thumb | numpad signal + LCtrl | `&tog NUM_HD_ULTRA`, LWin |
| Bottom thumb | calc + steno | Calc key + refresh |

Reach **`DEFAULT_HD_2`** via `&mo DEFAULT_HD_2` on the ultra numpad bottom-right key.

### `NAV_OVL` (nav overlay)

Momentary overlay with arrow / PgUp / PgDn on the **left-hand nav cluster** (and mirrored trans elsewhere). Hold to activate:

| Trigger | Binding |
|---|---|
| Hold **A** homerow | `&hms_m_a NAV_OVL 0` — hold = overlay, tap = `&ak_A` |
| Hold **.** on ultra numpad | `&lts_tp NAV_OVL KP_DOT` |
| Hold **I** on steno+numpad mods | `&lts NAV_OVL I` |

Renamed from `LCNUM` / `left_ctrl_num_layer` — the layer is nav keys, not a numpad.

### `FUNC` layer (settings + macros)

**Activation:** sticky **`&sl FUNC`** on **both** bottom inner thumb keys on `DEFAULT_HD` and `DEFAULT_HD_2`. Tap either thumb to latch; tap again to release.

Merged from **`FUNC_MACR`** (left sticky) + **`FUNC_HD`** (right sticky). Those were almost identical full layers; the real difference was **which hand** had the unique keys, not different macro nodes:

| Side | Was on `FUNC_MACR` | Was on `FUNC_HD` | Merged `FUNC` |
|---|---|---|---|
| Left | `bootloader`, `C_NEXT`, `td_emails` | `sys_reset`, lock keys | Both on rows 1–2 left |
| Right | BT/output | media, browser keys | Rows 0–3 right |
| Thumbs | `m_9_16`, `m_to_steno` | Insert, SpaceFn | Macro keys left; home/Insert/SpaceFn right |

Both stickies latch the **same** layer — reach macros on the left hand, settings on the right.

---

## Adaptive keys (urob)

Adaptive behaviors are defined in **`config/adaptive.dtsi`** using `zmk,behavior-adaptive-key` and referenced as **`&ak_*`** in the keymap (for example `&ak_Q`, `&ak_W`).

Each behavior has a default binding plus one or more trigger nodes (`akt_*`) that morph output when a specific prior key was released within `max-prior-idle-ms`:

```dts
ak_Q: ak_Q {
    compatible = "zmk,behavior-adaptive-key";
    #binding-cells = <0>;
    bindings = <&kp Q>;
    akt_q_e {
        trigger-keys = <E>;
        max-prior-idle-ms = <200>;
        bindings = <&kp G>;
    };
};
```

Hold-tap wrappers that embed adaptives (`hms_m_a`, `hm_m_c`, `hms_m_x`, `hms_m_m`) also use `&ak_*`.

Upstream docs: [urob/zmk-adaptive-key](https://github.com/urob/zmk-adaptive-key).

---

## Steno / Plover HID

The steno layer uses the **`plv`** behavior from **`config/behaviors.dtsi`**:

```dts
plv: plover_hid {
    compatible = "zmk,behavior-plover-hid";
    #binding-cells = <1>;
};
```

Keymap bindings look like `&plv PLV_TL`, `&plv PLV_X3`, etc. Include:

```dts
#include <dt-bindings/zmk/hid-io/plover_hid.h>
```

Steno mode is toggled via F17 / Ctrl+F17 (`steno_on` / `steno_off`); numpad signaling uses F18 / Ctrl+F18. Layer-tap behaviors that use `*_s_num` wrappers emit F18 on hold/toggle so host apps can detect numpad mode.

---

## Config layout

| File | Role |
|---|---|
| `dacman56.keymap` | Layer indices, conditional layers, key bindings |
| `adaptive.dtsi` | urob `&ak_*` adaptive-key behaviors |
| `behaviors.dtsi` | Hold-taps, tap dances, Plover `plv`, global `&sl` timeout |
| `macros.dtsi` | Text/steno macros + F17/F18 signaling helpers (`to_s`, `mo_s_num`, …) |
| `combos.dtsi` | 18 combos on default layers (+ numpad layers for Tab/Esc) |
| `dacman56.conf` | Kconfig (HID, adaptive timing, BT/debounce tuning) |
| `west.yml` | ZMK + module manifest |

Include order in the keymap matters: `combos` → `behaviors` → `macros` → `adaptive`.

**Plover aliases:** `PLV_X3` and `PLV_X4` map to different HID bit indices than the old fork, but existing Plover aliases (`+-` ← `X3`, `^-` ← `X4`) continue to work.

---

## Building

**GitHub Actions** builds three artifacts from **`build.yaml`**: `dacman56_left`, `dacman56_right`, and `settings_reset` on `nice_nano@1//zmk`.

For local builds, follow [ZMK user setup](https://zmk.dev/docs/user-setup), then:

```bash
west init -l config
west update
west build -s zmk/app -b nice_nano@1//zmk -d build/left -- -DSHIELD=dacman56_left
```

---

## Branches

| Branch | Purpose |
|---|---|
| **`vanilla`** | Upstream ZMK + modules; migration target (CI green) |
| **`Adaptive`** | Legacy fork-based production until merged from `vanilla` |

Merge **`vanilla` → `Adaptive`** only after on-keyboard validation (adaptives, numpad, steno, combos).

---

## Further reading

- **`docs/vanilla-migration-plan.md`** — status, checklist, SHA pinning
- **`config/west.yml`** — manifest and pin comments
