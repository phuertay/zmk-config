In keybdname.conf you need to add `CONFIG_ZMK_PLOVER_HID=y`

Make sure the west.yaml file compiles against the right repo

Here's a list of the PloverHID keycodes (from `app/include/dt-bindings/zmk/keys.h`, using `#define PLV_ST (ZMK_HID_USAGE(HID_USAGE_VENDOR_PLOVER, 9))`)

 | Key | Code | 
 | --- | --- |
 | PLV_SL | 0 | 
 | PLV_TL | 1 | 
 | PLV_KL | 2 | 
 | PLV_PL | 3 | 
 | PLV_WL | 4 | 
 | PLV_HL | 5 | 
 | PLV_RL | 6 | 
 | PLV_A | 7 | 
 | PLV_O | 8 | 
 | PLV_ST | 9 | 
 | PLV_E | 10 | 
 | PLV_U | 11 | 
 | PLV_FR | 12 | 
 | PLV_RR | 13 | 
 | PLV_PR | 14 | 
 | PLV_BR | 15 | 
 | PLV_LR | 16 | 
 | PLV_GR | 17 | 
 | PLV_TR | 18 | 
 | PLV_SR | 19 | 
 | PLV_DR | 20 | 
 | PLV_ZR | 21 | 
 | PLV_NM | 22 | 
 | PLV_X1 | 23 | 
 | PLV_X2 | 24 | 
 | PLV_X3 | 25 | 
 | PLV_X4 | 26 | 
 | PLV_X5 | 27 | 
 | PLV_X6 | 28 | 
 | PLV_X7 | 29 | 
 | PLV_X8 | 30 | 
 | PLV_X9 | 31 | 
 | PLV_X10 | 32 | 
 | PLV_X11 | 33 | 
 | PLV_X12 | 34 | 
 | PLV_X13 | 35 | 
 | PLV_X14 | 36 | 
 | PLV_X15 | 37 | 
 | PLV_X16 | 38 | 
 | PLV_X17 | 39 | 
 | PLV_X18 | 40 | 
 | PLV_X19 | 41 | 
 | PLV_X20 | 42 | 
 | PLV_X21 | 43 | 
 | PLV_X22 | 44 | 
 | PLV_X23 | 45 | 
 | PLV_X24 | 46 | 
 | PLV_X25 | 47 | 
 | PLV_X26 | 48 | 
 | PLV_X27 | 49 | 
 | PLV_X28 | 50 | 
 | PLV_X29 | 51 | 
 | PLV_X30 | 52 | 
 | PLV_X31 | 53 | 
 | PLV_X32 | 54 | 
 | PLV_X33 | 55 | 
 | PLV_X34 | 56 | 
 | PLV_X35 | 57 | 
 | PLV_X36 | 58 | 
 | PLV_X37 | 59 | 
 | PLV_X38 | 60 | 
 | PLV_X39 | 61 | 
 | PLV_X40 | 62 | 
 | PLV_X41 | 63 | 



## Summary

The Antecedent-Morph behavior (adaptive keys) sends different behaviors, depending on which key was most recently
released before the antecedent-morph behavior was pressed, if this occurs within a configurable time period.

## Usage

To load the module, either use a ZMK version from phuertay that includes it, or add the following entries
to `projects` in `config/west.yml`.

```yaml
manifest:
  remotes:
    - name: phzmk
      url-base: https://github.com/phuertay
  projects:
    - name: zmk
      remote: phzmk
      revision: main
      import: app/west.yml
    - name: zmk-antecedent-morph
      remote: phzmk
      revision: v1
  self:
    path: config
```


## Antecedent-Morph

The configuration of the behavior consists of an array of `antecedents`, key codes with implicit modifiers, as well as
of a delay `max-delay-ms` in milli-seconds. If none of the `antecedents` was released during the `max-delay-ms` before
the antecedent-morph behavior is pressed, the behavior invokes the `defaults` binding. If, however, the `n`-th of the
key codes (with implicit modifiers) listed in the array `antecedents` was released within `max-delay-ms`, the behavior
invokes the `n`-th of the bindings of the `bindings` property.

### Configuration

If the key A is assigned the behavior `&ad_a` defined as follows, for example,

```dts
/ {
    behaviors {
        ad_a: adaptive_a {
            compatible = "zmk,behavior-antecedent-morph";
            label = "ADAPTIVE_A";
            #binding-cells = <0>;
			defaults = <&kp A>;
            bindings = <&kp U>, <&kp O>;
			antecedents = <Q Z>;
			max-delay-ms = <250>;
        };
    };
};
```

then by default, pressing this key issues the key press `&kp A`. But if it is preceded within 250 milli-seconds by Q or
Z, then `&kp U` or `&kp O`, respectively, are issued instead.

### Behavior Binding

- Reference: `&ad_a`
- Parameter: None

Example:

```dts
&ad_a
```

### Dead Antecedents

If some binding somewhere issues an illegal key code in the range beyond 0x00ff, this illegal key code is recognized and
tested as an antecedent, but then immediately discarded from the event queue. It therefore functions similarly to a dead
key. Note that these illegal key codes are specified with the *usage page* 0x07 as in the following example. If some key
is bound to `&kp DEAD_ANTE`, then it does not print anything, but still turns a subsequent A into a U.

```dts
#define DEAD_ANTE 0x070100

/ {
    behaviors {
        ad_a: adaptive_a {
            compatible = "zmk,behavior-antecedent-morph";
            label = "ADAPTIVE_A";
            #binding-cells = <0>;
			defaults = <&kp A>;
            bindings = <&kp U>, <&kp O>;
			antecedents = <DEAD_ANTE Z>;
			max-delay-ms = <250>;
        };
    };
};
```
