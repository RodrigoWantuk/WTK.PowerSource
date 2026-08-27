# Schematic organization and net naming

## Principle

The EasyEDA project must contain **real electrical schematics**, not decorative block diagrams built from electrical wires/net labels.

An overview drawn with actual nets can accidentally merge domains and create hard shorts in the generated PCB netlist.

Documentation-only architecture diagrams belong in Markdown under `docs/` or must use non-electrical graphic primitives only.

## Planned eight schematic sheets

### 1. `CH_A_INPUT_BUCK`

Responsibilities:

- Channel-A input connector;
- TVS/polarity protection;
- eFuse footprint/bypass;
- input capacitors;
- XL4016 preregulator;
- Schottky;
- inductor;
- VPRE capacitors;
- VPRE DAC/control interface;
- hard VPRE limit/enable;
- external-buck fallback interface;
- relevant test points.

### 2. `CH_A_LINEAR_CV_CC`

Responsibilities:

- TIP36C pass stage;
- driver;
- analog CV loop;
- analog CC loop;
- loop-selection network;
- shunt interface;
- OVP/OCP;
- K_OUT_A and driver;
- reverse/backfeed protection;
- CC/CV-active and fault outputs.

### 3. `CH_A_MEAS_THERMAL`

Responsibilities:

- ADS1115_A;
- CD4051_A;
- voltage/current measurement conditioning;
- NTC networks;
- VIN/VPRE telemetry;
- fan driver/interface;
- measurement test points.

### 4. `CH_B_INPUT_BUCK`

Electrical equivalent of Sheet 1, entirely referenced to `GND_B`.

### 5. `CH_B_LINEAR_CV_CC`

Electrical equivalent of Sheet 2, entirely referenced to `GND_B`.

### 6. `CH_B_MEAS_THERMAL`

Electrical equivalent of Sheet 3, entirely referenced to `GND_B`.

### 7. `MCU_CONTROL_ISOLATION`

Responsibilities:

- STM32F103;
- 3V3/5V logic rails as applicable;
- DAC/control interfaces;
- 6N137-class PWM isolators;
- isolated I2C device;
- Channel-B enable/fault isolators;
- SWD/reset/boot;
- watchdog and control-side peripherals.

The isolation barrier must be represented by actual isolator components, not by direct wires.

### 8. `OUTPUT_SERIES_SYSTEM`

Responsibilities:

- output connectors/bornes;
- series/symmetric relay network;
- precharge/discharge elements if used;
- PE/chassis connections where present;
- system-level relay/interlock connections that genuinely involve both channels.

## Net-label rule

A net label is an electrical connection, not an annotation.

Use exact, domain-specific labels.

Examples:

```text
VIN_RAW_A      VIN_RAW_B
VIN_PROT_A     VIN_PROT_B
SW_A           SW_B
VPRE_A         VPRE_B
VSET_A         VSET_B
ISET_A         ISET_B
ISENSE_A       ISENSE_B
VSENSE_A       VSENSE_B
OUT_A_P        OUT_B_P
OUT_A_N        OUT_B_N
GND_A          GND_B
CC_ACTIVE_A    CC_ACTIVE_B
FAULT_HW_A     FAULT_HW_B
KOUT_CMD_A     KOUT_CMD_B
```

## Isolation-barrier naming

Never reuse one net name on both sides of an isolator.

Correct:

```text
I2C_SCL_A -> ISO1540 -> I2C_SCL_B
I2C_SDA_A -> ISO1540 -> I2C_SDA_B
PWM_VSET_B_MCU -> 6N137 -> PWM_VSET_B_ISO
```

Incorrect:

```text
SCL -> isolator -> SCL
```

Using one label on both sides risks telling the EDA/netlist that the two sides are already the same copper network.

## Ground symbols

Do not use a generic `GND` symbol throughout the project.

Use explicit nets/symbols:

```text
GND_A
GND_B
PE
CHASSIS
```

`OUT_A_N` and `OUT_B_N` are also not generic grounds because the low-side shunts create a measurable offset between each output negative and its internal channel ground.

## Inter-sheet connections

Use a controlled set of named nets between sheets rather than long fictional wires.

Example:

Sheet 1 creates `VPRE_A`; Sheet 2 receives `VPRE_A`.

Both are the same electrical net because the same label is intentionally used.

Local high-dv/dt or sensitive nets such as `SW_A`, internal FB nodes, compensation nodes, and op-amp summing junctions should remain local unless there is a real diagnostic reason to export them.

## Component annotation

Global automatic annotation is acceptable initially.

Do not force a page-number designator scheme if it fights EasyEDA's annotation workflow. A later controlled renumbering may be done once the schematic is stable.

## Electrical review rule

Before copying Channel A sheets into Channel B:

1. finish and review the A circuit;
2. verify pinouts against manufacturer datasheets;
3. verify every net name;
4. then duplicate and systematically change `_A` to `_B` and `GND_A` to `GND_B`;
5. audit that no A-domain label remains in the B sheets.

This is safer than designing A and B independently in parallel.
