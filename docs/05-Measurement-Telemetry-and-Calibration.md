# Measurement, telemetry, and calibration

## Measurement goals

The telemetry system supports:

- output-current display and protection supervision;
- output-voltage display and calibration;
- VPRE monitoring;
- raw input-voltage monitoring;
- multiple temperature sensors;
- diagnostic and bring-up measurements.

Fast CV/CC stability must not depend on telemetry update rate.

## Current shunt

The current baseline is a low-side shunt of approximately **50 mΩ** per channel.

A practical first implementation is:

```text
2 × 0.10 Ω / 5 W resistors in parallel
→ approximately 0.05 Ω / 10 W nominal combined rating
```

This intentionally uses a common resistor value and gives large thermal margin.

At 5 A:

```text
VSHUNT = 5 A × 0.05 Ω = 0.25 V
PSHUNT = 25 × 0.05 = 1.25 W
```

The shunt must have true Kelvin sense routing. High-current copper/wire and measurement traces must leave the resistor terminals independently so that solder, via, jumper, and trace drop are not interpreted as load current.

## Low-side sensing implications

Low-side sensing is acceptable because normal operation has no PC/USB/debugger connection and the channel domains are intended to float.

However, the design must recognize that `OUT_N` is lifted above the internal `GND` by the shunt drop.

An unintended external path from internal ground to earth/load negative can bypass some or all of the shunt. Service/debug procedures must therefore preserve isolation or keep outputs disabled when using non-isolated instruments.

## ADS1115

The preferred telemetry ADC is ADS1115-class, one device per isolated channel domain.

Reasons:

- 16-bit delta-sigma conversion;
- differential input capability;
- programmable gain;
- simple I2C interface;
- readily available low-cost modules for prototyping;
- enough resolution for the desired 5–10 mA current-readout target.

With a 50 mΩ shunt and ±512 mV input range, the nominal code size corresponds to roughly 0.3 mA per LSB before real-world noise, offset, shunt tolerance, and calibration. The desired 5–10 mA usable resolution therefore has substantial digital margin.

The project must qualify **effective** resolution, not advertise the theoretical ADC LSB as measurement accuracy.

## ADC channel priority

A representative per-channel allocation is:

```text
AIN0/AIN1  differential shunt measurement
AIN2       output-voltage measurement
AIN3       multiplexed slower telemetry
```

The final allocation is PROVISIONAL and may change when the exact analog front-end is drawn.

## Thermal and rail multiplexing

A CD4051-class analog multiplexer can feed slower measurements to the ADS1115.

Possible mux sources:

- NTC on commercial PSU / incoming power section;
- NTC on XL4016/buck hotspot;
- NTC near TIP36C;
- NTC on shared channel heatsink;
- spare temperature input;
- raw input voltage;
- VPRE;
- spare diagnostic channel.

Not every possible sensor must be populated. The mux gives the PCB inexpensive flexibility.

## NTC baseline

A conventional **10 kΩ B3950-class NTC** is a reasonable starting point because it is common, cheap, and easy to condition with ordinary resistor dividers.

Exact part number, divider resistance, self-heating, placement, and calibration are TUNE/OPEN until the mechanical design is known.

## Telemetry update rates

The firmware should not leave UI expectations undefined.

Current requirement:

- every supervised quantity: **≥10 Hz target** where practical;
- output voltage/current: **~50 Hz target** for responsive UI and diagnostics;
- NTCs: **5–10 Hz is sufficient**;
- VPRE/VIN: **≥10 Hz target**.

The exact schedule may be optimized, but these numbers are comfortably below ADS1115 maximum conversion rate and leave time for mux settling and isolated-I2C overhead.

## Mux settling

When switching a CD4051 channel, firmware must allow the analog path and ADC input network to settle before treating the next conversion as valid.

The settling delay and whether one conversion must be discarded are TUNE items that must be measured with the final resistor/capacitor network.

## Calibration

Calibration should be performed per channel and, where relevant, per range/mode.

Potential coefficients include:

- current zero offset;
- current gain/slope;
- voltage zero offset;
- voltage gain/slope;
- VPRE measurement gain/offset;
- temperature conversion coefficients where needed.

Calibration compensates static slope/offset error; it is not a substitute for validating dynamic waveform distortion, isolation delay asymmetry, or loop stability.

## Calibration storage

Internal STM32F103 Flash is sufficient for infrequent calibration commits.

Firmware should store versioned records with CRC rather than continuously emulate EEPROM during interactive adjustments.

A record should contain at least:

```text
format version
hardware revision
channel
measurement/range identifier
slope/gain
zero/offset
temperature reference if useful
generation counter
CRC
validity marker
```

The UI/calibration procedure may perform many temporary calculations in RAM, but Flash should be updated only when the calibration is explicitly committed.
