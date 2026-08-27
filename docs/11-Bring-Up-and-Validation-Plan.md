# Bring-up and validation plan

This document defines the minimum qualification program before the first hardware revision can be treated as production-ready.

The exact test order may change with prototype availability, but safety-critical gates must not be skipped.

## Stage 0 — unpowered inspection

- visual inspection;
- verify component orientation;
- verify XL4016, Schottky, TIP36C, relay, ADC/DAC pinouts;
- check GND_A/GND_B resistance/isolation;
- verify no accidental PE/chassis bonding;
- measure input-to-ground resistance for gross shorts;
- inspect shunt Kelvin routing;
- verify DNP/bypass population states.

## Gate 1 — input protection

With buck/output disabled:

- validate external fuse arrangement;
- power through current-limited bench supply;
- verify TVS does not conduct at normal input;
- verify eFuse bypass operation;
- if eFuse is populated, characterize enable, current limit, fault/status, and inrush behavior;
- confirm loss of MCU command cannot leave the optional eFuse in an unintended state where relevant.

## Gate 2 — buck preregulator

Characterize the integrated XL4016 stage independently of the TIP36 output section where practical.

Measure:

- VPRE command range;
- power-up default;
- hardware disable;
- maximum physically allowed command;
- efficiency;
- SW ringing;
- Schottky temperature;
- XL4016 temperature;
- inductor temperature/saturation behavior;
- output ripple;
- control-loop stability.

Mandatory load points include conventional mid-voltage operation and **low-voltage/high-current** points such as approximately 1.5–3 V at high current.

Verify the external-buck fallback interface before depending on the integrated stage for every prototype.

## Gate 3 — one-TIP36C SOA and foldback

This gate explicitly refers to **one TIP36C per channel**.

Instrument:

- VPRE;
- VOUT;
- TIP36C VCE;
- output current;
- K_OUT timing;
- relevant hardware-kill timing.

Apply controlled short/load steps and derive the real `VCE(t) × IC(t)` trajectory from fault onset until VPRE falls/output disconnects.

Compare the trajectory against transistor SOA at realistic case temperature.

Do not reuse assumptions from earlier two-transistor versions.

## Gate 4 — CV loop

- DC setpoint linearity;
- low/high output operation;
- line regulation versus VIN;
- load regulation;
- small and large load steps;
- no oscillation across representative capacitive/resistive loads;
- safe behavior with K_OUT open and when it closes.

## Gate 5 — CC loop

- current-setpoint linearity;
- transition CV→CC and CC→CV;
- short-circuit current;
- recovery from short;
- current-source mode with compliance limit;
- no software PID required for stable regulation.

## Gate 6 — current measurement

Validate the 50 mΩ shunt implementation:

- zero offset;
- 5 mA and 10 mA increment repeatability;
- 100 mA, 1 A, 2.5 A, 5 A reference points;
- shunt self-heating;
- calibration before/after temperature rise;
- Kelvin routing immunity to power-wire/trace drop.

The acceptance criterion is usable 5–10 mA-class resolution, not theoretical ADC LSB.

## Gate 7 — voltage and auxiliary telemetry

- VOUT calibration;
- VIN/VPRE calibration;
- CD4051 settling;
- NTC conversion and plausibility;
- telemetry scheduling at required rates;
- behavior when a sensor is open/shorted.

## Gate 8 — isolation and I2C_B recovery

- verify no DC continuity across the intended isolation barrier;
- exercise normal isolated I2C operation;
- induce bus timeout/stuck conditions where practical;
- verify GPIO/open-drain recovery sequence;
- verify STM32F1 BUSY recovery handling;
- verify bounded retries;
- verify persistent failure creates `COMM_B_FAULT`;
- verify K_OUT_B opens and VPRE_B becomes safe after persistent failure.

## Gate 9 — A/B setpoint fidelity

Where waveform/PWM-derived setpoint functionality is used, compare Channel A and Channel B after their reconstruction paths.

Measure:

- DC gain/offset;
- duty-dependent error;
- phase delay;
- step response;
- sine THD at representative frequencies, including near the intended upper low-frequency waveform range.

Repeat at the actual outputs to distinguish isolator/filter error from power-stage error.

## Gate 10 — ripple/noise

Acceptance target:

```text
≤25 mVpp in normal DC CV operation
```

Define test conditions in the report:

- bandwidth (e.g. 20 MHz);
- probe technique / spring ground;
- load current;
- output voltage;
- input supply;
- temperature;
- fan state.

A lower target such as <10 mVpp is desirable but not required until proven.

## Gate 11 — reverse power / battery backfeed

Connect a controlled external source/battery to the output under defined states.

Record:

- VOUT;
- VPRE;
- buck VIN;
- input branch current;
- K_OUT opening time;
- clamp/blocking current;
- semiconductor temperatures.

Test both negative forcing and positive backfeed scenarios.

Determine whether the final design needs additional blocking beyond K_OUT and local clamps.

## Gate 12 — thermal system

Characterize:

- TIP36C case/heatsink temperature;
- XL4016;
- Schottky;
- inductor;
- commercial PSU region;
- fan airflow/acoustics;
- NTC placement accuracy;
- fan hysteresis/curve;
- emergency hardware trip.

Test with the enclosure or a representative airflow restriction.

## Gate 13 — series/symmetric mode

- outputs individually disabled before transition;
- discharge/precharge behavior;
- relay contact sequencing;
- midpoint behavior;
- summed output voltage;
- one-channel fault while series link is active;
- contact DC voltage/current stress;
- safe return to independent mode.

## Gate 14 — power copper and wire reinforcement

At maximum intended current:

- measure temperature rise of PCB copper;
- measure voltage drop across critical paths;
- validate optional power-wire jumpers;
- inspect solder-joint heating;
- confirm jumper installation does not disturb Kelvin sensing.

## Release criterion

A PCB revision is not production-ready until failures in these gates are either:

- corrected and re-tested; or
- explicitly accepted as documented limitations with no safety ambiguity.
