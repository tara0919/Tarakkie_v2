# Enable Wireless Split for Tarakkie_v2 (Xiao BLE)

This plan addresses the development of a fully wireless split keyboard with swappable modules using the Seeed Xiao nRF52840.

## User Review Required

> [!IMPORTANT]
> **Modular Connector Specs**: You mentioned "2 magnet pogo pins" for the modules. For a module with active components (Trackball, Encoder, Keys), you typically need at least **4 lines** for I2C (VCC, GND, SDA, SCL) or more for direct GPIO/SPI.
> **Question**: Does "2 magnet pogo pins" mean two *sets* of pins? Or literally 2 pins? (2 pins is only enough for a simple switch matrix scan, insufficient for trackballs/encoders without complex modulation).

> [!WARNING]
> **PMW3610 Interface**: The PMW3610 trackball sensor typically uses **SPI** (4 signal wires + Power). Using it on a swappable module requires routing SPI pins through the connector.
> **Conflict**: If you plan to swap a Trackball (SPI) with an Encoder (GPIO) or Keypad (GPIO/I2C), the pin definitions in the firmware (Device Tree) will clash.
> **Solution Choice**: 
> 1.  **Multiple Firmware**: Create different firmware builds (e.g., `firmware_trackball`, `firmware_encoder`) and flash the one matching your current setup. (Safest, Standard ZMK way).
> 2.  **Unified Bus (Complexity High)**: Make all modules speak I2C (requires a microcontroller or translator on every module) so they share the same bus.

> [!CAUTION]
> **Reversible (180°) Connection**: Connecting a module 180° rotated creates a high risk of reversing VCC and GND, which will **destroy** the electronics immediately.
> **Check**: Have you designed the pinout to be rotation-symmetrical (e.g., VCC-Data-Data-VCC) or have mechanical keying to prevent reverse insertion?

## Modular Architecture & Pinout Proposal

### 1. Hardware Architecture
To accommodate the swappable modules with limited Xiao BLE pins, we will use an **I2C-based design**:
-   **Main Key Matrix**: Controlled by **MCP23017** (I2C IO Expander). This frees up almost all Xiao pins for the modules.
-   **Module Interface (Pogo Pins)**: Will carry I2C, SPI, and Power to support any module type.
-   **Firmware Strategy**: User will flash specific firmware for the attached module (e.g., "Trackball Mode", "Numpad Mode").

### 2. Pinout Assignment (12-pin Connector)
To support all modules (Trackball, Trackpad, Analog), the 12-pin magnetic connector will be wired as follows:

1.  **VCC** (3.3V)
2.  **GND**
3.  **D4 (SDA)** - I2C for Trackpad / Modules
4.  **D5 (SCL)** - I2C for Trackpad / Modules
5.  **D8 (SCK)** - SPI for Trackball
6.  **D9 (MISO)** - SPI for Trackball
7.  **D10 (MOSI)** - SPI for Trackball
8.  **D6 (CS)** - SPI Chip Select
9.  **D7 (INT)** - Interrupt (Trackball/Trackpad)
10. **D2 (Enc A)** - Encoder Signal A
11. **D3 (Enc B)** - Encoder Signal B
12. **D1 (Enc SW)** - Encoder Push Switch / Analog X

> [!TIP]
> **Analog Module Note**: When using the Analog stick, D1 and D0 will be used for X/Y. Since 13 pins (VCC+GND+11 GPIOs) are available on Xiao but only 12 are on the connector, we can use D1 for the Encoder switch in most modules, and swap it for Analog X in the Analog module.

### 3. ZMK Configuration Setup
#### [NEW] `Kconfig.defconfig`
- Enable `ZMK_SPLIT`.
- **Central Role**: Explicitly set `ZMK_SPLIT_ROLE_CENTRAL=y` for `SHIELD_TARAKKIE_V2_LEFT` only.
- Enable `ZMK_SPLIT_BLE_CENTRAL_BATTERY_LEVEL_PROXY` for battery reporting (Central only).

#### [NEW] `tarakkie_v2.conf`
- `CONFIG_ZMK_SLEEP=y` (Deep Sleep enabled).
- `CONFIG_ZMK_IDLE_SLEEP_TIMEOUT=900000` (15 mins).
- **Drivers**: Enable `CONFIG_I2C=y`, `CONFIG_GPIO_MCP230XX=y`, `CONFIG_SPI=y`, and `CONFIG_INPUT=y`.

#### [MODIFY] `tarakkie_v2_left.overlay` & `right.overlay`
- **Matrix**: Defined using `microchip,mcp23017` compatible node (address 0x20).
- **Input Properties**: Use `zephyr,axis-x/y` for PMW3610 and `zephyr,code` for `gpio-keys`.
- **Modules**: Trackball configuration as the default.

### 4. Detailed Module Specifications (Firmware Variants)

We will create separate overlay snippets for each module configuration.

#### Module 1: Trackball + Encoder
- **Components**: PMW3610 (SPI) + EC11 Encoder (GPIO A/B) + Push Switch (GPIO)
- **Pin Mapping (Draft)**:
    - PMW3610: SPI Bus (D8/D9/D10) + CS (D6) + INT (D7)
    - Encoder A/B: D2, D3
    - Push Switch: D1

#### Module 2: 4 Keys
- **Components**: 4 x Mechanical Switches
- **Pin Mapping**: D0, D1, D2, D3 (Direct GPIO) OR I2C Expander if needed (but GPIO likely sufficient)

#### Module 3: Analog Pad + Encoder + 1 Key
- **Components**: Analog Joystick (X/Y ADC) + Encoder (A/B) + Push SW + 1 Key
- **Pin Mapping**:
    - Analog X/Y: D0, D1 (Xiao supports ADC on A0-A3)
    - Encoder A/B: D2, D3
    - Key/Switch: Needs I2C IO Expander on module? Or reuse CS/INT pins as GPIO?
    - *Constraint*: We are tight on GPIOs here. Might need I2C for the key/switch.

#### Module 4: Trackpad + Encoder
- **Components**: Trackpad (I2C) + Encoder
- **Pin Mapping**:
    - Trackpad: I2C (D4/D5) + INT (D7)
    - Encoder A/B: D0, D1

## Verification
1.  **Build Check**: Verify that `west build` succeeds with the `nice_view` or `trackball` snippets active.
2.  **Pinout Check**: User to confirm they can wire the MCP23017 and Pogo pins as proposed above.
