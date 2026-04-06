# Integrated Board

Compact control and power board for two solenoids, built around a SAM D21 MCU, DRV8432 power drivers, and a reverse-polarity-protected 52 V input stage.

![3D board view](hardware/renders/3D_IC%20Mounted%20board_2026-04-06%20%281%29.png)

## Project Summary

This repository contains the hardware design files and support documents for a compact solenoid-driver board project. The board was developed around these main functions:

- `2` solenoid outputs driven by `DRV8432`
- MCU control using a `SAM D21`
- reverse polarity protection on the main power input
- compact mechanical packaging for a narrow board format

## Mechanical Targets

- Board size target: `31 mm x 121 mm`
- Maximum component height target: `7 mm`

## Main Design Blocks

- `MCU`: SAM D21 family controller
- `Power drivers`: DRV8432 motor/solenoid driver stages
- `Input protection`: reverse-polarity protection stage for the main supply rail
- `Outputs`: intended for direct solenoid drive

## Repository Layout

```text
.
|-- README.md
|-- docs
|   |-- notes
|   |   `-- reverse-polarity-protection-52V-14A.md
|   `-- references
|       |-- BSC093N15NS5.pdf
|       |-- FDMS86200.pdf
|       |-- ltc4359.pdf
|       |-- SQM120P10 datasheet.pdf
|       |-- WBDesign206.pdf
|       `-- XRS200N15TL.pdf
|-- hardware
|   |-- pcb
|   |   `-- PCB_IC Mounted board_2026-04-06.pdf
|   |-- renders
|   |   |-- 3D_IC Mounted board_2026-04-06 (1).png
|   |   `-- 3D_IC Mounted board_2026-04-06.png
|   `-- schematics
|       |-- SCH_IC Board_2026-04-06.pdf
|       |-- SCH_IC Board_2026-04-06 (1).pdf
|       |-- SCH_IC Board_2026-04-06 (2).pdf
|       `-- SCH_IC Board_2026-04-06 (3).pdf
|-- project
|   `-- easyeda
|       |-- ProDoc_Integrated Board_2026-04-06.epro
|       `-- ProDoc_Integrated Board_2026-04-06FINAL.epro
|-- DRV8432DKDR.pdf
|-- DRV8432EVM.pdf
|-- SAM-D21-DA1-Family-Data-Sheet-DS40001882.pdf
|-- SCH_IC Board_2026-04-06.pdf
`-- sparkfun-samd21-pro-breakout-ATSAMD21G18.pdf
```

## Key Files

- `project/easyeda/`: EasyEDA Pro project files
- `hardware/schematics/`: exported schematic PDFs
- `SCH_IC Board_2026-04-06.pdf`: root-level schematic export that remained locked during reorganization
- `hardware/pcb/`: PCB export
- `hardware/renders/`: 3D board renders
- `docs/notes/`: design notes created during the power-input sizing work
- `docs/references/`: supporting datasheets and comparison references

## Notes

- Multiple schematic PDFs are kept because they appear to be export or revision snapshots from the design process.
- The reverse-polarity sizing note captures the design exploration around the `LTC4359` and candidate MOSFETs.
- Some reference PDFs are still at the repo root because they were locked by another process during cleanup; they are still part of the project reference set.

## Preview Assets

- Top-side render: [hardware/renders/3D_IC Mounted board_2026-04-06 (1).png](hardware/renders/3D_IC%20Mounted%20board_2026-04-06%20%281%29.png)
- Bottom-side render: [hardware/renders/3D_IC Mounted board_2026-04-06.png](hardware/renders/3D_IC%20Mounted%20board_2026-04-06.png)
- PCB export: [hardware/pcb/PCB_IC Mounted board_2026-04-06.pdf](hardware/pcb/PCB_IC%20Mounted%20board_2026-04-06.pdf)
