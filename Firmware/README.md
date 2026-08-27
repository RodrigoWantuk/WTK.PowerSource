# Firmware

This directory contains the firmware for the single **STM32F103C8T6** controller used by WTK.PowerSource.

The firmware is responsible for user interaction, setpoint generation, telemetry, slow preregulator optimization, thermal supervision, relay sequencing, calibration, diagnostics, and fault reporting. It is **not** the sole safety layer: fast CV/CC regulation and critical OVP/OCP/thermal shutdown paths remain implemented in hardware.

## Planned directory layout

```text
Firmware/
├── README.md
├── config/
├── src/
│   ├── app/
│   ├── bsp/
│   ├── control/
│   ├── drivers/
│   ├── hardware/
│   ├── measurement/
│   ├── protection/
│   └── ui/
├── tests/
├── tools/
└── third_party/
```

The exact build system and dependency set will be committed when firmware implementation starts. The structure intentionally mirrors the organization used by other WTK embedded projects without pretending that the PowerSource firmware already exists.

## Core responsibilities

### `app/`

Top-level application state machine, startup/shutdown sequencing, operating-mode transitions, channel enable/disable logic, and coordination between UI and hardware services.

### `bsp/`

Board-specific STM32 initialization and pin/peripheral mappings: timers, PWM, I2C, ADC where used locally, GPIO, watchdog, SWD/debug configuration, and board revision definitions.

### `control/`

Setpoint and slow-control logic:

- `VSET` and `ISET` generation;
- VPRE target computation;
- current-source user mode and compliance-voltage handling;
- series/symmetric mode coordination;
- channel enable/disable sequencing;
- relay dwell/debounce/interlock behavior.

Fast current regulation must remain analog. Firmware must not implement a fast software PID that replaces the CC hardware loop.

### `drivers/`

Reusable device drivers, expected to include devices such as:

- ADS1115-class ADC;
- MCP4725-class DAC;
- isolated I2C path;
- display/front-panel peripherals when selected;
- NTC/multiplexer support;
- optional eFuse status/control interface.

### `hardware/`

Logical hardware abstraction for Channel A, Channel B, isolation behavior, relays, fan outputs, hardware-fault inputs, CC/CV-active signals, and preregulator enable paths.

### `measurement/`

Calibrated telemetry processing:

- output voltage;
- output current;
- VPRE and raw-input voltage;
- thermal sensors;
- calibration slope/offset records;
- plausibility checks and sensor-fault detection.

Initial targets are approximately **5–10 mA usable output-current resolution** and at least **10 Hz telemetry refresh for every supervised quantity**, with faster refresh for voltage/current where useful.

### `protection/`

Firmware-side supervision and fault state management. This layer must distinguish hardware faults from communication or telemetry faults.

Examples:

- `FAULT_HW_A/B`;
- overtemperature warning/shutdown state;
- persistent Channel-B communication failure;
- ADC/DAC invalid state;
- eFuse fault/status;
- backfeed detection result;
- watchdog/reset cause.

Critical hardware faults retain direct authority over K_OUT and/or preregulator enable even if firmware is stalled.

### `ui/`

Front-panel presentation and user controls. The UI should expose actual hardware state rather than hiding transitions between CV, CC, compliance limiting, fault, communication-degraded, and output-disabled states.

## Voltage-oriented and current-source operation

Both modes use the same analog CV/CC hardware.

### Voltage-oriented mode

```text
VSET = requested voltage
ISET = maximum allowed current
```

### Current-source mode

```text
ISET = requested current
VSET = compliance-voltage ceiling
```

The analog control loop automatically settles at the current target when the load can support it. If the compliance voltage is reached first, the channel becomes voltage-limited. Firmware configures and displays this behavior; it does not continuously hunt the voltage in software to force the current.

## VPRE management

Firmware should keep the switching preregulator only high enough to provide useful headroom to the linear stage, reducing TIP36C dissipation.

Normal behavior is conceptually:

```text
VPRE_target ≈ measured/output target + required linear-stage headroom
```

The exact headroom and transition rules remain TUNE parameters.

During CC/short conditions, hardware must remain safe before firmware reacts. Firmware-side VPRE reduction is an optimization and secondary protection mechanism, not the only SOA protection.

## Channel-B isolation and I2C recovery

Only one MCU exists, located in the Channel-A/control domain. Channel B is electrically isolated.

The STM32F103 I2C peripheral requires explicit recovery handling. The firmware design must include:

1. timeout for every isolated-I2C transaction;
2. bus recovery using GPIO/open-drain clocking and STOP generation when appropriate;
3. the STM32F1-specific recovery sequence required for a stuck `BUSY` condition;
4. peripheral reinitialization after recovery;
5. bounded retry count;
6. `COMM_B_FAULT` after persistent failure;
7. fail-safe disable of `K_OUT_B` and `VPRE_ENABLE_B` after persistent loss of Channel-B communication.

The global watchdog is not considered sufficient coverage for an I2C peripheral lock-up while the CPU remains alive.

## Startup safety baseline

A safe startup should conceptually follow:

```text
reset
  ↓
K_OUT_A/B open
VPRE disabled/minimum
  ↓
initialize MCU and watchdog
  ↓
initialize local + isolated communications
  ↓
set DAC/PWM commands to safe values
  ↓
read hardware-fault and thermal state
  ↓
validate telemetry
  ↓
allow user-requested output enable
```

Any reset, watchdog event, unrecoverable communication error, invalid setpoint state, or critical hardware fault must return the affected output to a safe disabled state.

## Calibration storage

Calibration records should be versioned and protected by CRC. Internal STM32 Flash is sufficient for infrequent calibration commits; firmware must not rewrite calibration continuously during interactive adjustment.

A suitable record should include at least:

- format/version;
- channel;
- voltage/current range or mode identifier;
- gain/slope coefficient;
- offset coefficient;
- temperature reference where useful;
- CRC;
- validity marker/generation counter.

## Testing direction

Host-side tests should be used for logic that can be separated from the STM32 peripherals, especially:

- setpoint conversions;
- calibration math;
- operating envelope calculations;
- CV/CC/compliance UI-state logic;
- relay and series-mode state machines;
- I2C recovery state-machine decisions;
- fault classification;
- thermal hysteresis;
- configuration record validation.

Hardware-in-the-loop/bench tests remain mandatory for analog stability, SOA, ripple, relay behavior, isolated communications, and protection response.

## Current status

No production firmware baseline is committed yet. Hardware interfaces, GPIO allocation, display choice, build profiles, and exact driver selection should be frozen against the first schematic revision before implementation is treated as stable.
