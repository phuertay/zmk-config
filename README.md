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
 | PLV_STR | 9 | 
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
 | PLV_NUM | 22 | 
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
