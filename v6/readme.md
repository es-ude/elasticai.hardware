# Elastic Node Hardware Platform v6 (ENV6)
This folder includes the hardware designs and documentation.

## State:
Still in development. Available boards are:
- [x] Main board (MCU, WLAN, ...)
- [ ] FPGA Board with CologneChip GateMate M1A1
- [ ] FPGA Board with CologneChip GateMate M1A2
- [ ] FPGA Board with Lattice ICE40UP5K
- [ ] FPGA Board with AMD Spartan-7 S15
- [ ] FPGA Board with AMD Spartan-7 S50
- [ ] FPGA Board with AMD Artix-7 100T

## Changes to ENV5
- Removing the ESP32 chip and replace with RPi Radio Module
- Upgrading MCU from RP2040 to RP2354B (more GPIOs). Taking the internal flash version is optional.
- Upgrade the current sensor for runtime maintenance from PAC1934 to PAC1954.
- Refactored the power management path (new battery charger, programmable 4-channel power IC).
- Removed all LEDs and replace with one WS2812b LED.
- Add more debugging features to the FPGA (FLASH_DONE, JTAG, ...)

## Key new features
- Split between main board (MCU, WLAN, USB, Power, PAC-sensor), Application layer and FPGA board.
- Automatic identification of the connected FPGA board (further details below).
- Add a microSD card slot to the design.
- Add connectors for connecting power lines to external high-speed current measurement unit (e.g. Nordic PKK2)


## Auto-Identification of the FPGA Board ID

The MCU board identifies which swappable FPGA board is attached by measuring a resistor divider with the ADC sampling using the RP235x. Here, we assume using 1 % resistors (E12 values), 12-bit ADC, total noise/offset/INL error ±16 LSB. In order to detect the FPGA device in the right windows, an oversampling is done (x16) in order to decrease noise during read-out. For the worst-case tolerance of both resistors plus ±16 LSB, the minimum gap between windows is ~73 LSB.

### Setup

```
 3.3 V (ADC_AVDD)
     |
   [R_up = 10 kΩ, 1 %]       <- MCU board (fixed)
     |
     +------ ADC pin (+ 100 nF to GND)
     |
   [R_down = ID resistor]    <- FPGA board (unique per board)
     |
    GND
```
`Code = 4095 · R_down / (R_down + R_up)`


### ID Table

| ID   | R_down (FPGA board) | Nominal code | Window (min … max)   | FPGA target               |
|------|---------------------|--------------|----------------------|---------------------------|
| –    | short to GND        | 0            | 0 … 16 → fault       | None                      |
| 1    | 1.0 kΩ              | 372          | 350 … 395            | Lattice ICE40UP5K         |
| 2    | 1.5 kΩ              | 534          | 509 … 560            | CologneChip GateMate M1A1 |
| 3    | 2.2 kΩ              | 738          | 710 … 767            | CologneChip GateMate M1A2 |
| 4    | 3.3 kΩ              | 1016         | 985 … 1047           | reserved                  |
| 5    | 4.7 kΩ              | 1309         | 1275 … 1343          | reserved                  |
| 6    | 6.8 kΩ              | 1658         | 1622 … 1693          | reserved                  |
| 7    | 10 kΩ               | 2048         | 2011 … 2084          | AMD Spartan-7 S15         |
| 8    | 15 kΩ               | 2457         | 2421 … 2493          | AMD Spartan-7 S50         |
| 9    | 22 kΩ               | 2815         | 2781 … 2849          | AMD Artix-7-100T          |
| 10   | 33 kΩ               | 3143         | 3112 … 3173          | reserved                  |
| 11   | 47 kΩ               | 3377         | 3349 … 3404          | reserved                  |
| 12   | 68 kΩ               | 3570         | 3545 … 3595          | reserved                  |
| 13   | 100 kΩ              | 3723         | 3700 … 3745          | reserved                  |
| 14   | 150 kΩ              | 3839         | 3818 … 3860          | reserved                  |
| –    | open (no board)     | ≈ 0          | > 3900 → no board    | None                      |
