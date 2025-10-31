# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Keyball keyboard series firmware repository based on QMK firmware. Keyball keyboards are split keyboards with integrated trackballs (34mm PMW3360 optical sensor).

**Supported Models:**
- Keyball39: 39 keys + trackball
- Keyball44: 44 keys + trackball
- Keyball46: 46 keys + trackball
- Keyball61: 61 keys + trackball
- ONE47: 47 keys + trackball

## Build Commands

**Prerequisites:**
This repository is designed to be symlinked into QMK firmware's keyboards directory. The official workflow is:
```bash
# Clone QMK firmware (0.22.14 verified)
git clone https://github.com/qmk/qmk_firmware.git --depth 1 --recurse-submodules --shallow-submodules -b 0.22.14 qmk

# Create symlink from QMK's keyboards directory to this repo
cd qmk/keyboards
ln -s /path/to/this/keyball/qmk_firmware/keyboards/keyball keyball
```

**Build firmware:**
```bash
# From QMK firmware root directory
make SKIP_GIT=yes keyball/keyball44:default
make SKIP_GIT=yes keyball/keyball39:default
make SKIP_GIT=yes keyball/keyball46:default
make SKIP_GIT=yes keyball/keyball61:default
```

**Flash firmware:**
```bash
make SKIP_GIT=yes keyball/keyball44:default:flash
```

## Standard Keymaps

Each keyboard provides three standard keymaps:
- `via` - For use with Remap (https://remap-keys.app/) or VIA configurator
- `test` - For testing keyboard operation during build
- `default` - Base template for custom firmware

## Architecture

**Directory Structure:**
- `drivers/pmw3360/` - PMW3360 optical sensor driver for trackball
- `lib/keyball/` - Core Keyball library with trackball functionality
- `lib/oledkit/` - OLED display utilities
- `lib/duplexmatrix/` - Split keyboard matrix handling
- `keyball{39,44,46,61}/` - Individual keyboard implementations
- `one47/` - ONE47 keyboard implementation

**Key Files:**
- `lib/keyball/keyball.c` - Core trackball and scroll functionality
- `lib/keyball/keyball.h` - API definitions and configuration
- `lib/keyball/keycodes.md` - Special keycode documentation
- `{keyboard}/rules.mk` - Build configuration for each keyboard
- `{keyboard}/info.json` - QMK keyboard metadata

## Keyball-Specific Features

**Special Keycodes** (defined in `lib/keyball/keycodes.md`):
- `KBC_RST` / `KBC_SAVE` - Reset/save configuration to EEPROM
- `CPI_I100` / `CPI_D100` / `CPI_I1K` / `CPI_D1K` - Adjust trackball CPI (100-12000)
- `SCRL_TO` / `SCRL_MO` - Toggle/momentary scroll mode
- `SCRL_DVI` / `SCRL_DVD` - Adjust scroll divider (speed)
- `AML_TO` / `AML_I50` / `AML_D50` - Automatic mouse layer control
- `SSNP_VRT` / `SSNP_HOR` / `SSNP_FRE` - Scroll snap modes (vertical/horizontal/free)

**Scroll Snap Modes:**
- Vertical (default): Restricts scrolling to vertical direction
- Horizontal: Restricts scrolling to horizontal direction
- Free: No restriction, allows diagonal scrolling

**Configuration Macros** (in keymap `config.h`):
- `KEYBALL_SCROLL_DIV_DEFAULT` - Default scroll divider (default: 4 = 1/8 speed)
- `KEYBALL_SCROLLSNAP_ENABLE` - Enable/disable scroll snap (default: enabled)
- `KEYBALL_SCROLLBALL_INHIVITOR` - Delay after scroll mode toggle (default: 50ms)

## Common API Functions

**Core functions** (from `lib/keyball/keyball.h`):
- `keyball_set_scroll_mode(bool enable)` - Enable/disable scroll mode
- `keyball_get_scroll_mode()` - Get current scroll mode state
- `layer_state_set_user(layer_state_t state)` - Typically used to auto-enable scroll mode on specific layers

**Example pattern** (from default keymaps):
```c
layer_state_t layer_state_set_user(layer_state_t state) {
    // Auto enable scroll mode when highest layer is 3
    keyball_set_scroll_mode(get_highest_layer(state) == 3);
    return state;
}
```

## Hardware Details

- **MCU:** ATmega32U4
- **Bootloader:** Caterina (Pro Micro compatible)
- **Communication:** Split keyboard using serial
- **Sensor:** PMW3360 optical sensor via SPI
- **Display:** OLED support (optional, enable in keymap rules.mk)

## Important Build Settings

From `rules.mk`:
- `LTO_ENABLE = yes` - Required for firmware size optimization
- `POINTING_DEVICE_ENABLE = yes` with `POINTING_DEVICE_DRIVER = custom`
- `SPLIT_KEYBOARD = yes` for split models
- OLED is disabled by default in keyboard-level rules.mk, enable in keymap-level rules.mk if needed
