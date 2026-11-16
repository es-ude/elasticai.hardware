# Elastic Node Hardware Platform v5 (ENv5)
This folder includes the hardware designs and documentation. 
The firmware can be used from the elasticAI.runtime ENv5 ([repo](https://github.com/es-ude/elastic-ai.runtime.enV5)).
For controlling the hardware from PC with Python CLI, please use the elasticAI.experiment-framework ([repo](https://github.com/es-ude/elastic-ai.experiment-framework)).

## State
Still active. Further details are in the curresponding readme's.
- Test version ([docu](test_version/readme.md), closed)
- Revision 1 ([docu](revision_1/readme.md), closed)
- Revision 2 ([docu](revision_2/readme.md), active)
- Tiny ([docu](tiny/readme.md), closed)

## Improvements to ENv4
- Using Cortex-M0 MCU instead of AVR 8-bit MCU, to gain more performance and lower power consumption.
- Using SPI as the communication interface between MCU and the FPGA to save more IO pins.
- Simplify the power supply circuit to reduce the space and improve reliability.
- Adding an energy meter chip to ease the energy measurement.
- Optimize the PCB layout to reduce the PCB size.

## Limitations
- ENv5 is focused to use for IoT applications. It is not very flexible to open new applications (e.g. medical, ...).
- ESP wireless chip has high programming overhead (replace with Pi Radio Module).
- No feedback about the finished FPGA programming state (no return of DONE).
- The SRAM is not used in any application (can be removed).
- Extension Pin Header includes a CS line which is connected to the Flash (replace with unused line).
- LEDs on MCU and on FPGA are not really used (can be unified).
- Two FPGA clocks are available, but the 32 MHz is not used (replace with one programmable soruce). 
- Missing logic reset-line to the FPGA (add with unused GPIO between MCU and FPGA).
- Button for ``FPGA_RESET`` is for controlling the programming state of the FPGA. It should be only controlled via a MCU GPIO.
