# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is a ZMK firmware configuration for the **roBa** custom split keyboard. The keyboard is a split design with:
- 44 keys total across left and right halves
- Seeeduino XIAO BLE controllers
- PMW3610 trackball sensor on the right half
- Rotary encoder on the left half
- 4 layers: default, NUM, ARROW, and Bluetooth
- Custom combos for common key combinations (Tab, Escape, quotes, language switching)

## Architecture

### Key Files

**Keymap Configuration:**
- `config/roBa.keymap` - Main keymap defining layers, combos, macros, and behaviors
- `config/roBa.json` - Keymap JSON representation for keymap-drawer

**Hardware Definitions:**
- `boards/shields/roBa/roBa.dtsi` - Core hardware definition (matrix transform, physical layout, sensors)
- `boards/shields/roBa/roBa_R.overlay` - Right half specific config (trackball, SPI setup)
- `boards/shields/roBa/roBa_L.overlay` - Left half specific config
- `boards/shields/roBa/roBa_R.conf` - Right half features (PMW3610 trackball driver config, ZMK Studio)
- `boards/shields/roBa/roBa_L.conf` - Left half features (minimal config)

**Dependencies:**
- `config/west.yml` - West manifest defining ZMK version (v0.3-branch) and external driver (zmk-pmw3610-driver)

**Build Configuration:**
- `build.yaml` - GitHub Actions matrix for building firmware (right half with studio-rpc-usb-uart, left half, settings_reset)

### Hardware Architecture

**Split Keyboard Design:**
- Uses `zmk,matrix-transform` with 11 columns × 4 rows
- Right half: columns 6-10 (col-offset = 6 in overlay)
- Left half: columns 0-4
- Diode direction: col2row
- GPIO pins defined per-half in overlay files

**Trackball Integration (Right Half Only):**
- PMW3610 sensor connected via SPI0
- Custom ZMK driver from kumamuk-git/zmk-pmw3610-driver
- Configured with automouse-layer (layer 4) and scroll-layers (layer 5) in keymap
- Extensive tuning options in roBa_R.conf (CPI, polling rate, power management, smart algorithm)
- Input listener setup in roBa_R.overlay for sensor integration

**Encoder (Left Half):**
- ALPS EC11 rotary encoder
- Sensor bindings defined per-layer in keymap

### Custom Behaviors

**`to_layer_0` macro:**
- Macro that switches to layer 0 and sends a key press
- Used to ensure escape or other keys return to base layer

**`lt_to_layer_0` behavior:**
- Custom hold-tap that activates momentary layer on hold, but uses `to_layer_0` macro on tap
- Ensures taps always return to base layer

## Build Process

**Firmware builds automatically via GitHub Actions:**
- Workflow: `.github/workflows/build.yml`
- Triggered on: push, pull_request, workflow_dispatch
- Uses: `zmkfirmware/zmk/.github/workflows/build-user-config.yml@main`
- Builds 3 artifacts: roBa_R (with Studio), roBa_L, settings_reset

**Build targets (from build.yaml):**
1. `seeeduino_xiao_ble` + `roBa_R` + `studio-rpc-usb-uart` snippet
2. `seeeduino_xiao_ble` + `roBa_L`
3. `seeeduino_xiao_ble` + `settings_reset`

**Manual build (local):**
ZMK typically requires West for local builds, which is not configured in this repo. This is designed for cloud builds via GitHub Actions.

## Keymap Visualization

**Automatic keymap drawings:**
- Workflow: `.github/workflows/draw.yml`
- Tool: keymap-drawer (caksoylar/keymap-drawer)
- Currently: Manual trigger only (`workflow_dispatch`)
- Outputs: `keymap-drawer/roBa.svg` and `keymap-drawer/roBa.yaml`
- To enable auto-generation on keymap changes, uncomment the `push:` trigger in draw.yml

**To manually regenerate keymap drawing:**
Trigger the "Draw Keymap" workflow from GitHub Actions tab

## Editing the Keymap

**Layer structure (config/roBa.keymap):**
- Layer 0: `default_layer` - Base alphas with mod-taps
- Layer 1: `NUM` - Numbers and symbols
- Layer 2: `ARROW` - Arrow keys, function keys, mouse clicks
- Layer 3: `Bluetooth` - BT profile selection, bootloader access
- Layers 4-5: Implicit automouse and scroll layers (defined in trackball config)

**Key features to preserve:**
- `&mt` behavior configured with `flavor = "balanced"` and `quick-tap-ms = <0>`
- Trackball automouse-layer and scroll-layers must match layer numbers
- Combos use key-positions (matrix indices), not key names
- Sensor bindings for encoder per-layer

**When modifying hardware:**
- Update both .dtsi (shared) and .overlay files (per-half)
- Update .conf files for feature flags
- Trackball tuning is extensive - refer to PMW3610 driver docs before changing

## ZMK Studio

The right half is configured with ZMK Studio support:
- Enabled via `CONFIG_ZMK_STUDIO=y` in roBa_R.conf
- Locking disabled: `CONFIG_ZMK_STUDIO_LOCKING=n`
- Requires `studio-rpc-usb-uart` snippet in build.yaml
- Allows runtime keymap editing via ZMK Studio web interface
