# Project scope and requirements

This document defines the current engineering scope of WTK.PowerSource. It is a design baseline, not a declaration that every target has already been qualified on hardware.

## Status vocabulary

The project uses the following terms consistently:

- **FROZEN** — architectural decision that should not change silently.
- **PROVISIONAL** — current engineering choice, expected to work but still replaceable without changing the project concept.
- **TUNE** — value or threshold that must be finalized on the bench.
- **OPEN** — decision intentionally not closed yet.
- **DNP** — footprint/function may exist in the design but the part is allowed to remain unpopulated.

## Product concept

WTK.PowerSource is a dual-channel programmable bench power supply with two isolated power domains and one central STM32F103-class controller.

Each channel uses a switching preregulator followed by a linear pass stage. The switching stage performs most of the voltage conversion; the linear stage performs the final CV/CC regulation and isolates the load from much of the switching behavior.

The design priorities are:

- low component cost;
- practical sourcing in Brazil in small quantities;
- repairability and manual assembly;
- analog protection that does not depend on firmware timing;
- independent, floating output channels;
- series/symmetric operation;
- a two-layer 1 oz PCB without making 2 oz copper mandatory.

## Electrical requirements

| Requirement | Current baseline | Status |
|---|---:|---|
| Number of output channels | 2 | FROZEN |
| Control MCUs | 1 × STM32F103C8T6-class | FROZEN |
| Initial input supply | 24 VDC isolated per channel | FROZEN for Rev.1 |
| Future input ceiling | 36 VDC maximum per channel | FROZEN design envelope |
| Future desired output | approximately 30 V/channel when powered appropriately | PROVISIONAL future target |
| Rev.1 maximum output voltage | maximum naturally available after real circuit drops | FROZEN philosophy |
| Artificial 22/24 V firmware ceiling | none | FROZEN |
| Maximum output current | up to 5 A/channel, subject to power/thermal envelope | PROVISIONAL target |
| Initial channel power class | approximately 60 W/channel | PROVISIONAL target |
| DC output ripple/noise acceptance | ≤25 mVpp target under defined test conditions | PROVISIONAL / bench gate |
| Preferred current resolution | approximately 5–10 mA usable resolution | PROVISIONAL / bench gate |
| PCB | 2 layers, 1 oz copper | FROZEN baseline |
| High-current reinforcement | wide copper + optional soldered power-wire jumpers | FROZEN allowed technique |

## No artificial maximum-output-voltage limit

The first hardware revision is not required to guarantee 22 V, 23 V, or 24 V at the output from a nominal 24 V source.

The actual maximum regulated voltage is intentionally allowed to be determined by:

- commercial PSU voltage and sag;
- preregulator dropout;
- TIP36C headroom;
- shunt drop;
- relay/contact resistance;
- copper and wire drop;
- load current;
- temperature.

The correct Rev.1 behavior is therefore:

```text
VOUT_MAX = maximum stable regulated output achievable from the real 24 V system
```

If the measured result later proves insufficient, the project may evaluate a higher-voltage commercial supply or a boost stage ahead of the preregulator. No boost is required in the current baseline merely to recover the last one or two volts.

## Future 36 V input / ~30 V output envelope

The design should avoid unnecessary component choices that make a later 30 V/channel revision difficult, but Rev.1 remains a 24 V-input project.

The future envelope means:

- **36 VDC is the maximum intended source voltage**, not a nominal 36 V requirement;
- approximately 30 V/channel is the future output goal;
- the PCB should use sensible voltage ratings and spacing for this range;
- the project does not need to be designed for a 60 V input bus.

The XL4016 itself sits near the upper end of its comfortable operating range at 36 V, so a later high-voltage revision may choose a different buck stage while retaining the rest of the architecture.

## CV and current-source modes

The analog hardware supports both voltage-oriented and current-oriented user workflows.

### Voltage-oriented mode

```text
VSET = requested output voltage
ISET = current limit
```

The CV loop normally controls until the current limit is reached, after which the analog CC loop takes control.

### Current-source mode

```text
ISET = requested current
VSET = compliance-voltage ceiling
```

The fast regulation is still performed by the analog CC loop. Firmware does **not** run a fast PID that repeatedly increases voltage until the requested current appears.

If the load requires more voltage than the configured compliance ceiling, the channel becomes voltage-limited and the requested current cannot be reached. The UI should make that state explicit.

## Series and symmetric operation

The channels remain electrically isolated so they can be used independently or connected in series by the system switching arrangement.

Conceptually:

```text
A+  ───────── positive extreme
A-  ─┐
     ├── COM when series link is engaged
B+  ─┘
B-  ───────── negative extreme
```

This permits:

- independent floating outputs;
- summed differential voltage;
- `+V / COM / -V` symmetric use.

The maximum series voltage is the sum of the actual maximum voltage available from both channels; the design does not impose an artificial software number.

## Normal-use connectivity

During normal operation the instrument is expected to have:

- AC mains feeding the two isolated commercial DC supplies;
- no PC connection;
- no USB connection;
- no ST-Link/SWD connection.

This is important because the current baseline uses low-side current sensing and relies on the output domains remaining intentionally floating.

Debug and service procedures must not silently defeat that assumption.

## Output relay

Each channel retains a physical fail-safe output relay `K_OUT`.

The relay is normally open when de-energized and may open because of:

- user OUTPUT OFF;
- startup/reset;
- watchdog or invalid firmware state;
- OVP;
- severe OCP;
- thermal fault;
- eFuse fault where applicable;
- backfeed/reverse-power detection;
- persistent Channel-B communication failure;
- series-mode reconfiguration;
- explicit safety sequencing.

Critical hardware faults must be able to remove relay drive without waiting for firmware.

## Archived feature: external analog power amplifier

The possibility of adding analog inputs and a complementary sink stage so the supply could behave as a signal/power amplifier was evaluated and is intentionally **archived**.

It is not part of the current schematic baseline. It may be reconsidered only after the normal power-supply PCB is laid out and spare area/thermal margin are known.
