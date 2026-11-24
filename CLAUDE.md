# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

ZMK user-config repo for the **crosses** 36-key split keyboard (nice!nano, trackballs). No source code — only devicetree/Kconfig config compiled into firmware by ZMK.

## Git workflow

`main` is the stable, known-good config. Never commit to `main` directly: all new work goes on a feature branch and gets merged into `main` once it's been tested on the keyboard.

## Building

- CI only: `.github/workflows/build.yml` reuses `zmkfirmware/zmk/.github/workflows/build-user-config.yml@v0.3.0` and runs on **`workflow_dispatch` only** (not on push). To build a branch: `gh workflow run build.yml --ref <branch>`, fetch artifacts with `gh run download`.
- `build.yaml` = build matrix: `crosses_left`, `crosses_right` (right half is the central, built with ZMK Studio: `studio-rpc-usb-uart` snippet + `CONFIG_ZMK_STUDIO=y`), plus `settings_reset`.
- No local build/lint/test setup in the repo. Local build would need a west workspace from `config/west.yml`.

## Dependencies (`config/west.yml`)

- `zmk` pinned to a specific commit.
- `gggw-zmk-keebs` (branch `zephyr-4.1`): provides the crosses shields, the `gggw_crosses_36_layout` physical layout, trackball nodes (`trackball_peripheral`, `trackball_peripheral_split`, `trackball_peripheral_listener`) and `zip_*` input processors referenced by the overlays. Shield/board definitions are **not** in this repo.
- `urob/zmk-helpers` v0.3: `ZMK_HOLD_TAP`, `ZMK_COMBO`, `ZMK_TAP_DANCE`, `ZMK_MOD_MORPH` macros and key-position labels (`LT0`…`RH2` from `zmk-helpers/key-labels/36.h`).

## Keymap structure

`config/crosses.keymap` includes, in order:
1. `definitions.dtsi` — `XXX`/`___` aliases, layer index defines (`BASE 0`, `LOWER 1`, `RAISE 2`, `BASE_G 3`, `LOW_G 4`), key-position diagram, home-row-mod macros (`MAKE_HRM`, `HML_*`/`HMR_*`), combo timing constants.
2. `behaviors.dtsi` — mod-morphs, tap-dances, combos (combos use position labels, not numeric indices).
3. `links.dtsi` — currently empty placeholder.

Layer defines must match the order of layer nodes in the `keymap` block; adding/reordering a layer means updating `definitions.dtsi` and every `&mo`/`&to`/`&lt`/combo/listener reference.

Overlays (`crosses_left.overlay`, `crosses_right.overlay`) configure trackball input-processors and per-layer input listeners (e.g. scroll vs. pointer by layer). Overlays are compiled per shield and do **not** include `definitions.dtsi`, so layer names used there (e.g. `SCROLL` in the right overlay) must be defined or replaced with numbers.

## Other files

- `config/crosses.conf` — Kconfig (`CONFIG_ZMK_DISPLAY=y`).
- `config/info.json` — physical layout metadata for keymap tooling.
- `crosses.svg` — rendered keymap image shown in README; regenerate manually when the keymap changes.
- `zephyr/module.yml` — makes the repo a Zephyr module (`board_root: .`).
