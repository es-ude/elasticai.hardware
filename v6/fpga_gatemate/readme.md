# ENv6 FPGA Board: CologneChip GateMate M1A1/2

FPGA board (shield, rev 1.0) for the ENv6 base board with the CologneChip GateMate CCGM1A1 (schematic of the M1A1; the M1A2 is not designed yet). FPGA board ID: **2** (R1 + R2 + R3 = 750 Ω + 750 Ω + 27 Ω ≈ 1.5 kΩ to GND on `FPGA_DETECT`, see the [auto-identification](../readme.md#auto-identification-of-connected-boards)).

![Block diagram of the ENv6 FPGA board with GateMate M1A1](../0_docu/env6_fpga_gatemate_overview.svg)

## Contents (KiCad sheets)

| Sheet         | Content                                                                                                          |
|---------------|------------------------------------------------------------------------------------------------------------------|
| `fpga_config` | IS25LP064D quad-SPI flash (U8), configuration mode pins `CFG_MD0 … 3`, pull-ups, SerDes header (J1, not fitted)  |
| `fpga_power`  | Supply decoupling, Si5351A clock generator (U7) with 25 MHz crystal (Y1), test points TP1/TP2                    |
| `fpga_io`     | IO banks of the FPGA, status LED (D1 with R4 = 750 Ω)                                                            |
| `translation` | Five TXB0108 level shifters (U1 … U5) between the FPGA (`VDD_IO`) and the connector (`VDDD_MCU`)                 |
| `connectors`  | ENv6 connector (base board and application board), ID resistors R1 … R3                                          |

### Configuration and clock

- **Mode pins:** `CFG_MD0 … 3` have 10 kΩ pull-downs fitted (R15 … R18); the pull-ups to `VDD_IO` (R11 … R14) are not fitted. The default setting is therefore `0000`. A different mode is selected by fitting a pull-up and removing the pull-down.
- **Flash:** The IS25LP064D (U8) is connected to the SPI configuration pins of the FPGA and, via level shifter U3, to the SPI1/QSPI lines of the MCU. Pull-ups (R20 … R24, 10 kΩ to `VDD_IO`) are on `CFG_DONE`, `CFG_FAILED_N`, flash CS and the flash IO2/IO3 lines.
- **Clock:** The Si5351A (U7) is controlled via I2C directly from the connector (A1, A2) and enabled via `EN_FPGA_OSC` (A13). `CLK0` drives the FPGA clock input `CLK0` (`FPGA_SYS_CLK`, test point TP1) through a 27 Ω series resistor; `CLK7` is available at test point TP2. Spread spectrum is disabled (`SSEN` pulled low by R10, R9 not fitted). The crystal load capacitors C41 and C45 (12 pF) are placed but not fitted; the internal load capacitance of the Si5351A is used.
- **SerDes:** The SerDes lines (`SER_CLK`, `SER_TX`, `SER_RX`) are only routed to the pin header J1, which is not fitted. R19 (750 Ω) is the SerDes termination resistor.

## Power Supply

| Rail   | Pins      | Net label | Use                                                                                             |
|--------|-----------|-----------|-------------------------------------------------------------------------------------------------|
| `PSU1` | C3 … C8   | `1V0`     | FPGA core, PLL and SerDes (`VDD_CORE`)                                                          |
| `PSU2` | C11 … C16 | `2V5`     | All IO banks (`VDD_IO`), flash, `VCCA` of the level shifters, Si5351A output supplies, pull-ups |

`VDD_PLL` (R27), `VDD_SER_PLL` (R26) and `VDD_SER` (R25) are fed from `1V0` through a 27 Ω series resistor each. On the FPGA side each of these three nodes has 3 × 1 µF, 10 µF and 100 nF to GND as an RC filter. Replace the resistors by 0 Ω if the voltage drop at the pins gets too large (the datasheet allows 0.95 … 1.05 V in economy mode).
