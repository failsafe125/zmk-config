# Antari macropad — ZMK config

ZMK firmware for the Antari2040 macropad PCB (https://github.com/nendezkombet/Antari2040),
running on either a Seeed XIAO nRF52840 (BLE) or a XIAO RP2040.
The original project only ships QMK/Vial firmware for the RP2040.
This config was reverse-engineered from its QMK `info.json`.

## Hardware facts (things the original repo doesn't tell you)

- The pad is 12 keys (4 wide x 3 tall) plus one EC11 encoder.
  The encoder push button is wired into the matrix as a 13th key,
  so the matrix is electrically 3 rows x 5 columns.
- Pin map (XIAO D-pin names, same pads on both chips):
  - Columns 0-4: D6, D5, D4, D3, D9
  - Rows 0-2: D0, D1, D2
  - Encoder A/B: D8 / D7
  - RGB data (12x SK6812 MINI): D10
- The 220 ohm resistor footprint near the XIAO is the LED data line.
  If it is not populated, the LEDs will never light. 220-470 ohm works.
- The LV/HV solder jumper selects LED voltage. Bridge LV (3.3V):
  it works on battery and matches the XIAO's 3.3V data signal.
  HV is 5V from USB only.
- The empty diode spots next to the encoder footprint: populate one
  (1N4148) when installing the encoder or the encoder press won't register.
  The BOM only counts 12 diodes; the encoder press needs a 13th.
- Battery pads (+/-) and the slide switch footprint are optional,
  only needed for wireless use (nRF52840 only).

## Repo layout

```
build.yaml                              builds firmware for BOTH boards
config/
  antari.keymap                         layers (shared by both boards)
  antari.conf                           features: encoder, RGB, ZMK Studio
  boards/shields/antari/
    antari.overlay                      matrix, transform, encoder, physical layout
    Kconfig.shield
    Kconfig.defconfig
    boards/
      seeeduino_xiao_ble.overlay        LED strip via SPI (nRF)
      seeeduino_xiao_rp2040.overlay     LED strip via PIO (RP2040)
```

## Building and flashing

Push to GitHub. Actions builds two .uf2 files (one per board), attached
as an artifact on the workflow run.

1. Double-tap the reset button on the XIAO.
2. A USB drive appears: XIAO-SENSE = nRF52840, RPI-RP2 = RP2040.
3. Drag the matching .uf2 onto the drive. It flashes and reboots itself.

nRF board: USB + Bluetooth. RP2040 board: USB only, no radio,
the Bluetooth keys on layer 2 do nothing.

## Layers

Layer 0 (default): media prev/play/next on top row, volume and arrows
below. Encoder twist = volume, press = mute.

Layer 1 (numbers): tap top-left key to toggle in and out. Right 3x3
grid becomes 7-8-9 / 4-5-6 / 1-2-3, bottom-left is 0.

Layer 2 (hold 2nd-row-left key): Bluetooth profile select and clear,
bootloader, ZMK Studio unlock, RGB toggle / effect / hue.
Encoder twist = LED brightness on this layer.

Layer 3 (hold 3rd-row-left key): numpad digits with / * - and
+ on the encoder press.

## Live remapping (ZMK Studio)

Open https://zmk.studio in Chrome or Edge with the pad on USB.
To unlock: hold the 2nd-row-left key and tap the top row 3rd key.
Studio changes are saved on the board and override this repo's keymap
until restored from inside Studio.

## Troubleshooting notes from the build

- "No shield named 'antari'": the shield folder must be at
  config/boards/shields/antari/ (not config/shields/).
- Keys type numbers instead of arrows: you're on layer 1,
  tap the top-left key.
- One dead key with working neighbors: check that key's hotswap
  socket seating, then its diode joints. Row/column both working
  means the fault is local to that key.
- All LEDs dead: 220 ohm resistor missing, LV jumper not bridged,
  or LED #1 bad (it gates the whole chain).
- ZMK USB logging (CONFIG_ZMK_USB_LOGGING=y) conflicts with this
  board's defaults unless CONFIG_SERIAL, CONFIG_CONSOLE and
  CONFIG_UART_CONSOLE are also set to y. Only enable for debugging.

## Case

Two printable options were generated from the original cutting file
(antari2040_cutting.ai): a faithful 4-plate stack (switch plate, two
mid frames, bottom) and a one-piece tray that replaces the bottom
three layers and the standoffs (M2x8 self-tapping screws instead).
Switch cutouts are traced at 13.9mm; FDM printers may need ~0.15mm
hole compensation or a 100.5% XY scale for switches to seat.
