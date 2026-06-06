# Vanilla ZMK migration plan

Branch: `vanilla`  
Target: upstream `zmkfirmware/zmk` with imported modules instead of `phuertay/zmk`.

Migrates adaptive keys to **[urob/zmk-adaptive-key](https://github.com/urob/zmk-adaptive-key)** and simplifies the keymap by removing dead layers. Plover HID via **[petercpark/zmk-hid-io-plover-hid](https://github.com/petercpark/zmk-hid-io-plover-hid)**.

### Status (2026-06-06)

| Item | State |
|---|---|
| Implementation on `vanilla` | **Done** — merged in [#17](https://github.com/phuertay/zmk-config/pull/17) |
| CI | **Green** — `dacman56_left`, `dacman56_right`, `settings_reset` |
| Hardware validation | **Pending** — see checklist below |
| Merge `vanilla` → `Adaptive` | **Blocked** until hardware sign-off ([#16](https://github.com/phuertay/zmk-config/pull/16)) |
| User-facing docs | **`README.md`** describes the current `vanilla` stack |

Sections below that describe “today” or `&ad_*` / 30 layers document the **pre-migration** fork config and the planning rationale. The live config uses **`&ak_*`**, **11 layers**, and **`&plv PLV_*`**.

### Hardware checklist (before merging to `Adaptive`)

- [ ] All 14 adaptive keys morph correctly
- [ ] Hold-tap adaptives (`hms_m_a`, `hm_m_c`, `hms_m_x`, `hms_m_m`)
- [ ] Ultra numpad + steno numpad mods (`NUM_HD_ULTRA`, `NUM_HD_STENO_MODS`)
- [ ] **`DEFAULT_HD_2`** — plain Tab/R layout, reachable via `&mo` from numpad; Excel `m_excel_go_to` overlay
- [ ] Steno / Plover output (`+-` / `^-` via `PLV_X3` / `PLV_X4`)
- [ ] Combos and macros on trimmed layer set

---

## Goals

1. Stop depending on `phuertay/zmk`.
2. Use **urob adaptive-key** with inline trigger bindings (drop most `m_d*` adaptive macros).
3. **Delete dead layers** — especially the 13 `adaptive_*` overlay layers that are never activated.
4. Keep only **NUM_HD_ULTRA** + **NUM_HD_STENO_MODS** for numpad (drop legacy num variants).
5. Track **`zmkfirmware/zmk@main`** + **`urob/zmk-adaptive-key@main`** (matches current fork and `nice_nano@1//zmk` board IDs).
6. Validate on CI, then hardware.

---

## ZMK version target

**Decision: `main` on both by default.** Pin to commit SHAs only if an upstream push breaks CI.

| Option | When to use |
|---|---|
| `zmkfirmware/zmk@main` + `urob/zmk-adaptive-key@main` | **Default** — matches `phuertay/zmk` (Zephyr 4.1) and `build.yaml` board IDs |
| Pin ZMK + urob to commit SHAs | If a push breaks CI and you need a stable build while investigating |
| `v0.3.0` / `v0.3` tags on both | Fallback only — older Zephyr 3.5; requires reverting `build.yaml` to `nice_nano_v2` |

Why not `v0.3.0` as the default: your fork and `build.yaml` already use Zephyr 4.1 / `nice_nano@1//zmk`. Pinning to `v0.3.0` would be a downgrade. urob also CI-tests against ZMK `main` on a schedule.

### Verified revisions (2026-06-06)

These are the commits `west update` resolves when both projects use `revision: main` today. They are also the pair urob’s [test-main workflow](https://github.com/urob/zmk-adaptive-key/actions/workflows/test-main.yml) last exercised successfully ([run 26915720317](https://github.com/urob/zmk-adaptive-key/actions/runs/26915720317), 2026-06-03 — still current `main` tips as of this verification).

| Project | SHA | Commit link |
|---|---|---|
| [zmkfirmware/zmk](https://github.com/zmkfirmware/zmk) | `773dec58eaacaef4703b3e4595e50bd71f6cad3d` | [773dec58](https://github.com/zmkfirmware/zmk/commit/773dec58eaacaef4703b3e4595e50bd71f6cad3d) |
| [urob/zmk-adaptive-key](https://github.com/urob/zmk-adaptive-key) | `e5b335a7b1076cb122582f5a892b118b02a3be34` | [e5b335a7](https://github.com/urob/zmk-adaptive-key/commit/e5b335a7b1076cb122582f5a892b118b02a3be34) |
| [petercpark/zmk-hid-io-plover-hid](https://github.com/petercpark/zmk-hid-io-plover-hid) | `d895457418590dddf628c5ae907b39deec7095ff` | [d895457](https://github.com/petercpark/zmk-hid-io-plover-hid/commit/d895457418590dddf628c5ae907b39deec7095ff) |

```yaml
# config/west.yml — see the file in-repo for copy-paste pin comments
# (LAST KNOWN GOOD + LATEST GREEN CI lines per project)
```

### How to find the newest working revision

Use this when CI turns red after you changed nothing locally — an upstream push on `main` is the usual cause.

#### 1. Find the last green build (your config)

1. Open **[phuertay/zmk-config → Actions](https://github.com/phuertay/zmk-config/actions)**.
2. Click the newest run with a **green checkmark**.
3. Open any **Build** job, e.g. `Build (nice_nano@1//zmk, dacman56_left)` (right/settings_reset are equivalent).
4. Expand the **`West Update`** step.
5. In the log, **Ctrl+F** for the anchor line, then read the **next** `HEAD is now at` line:

   | Project | Search for this anchor | Example line after it |
   |---|---|---|
   | ZMK | `=== updating zmk (zmk):` | `HEAD is now at 773dec5 docs: …` |
   | urob | `=== updating zmk-adaptive-key (zmk-adaptive-key):` | `HEAD is now at e5b335a Bump …` |

   Ignore `HEAD is now at` under `zephyr`, `hal_stm32`, `lvgl`, etc.

6. The log shows a **7-character** prefix. Resolve the **full 40-char SHA** on [zmk commits](https://github.com/zmkfirmware/zmk/commits/main) or [urob commits](https://github.com/urob/zmk-adaptive-key/commits/main) — click the matching commit, copy SHA from the URL.

Copy-paste pin lines live in **`config/west.yml`** (header comment + per-project `LAST KNOWN GOOD` / `LATEST GREEN CI` lines).

#### 2. Resolve SHAs locally (optional)

From a clean workspace with your `config/west.yml`:

```bash
west init -l config
west update --fetch-opt=--filter=tree:0
cd zmk && git rev-parse HEAD          # full ZMK SHA
cd ../zmk-adaptive-key && git rev-parse HEAD   # full urob SHA
```

See [ZMK user setup](https://zmk.dev/docs/user-setup) for a full local environment.

#### 3. Check urob’s module tests against ZMK main

urob runs scheduled CI against ZMK `main`: **[urob/zmk-adaptive-key → Run tests (main)](https://github.com/urob/zmk-adaptive-key/actions/workflows/test-main.yml)**. If your build fails but that workflow is green, the problem is likely in your keymap/config, not the module pairing.

#### 4. Pin in `west.yml`

1. Comment out or remove `revision: main` on the broken project.
2. Uncomment and set the `revision: <full-sha>` line from step 1 or 2 (use the **full** 40-character SHA).
3. Add/update the date in the end-of-line comment.
4. Push and confirm CI is green again.

#### 5. Return to floating `main`

When you want upstream fixes, set `revision: main` again, push, and watch CI. If it breaks, repeat from step 1.

Official background on why pinning matters: **[Pin your ZMK version](https://zmk.dev/blog/2025/06/20/pinned-zmk)** · **[ZMK releases](https://github.com/zmkfirmware/zmk/releases)** · **[Zephyr 4.1 / board variant notes](https://zmk.dev/blog/2025/12/09/zephyr-4-1)** (relevant because this config uses `nice_nano@1//zmk` in `build.yaml`).

---

## How adaptives worked before migration (historical)

There were **two separate mechanisms** in the fork keymap, and only one actually did anything:

### 1. Runtime behaviors (active)

On `default_layer_hd`, keys use `&ad_Q`, `&ad_W`, etc. defined in `config/adaptive.dtsi`. These are `zmk,behavior-antecedent-morph` instances — morphing happens **in firmware** based on the previous key. **This is your real adaptive system.**

### 2. Overlay layers (dead code)

Layers `adaptive_q` … `adaptive_dot` (indices 2–14, `#define AL_Q` … `AL_DOT`) show visual "what would be typed" layouts with `&m_dqt`, `&kp G`, etc.

**Nothing ever activates these layers.** There are zero `&mo AL_*`, `&sl AL_*`, or `&to AL_*` bindings anywhere in the config. The `AL_*` defines only match layer indices; they are never referenced.

**Conclusion:** All 13 `adaptive_*` layers (~175 lines) and all `AL_*` defines can be **deleted** with zero functional impact. urob migration does not need replacement overlay layers.

---

## Adaptive binding naming — **decision: `&ak_*`**

Rename all adaptive behavior nodes and keymap references from `&ad_*` to `&ak_*` (urob convention).

```dts
// adaptive.dtsi
ak_Q: ak_Q {
    compatible = "zmk,behavior-adaptive-key";
    ...
};

// dacman56.keymap
&ak_Q  &ak_W  &ak_E  ...
```

Also update hold-tap wrappers: `&ad_A` → `&ak_A`, etc. in `hms_m_a`, `hm_m_c`, `hms_m_x`, `hms_m_m`.

Inline urob trigger bindings in `adaptive.dtsi` and delete adaptive-only `m_d*` macros.

**Not the same as Plover remapping:** Plover `PLV_X3`/`PLV_X4` are bit-position changes in the HID report (see Phase 4 / PLV section below), not a naming choice.

---

## Layer audit (30 layers today)

Layer indices are assigned **in keymap file order**. Current mapping:

| Idx | Define | Layer name | Status | Evidence |
|---|---|---|---|---|
| 0 | `DEFAULT_HD` | `default_layer_hd` | **KEEP** | Primary typing layer |
| 1 | `STENO` | `steno` | **KEEP** | `&m_to_steno`, steno thumb keys |
| 2 | `AL_Q` | `adaptive_q` | **DELETE** | Never activated; `AL_Q` unused |
| 3 | `AL_W` | `adaptive_w` | **DELETE** | Never activated |
| 4 | `AL_E` | `adaptive_e` | **DELETE** | Never activated |
| 5 | `AL_R` | `adaptive_r` | **DELETE** | Never activated |
| 6 | `AL_O` | `adaptive_o` | **DELETE** | Never activated |
| 7 | `AL_P` | `adaptive_p` | **DELETE** | Never activated |
| 8 | `AL_SEMI` | `adaptive_SEMI` | **DELETE** | Never activated |
| 9 | `AL_Z` | `adaptive_z` | **DELETE** | Never activated |
| 10 | `AL_X` | `adaptive_x` | **DELETE** | Never activated |
| 11 | `AL_C` | `adaptive_c` | **DELETE** | Never activated |
| 12 | `AL_M` | `adaptive_m` | **DELETE** | Never activated |
| 13 | `AL_FSLH` | `adaptive_fslh` | **DELETE** | Never activated |
| 14 | `AL_DOT` | `adaptive_dot` | **DELETE** | Never activated |
| 15 | `NUM_HD` | `unused_numeric_layer_hd` | **DELETE** | Legacy numpad; default uses ULTRA |
| 16 | `NUM_HD_ULTRA` | `numeric_layer_ultra_KP_N` | **KEEP** | Main numpad (`&lt_s NUM_HD_ULTRA`, thumbs) |
| 17 | `NUM_HD_MID` | `unused_num_hd_mid` | **DELETE** | `NUM_HD_MID` never referenced |
| 18 | `NUM_HD_STENO_MODS` | `num_hd_steno_mods` | **KEEP** | Conditional: STENO + NUM_HD_ULTRA |
| 19 | `LCNUM` | `left_ctrl_num_layer` | **KEEP** | Active via `&lts LCNUM` on default + steno mods — confirmed |
| 20 | `FN` | `fn_layer` | **DELETE** | `&mo FN` only on dead NUM_HD layers; combos list `FN` but work on DEFAULT_HD alone |
| 21 | `NUM_L_HD` | `numeric_layer_left_hd` | **DELETE** | Left numpad chain — confirmed drop |
| 22 | `RCNUM_L` | `right_ctrl_num_layer` | **DELETE** | Part of left numpad chain |
| 23 | `FN_L` | `fn_layer_left` | **DELETE** | Part of left numpad chain |
| 24 | `DEFAULT_HD_2` | `default_layer_hd_2` | **KEEP** | Alternate default (plain Tab/R); see README |
| 25 | `SPFN_HD` | `spacefn_layer_hd` | **KEEP** | Default thumbs: `&lts SPFN_HD BSPC/ENTER` |
| 26 | `MOUSE` | `mouse_layer` | **REVIEW** | `&tog MOUSE` from func layers |
| 27 | `FUNC_HD` | `function_layer_right` | **KEEP** | `&sl FUNC_HD` from default thumb |
| 28 | `FUNC_MACR` | `function_layer_macros` | **KEEP** | `&sl FUNC_MACR` from default thumb |
| 29 | `FUNC_STENO_MODS` | `function_steno_mods` | **KEEP** | Conditional: STENO + FUNC_HD or FUNC_MACR |

### Proposed simplified layer stack (after cleanup)

Renumber after deletions — **11 layers** down from 30:

| New idx | Define | Layer | Notes |
|---|---|---|---|
| 0 | `DEFAULT_HD` | `default_layer_hd` | Primary; numpad layer-taps |
| 1 | `STENO` | `steno` | |
| 2 | `NUM_HD_ULTRA` | `numeric_layer_ultra_KP_N` | Only numpad layer |
| 3 | `NUM_HD_STENO_MODS` | `num_hd_steno_mods` | Conditional overlay |
| 4 | `LCNUM` | `left_ctrl_num_layer` | Nav cluster on homerow — confirmed |
| 5 | `DEFAULT_HD_2` | `default_layer_hd_2` | Alternate default; `&mo` from ultra numpad |
| 6 | `SPFN_HD` | `spacefn_layer_hd` | SpaceFn thumbs |
| 7 | `MOUSE` | `mouse_layer` | Toggled from func layers — confirmed |
| 8 | `FUNC_HD` | `function_layer_right` | BT/settings |
| 9 | `FUNC_MACR` | `function_layer_macros` | Macros/emails |
| 10 | `FUNC_STENO_MODS` | `function_steno_mods` | Conditional overlay |

**Dropped (confirmed):** `NUM_HD`, `NUM_HD_MID`, all `adaptive_*`, `FN`, `NUM_L_HD`, `RCNUM_L`, `FN_L`.

**Kept:** `DEFAULT_HD_2` (restored after initial migration — plain typing + Excel macro overlay).

**Update after renumbering:** all `#define` indices, combo `COMB_LAY_NUM`, conditional layers, and every `&mo`/`&lt_s`/`&tog`/`&sl` reference.

### Combo layer update

```dts
// combos.dtsi — today
#define COMB_LAY_NUM NUM_HD NUM_HD_ULTRA NUM_HD_STENO_MODS

// after cleanup
#define COMB_LAY_NUM NUM_HD_ULTRA NUM_HD_STENO_MODS
```

Remove `FN` from `COMB_LAY_ALL` if `fn_layer` is deleted (combos on default already work with just `DEFAULT_HD`).

---

## Schema mapping: antecedent-morph → urob adaptive-key

### Old

```dts
ad_W: adapt_W {
    compatible = "zmk,behavior-antecedent-morph";
    #binding-cells = <0>;
    max-delay-ms = <200>;
    defaults = <&kp W>;
    antecedents = <Q E>;
    bindings = <&m_dqt>, <&m_ddw>;
};
```

### New (with inline bindings — no `m_dqt`/`m_ddw` macros)

```dts
ak_W: ak_W {
    compatible = "zmk,behavior-adaptive-key";
    #binding-cells = <0>;
    bindings = <&kp W>;

    akt_w_q {
        trigger-keys = <Q>;
        max-prior-idle-ms = <200>;
        bindings = <&kp BSPC &kp SQT>;
    };
    akt_w_e {
        trigger-keys = <E>;
        max-prior-idle-ms = <200>;
        bindings = <&kp BSPC &kp D &kp W>;
    };
};
```

Full conversion table for all 14 keys is unchanged from the prior plan revision (see git history or section below).

---

## Full behavior conversion table

All triggers use `max-prior-idle-ms = <200>`.

| Behavior | Default | Triggers → inline bindings |
|---|---|---|
| `ak_Q` | `&kp Q` | `E` → `&kp G` |
| `ak_W` | `&kp W` | `Q` → `BSPC SQT`; `E` → `BSPC D W` |
| `ak_E` | `&kp E` | `Q` → `BSPC G E`; `W` → `BSPC Y`; `R` → `&kp C` |
| `ak_U` | `&kp U` | `O` → `&kp M` |
| `ak_I` | `&kp I` | `O` → `M K` |
| `ak_O` | `&kp O` | `I` → `M K`; `P` → `&kp L`; `U` → `&kp M` |
| `ak_P` | `&kp P` | `O` → `&kp L` |
| `ak_A` | `&kp A` | `SEMI` → `BSPC G A`; `FSLH` → `BSPC B A` |
| `ak_S` | `&kp S` | `FSLH` → `BSPC B S` |
| `ak_SEMI` | `&kp SEMI` | `E` → `G C` |
| `ak_Z` | `&kp Z` | `X` → `BSPC V Y`; `C` → `&kp G` |
| `ak_X` | `&kp X` | `C` → `&kp Y` |
| `ak_C` | `&kp C` | `Z` → `BSPC G C`; `FSLH` → `BSPC B C` |
| `ak_M` | `&kp M` | `FSLH` → `BSPC J M` |
| `ak_FSLH` | `&kp FSLH` | `E` → `&kp B`; `M` → `&kp J`; `DOT` → `&kp L` |

### Macros to delete from `adaptive.dtsi`

Once inlined, remove these if nothing else references them:

`m_dba`, `m_dbc`, `m_dbs`, `m_ddw`, `m_dga`, `m_dgc`, `m_dge`, `m_djm`, `m_dkk`, `m_dqt`, `m_dvy`, `m_gc`, `m_mk`

Keep `m_dg_gr` only if still used outside adaptives (check `adaptive_fslh` overlay — being deleted anyway).

---

## Phased implementation

### Phase 1 — Build infrastructure

- `config/west.yml` → vanilla ZMK `main` + `zmk-adaptive-key@main` (with SHA-pin comments; see above)
- `.github/workflows/build.yml` → `zmkfirmware/zmk/.../build-user-config.yml@main`

### Phase 2 — Rewrite `adaptive.dtsi`

- Convert all 14 behaviors to `zmk,behavior-adaptive-key`
- Inline trigger bindings; delete adaptive-only `m_*` macros
- Rename to `&ak_*` in `adaptive.dtsi` and keymap
- Add Kconfig: `CONFIG_ZMK_ADAPTIVE_KEY_WAIT_MS=7`, `CONFIG_ZMK_ADAPTIVE_KEY_TAP_MS=2`

### Phase 3 — Keymap simplification

1. Delete all 13 `adaptive_*` layer blocks + `AL_*` defines
2. Delete `unused_numeric_layer_hd`, `unused_num_hd_mid`
3. Delete confirmed-unused layers: `FN`, `DEFAULT_HD_2`, `NUM_L_HD` / `RCNUM_L` / `FN_L` chain; remove `&tog NUM_L_HD` from func layers
4. Renumber `#define` layer indices top-to-bottom
5. Fix all layer references in keymap, combos, macros, conditional layers
6. Remove `default_layer_hd_2` if redundant with primary default (merge any unique bindings first)

**Acceptance:** ~300 fewer lines in keymap; CI builds; layer count ≤ 11.

### Phase 4 — Plover HID module

- Add `petercpark/zmk-hid-io-plover-hid@main` to west.yml
- Replace fork Kconfig:

  ```conf
  # remove
  CONFIG_ZMK_PLOVER_HID=y
  CONFIG_ZMK_HID_KEYBOARD_REPORT_SIZE=20

  # add
  CONFIG_USB_DEVICE_HID=y
  CONFIG_USB_HID_DEVICE_COUNT=2
  CONFIG_ZMK_HID_IO=y
  CONFIG_ZMK_HID_IO_PLOVER_HID=y
  ```

- `#include <dt-bindings/zmk/hid-io/plover_hid.h>`
- `&kp PLV_*` → `&plv PLV_*` on steno layer and steno macros
- Plover side: existing `+-`/`*3`,`X3` and `^-`/`*4`,`X4` aliases — no change required

### Phase 5 — Validation

- CI: all three build targets in `build.yaml`
- Hardware: all 14 adaptive keys + hold-tap wrappers
- Hardware: NUM_HD_ULTRA + steno + steno-mods overlay
- Hardware: Plover (phase 4)

---

## Files touched

| File | Changes |
|---|---|
| `config/west.yml` | Vanilla ZMK + urob (+ petercpark later) |
| `.github/workflows/build.yml` | Upstream workflow |
| `config/adaptive.dtsi` | urob rewrite + macro cleanup |
| `config/dacman56.keymap` | Delete dead layers, renumber indices |
| `config/combos.dtsi` | Update `COMB_LAY_NUM`, possibly `COMB_LAY_ALL` |
| `config/macros.dtsi` | Remove `m_excel_go_to` DEFAULT_HD_2 refs if layer dropped |
| `config/dacman56.conf` | Adaptive Kconfig; later Plover Kconfig |
| `config/behaviors.dtsi` | Unchanged unless layer-hold behaviors need index fixes |

---

## Risks

| Risk | Mitigation |
|---|---|
| Layer renumber breaks a hidden path | Grep all `&mo`, `&lt`, `&tog`, `&sl`, `&to` after edit |
| Removing `DEFAULT_HD_2` breaks Excel macro | Rewrite `m_excel_go_to` or keep layer |
| Upstream `main` push breaks CI | Pin ZMK and/or urob SHAs in `west.yml` (see version section) |
| urob press-vs-release timing feel | Tune `max-prior-idle-ms` on hardware |

---

## Rollback

Stay on `Adaptive` branch (fork) for production until vanilla is hardware-validated.

---

## `default_layer_hd_2` — purpose and differences

`default_layer_hd_2` is a near-copy of `default_layer_hd` for **plain typing** without numpad F18 signaling:

| Position | `default_layer_hd` | `default_layer_hd_2` |
|---|---|---|
| Row 1, col 0 | `&lt_s NUM_HD_ULTRA TAB` (tap = Tab, hold = numpad) | `&kp TAB` (plain Tab) |
| Row 1, col 4 | `&lt_s_tap NUM_HD_ULTRA R` (R with numpad signal) | `&kp R` |
| Row 2, col 4 | `&lt_s NUM_HD_ULTRA F` | `&lt NUM_HD_ULTRA F` (no numpad signal on hold) |
| Thumb, RBKT | `&to_tap_s NUM_HD_ULTRA RBKT` | `&hm TILDE RBKT` |
| Thumb, R-outer | `&lay_or_lay_s_num …` + `&kp LCTL` | `&tog NUM_HD_ULTRA` + `&kp LWIN` |
| Thumb, bottom-R | `&m_to_steno` + `&sl FUNC_HD` | `&kp C_AC_REFRESH` + `&sl FUNC_HD` |
| Thumb, bottom-L | `&m_calc` | `&kp C_AL_CALC` |

**Live references:**

- `numeric_layer_ultra_KP_N`: `&mo DEFAULT_HD_2` on bottom-right (momentary alt default while in numpad)
- `m_excel_go_to`: momentarily `&mo DEFAULT_HD_2` during Excel Go To (Ctrl+W)
- Combos: `COMB_LAY_ALL` includes both `DEFAULT_HD` and `DEFAULT_HD_2`

---

## `numeric_layer_left_hd`, `RCNUM_L`, `FN_L` — what they are

These three layers form a **left-hand numpad mode**, separate from `NUM_HD_ULTRA`:

```
FUNC_HD / FUNC_MACR  ──&tog NUM_L_HD──►  numeric_layer_left_hd (NUM_L_HD)
                                              │
                              hold &lts RCNUM_L ENTER ──►  RCNUM_L (RC arrow overlay)
                              hold &mo FN_L ──►  FN_L (F-key overlay on right half)
```

### `numeric_layer_left_hd` (NUM_L_HD)

- **Purpose:** Numpad/operators laid out for the **left side** of the keyboard (vs ultra layer which is right-heavy).
- **Reachable from:** `&tog NUM_L_HD` on `function_layer_right` and `function_layer_macros` only — not from default or ultra numpad.
- **Notable bindings:** Excel-style `&fm F*` shifted digits, `&m_excel_go_to`, `&lay_or_lay DEFAULT_HD_2 NUM_HD`, nav cluster on right via `&lts RCNUM_L ENTER`.
- **If dropped:** You lose the toggleable left-numpad layout; **`NUM_HD_ULTRA` is unaffected** (that is your main num layer).

### `RCNUM_L` (right_ctrl_num_layer)

- **Purpose:** While on `NUM_L_HD`, holding Enter (`&lts RCNUM_L ENTER`) overlays **Ctrl+arrow** navigation on the right half (PgUp, Up, PgDn, etc.).
- **Only referenced from:** `numeric_layer_left_hd` — not from default or ultra.
- **If dropped:** Remove the hold-tap on Enter in `NUM_L_HD`; no impact elsewhere.

### `FN_L` (fn_layer_left)

- **Purpose:** While on `NUM_L_HD`, `&mo FN_L` on thumb shows **F13–F24** on the right half (mirror of `fn_layer` concept but for left-numpad context).
- **Only referenced from:** `numeric_layer_left_hd`.
- **If dropped:** Remove `&mo FN_L` thumb binding; no impact elsewhere.

**Summary:** This entire chain is **confirmed for deletion**. Remove `&tog NUM_L_HD` from `FUNC_HD` and `FUNC_MACR` during phase 3.

---

## Plover HID: `PLV_X3` and `PLV_X4` (Phase 4)

Plover HID sends a **64-bit bitmap** — each steno key is one bit index (0–63). Your steno layer uses:

```dts
&kp PLV_X3  ...  // left column, row 3
&kp PLV_X4  ...  // left column, row 4
```

**In your fork** (`phuertay/zmk`), `PLV_X3` = bit **25**, `PLV_X4` = bit **26** (sequential X-key numbering from 23 upward).

**In petercpark's module**, the header assigns standard steno keys 0–22 first, then **extra** keys (SL2, ST2–ST4, NM2–NMC) at indices 23–37, and only then **X-keys**:

| Name | phuertay fork (bit) | petercpark module (bit) |
|---|---|---|
| `PLV_X3` | 25 | **40** |
| `PLV_X4` | 26 | **41** |
| (at 25–26 in module) | — | `PLV_ST3`, `PLV_ST4` (extra star keys) |

**Your Plover keymap already aliases both naming schemes:**

```
"+-" ← *3, X3
"^-" ← *4, X4
```

So today the fork sends `*3`/`*4` (bits 25/26) → Plover outputs `+-`/`^-`. After migration petercpark sends `X3`/`X4` (bits 40/41) → **same logical strokes**. No Plover config change required; optionally simplify aliases to `["X3"]` / `["X4"]` only after migration.

Standard keys (bits 0–22) are identical on both firmwares.

---

## Decisions (confirmed)

1. ~~Drop `DEFAULT_HD_2`?~~ **No — keep.** Restored on `vanilla` (plain Tab/R alt layout + Excel macro).
2. ~~Drop `NUM_L_HD` / `RCNUM_L` / `FN_L`?~~ **Yes** — remove left numpad chain and func-layer toggles.
3. ~~Drop `MOUSE` layer?~~ **No — keep.**
4. ~~Keep `LCNUM` nav cluster?~~ **Yes — keep.**
5. ~~ZMK / urob version?~~ **`main` on both** — pin SHAs in `west.yml` if a push breaks CI (see [verified revisions](#verified-revisions-2026-06-06) and [how to find the newest working revision](#how-to-find-the-newest-working-revision)).

---

## PR sequence

1. ~~**Planning** ([#16](https://github.com/phuertay/zmk-config/pull/16)) — this document + `west.yml` pin comments~~
2. ~~**Implementation** ([#17](https://github.com/phuertay/zmk-config/pull/17)) — phases 1–4 merged to `vanilla`~~
3. **Promote to production** ([#16](https://github.com/phuertay/zmk-config/pull/16)) — merge `vanilla` → `Adaptive` after hardware sign-off
