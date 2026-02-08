# SP3000 Scanner Keypad — ZMK Firmware

Custom ZMK firmware for the Ceynetics MICRPAD BLE (CS2025A Rev.A), a Bluetooth/USB-C replacement keyboard for the Fuji Frontier SP3000 film scanner.

## Hardware Summary

| Component | Part | Notes |
|-----------|------|-------|
| MCU | E73-2G4M08S1C (nRF52840) | BLE 5.0 + USB-C |
| Matrix | 5 rows × 10 columns | row2col diodes (1N4148WS) |
| Switches | 43× Kailh MX hotswap | CPG151101S11-16 |
| Encoder | EC11E09244BS | Rotary + push, 80 steps |
| LEDs | 43× SK6812Mini-E | Via 74AHCT1G125 level shifter |
| Battery | BQ24075 charger + MAX17048 fuel gauge | TPS63020 buck-boost PSU |
| Crystal | 32.768 KHz | On XL1/XL2 (P0.00/P0.01) |

## GPIO Pin Mapping

Extracted from KiCad schematics by matching global label coordinates to E73 module pin positions.

### Matrix

| Signal | Pin | Devicetree | Signal | Pin | Devicetree |
|--------|-----|------------|--------|-----|------------|
| ROW0 | P1.00 | `&gpio1 0` | COL0 | P1.10 | `&gpio1 10` |
| ROW1 | P1.04 | `&gpio1 4` | COL1 | P0.28 | `&gpio0 28` |
| ROW2 | P1.06 | `&gpio1 6` | COL2 | P1.13 | `&gpio1 13` |
| ROW3 | P1.09 | `&gpio1 9` | COL3 | P0.29 | `&gpio0 29` |
| ROW4 | P1.11 | `&gpio1 11` | COL4 | P0.31 | `&gpio0 31` |
| | | | COL5 | P0.30 | `&gpio0 30` |
| | | | COL6 | P0.07 | `&gpio0 7` |
| | | | COL7 | P0.13 | `&gpio0 13` |
| | | | COL8 | P0.24 | `&gpio0 24` |
| | | | COL9 | P0.09 | `&gpio0 9` |

### Peripherals

| Signal | Pin | Notes |
|--------|-----|-------|
| ENC_A | P0.22 | Encoder channel A |
| ENC_B | P0.20 | Encoder channel B |
| LED_DATA | P1.02 | SPI MOSI → level shifter → SK6812 chain |
| I2C SDA | P0.02 | MAX17048 fuel gauge |
| I2C SCL | P0.03 | MAX17048 fuel gauge |

## Switch-to-Matrix Map

Verified from KiCad KEY_MATRIX.kicad_sch wiring:

```
         C0    C1    C2    C3    C4    C5    C6    C7    C8    C9
  R0:   SW2   SW7   SW11  SW16  SW21  SW26  SW31  SW36  SW41  ---
  R1:   SW3   SW8   SW12  SW17  SW22  SW27  ---   SW37  ---   SW34
  R2:   SW4   SW9   SW13  SW18  SW23  SW28  SW33  SW38  SW43  SW35
  R3:   SW5   SW1   SW14  SW19  SW24  SW29  ---   SW39  ---   SW42
  R4:   SW6   SW10  SW15  SW20  SW25  SW30  ---   SW40  ---   SW44
```

Empty positions (7 of 50): R0C9, R1C6, R1C8, R3C6, R3C8, R4C6, R4C8

## Keymap Status

**Current: PLACEHOLDER keycodes.** Based on SP3000 manual (PP3-B1271E2 §2.3):

- Color correction keys → KP4–KP9 (manual: "Used as numeral keys 4–9")
- Density correction keys → KP0–KP3 (manual: "Used as numeral keys 0–3")
- F1–F6 mapped directly; arrows, Enter, Tab, Escape as standard
- BT/RGB control layer on hold of R0C9

## Next Steps

### 1. Capture original keycodes (CRITICAL)

```bash
# On any Linux box with the original keyboard plugged in:
sudo evtest /dev/input/eventX
# Press every key, record the scancodes, update sp3000_keypad.keymap
```

### 2. Verify encoder push wiring

SW45 push may be at one of the empty matrix positions or on dedicated GPIO.

### 3. Build and flash

```bash
# GitHub Actions: push repo, UF2 builds automatically
# Local: west build -b sp3000_keypad -- -DZMK_CONFIG=/path/to/config
```

## File Structure

```
config/
├── boards/arm/sp3000_keypad/
│   ├── sp3000_keypad.dts          # Devicetree (GPIO, peripherals)
│   ├── sp3000_keypad.keymap       # Key bindings (EDIT THIS)
│   ├── sp3000_keypad_defconfig    # Kconfig defaults
│   ├── sp3000_keypad.yaml         # Board metadata
│   ├── Kconfig.board              # Board selection
│   ├── Kconfig.defconfig          # Board defaults
│   └── board.cmake                # CMake config
├── west.yml                       # West manifest
build.yaml                         # GitHub Actions build config
```
