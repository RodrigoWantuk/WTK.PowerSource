# Open items and decision log

This document separates unresolved engineering work from decisions that have already been made.

## Current open items

### Exact commercial PSU

OPEN.

Rev.1 uses nominal 24 V isolated supplies. Need to select real units and measure:

- voltage sag;
- current-limit/hiccup mode;
- thermal behavior;
- inrush interaction;
- usable continuous power.

### Exact maximum output voltage

Intentionally OPEN as a measured result.

There is no artificial 22 V or 24 V ceiling in the design. Maximum output is whatever the real 24 V system can regulate safely.

### XL4016 loose-IC sourcing

OPEN/high risk.

Integrated buck is preferred, but external-module fallback must be preserved.

### eFuse PN

OPEN.

MP5046 family is a technical candidate and DNP is explicitly allowed.

### eFuse current threshold

TUNE after maximum normal input current is measured.

Philosophy is frozen: threshold near the legitimate maximum, passive fuse with more margin.

### Passive fuse rating

TUNE after PSU/inrush characterization.

### Output relay PN

OPEN.

Must satisfy real DC current, open-contact voltage, mechanical life, coil-drive, and series-mode requirements.

### CV/CC op-amp and compensation values

OPEN/TUNE.

The analog architecture is frozen; compensation and exact amplifiers require schematic/bench analysis.

### Small active output sink

OPEN.

A small sink may improve negative transient/discharge behavior. Full complementary power-amplifier operation is archived.

### Output capacitance / switchable bulk

OPEN/TUNE.

Must be chosen with loop stability and waveform behavior.

### Thermal thresholds and fan curve

TUNE.

### Exact NTC count and placement

OPEN until mechanical/PCB layout.

### Reverse-power blocking topology

OPEN until battery/backfeed testing identifies the real current path and K_OUT timing.

### Display/front-panel hardware

OPEN.

Not required to close the power architecture.

### GPIO allocation

OPEN until schematic interfaces are finalized.

## Frozen decisions

- two isolated power channels;
- one STM32F103-class MCU;
- no second MCU for Channel B;
- initial nominal 24 V isolated PSU per channel;
- future source ceiling 36 V/channel;
- future desired output around 30 V/channel when powered appropriately;
- no artificial Rev.1 maximum-output-voltage number;
- no boost merely to recover top-end voltage in Rev.1;
- buck preregulator + linear pass architecture;
- one TIP36C per channel initially;
- analog CV and CC loops;
- hardware OVP/OCP/thermal authority independent of normal firmware control;
- fail-safe normally-open K_OUT per channel;
- current-source mode implemented through ISET + voltage compliance, not a software current PID;
- normal operation without PC/USB/debugger connection;
- low-side current-shunt architecture acceptable under that isolation assumption;
- target 5–10 mA current-readout resolution;
- ripple/noise acceptance target approximately 25 mVpp pending qualification;
- multiple NTC/thermal telemetry planned;
- 2-layer 1 oz PCB;
- wide copper and optional soldered power wires/jumpers are allowed;
- eFuse optional/DNP-capable;
- passive external fuse expected even when eFuse is populated;
- eFuse threshold near normal maximum current, passive fuse with larger margin;
- Channel-B communication failure is fail-safe;
- STM32F103 isolated-I2C timeout/recovery separate from global watchdog;
- external analog power-amplifier feature archived.

## Major decisions removed from the project

### Two MCUs

Rejected because of cost, duplicated firmware, synchronization, and BOM impact.

### INA180

Removed because practical Brazil availability is poor for this project.

The architecture moved to a calibrated shunt plus ADS1115 telemetry and conventional analog circuitry for fast CC/protection.

### Fixed 22 V maximum output

Removed.

The project accepts the real dropout produced by a 24 V source and will revisit source voltage/boost only if bench results justify it.

### Mandatory 2 oz copper

Rejected.

The board remains 1 oz with geometry and wire reinforcement used where needed.

### LTC3780 buck-boost as mandatory preregulator

Removed from current direction.

With an initial 24 V supply and willingness to accept natural top-end dropout, a simpler buck is preferred.

### External analog signal-amplifier mode

Archived, not deleted permanently.

It may be reconsidered after PCB layout if spare area and thermal margin exist.

## Documentation maintenance rule

When a decision moves from OPEN/TUNE to FROZEN, update the relevant technical document **and** this decision log in the same commit.

When a frozen architectural decision is intentionally changed, record the old and new state here rather than silently editing history.
