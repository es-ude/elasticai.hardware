# ENv6 Base Board – Schematic Review

Date: 2026-11-24

## Summary

The schematic of ENv6.main has two errors that should be fixed before the layout is frozen, two items that need a decision, and several minor notes. The connector pin assignment matches the readme and the GPIO map is complete.

**Scope:** the four KiCad sheets `MCU`, `Power`, `Peripherals` and `connector` (KiCad 10, file version 20260306) plus the top-level sheet. The PCB layout was not reviewed.

**Method:** `kicad-cli` was not available, so a script rebuilt the netlist from symbols, wires, labels, junctions and no-connect markers. The result was checked against `readme.md` and the manufacturer data sheets of the PAC1954 and LP873300. This does not replace a real ERC run in KiCad.

| Severity | Count | Items |
| --- | --- | --- |
| Error | 2 | `FPGA_DETECT` label typo; JTAG header J402 not connected |
| To verify | 2 | PAC1954 channel 1 polarity; TLV73333 output capacitor |
| Note | 7 | see Notes |

## Errors

### 1. Board detection `FPGA_DETECT` is not connected (`MCU.kicad_sch`)

The pull-up R401 (10 kΩ) and the filter capacitor C420 (1 µF) sit on a global label spelled `FPGA_DETCT`. MCU pin GP46 and connector pin B22 use `FPGA_DETECT`. The two names are different nets, so the ADC input has no pull-up and the detection of the FPGA board cannot work.

- **Fix:** rename the label `FPGA_DETCT` to `FPGA_DETECT`.
- **Why the ERC missed it:** the rule `single_global_label` is set to "ignore" in `main.kicad_pro`. Setting it to "warning" would flag such typos.
- The counterpart `APP_DETECT` (R402, C419) is correct.

### 2. JTAG probe header J402 is not connected (`MCU.kicad_sch`)

The four pins of J402 use outdated local labels `JTAG_TDO_GP17`, `JTAG_TCK_GP18`, `JTAG_TDI_GP16` and `JTAG_TMS_GP19`. Each appears only once in the project. The real signals are `JTAG_TCK_GP26`, `JTAG_TDO_GP27`, `JTAG_TDI_GP28` and `JTAG_TMS_GP30`. The old names also point at GP16 and GP17, which are the QSPI lines IO2 and IO3.

| J402 pin | Current label | Should be |
| --- | --- | --- |
| 1 | `JTAG_TDO_GP17` | `JTAG_TDO_GP27` |
| 2 | `JTAG_TCK_GP18` | `JTAG_TCK_GP26` |
| 3 | `JTAG_TDI_GP16` | `JTAG_TDI_GP28` |
| 4 | `JTAG_TMS_GP19` | `JTAG_TMS_GP30` |

J402 is marked DNP, which is why the error is easy to overlook. If the header is fitted, an external FPGA JTAG probe cannot reach the FPGA.

## Items to verify

### PAC1954 channel 1 has swapped polarity (`Peripherals.kicad_sch`)

On channel 1, SENSE1+ (pin 11) is wired to the load side (`PSU4`) and SENSE1– (pin 12) to the source side (`PSU_OUT4`). Channels 2 to 4 are wired correctly: SENSE+ on the source side. The pin assignment 11 = SENSE1+, 12 = SENSE1– matches the [PAC195X data sheet](https://cdn.sparkfun.com/assets/4/4/c/b/9/PAC195X_Datasheet.pdf) and the [reference schematic](https://onlinedocs.microchip.com/oxy/GUID-235ED391-98DA-4853-8086-D8B4EA690956-en-US-2/GUID-BF92F4DE-EA74-4E74-841F-06EF01F9BD72-high.pdf).

Without a bipolar configuration, channel 1 will read zero or negative values. Swap `PAC_SenseP_1` and `PAC_SenseN_1` at the chip, or configure the channel as bipolar in firmware.

| Channel | SENSE+ net | SENSE– net | Result |
| --- | --- | --- | --- |
| 1 | `PAC_SenseN_1` (load, `PSU4`) | `PAC_SenseP_1` (source, `PSU_OUT4`) | swapped |
| 2 | `PAC_SenseP_2` (source, `PSU_OUT3`) | `PAC_SenseN_2` (load, `PSU3`) | ok |
| 3 | `PAC_SenseP_3` (source, `PSU_OUT1`) | `PAC_SenseN_3` (load, `PSU1`) | ok |
| 4 | `PAC_SenseP_4` (source, `PSU_OUT2`) | `PAC_SenseN_4` (load, `PSU2`) | ok |

### No output capacitor directly at the TLV73333 (`Power.kicad_sch`)

The LDO output (pin 5) connects only to R307 (1 Ω) and the header J301. The only nearby capacitor, C322 (1 µF), sits behind R307 on the `VDD_MCU` side. A capacitor that far from the output, with 1 Ω in between, is marginal for LDO stability. Place a 1 µF capacitor directly at pin 5.

R307 also drops about 0.3 V at 300 mA, which matters during WLAN transmit. Consider 0 Ω or a smaller value if the shunt is not needed.

## Notes

| Item | Sheet | Note |
| --- | --- | --- |
| LP873300 `EN` (IC301 pin 6) | Power | Marked no-connect. The [data sheet](https://www.ti.com/lit/pdf/SNVSBK2) allows a floating pin. Whether the four rails start without I2C setup depends on the OTP configuration; check the default of this orderable part. |
| PAC1954 `SLOW/ALERT1` (U201 pin 1) | Peripherals | The only connection is R207, which is DNP. In the fitted state the pin floats. |
| microSD DAT1 / DAT2 (J201) | Peripherals | No pull-ups. The SD specification recommends them in SPI mode as well. |
| USB-C shield (J302 S1) | Power | Connected to GND through R301 (5.1 kΩ). This looks copied from the CC resistors; a direct connection or an RC network is usual. D+ / D– have no ESD protection. |
| WS2812B supply (D401) | MCU | Runs from 3.3 V (`VDD_MCU`). The data sheet minimum is 3.5 V, so operation is outside the specification, although it often works. |
| Rail numbering | Peripherals, connector | Connector pins C19–24 carry `PSU4` and C27–32 carry `PSU3`, so the order is swapped. PAC1954 channels map as K1 = `PSU4`, K2 = `PSU3`, K3 = `PSU1`, K4 = `PSU2`. Correct, but easy to mix up in firmware. |
| No `PWR_FLAG` symbols | all | The ERC may report "power pin not driven" on `VSYS` or `USB_VCC`. |

## Verified and next steps

Checked and without findings:

- Connector B401 pins match the tables in `readme.md`: I2C, SPI1, QSPI, JTAG, FPGA configuration, unused GPIOs, ADC, detection, `VSYS` and the `PSU` pins.
- Every GPIO from GP0 to GP47 has exactly one assignment.
- No duplicate reference designators; every part has a footprint; no open pins without a no-connect marker.
- USB series resistors (27 Ω), CC resistors (5.1 kΩ) and the RP2354B supply pins are plausible.
- The GND, AGND and OSCGND domains are separated cleanly by net ties.

Next steps:

- [ ] Rename `FPGA_DETCT` to `FPGA_DETECT`
- [ ] Reconnect J402 to the `JTAG_*_GP26/27/28/30` nets
- [ ] Decide on the PAC1954 channel 1 polarity (swap wiring or configure bipolar)
- [ ] Add a 1 µF output capacitor at TLV73333 pin 5
- [ ] Set `single_global_label` to "warning" in the ERC settings and run a full ERC in KiCad
- [ ] Update the PCB from the schematic and re-run the DRC after the net changes

Sources: [PAC195X data sheet (Microchip)](https://cdn.sparkfun.com/assets/4/4/c/b/9/PAC195X_Datasheet.pdf), [PAC1934 reference schematic](https://onlinedocs.microchip.com/oxy/GUID-235ED391-98DA-4853-8086-D8B4EA690956-en-US-2/GUID-BF92F4DE-EA74-4E74-841F-06EF01F9BD72-high.pdf), [LP8733xx data sheet (TI)](https://www.ti.com/lit/pdf/SNVSBK2).
