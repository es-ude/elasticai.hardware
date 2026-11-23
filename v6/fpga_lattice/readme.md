# ENv6 FPGA Board: Lattice iCE40UP5K

FPGA board (shield, rev 1.0) for the ENv6 base board. FPGA board ID: **1** (R5 = 1 kΩ, see the [auto-identification](../readme.md#auto-identification-of-connected-boards)).

![Block diagram of the ENv6 FPGA board with iCE40UP5K](../0_docu/env6_fpga_lattice_overview.svg)

## Contents

- Lattice iCE40UP5K-SG48 (U1)
- W25Q16 configuration flash (U2)
- Si5351A clock generator (J1) and crystal (`FPGA_OSC_EN` signal from the connector)
- Status LED (D1)
- ENv6 connector: SPI1 for flashing and user logic, configuration lines (`PR_DONE`, `PR_RST`, `RST_LOGIC`), I2C, 12 FPGA IOs (`FPGA_IO_A_0 … 11`) plus further FPGA IOs towards the application board
