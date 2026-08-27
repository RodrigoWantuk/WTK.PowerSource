# CV, CC, and output control

## Control philosophy

The fast output-regulation loops are analog.

Firmware generates setpoints and supervises the system, but a software scheduling delay, I2C timeout, UI stall, or watchdog interval must not be the element that prevents destructive output current.

The channel therefore has two simultaneous analog limits:

```text
CV loop: output voltage must not exceed VSET
CC loop: output current must not exceed ISET
```

Whichever loop demands less pass-transistor conduction takes control.

## Linear pass stage

The current baseline uses one **TIP36C** per channel as the series pass transistor.

The pass stage receives `VPRE` from the buck and produces the regulated output node.

The driver topology and compensation network remain PROVISIONAL/TUNE until the first hardware loop is characterized.

Important constraints:

- low dropout/headroom is valuable because the initial source is only 24 V;
- base-drive resistors must not create unnecessary volt-level drops at high base current;
- the TIP36C must be operated inside its DC/transient SOA at actual case temperature;
- thermal protection does not replace SOA-safe electrical foldback.

## CV loop

The CV amplifier compares a programmed voltage reference with a scaled measurement of the actual regulated output.

The loop should sense an internal regulated node in a way that remains stable when K_OUT is open. The design must avoid the classic failure mode where opening the output relay removes feedback, causing the internal regulator to saturate high and create an overshoot when the relay closes again.

Any future remote/Kelvin output sense must be designed explicitly; it must not be inferred by simply moving a feedback wire past K_OUT.

## CC loop

The CC amplifier compares the current-setpoint reference with the conditioned shunt measurement.

The analog CC loop is the primary normal current regulator from near zero up to the project current limit.

The firmware may use a `CC_ACTIVE` signal for:

- UI state;
- VPRE foldback/optimization;
- logging;
- thermal prediction;

but the current limit itself must remain functional without that firmware reaction.

## Current-source user mode

Current-source mode is a firmware/UI interpretation of the same CV/CC hardware.

The user configures:

```text
requested current = ISET
maximum allowed voltage = VSET / compliance
```

Example:

```text
ISET = 1.000 A
compliance = 12.0 V
```

With a 5 Ω load, the analog loop can settle near 5 V / 1 A.

With a 20 Ω load, 1 A would require 20 V, so the channel reaches 12 V compliance first and can deliver only approximately 0.6 A.

The firmware should present this as a compliance/voltage-limited state rather than hiding it.

## No firmware current PID

The following approach is explicitly rejected for normal regulation:

```text
ADC reads low current
→ firmware increases VSET
→ ADC reads again
→ firmware increases VSET
→ ...
```

That would create a slow discrete-time current loop with avoidable latency, overshoot, and dependence on MCU scheduling. The analog CC loop already solves the problem correctly.

## Setpoint generation

Current architecture direction:

- `VSET` — PWM plus reconstruction/filtering is acceptable and useful where waveform generation is desired;
- `ISET` — PWM/filter or equivalent low-cost setpoint generation is acceptable because it is normally slow;
- `VPRE` — DAC-oriented control is preferred because it is slow and benefits from deterministic absolute command values.

Exact DAC/PWM scaling remains to be finalized against the schematic.

## PWM resolution

A representative STM32F103 timer configuration discussed for setpoint generation is approximately 70.3125 kHz with around 10-bit duty resolution.

This gives a current-setpoint granularity naturally close to the desired 5 mA class if 0–5 A maps across the full code range.

Voltage resolution in a high-voltage range is coarser and may use calibration/dithering where justified, but UI display resolution must not be confused with guaranteed physical output resolution.

## Waveform generation

Internally generated low-frequency waveform operation remains compatible with the architecture, but it is not the same as the archived external analog-amplifier feature.

The preregulator must not track the waveform cycle-by-cycle. It should be placed above the required waveform peak plus headroom, leaving the linear stage to reproduce the fast variation.

This creates a separate thermal envelope because instantaneous `VPRE - VOUT` can be large during part of the waveform cycle.

Waveform current limits and maximum amplitude/frequency therefore require their own bench qualification.

Sine/triangle behavior is expected to be easier than a fast square wave because the reconstruction and loop bandwidth intentionally limit high harmonics.

## Output relay K_OUT

Each channel has a normally-open relay used as a physical disconnect.

K_OUT is retained because it provides:

- real OUTPUT OFF;
- safe boot while setpoints initialize;
- hardware fault isolation;
- safer series-mode switching;
- a response path for backfeed/reverse-power detection;
- a final physical disconnect if the linear control stage behaves unexpectedly.

K_OUT must default open when power/logic is absent.

The electrical enable should conceptually require both:

```text
MCU_ENABLE = true
AND
HW_OK = true
```

A hardware fault must be able to remove coil drive even if the MCU remains high.

## Output-capacitance strategy

Large output capacitance improves DC transient behavior but can make low-frequency waveform operation and controlled output discharge more difficult.

A possible architecture is a small always-connected `CFAST` plus switchable `CBULK`. This remains PROVISIONAL/TUNE and must not be frozen until the CV/CC loop is characterized.

## Output-current sink

A small active sink was previously considered to improve negative-going waveform response and capacitor discharge. A full complementary high-current amplifier stage is archived.

Whether a small sink remains worthwhile for the ordinary supply is OPEN and should be decided from transient tests rather than assumed as a required feature.
