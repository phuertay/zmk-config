# Term layout reference

`kbdterm.c` is the **only authoritative** Term layout source for this repo.

- Extracted from Windows `kbtrm001.dll` (see file header).
- Do not infer Term key→character mappings from QWERTY labels or other Hands Down variants.
- When changing adaptives, read [`../term-layout.md`](../term-layout.md) and verify triggers against **keycap labels** pressed (see `dacman56.keymap` bindings).

To refresh: re-export from `kbtrm001.dll` and replace `kbdterm.c`, then update `docs/term-layout.md`.
