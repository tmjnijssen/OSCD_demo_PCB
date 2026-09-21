# OSCD Demo PCB
[![DOI](https://zenodo.org/badge/1336786561.svg)](https://doi.org/10.5281/zenodo.22870261)

A demonstration / teaching PCB built by the [Open Science Community Delft](https://www.tudelft.nl/en/library/open-science-community-delft) to show what an open hardware project looks like in practice: full sources, an open licence, and everything needed to have the board made and reprogrammed by anyone.

It is a round, ~93 mm ESP32-C3 board shaped like the OSCD logo, with a ring of 12 addressable RGB LEDs, three sensors on a shared I²C bus, capacitive touch pads, and three push buttons. Plug it into USB-C, and the demo firmware lights the ring in response to tilt, touch, ambient light, and button presses.

| | |
|---|---|
| **Revision** | 1.0 |
| **Designer** | Dr. ir. Tim M. J. Nijssen, Open Science Community Delft |
| **Licence** | [CERN-OHL-P v2](LICENSE) (permissive) |
| **EDA tool** | KiCad (2026-era release — see [Opening the design](#opening-the-design)) |

---

## Features

| Block | Part | Interface |
|---|---|---|
| Microcontroller | ESP32-C3-MINI-1 (RISC-V, Wi-Fi + BLE, native USB) | — |
| RGB LEDs | 12 × WS2812B-2020, in a 63 mm ring | 1-wire, `GPIO8` |
| IMU | LSM6DS3 — 3-axis accelerometer + gyroscope | I²C `0x6A` |
| Ambient light | APDS-9306-065 | I²C `0x52` |
| Capacitive touch | MPR121 — 12 electrodes (6 buttons + 6-pad slider) | I²C `0x5A`, IRQ on `GPIO10` |
| Buttons | 3 × user push button, plus RESET and BOOT | `GPIO0/1/2` |
| Power | USB-C 5 V → HT7533-1 LDO → 3.3 V | — |
| Breakout | 2 × 2×5 pin header (5 V, 3.3 V, GND, 9 GPIO) | 2.54 mm |
| Test points | 5 V, 3.3 V, SDA, SCL (each via 100 Ω) | — |

There is no USB-UART bridge — the ESP32-C3's native USB peripheral is wired straight to the USB-C connector, protected by a USBLC6-2SC6 ESD array.

## Quick start

1. **Power it.** Connect a USB-C cable to `J1`. The board is bus-powered; no battery or jumper setting is needed.
2. **Install the toolchain.** Arduino IDE with the *esp32* board package by Espressif, board **ESP32C3 Dev Module**, and **Tools → USB CDC On Boot: Enabled**.
3. **Flash the demo.** Open [`firmware_examples/test_sensors/test_sensors.ino`](firmware_examples/test_sensors/test_sensors.ino) and upload it. Because there is no auto-reset circuit you must enter the bootloader by hand: **hold BOOT (SW2), tap RESET (SW1), release BOOT**, then click Upload. Tap RESET again when the upload finishes.
4. **Watch it.** The ring plays a rainbow sweep at boot, then reacts to tilt, touch, and the three buttons. Open the Serial Monitor at 115200 baud for sensor readings.

Full setup notes, required libraries, and troubleshooting are in [`firmware_examples/README.md`](firmware_examples/README.md).

## Pinout

| ESP32-C3 pin | Function on this board | Also on |
|---|---|---|
| `GPIO0` | Button **SW4** (active low, RC-debounced) | `J2.6` |
| `GPIO1` | Button **SW3** (active low, RC-debounced) | `J2.7` |
| `GPIO2` | Button **SW5** (active low, RC-debounced) | `J2.3` |
| `GPIO3` | APDS-9306 interrupt | `J2.4` |
| `GPIO4` | LSM6DS3 `INT1` | `J2.9` |
| `GPIO5` | LSM6DS3 `INT2` | `J3.3` |
| `GPIO6` | **I²C SCL** (4.7 kΩ pull-up) | `J3.4`, `TP5` |
| `GPIO7` | **I²C SDA** (4.7 kΩ pull-up) | `J3.5`, `TP6` |
| `GPIO8` | WS2812B data out → `D1` | `J3.6` |
| `GPIO9` | BOOT button **SW2** (strapping pin) | `J3.7` |
| `GPIO10` | MPR121 `IRQ` | `J2.8` |
| `GPIO18/19` | USB D− / D+ | — |
| `GPIO20/21` | UART0 RX / TX | `J3.8`, `J3.9` |
| `EN` | RESET button **SW1** | `J2.5` |

**Header pinouts** (2×5, 2.54 mm, odd/even numbering):

```
J2:  1 +5V    2 +3V3    3 GPIO2   4 GPIO3   5 EN
     6 GPIO0  7 GPIO1   8 GPIO10  9 GPIO4  10 GND

J3:  1 +5V    2 +3V3    3 GPIO5   4 GPIO6   5 GPIO7
     6 GPIO8  7 GPIO9   8 GPIO20  9 GPIO21 10 GND
```

**LED ring order:** `D1` sits at the 9 o'clock position and the chain `D1 → D2 → … → D12` runs clockwise, 30° apart.

**Touch electrodes:** `TP2` is the inner 6-pad slider on `ELE0…ELE5`; `TP1` is the outer ring of 6 buttons on `ELE6…ELE11`.

## Repository layout

```
OSCD_demo_PCB.kicad_pro     KiCad project
OSCD_demo_PCB.kicad_sch     Schematic (single sheet)
OSCD_demo_PCB.kicad_pcb     Board layout (4 layers)
schematic.pdf               Rendered schematic, for reading without KiCad
assets/
  OSCD_demo_PCB_symbols.kicad_sym    Project-local symbols
  OSCD_demo_PCB_footprints.pretty/   Project-local footprints (ESP32 module,
                                     touch pads, WS2812B-2020, logo, switches)
  graphics/                          OSCD logo artwork (DXF)
bom/ibom.html               Interactive HTML BOM for hand assembly
production/                 Fabrication outputs (gerbers, drill, BOM, CPL)
firmware_examples/          Arduino sketches
docs/                       Hardware and manufacturing documentation
```

## Opening the design

The sources were saved by a 2026-era KiCad release (schematic file format `20260306`, board format `20260206`). Open them with that release or newer; older KiCad versions will refuse the files.

Symbol and footprint libraries are project-local and resolved through `${KIPRJMOD}` in `sym-lib-table` / `fp-lib-table`, so cloning the repository and opening `OSCD_demo_PCB.kicad_pro` is enough — nothing has to be installed into your global libraries.

## Documentation

- **[docs/hardware.md](docs/hardware.md)** — how each subsystem works, jumper/configuration options, and design notes.
- **[docs/manufacturing.md](docs/manufacturing.md)** — board specification, ordering, and how to regenerate the fabrication files.
- **[firmware_examples/README.md](firmware_examples/README.md)** — toolchain setup, upload procedure, and what the demo does.

## Licence

Hardware and firmware are released under the **CERN Open Hardware Licence Version 2 – Permissive** ([`LICENSE`](LICENSE)). You may use, study, modify, manufacture and sell boards based on this design, including in closed-source products, provided you keep the copyright and licence notices and do not imply endorsement.

The OSCD logo and name are project branding; please replace them with your own if you distribute modified boards.

## Credits

Designed by Dr. ir. Tim M. J. Nijssen for the Open Science Community Delft.

