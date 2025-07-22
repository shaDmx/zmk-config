# ZMK Keyboard Firmware Configuration

This repository contains custom ZMK (Zephyr Mechanical Keyboard) firmware configurations for split mechanical keyboards. The codebase supports two keyboard models: Corne Choc Pro BT and Piantor Pro BT, both with advanced features like homerow mods, tap dance behaviors, encoders, and mouse emulation.

## Repository Structure

### Core Configuration Files

- `build.yaml` - GitHub Actions build matrix configuration defining firmware build targets
- `config/west.yml` - West manifest file specifying ZMK framework dependencies
- `zephyr/module.yml` - Zephyr module configuration for custom board definitions

### Board Definitions

Each keyboard has its own board definition directory under `boards/arm/`:

#### Corne Choc Pro BT (`boards/arm/corne_choc_pro/`)
- `corne_choc_pro.dtsi` - Device tree source defining hardware configuration
- `corne_choc_pro-layouts.dtsi` - Physical layout definitions
- `corne_choc_pro_left.dts` / `corne_choc_pro_right.dts` - Left/right board definitions
- `corne_choc_pro.keymap` - Default keymap (simpler configuration)
- Hardware features: 4 encoders, nice!view display support, nRF52840 MCU

#### Piantor Pro BT (`boards/arm/piantor_pro_bt/`)
- Similar structure to Corne but for Piantor keyboard layout
- 42-key layout vs Corne's 44-key layout

### Active Keymaps

The primary keymaps are located in `config/`:
- `config/corne_choc_pro.keymap` - Advanced Corne keymap with custom behaviors
- `config/piantor_pro_bt.keymap` - Piantor keymap (uses board default)
- `config/corne_choc_pro.json` - Physical layout definition for keymap editor

## Key Features

### Advanced ZMK Behaviors

1. **Homerow Mods**: Sophisticated home row modifiers with separate left/right configurations
   - Left homerow mods (`hml`): GUI-A, Alt-S, Shift-D, Ctrl-F
   - Right homerow mods (`hmr`): Ctrl-J, Shift-K, Alt-L, GUI-;
   - Optimized timing: 280ms tapping term, 175ms quick-tap, 150ms prior idle

2. **Tap Dance Behaviors**:
   - `copy_paste`: Single tap = Ctrl+C, Double tap = Ctrl+V
   - `copy_paste_shift`: Single tap = Shift+Ctrl+C, Double tap = Shift+Ctrl+V

3. **Layer System**:
   - Layer 0: QWERTY (default)
   - Layer 1: NUMBER (numbers, F-keys, navigation)
   - Layer 2: SYMBOL (symbols, punctuation)
   - Layer 3: SETTINGS (Bluetooth, system controls)
   - Layer 4: MOUSE (mouse emulation with movement and scrolling)
   - Layers 5-9: Extra layers for future expansion

### Hardware Integration

- **Encoders**: 4 encoders per board for volume, page navigation, media control, brightness
- **RGB**: RGB underglow support with toggle controls
- **Nice!View Display**: OLED display integration
- **Bluetooth**: Full BLE support with device switching (5 profiles)
- **Mouse Emulation**: Pointer movement, clicking, and scrolling

## Build System

### GitHub Actions Matrix

The `build.yaml` defines multiple build configurations:

```yaml
# Studio-enabled builds with nice!view display
- board: corne_choc_pro_left
  snippet: studio-rpc-usb-uart
  shield: nice_view
  cmake-args: -DCONFIG_ZMK_STUDIO=y

# Settings reset builds for troubleshooting
- board: corne_choc_pro_left
  shield: settings_reset
  snippet: studio-rpc-usb-uart
```

Each keyboard builds 4 firmware variants:
1. Left half with nice!view + ZMK Studio support
2. Right half with nice!view
3. Left half with settings reset capability
4. Right half with settings reset capability

### Build Commands

For local development:
```bash
# Build specific board/shield combination
west build -b corne_choc_pro_left -- -DSHIELD=nice_view -DCONFIG_ZMK_STUDIO=y

# Clean build
west build -b corne_choc_pro_left --pristine

# Build settings reset firmware
west build -b corne_choc_pro_left -- -DSHIELD=settings_reset
```

## Development Workflow

### Making Keymap Changes

1. **Edit the keymap**: Modify `config/corne_choc_pro.keymap` or the respective board keymap
2. **Test locally** (if west toolchain installed):
   ```bash
   west build -b corne_choc_pro_left
   ```
3. **Commit and push**: GitHub Actions will automatically build firmware
4. **Download firmware**: Get `.uf2` files from Actions artifacts
5. **Flash**: Put keyboard in bootloader mode and drag firmware files

### Advanced Customization

#### Adding New Behaviors
Create custom behaviors in the keymap file:
```c
/ {
    behaviors {
        my_behavior: my_custom_behavior {
            compatible = "zmk,behavior-hold-tap";
            // configuration
        };
    };
};
```

#### Modifying Hardware Configuration
Edit the `.dtsi` files to:
- Change GPIO pin assignments
- Add/remove encoder support
- Modify display configurations
- Adjust power management settings

### Testing and Debugging

1. **Settings Reset**: Use settings reset firmware to clear NVRAM
2. **ZMK Studio**: Enable studio mode for real-time keymap editing
3. **Bootloader Mode**: Hold reset while plugging in USB
4. **Bluetooth Clearing**: Use `&bt BT_CLR` key combination

## Common Tasks

### Add a New Key Binding
1. Locate the appropriate layer in the keymap
2. Replace `&trans` with desired key code (e.g., `&kp ESC`)
3. Commit and rebuild

### Adjust Homerow Mod Timing
Modify these parameters in the behavior definition:
- `tapping-term-ms`: How long to hold for modifier
- `quick-tap-ms`: Rapid tap threshold
- `require-prior-idle-ms`: Idle time before activation

### Add Bluetooth Device
1. Use `&bt BT_SEL 0-4` to select profile slots
2. Pair normally with target device
3. Use `&bt BT_CLR` to clear current profile if needed

### Enable/Disable Features
Modify the board configuration files or use CMake flags:
- `-DCONFIG_ZMK_RGB_UNDERGLOW=y` - Enable RGB
- `-DCONFIG_ZMK_DISPLAY=y` - Enable display
- `-DCONFIG_ZMK_STUDIO=y` - Enable studio mode

## Dependencies

- **ZMK Framework**: Main firmware framework (auto-fetched via west.yml)
- **Zephyr RTOS**: Real-time operating system
- **nRF5 SDK**: Nordic semiconductor SDK for nRF52840
- **West**: Zephyr's meta-tool for project management

## Hardware Requirements

### Corne Choc Pro BT
- 2x nRF52840 microcontrollers
- 44x MX/Choc switches
- Optional: 4x rotary encoders
- Optional: 2x nice!view displays
- Optional: RGB underglow LEDs

### Piantor Pro BT
- 2x nRF52840 microcontrollers  
- 42x MX/Choc switches
- Similar optional features as Corne

## Troubleshooting

### Build Issues
- Ensure west toolchain is properly installed
- Check for syntax errors in `.keymap` files
- Verify board definitions match hardware

### Firmware Issues
- Flash settings reset firmware if behavior is erratic
- Check battery levels on wireless boards
- Verify Bluetooth pairing status

### Layout Issues
- Use ZMK Studio for real-time debugging
- Check layer activation key combinations
- Verify homerow mod timing and trigger positions

This configuration represents an advanced ZMK setup with extensive customization for productivity-focused keyboard layouts.