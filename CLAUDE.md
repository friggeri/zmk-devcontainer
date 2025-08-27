# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a ZMK (Zephyr Mechanical Keyboard) firmware development environment for a custom keyboard with analog joystick support. The workspace contains multiple git submodules:

- **ixkb-config**: Keyboard definition and configuration (board: ixkb)
- **zmk**: Fork of ZMK firmware with custom HID joystick support (branch: feat/hid-joystick)
- **zmk-analog-joystick-driver**: ADC driver that parses analog input and generates joystick events
- **zmk-joystick-listener**: Listens to joystick events and calls custom HID functions

## Build System and Commands

### West Configuration
The project uses West build system with Zephyr RTOS. Configuration is in `ixkb-config/config/west.yml`.

### Build Commands
```bash
# Build the firmware (from /workspaces/zmk-dev)
west build -s zmk/app -b ixkb -- -DZMK_CONFIG=/workspaces/zmk-dev/ixkb-config/boards/arm/ixkb

# Build with pristine build directory
west build -p -s zmk/app -b ixkb -- -DZMK_CONFIG=/workspaces/zmk-dev/ixkb-config/boards/arm/ixkb
```

### Testing
```bash
# Run a single test (from zmk/app directory)
./run-test.sh tests/<test-path>

# Run all tests
./run-test.sh all

# Run BLE-specific tests
./run-ble-test.sh <test-path>

# Environment variables for testing:
# ZMK_TESTS_VERBOSE=1       - More verbose output
# ZMK_TESTS_AUTO_ACCEPT=1   - Auto-accept test snapshots
```

## Architecture and Key Components

### HID Joystick Implementation
The custom joystick support spans multiple components:

1. **Hardware Layer** (`zmk-analog-joystick-driver/`):
   - Reads ADC values from analog joystick
   - Converts to logical joystick events
   - Driver bindings in `dts/bindings/joystick/zmk,joystick.yaml`

2. **Event Processing** (`zmk-joystick-listener/`):
   - Listens to joystick input events
   - Processes and forwards to HID layer
   - Configuration in `dts/bindings/zmk,joystick-listener.yaml`

3. **HID Layer** (`zmk/app/`):
   - Custom HID joystick report structures in `include/zmk/hid.h`
   - Implementation in `src/hid.c`
   - BLE HOG support in `src/hog.c`
   - USB HID support in `src/usb_hid.c`
   - Endpoint management in `src/endpoints.c`

### Key Files for Joystick Support
- `zmk/app/src/endpoints.c`: Contains `zmk_hid_joystick_report` handling
- `zmk/app/include/zmk/hid.h`: HID report structures
- `zmk/app/src/hog.c`: BLE HID-over-GATT implementation
- `zmk-analog-joystick-driver/drivers/input_analog_joystick/`: ADC driver
- `zmk-joystick-listener/src/joystick.c`: Event listener

### Board Configuration
- Board definition: `ixkb-config/boards/arm/ixkb/`
- Keymap: `ixkb-config/boards/arm/ixkb/ixkb.keymap`
- Device tree: `ixkb-config/boards/arm/ixkb/ixkb.dts`
- Build configuration: `ixkb-config/build.yaml`

### ZMK Module System
- Modules are registered via `zephyr/module.yml` in each module directory
- West manages dependencies through `west.yml` manifests
- Extra modules can be specified with `ZMK_EXTRA_MODULES` environment variable

## Development Workflow

### Working with Joystick Features
1. ADC configuration is in the device tree (`ixkb.dts`)
2. Joystick behavior is configured through Kconfig options
3. HID reports are sent via BLE (HOG) or USB endpoints
4. Use `CONFIG_ZMK_BLE` and `CONFIG_ZMK_USB` to enable respective transports

### Code Organization
- Core ZMK functionality: `zmk/app/src/`
- Behaviors: `zmk/app/src/behaviors/`
- HID implementation: `zmk/app/src/hid.c`, `zmk/app/src/hog.c`, `zmk/app/src/usb_hid.c`
- Events: `zmk/app/src/events/`
- Drivers: Custom drivers in module directories

### Configuration System
- Kconfig files define configuration options
- Device tree overlays configure hardware
- West manifests manage dependencies
- CMakeLists.txt files control the build process