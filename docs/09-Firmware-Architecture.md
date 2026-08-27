# Firmware architecture

## Core principle

Firmware controls the instrument but is not the sole fast safety layer.

Analog CV/CC and critical hardware OVP/OCP/thermal paths must remain effective even if the MCU stalls.

The MCU is responsible for:

- UI and user commands;
- setpoint generation;
- slow VPRE management;
- measurement/telemetry;
- thermal/fan control;
- relay sequencing;
- fault classification and reporting;
- calibration;
- communication recovery;
- diagnostics.

## MCU

Current baseline: **one STM32F103C8T6-class device / Blue Pill class module**.

A second MCU is explicitly rejected in the current architecture.

## Suggested source layout

```text
Firmware/src/
├── app/
├── bsp/
├── control/
├── drivers/
├── hardware/
├── measurement/
├── protection/
└── ui/
```

### app

Top-level application state machine:

- boot;
- self-check;
- output disabled;
- normal CV/CC operation;
- current-source mode;
- series/symmetric transitions;
- fault state;
- recovery/re-arm;
- shutdown.

### bsp

Board-specific definitions:

- GPIO assignment;
- timer/PWM configuration;
- I2C;
- ADC where locally used;
- watchdog;
- SWD/debug;
- board revision.

### control

- VSET/ISET conversion;
- VPRE target calculation;
- output-enable sequencing;
- series relay state machine;
- compliance-voltage handling;
- fan target where kept with control logic.

### drivers

Expected drivers include:

- ADS1115;
- MCP4725-class DAC;
- isolated-I2C handling;
- CD4051 mux abstraction;
- display/front-panel devices once selected;
- optional eFuse status/control.

### hardware

Logical channel interface:

- hardware-fault inputs;
- K_OUT command;
- buck enable/kill;
- CC/CV-active inputs;
- fan output;
- series relay;
- Channel-B isolation abstraction.

### measurement

- raw ADC acquisition;
- calibrated voltage/current conversion;
- NTC conversion;
- telemetry scheduling;
- plausibility checks;
- filter/averaging for UI.

### protection

- fault classification;
- software interlocks;
- persistent communication failure handling;
- sensor-fault handling;
- thermal-warning state;
- eFuse status;
- safe reset behavior.

### ui

The UI should expose actual regulation state:

- CV;
- CC;
- compliance-limited current-source mode;
- OUTPUT OFF;
- hardware fault;
- communication fault;
- thermal warning;
- series/symmetric configuration.

## Current-source operation

Firmware does not adjust voltage iteratively to chase current.

Instead:

```text
ISET = requested current
VSET = compliance ceiling
```

The analog loops regulate at the requested current if physically possible.

Firmware reads `CC_ACTIVE/CV_ACTIVE` or equivalent state and presents the result.

## VPRE management

Firmware should slowly optimize preregulator voltage to reduce TIP36C power.

In normal DC operation:

```text
VPRE_TARGET ≈ required output + headroom
```

Headroom is a calibrated/TUNE value.

During CC or large transients, hardware must already keep the transistor safe. Firmware reduction of VPRE is a secondary/thermal optimization, not the only short-circuit protection.

## Telemetry scheduling

Recommended minimum behavior:

- voltage/current: ~50 Hz target;
- VIN/VPRE: ≥10 Hz;
- NTCs: 5–10 Hz;
- every supervised signal: avoid long blind intervals.

The scheduler must account for ADS1115 conversion time and CD4051 settling.

## I2C_B recovery

The isolated Channel-B I2C path has its own timeout/recovery state machine.

A recommended sequence is:

```text
transaction starts
    ↓
timeout?
    ├─ no -> normal completion
    └─ yes
         ↓
disable/release peripheral
         ↓
GPIO open-drain bus recovery / STOP
         ↓
STM32F1 BUSY recovery sequence if required
         ↓
reinitialize I2C
         ↓
retry bounded number of times
         ↓
still failing -> COMM_B_FAULT
```

A global watchdog does not replace this mechanism.

## COMM_B_FAULT

Persistent Channel-B bus failure causes:

- K_OUT_B open;
- VPRE_B forced disabled/safe;
- clear UI indication;
- no automatic continuation into a blind energized state.

Local analog Channel-B protections remain operational regardless.

## Startup sequence

Conceptual safe boot:

```text
reset
 ↓
K_OUT_A/B open
buck/VPRE disabled or safe minimum
 ↓
initialize clocks/watchdog/GPIO
 ↓
initialize local and isolated buses
 ↓
load and validate calibration/configuration
 ↓
program safe DAC/PWM setpoints
 ↓
read hardware faults and telemetry
 ↓
allow user-controlled output enable
```

No power-on state should depend on an uninitialized DAC/PWM value accidentally being benign.

## Calibration storage

Use versioned CRC-protected internal-Flash records.

Calibration is committed infrequently. The design does not require an external EEPROM merely to avoid STM32 Flash wear.

## Host-side testing

Pure logic should be testable without hardware, including:

- calibration math;
- setpoint conversions;
- power-envelope calculations;
- current-source compliance logic;
- relay state machines;
- I2C recovery-state decisions;
- thermal hysteresis;
- fault classification;
- configuration CRC/version handling.

Analog stability, SOA, isolation hardware, and ripple remain bench/HIL validation items.
