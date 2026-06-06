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

## Layer map (11 layers)

| Index | Define | Layer name |
|---:|---|---|
| 0 | `DEFAULT_HD` | `default_layer_hd` — primary layout (numpad layer-taps, steno macro) |
| 1 | `STENO` | `steno` |
| 2 | `NUM_HD_ULTRA` | `numeric_layer_ultra_KP_N` |
| 3 | `NUM_HD_STENO_MODS` | conditional: steno + ultra numpad |
| 4 | `LCNUM` | `left_ctrl_num_layer` |
| 5 | `DEFAULT_HD_2` | `default_layer_hd_2` — alternate default (plain Tab/R, no numpad signals) |
| 6 | `SPFN_HD` | `spacefn_layer_hd` |
| 7 | `MOUSE` | `mouse_layer` |
| 8 | `FUNC_HD` | `function_layer_right` |
| 9 | `FUNC_MACR` | `function_layer_macros` |
| 10 | `FUNC_STENO_MODS` | conditional: steno + func layers |

Conditional layers are defined in `config/dacman56.keymap`. Legacy overlay layers (`adaptive_*`) and the left numpad chain were removed during the vanilla migration — see **`docs/vanilla-migration-plan.md`**.

### `DEFAULT_HD` vs `DEFAULT_HD_2`

Both layers share the same adaptive home row (`&ak_*`). **`DEFAULT_HD_2`** is for plain typing without numpad F18 signaling:

| | `DEFAULT_HD` | `DEFAULT_HD_2` |
|---|---|---|
| Tab | `&lt_s NUM_HD_ULTRA TAB` | `&kp TAB` |
| R / F | numpad layer-taps | `&kp R`, `&lt NUM_HD_ULTRA F` |
| RBKT thumb | numpad `to_tap_s` | `&hm TILDE RBKT` |
| Outer thumb | numpad signal + LCtrl | `&tog NUM_HD_ULTRA`, LWin |
| Bottom thumb | calc + steno macros | Calc key + refresh |

Reach **`DEFAULT_HD_2`** from the ultra numpad layer (`&mo DEFAULT_HD_2` on the bottom-right key). The **`m_excel_go_to`** macro momentarily activates it during Excel Go To (Ctrl+W).

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

Steno mode is toggled via F17 / Ctrl+F17 (`steno_on` / `steno_off`); numpad signaling uses F18 / Ctrl+F18.

**Plover aliases:** `PLV_X3` and `PLV_X4` map to different HID bit indices than the old fork, but existing Plover aliases (`+-` ← `X3`, `^-` ← `X4`) continue to work. Details in **`docs/vanilla-migration-plan.md`** (Plover HID section).

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

- **`docs/vanilla-migration-plan.md`** — migration history, layer audit, SHA pinning, hardware checklist
- **`config/west.yml`** — manifest and pin comments
