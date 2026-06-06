# Vanilla ZMK migration plan

Branch: `vanilla`  
Target: upstream `zmkfirmware/zmk` with imported modules instead of `phuertay/zmk`.

This plan migrates adaptive keys from **antecedent-morph** (baked into your fork) to **[urob/zmk-adaptive-key](https://github.com/urob/zmk-adaptive-key)**. Plover HID is a separate follow-up phase using **[petercpark/zmk-hid-io-plover-hid](https://github.com/petercpark/zmk-hid-io-plover-hid)**.

---

## Goals

1. Stop depending on `phuertay/zmk` for adaptive and Plover functionality.
2. Keep existing keymap bindings (`&ad_Q`, `&ad_W`, …) wherever possible to minimize layer churn.
3. Preserve adaptive timing feel (`200 ms` window, macro tap/wait timings).
4. Validate on CI before flashing hardware.

## Non-goals (for initial merge)

- Rewriting the `adaptive_*` practice/overlay layers in `dacman56.keymap` (they can stay as-is).
- Changing homerow-mod or combo behavior unrelated to adaptive migration.
- Migrating Plover HID in the same PR (recommended as phase 2 to isolate risk).

---

## Current state

| Component | Today |
|---|---|
| ZMK source | `phuertay/zmk@main` via `config/west.yml` |
| Adaptive engine | `behavior_antecedent_morph.c` inside fork |
| Adaptive config | `config/adaptive.dtsi` (`zmk,behavior-antecedent-morph`) |
| Adaptive bindings | 14 keys on default layers: `&ad_Q` … `&ad_FSLH` |
| Hold-tap wrappers | `hms_m_a`, `hm_m_c`, `hms_m_x`, `hms_m_m` tap side uses `&ad_*` |
| Plover | `CONFIG_ZMK_PLOVER_HID=y`, `&kp PLV_*` (fork-only) |
| CI workflow | `phuertay/zmk/.github/workflows/build-user-config.yml@main` |

## Target state

| Component | After migration |
|---|---|
| ZMK source | `zmkfirmware/zmk@v0.3` (pin to release tag) |
| Adaptive engine | `urob/zmk-adaptive-key@v0.3` |
| Adaptive config | `config/adaptive.dtsi` rewritten (`zmk,behavior-adaptive-key`) |
| Plover (phase 2) | `petercpark/zmk-hid-io-plover-hid@main` + keymap/config updates |
| CI workflow | `zmkfirmware/zmk/.github/workflows/build-user-config.yml@main` |

Pin **the same ZMK release tag** for both `zmk` and `zmk-adaptive-key` (urob tags track ZMK releases).

---

## Schema mapping: antecedent-morph → adaptive-key

### Old (antecedent-morph)

```dts
ad_Q: adapt_Q {
    compatible = "zmk,behavior-antecedent-morph";
    #binding-cells = <0>;
    max-delay-ms = <200>;
    defaults = <&kp Q>;
    antecedents = <E>;
    bindings = <&kp G>;
};
```

### New (urob adaptive-key)

```dts
ad_Q: adapt_Q {
    compatible = "zmk,behavior-adaptive-key";
    #binding-cells = <0>;
    bindings = <&kp Q>;   /* default when no trigger matches */

    akt_q_e {
        trigger-keys = <E>;
        max-prior-idle-ms = <200>;
        bindings = <&kp G>;
    };
};
```

### Rules

| antecedent-morph | adaptive-key |
|---|---|
| `defaults = <…>` | top-level `bindings = <…>` |
| `antecedents = <A B C>` | three child triggers, **in the same order** |
| `bindings = <X Y Z>` | each trigger’s `bindings = <…>` |
| `max-delay-ms = <N>` | each trigger’s `max-prior-idle-ms = <N>` |
| separate `m_*` macro behaviors | optional inline sequences (see below) |

### Inline macros (recommended)

urob supports inline behavior sequences, which can replace many `ZMK_MACRO_F` helpers used only by adaptive keys:

```dts
/* was: bindings = <&m_dge>;  where m_dge = BSPC G E */
bindings = <&kp BSPC &kp G &kp E>;
```

Keep standalone `m_*` macros when they are referenced from `adaptive_*` overlay layers in `dacman56.keymap`.

### Semantic note

urob triggers on the **last key pressed** before the adaptive key (documented behavior). Your fork’s antecedent-morph docs say “released,” but the implementation also timestamps key-down events. After migration, verify typing feel on hardware; tune `max-prior-idle-ms` if needed.

---

## Full behavior conversion table

All timings use `max-prior-idle-ms = <200>` unless you introduce `ADAPTIVE_DELAY_FAST` (155) for specific keys later.

| Behavior | Default | Trigger 1 | Binding 1 | Trigger 2 | Binding 2 | Trigger 3 | Binding 3 |
|---|---|---|---|---|---|---|---|
| `ad_Q` | `&kp Q` | `E` | `&kp G` | — | — | — | — |
| `ad_W` | `&kp W` | `Q` | `BSPC SQT` | `E` | `BSPC D W` | — | — |
| `ad_E` | `&kp E` | `Q` | `BSPC G E` | `W` | `BSPC Y` | `R` | `&kp C` |
| `ad_U` | `&kp U` | `O` | `&kp M` | — | — | — | — |
| `ad_I` | `&kp I` | `O` | `M K` | — | — | — | — |
| `ad_O` | `&kp O` | `I` | `M K` | `P` | `&kp L` | `U` | `&kp M` |
| `ad_P` | `&kp P` | `O` | `&kp L` | — | — | — | — |
| `ad_A` | `&kp A` | `SEMI` | `BSPC G A` | `FSLH` | `BSPC B A` | — | — |
| `ad_S` | `&kp S` | `FSLH` | `BSPC B S` | — | — | — | — |
| `ad_SEMI` | `&kp SEMI` | `E` | `G C` | — | — | — | — |
| `ad_Z` | `&kp Z` | `X` | `BSPC V Y` | `C` | `&kp G` | — | — |
| `ad_X` | `&kp X` | `C` | `&kp Y` | — | — | — | — |
| `ad_C` | `&kp C` | `Z` | `BSPC G C` | `FSLH` | `BSPC B C` | — | — |
| `ad_M` | `&kp M` | `FSLH` | `BSPC J M` | — | — | — | — |
| `ad_FSLH` | `&kp FSLH` | `E` | `&kp B` | `M` | `&kp J` | `DOT` | `&kp L` |

Hold-tap wrappers (`hms_m_a`, `hm_m_c`, `hms_m_x`, `hms_m_m`) stay unchanged except they will reference the rewritten `&ad_*` nodes.

---

## Phased implementation

### Phase 1 — Build infrastructure

**Files:** `config/west.yml`, `.github/workflows/build.yml`

Replace fork remotes with vanilla ZMK + urob module:

```yaml
manifest:
  remotes:
    - name: zmkfirmware
      url-base: https://github.com/zmkfirmware
    - name: urob
      url-base: https://github.com/urob
    - name: ph
      url-base: https://github.com/phuertay
  projects:
    - name: zmk
      remote: zmkfirmware
      revision: v0.3
      import: app/west.yml
    - name: zmk-adaptive-key
      remote: urob
      revision: v0.3
    - name: zmk-keyboard-dacman56
      remote: ph
      revision: main
  self:
    path: config
```

Switch CI workflow:

```yaml
uses: zmkfirmware/zmk/.github/workflows/build-user-config.yml@main
```

**Acceptance:** GitHub Actions fetches upstream ZMK and urob module; build may fail until phase 2.

---

### Phase 2 — Rewrite `adaptive.dtsi`

**File:** `config/adaptive.dtsi`

1. Replace `ZMK_ADAPTIVE` macro with a `ZMK_ADAPTIVE_KEY` macro targeting `zmk,behavior-adaptive-key`.
2. Convert all 14 behaviors using the table above.
3. Either inline macro sequences in triggers **or** keep `m_*` macro behaviors (lowest risk: keep macros first, inline later).
4. Leave homerow-mod wrappers and non-adaptive macros untouched initially.

**Suggested macro skeleton:**

```dts
#define ADAPTIVE_DELAY 200

#define ZMK_ADAPTIVE_KEY(name, default_binding) \
    ad_##name: adapt_##name { \
        compatible = "zmk,behavior-adaptive-key"; \
        #binding-cells = <0>; \
        bindings = default_binding;
        /* child triggers appended via macro args or written explicitly */
    };
```

Because triggers are child nodes, explicit per-key definitions may be clearer than a complex macro.

**Acceptance:** Firmware compiles; no references to `zmk,behavior-antecedent-morph` remain.

---

### Phase 3 — Kconfig tuning

**File:** `config/dacman56.conf`

Add urob timing to approximate current macro behavior (`wait-ms = 7`, `tap-ms = 2`):

```conf
CONFIG_ZMK_ADAPTIVE_KEY_WAIT_MS=7
CONFIG_ZMK_ADAPTIVE_KEY_TAP_MS=2
```

Optional (only if build warns or triggers are trimmed):

```conf
# CONFIG_ZMK_ADAPTIVE_KEY_MAX_TRIGGER_CONDITIONS=32  # default
# CONFIG_ZMK_ADAPTIVE_KEY_MAX_BINDINGS=4             # default; longest inline seq is 3 keys + BSPC
```

**Do not remove Plover config yet** — fork Plover settings are still required until phase 4. That means phase 1–3 either:

- **Option A (recommended):** migrate adaptive only on `vanilla`, accept that Plover breaks until phase 4, or
- **Option B:** add `petercpark/zmk-hid-io-plover-hid` in the same PR as phase 4 before merging.

For a clean incremental path, use **Option A** and treat steno layers as temporarily broken on `vanilla` until phase 4.

---

### Phase 4 — Plover HID module (follow-up)

**Files:** `config/west.yml`, `config/dacman56.conf`, `config/dacman56.keymap`, `config/macros.dtsi`

1. Add to `west.yml`:

   ```yaml
   - name: zmk-hid-io-plover-hid
     remote: petercpark
     revision: main
   ```

2. Replace fork Plover Kconfig:

   ```conf
   # remove:
   # CONFIG_ZMK_PLOVER_HID=y
   # CONFIG_ZMK_HID_KEYBOARD_REPORT_SIZE=20

   CONFIG_USB_DEVICE_HID=y
   CONFIG_USB_HID_DEVICE_COUNT=2
   CONFIG_ZMK_HID_IO=y
   CONFIG_ZMK_HID_IO_PLOVER_HID=y
   ```

3. Keymap changes:
   - `#include <dt-bindings/zmk/hid-io/plover_hid.h>`
   - Include `dts/behaviors/plover_hid.dtsi` from the module (or define `plv` behavior locally)
   - Replace `&kp PLV_*` → `&plv PLV_*` in steno layer and macros

4. **Remap extended keys** — indices differ for X-keys:

   | Key | phuertay fork index | petercpark module index |
   |---|---|---|
   | `PLV_X3` | 25 | 40 |
   | `PLV_X4` | 26 | 41 |

   Standard steno keys (0–22) align.

**Acceptance:** Plover sees STN device; steno layer produces correct strokes.

---

### Phase 5 — Validation

#### CI

- [ ] `dacman56_left` build succeeds
- [ ] `dacman56_right` build succeeds
- [ ] `settings_reset` build succeeds

#### Hardware — adaptive keys

Exercise each adaptive key with its trigger sequences from the conversion table:

- [ ] Default output when typed alone
- [ ] Each trigger path (e.g. `Q`→`W`, `E`→`W`)
- [ ] Hold-tap + adaptive combos (`&hms_m_a`, `&hm_m_c`, `&hms_m_x`, `&hms_m_m`)
- [ ] Rapid rolling vs slow typing at the 200 ms boundary

#### Hardware — Plover (phase 4)

- [ ] USB exposes Plover HID interface
- [ ] Steno layer keys register in Plover
- [ ] Steno macros in `macros.dtsi` still work

---

## Files touched (summary)

| File | Phase | Change |
|---|---|---|
| `config/west.yml` | 1, 4 | Vanilla ZMK + modules |
| `.github/workflows/build.yml` | 1 | Upstream build workflow |
| `config/adaptive.dtsi` | 2 | Full rewrite to adaptive-key |
| `config/dacman56.conf` | 3, 4 | Adaptive Kconfig; later Plover Kconfig |
| `config/dacman56.keymap` | 4 | Plover binding syntax only |
| `config/macros.dtsi` | 4 | Plover binding syntax only |
| `README.md` | 5 | Update adaptive + Plover setup docs |

**Unchanged:** `build.yaml`, combo/macro logic (except Plover syntax), layer IDs, shield repo.

---

## Risks and mitigations

| Risk | Mitigation |
|---|---|
| urob triggers on key **press** vs expected **release** timing | Hardware test; adjust `max-prior-idle-ms` |
| Hold-tap init-order issues (known with antecedent-morph module) | urob advertises hold-tap compatibility; test `hms_m_*` early |
| Plover index mismatch for `PLV_X3`/`PLV_X4` | Explicit remapping table in phase 4 |
| ZMK `main` vs tagged release drift | Pin `v0.3` for both zmk and zmk-adaptive-key |
| Adaptive-only branch breaks steno until phase 4 | Keep `Adaptive` branch as production fallback |

---

## Rollback

- **Production:** stay on `Adaptive` branch (`phuertay/zmk` fork).
- **If vanilla fails:** revert `west.yml` + `adaptive.dtsi` + workflow; no hardware flash required.
- **Partial rollback:** keep vanilla ZMK but temporarily re-add ssbb `zmk-antecedent-morph` with old `adaptive.dtsi` if urob semantics feel wrong (drop-in compatible string migration path).

---

## Suggested PR sequence

1. **PR 1 (`vanilla`):** Phases 1–3 — vanilla ZMK + urob adaptive-key (Plover broken temporarily).
2. **PR 2 (`vanilla` or `vanilla-plover`):** Phase 4 — Plover HID module.
3. **PR 3:** README + cleanup (remove unused `m_*` macros if fully inlined).

Merge to `Adaptive` only after hardware sign-off on both PRs.
