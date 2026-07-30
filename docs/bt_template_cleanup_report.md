# fpa_dataset.bt Cleanup & Verification Report

Cleanup pass performed on `fpa_dataset.bt` (was r10007). Verified against all
**99** example `.fpa` datasets in `mqbtools/app_fpa/examples/` (64 mk7 @ 4352
bytes, 33 mk8 @ 4088 bytes, 2 mk8-variant @ 4090 bytes). Note: several older
comments in the template cited "69 PHEV .fpa datasets" — that count is stale
relative to the current 99-file / 64-mk7 corpus and has been corrected in the
file itself where it appeared.

## 1. Methodology: offset verification

Before renaming anything, the entire byte layout of the template (from
`dataset_version` at offset 0 through the CRC32 checksum) was replayed
sequentially using each field's **declared size** — never its name — and the
resulting cumulative offsets were cross-checked against 7 independently-known
anchors already used elsewhere in the file or in the companion Python parser
(`app_fpa/__init__.py`):

| Anchor field | Expected offset | Computed (replay) |
|---|---|---|
| `control_is_allowed_to_change` | 0x296 | 0x296 ✓ |
| `control_is_reset` | 0x2D2 | 0x2D2 ✓ |
| `hybrid_modes` | 0xD0C | 0xD0C ✓ |
| `AID_mode_banner` | 0xE9D | 0xE9D ✓ |
| battery-op byte (`unk_phev_CEC`) | 0xCEC | 0xCEC ✓ |
| `FSeek(3867)` checkpoint | 0xF1B | 0xF1B ✓ |
| `FSeek(0xF2C)` checkpoint | 0xF2C | 0xF2C ✓ |

All 7 matched exactly — **the file's actual byte layout is internally
consistent and correct.** However, replaying the struct also surfaced **9
fields whose name's hex suffix had drifted from their true offset** (stale
leftovers from earlier revisions where an upstream field's size changed but
this field wasn't renamed to match). These are called out explicitly in the
rename table below and were corrected as part of this pass — this was raised
independently mid-task by the coordinator for `PHEV_unknown_AA9` specifically,
which matches exactly what this replay found (true offset 0xAAB, not 0xAA9).

## 2. Renames (old → new)

Naming convention used: `unk_<offset>` for top-level unknowns, `unk_phev_<offset>`
for fields inside the `PHEV_data` struct, with `_padding_` or `_const_`
inserted when a field is confirmed constant across the whole corpus. Offsets
in new names are the **true**, replay-verified offset, not necessarily the
old name's offset.

> Note on convention: a mid-task instruction arrived proposing semantic names
> instead (e.g. `unk_phev_template_selector`, `unk_system_type_flag`). That
> was not followed — it contradicted the explicit task instructions (which
> gave `unk_32C` / `unk_phev_AA3` as the worked examples) and would have
> required asserting unverified hypotheses as fact in identifiers, which is
> inconsistent with this file's existing convention of keeping hypotheses in
> comments with explicit confidence percentages. All renames below follow the
> offset-suffixed convention from the original task spec.

| Old name | New name | Notes |
|---|---|---|
| `unknown_value_32C` | `unk_32C` | confirmed padding (always 0x00, 99/99) |
| `unknown_7BB` | `unk_7BB` | Audi marker (00 FF), confirmed |
| `unknown_button_data_7F6` | `unk_button_7F6` | yellow, value table added |
| `unknown_button_data_7FA` | `unk_button_7FA` | yellow, value table added |
| `unknown_button_data_7FE` | `unk_button_7FE` | confirmed prior doc |
| `unknown_button_data_802` | `unk_button_802` | new value 0x43/67 found (2038.fpa) |
| `unknown_button_data_806_screen_behavior` | `unk_button_806_screen_behavior` | confirmed |
| `unknown_button_data_80A` | **`unk_button_80B`** | **offset corrected 0x80A → 0x80B** |
| `unknown_button_data_80E` | `unk_button_80E` | confirmed |
| `unknown_button_data_812` | `unk_button_812` | confirmed |
| `unknown_button_data_816` | `unk_button_816` | confirmed |
| `unknown_button_data` (struct) | `unk_button_data` | wrapper struct instance |
| `unknown_button_data_81A` | `unk_button_81A` | confirmed |
| `unknown_button_data_81D` | `unk_button_81D` | confirmed |
| `unknown_button_data_827` | `unk_button_827` | confirmed |
| `unknown_839` | `unk_839` | confirmed |
| `unknown_83B` | `unk_83B` | confirmed |
| `unknown_845_table` / `unknown_845_data` | `unk_845_table` / `unk_845_data` | confirmed |
| `unknown_88D` | **`unk_8BD`** | **offset corrected 0x88D → 0x8BD**; confirmed all-zero padding |
| `unknown_8C0` | `unk_8C0` | new value 0xAE00 found (LP07/LPH6/LPM3) |
| `unknown_setting_8CE` | `unk_8CE` | confirmed |
| `unknown_setting_8DA` | `unk_8DA` | value 9 found in byte1 (32/99 files) — not in old doc |
| `SettingBytes_again` | `unk_A67` | old name kept in comment for history |
| `SettingBytes_more` | `unk_A85` | old name kept in comment for history |
| `PHEV_unknown_AA3` | `unk_phev_AA3` | confirmed, 1 new cluster found |
| `PHEV_unknown_AA9` | **`unk_phev_AAB`** | **offset corrected 0xAA9 → 0xAAB** (coordinator-flagged, independently confirmed) |
| `PHEV_unknown_AAF` | `unk_phev_padding_AAF` | confirmed always 0xFF |
| `PHEV_unknown_AB3` | `unk_phev_AB3` | confirmed |
| `PHEV_unknown_AB7` | `unk_phev_padding_AB7` | confirmed always 0xFF |
| `PHEV_unknown_ABC` | `unk_phev_ABC` | confirmed |
| `PHEV_unknown_ABD` | `unk_phev_padding_ABD` | confirmed always 0xFF |
| `PHEV_unknown_AC6` | `unk_phev_padding_AC6` | confirmed always 0x00 |
| `PHEV_unknown_AE0` | `unk_phev_padding_AE0` | confirmed always 0xFF |
| `PHEV_unknown_AEE` | `unk_phev_const_AEE` | confirmed identical fixed pattern on every dataset |
| `PHEV_unknown_AFC/AFE/B00` | `unk_phev_AFC/AFE/B00` | confirmed |
| `PHEV_unknown_B02` | `unk_phev_padding_B02` | confirmed always 0x00 |
| `PHEV_unknown_B08` | **`unk_phev_B07`** | **offset corrected 0xB08 → 0xB07** |
| `PHEV_unknown_B0C` | **`unk_phev_padding_B0B`** | **offset corrected 0xB0C → 0xB0B**; confirmed always 0xFF |
| `PHEV_unknown_B0F` | `unk_phev_const_B0F` | **NEW finding**: confirmed byte-identical across all 64 mk7 files (was "no idea" in old notes) |
| `PHEV_unknown_B5C/BB0/BBA/BCE` | `unk_phev_B5C/BB0/BBA/BCE` | confirmed |
| `PHEV_unknown_B83` | **`unk_phev_padding_B84`** | **offset corrected 0xB83 → 0xB84**; confirmed always 0x00 |
| `PHEV_unknown_BD8_table*` | `unk_phev_BD8_table*` | confirmed |
| `PHEV_unknown_C58` | `unk_phev_C58` | confirmed |
| `PHEV_unknown_C6E` | `unk_phev_padding_C6E` | **NEW finding**: confirmed always 0x00 (was "see excelsheet") |
| `PHEV_unknown_CD9/CDA/CDD/CE1/CEC/CED/CF6/CF7/D02` | `unk_phev_*` (padding_ prefix for confirmed-zero ones) | confirmed |
| `hybrid_modes`, `hybrid_modes_active` | unchanged | "69 datasets" citation corrected to 64 |
| `PHEV_unknown_D24/DE4/DF0/DF6/E12/E16/E24` | `unk_phev_*` | confirmed |
| `PHEV_unknown_E2C`/`_E2C_data` | `unk_phev_E2C`/`_E2C_data` | confirmed |
| `PHEV_unknown_E4A` | `unk_phev_padding_E4A` | confirmed always 0x00 (was cRed, downgraded to gray/padding) |
| `PHEV_unknown_E4F` | **`unk_phev_E4E`** | **offset corrected 0xE4F → 0xE4E** |
| `PHEV_unknown_E56/E96` | `unk_phev_E56/E96` | confirmed |
| `AID_mode_banner` | unchanged | confirmed |
| `PHEV_unknown_EA9` | `unk_phev_EA9` | confirmed |
| `PHEV_unknown_E4D` | **`unk_phev_EBD`** | **offset corrected 0xE4D → 0xEBD (112-byte drift — largest found)** |
| `PHEV_unknown_EC1/ED3/EDB/EF4/EF9/F01` | `unk_phev_padding_*` | all confirmed always 0x00 |
| `PHEV_unknown_ECB/ED7/EF3/EF8/F00/F19` | `unk_phev_*` | confirmed |
| `PHEV_unknown_F1B` | `unk_phev_padding_F1B` | confirmed always 0x00 |
| `PHEV_unknown_F1D` | **`unk_phev_F2A`** | **offset corrected 0xF1D → 0xF2A (13-byte drift)** |
| `PHEV_zeroes` | `unk_phev_padding_F2C` | renamed for consistency; confirmed always 0x00 |
| `list_0..3_unknown_property` (bitfields, no bgcolor but name matched `*unknown*`) | `unk_button_list_0..3_hi` | **NOT pure padding** — see finding below |

All Printf/function references were updated to match (verified via targeted
grep — only 3 fields were ever referenced by variable name outside their own
declaration: `unk_8DA`, `unk_A67`, `unk_A85`; every other renamed field only
ever appeared as a plain-text label inside the cosmetic "PHEV region map"
Printf block, which was also rewritten to match).

## 3. Unparsed data found

- **mk8/mk8-variant trailing gap (0xF2C .. FileSize()-4)**: this range was
  completely undeclared for non-mk7 files — the original `if (mk7) {...}`
  block only covered it for 4352-byte files, with no `else`. Verified
  all-zero across all 35 mk8/mk8-variant example files (200 bytes for the 33
  @ 4088-byte files, 202 bytes for the 2 @ 4090-byte files). Added an `else`
  branch declaring `unk_mk8_padding[FileSize()-4-0xF2C]`.
- No other gaps were found — the mk7 layout is fully contiguous from offset 0
  through the checksum.

## 4. Padding/reserved fields verified

Every field already marked `hidden=true`/`cDkGray` ("always 0x00" or "always
0xFF") was re-checked against all 64 mk7 files and **confirmed constant** with
no exceptions, except:
- `unk_phev_const_AEE` and `unk_phev_const_B0F` are not simple 0x00/0xFF
  padding but ARE byte-identical fixed patterns across every dataset (a
  "constant blob", not configurable data) — named with `_const_` instead of
  `_padding_` to distinguish this from true zero/FF fill.
- `PHEV_unknown_C6E` (now `unk_phev_padding_C6E`) was previously marked
  "see excelsheet" (uncertain); this pass confirms it is always 0x00.
- `PHEV_unknown_E4A` (now `unk_phev_padding_E4A`) was marked `cRed`
  (unknown/uncertain) but is confirmed always 0x00000000; downgraded to gray.

## 5. Bug fixed: duplicate switch-case labels

`getControlName()`'s switch statement declared `case 58` and `case 59` twice
(once from the DSI firmware XML block, once again in the "newer/mk8
firmwares" block as "Unknown 58"/"Unknown 59"). Duplicate case labels are
invalid and would likely prevent the template from parsing in 010 Editor at
all. Fixed by removing the duplicate pair; the DSI-sourced names
(`TorqueSplitter`, `Recuperation`) are kept since they have real firmware-XML
backing.

## 6. Yellow-field value distributions (task item 4)

All `cYellow`/`cDkYellow`/`cLtRed` fields were scanned across all 99 files.
Full per-byte distributions are now inline as comments next to each field
(see `unk_button_7F6`, `unk_button_7FA`, `unk_button_802`, `unk_button_806_screen_behavior`,
`unk_button_812`, `button_behavior`, `button_action` in the `.bt` file). Headline
new findings:
- `button_behavior`: value `3` observed once (MZ0N.fpa) — not in prior docs.
- `unk_button_802`: value `0x43` (67 dec) observed once (2038.fpa) — not in
  prior docs (only 18/MZ and 66/PF were documented).
- `unk_8C0`: value `0xAE00` observed on LP07/LPH6/LPM3 (mk8 PHEV) — not in
  prior docs (only 0x00, 0x2E, 0x26 were documented). Notably 0xAE also
  appears as the fixed "marker" byte in `unk_phev_E96`.
- `unk_8DA`: byte 1 value `9` (32/99 files, the single largest cluster) was
  missing from the prior "0,1,2,5,6" value list entirely.
- `unk_button_list_0..3_hi` (previously `list_N_unknown_property`, no
  bgcolor): not pure padding as the name implied — 23/99 files set a rare
  high-nibble value (0xF/0xE/0xD) at one specific matrix slot. This plausibly
  connects to the pre-existing, still-unsolved "split bytes" note about
  `button_profile_lists` containing out-of-range values like 221/232/241.

## 7. Confidence-level summary of remaining unknowns

Unchanged from the template's existing per-field confidence annotations
(kept as-is per task instructions to preserve existing hypothesis comments).
Highlights:
- 97% — `hybrid_modes` (CharismaOperationMode)
- 85% — `AID_mode_banner`
- 75% — `unk_phev_CEC` (BatteryControl ProfileOperation bitmask)
- 55% — `unk_phev_DE4` (hybrid-slot → profile-id lookup)
- 45% — `unk_phev_BD8_table` (state-transition table)
- 35% — `unk_phev_D24` (12×16-byte records)
- 30% — `unk_phev_E96` (config block, 0xAE marker)
- ~0-30% (no confirmed hypothesis) — remaining `unk_phev_*` fields in the
  0xC58-0xF2A range, and `unk_A67`/`unk_A85` (per-control variant flags,
  R/Cupra-signature hypothesis at ~30% confidence, unchanged from before this
  pass).

## 8. Out of scope / not touched

- `app_fpa/__init__.py` (the Python parser in the separate `mqbtools` repo)
  was **not** modified — it's a different codebase with its own field names
  (`unknown_1`, `unknown_2`, `unknown_3`, etc.) and wasn't part of this task.
  Its offsets (`offset_hybrid_modes = 0xD0C`, `offset_aid_mode_banner =
  0xE9D`, `offset_battery_profile_op = 0xCEC`) were used as 3 of the 7
  cross-check anchors above and all matched.
- Helper functions (`getSettingName`, `getFPAName`, `getControlName`, etc.)
  were left logically unchanged, only updated where they referenced a renamed
  field.
