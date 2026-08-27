# PCB

This directory contains the schematic, PCB-layout, fabrication, rendering, and revision artifacts for WTK.PowerSource.

The electrical architecture and current engineering baseline are documented under [`../docs`](../docs). The canonical editable PCB source is expected to be maintained in **EasyEDA Pro** unless the project explicitly changes EDA tools.

## Directory layout

```text
PCB/
├── README.md
├── source/                 # complete EasyEDA Pro source/export
├── fabrication/            # Gerbers, drill files, fabrication BOMs and release packages
├── renders/                # top/bottom/3D renders and assembly references
└── revisions/              # hardware revision notes and change logs
```

## Current PCB baseline

- Two-layer FR-4.
- **1 oz copper** baseline; 2 oz is deliberately not required.
- Wide traces and copper pours for high-current paths.
- Top/bottom copper sharing and via stitching where useful.
- Dedicated PTH points may be used for short soldered power-wire jumpers when carrying the full current in PCB copper would unnecessarily constrain routing or fabrication.
- Kelvin sense routing must remain electrically separate from power-current routing, especially around the low-side current shunt.
- `GND_A`, `GND_B`, output-negative nodes, PE, and chassis are distinct electrical concepts and must never be merged by naming convenience.
- The two isolated channel domains must remain auditable throughout schematic capture and PCB layout.
- Manual assembly, repair, transistor replacement, relay replacement, and accessible test points are design priorities.

## Planned schematic sheets

The current schematic organization is expected to use eight real electrical sheets:

1. `CH_A_INPUT_BUCK`
2. `CH_A_LINEAR_CV_CC`
3. `CH_A_MEAS_THERMAL`
4. `CH_B_INPUT_BUCK`
5. `CH_B_LINEAR_CV_CC`
6. `CH_B_MEAS_THERMAL`
7. `MCU_CONTROL_ISOLATION`
8. `OUTPUT_SERIES_SYSTEM`

No decorative overview sheet should use electrical wires, power symbols, or net labels that could accidentally participate in the netlist. Documentation-only block diagrams belong under `docs/`.

## Net naming

Channel/domain suffixes are mandatory where ambiguity could cause a cross-domain short. Examples:

```text
VIN_RAW_A      VIN_RAW_B
VIN_PROT_A     VIN_PROT_B
VPRE_A         VPRE_B
VSET_A         VSET_B
ISET_A         ISET_B
OUT_A_P        OUT_B_P
OUT_A_N        OUT_B_N
GND_A          GND_B
FAULT_HW_A     FAULT_HW_B
CC_ACTIVE_A    CC_ACTIVE_B
```

Signals on opposite sides of an isolator must use different net names. For example:

```text
I2C_SCL_A -> isolator -> I2C_SCL_B
```

Never reuse one net label on both sides of an isolation barrier.

## Power-path notes

The initial design uses isolated 24 V supplies, but the board should avoid unnecessary redesign if a later revision uses up to 36 V input and approximately 30 V output per channel.

Power paths should be laid out around the actual thermal/mechanical assembly:

- input protection / optional eFuse;
- XL4016-class preregulator and Schottky;
- inductor and VPRE capacitors;
- TIP36C pass transistor;
- current shunt;
- K_OUT relay;
- output connector;
- optional power-wire reinforcement points.

A single heatsink assembly per channel is preferred where practical, but semiconductor tabs must be electrically isolated from the heatsink when their tabs belong to different circuit nodes.

## Fabrication releases

Suggested release structure:

```text
PCB/fabrication/
├── Rev1/
│   ├── Gerber_*.zip
│   ├── BOM_*.csv
│   ├── PickPlace_*.csv       # if assembly is ever used
│   └── Assembly_*.pdf
└── Rev2/
```

Do not publish a fabrication revision until the matching schematic, PCB, BOM, DNP list, and documentation describe the same electrical revision.

## Pre-release checklist

Before a PCB revision is considered ready for fabrication:

1. Run schematic ERC and PCB DRC.
2. Audit `GND_A` versus `GND_B` isolation explicitly.
3. Verify every power semiconductor pinout and metal-tab connection against the selected manufacturer datasheet.
4. Verify relay contact ratings for DC use and series-mode voltage.
5. Verify the external-fuse and optional-eFuse bypass population states.
6. Verify shunt Kelvin routing independently of the high-current copper path.
7. Verify creepage/clearance around the channel isolation barrier and any off-board wiring.
8. Verify that K_OUT is fail-safe de-energized/open.
9. Verify power-wire jumper footprints and assembly notes if populated.
10. Verify heatsink isolation hardware and mounting clearances.
11. Export Gerbers and inspect them in an independent viewer.
12. Export a BOM from the exact same source revision.
13. Record every DNP/TUNE/OPEN part used by the prototype.

The current engineering specification remains pre-prototype; bench qualification gates in [`../docs`](../docs) take precedence over optimistic schematic targets.
