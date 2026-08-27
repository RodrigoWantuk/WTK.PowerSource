# Electrical architecture

## System partitioning

WTK.PowerSource consists of two isolated power domains and one control MCU.

```text
                         STM32F103
                         domain A
                            │
                  control / supervision
                            │
                     isolation barrier
                            │
          ┌─────────────────┴─────────────────┐
          │                                   │
      CHANNEL A                           CHANNEL B
      GND_A domain                       GND_B domain
          │                                   │
   isolated PSU A                        isolated PSU B
          │                                   │
   input protection                      input protection
          │                                   │
     buck preregulator                    buck preregulator
          │                                   │
        VPRE_A                              VPRE_B
          │                                   │
       TIP36C                               TIP36C
          │                                   │
      current shunt                     current shunt
          │                                   │
       K_OUT_A                              K_OUT_B
          │                                   │
       output A                             output B
```

## Ground and output-node definitions

The schematic and PCB must preserve explicit names for electrically distinct nodes.

### Channel A

- `GND_A` — internal power/control reference for Channel A and the MCU domain.
- `OUT_A_P` — positive output terminal.
- `OUT_A_N` — negative output terminal on the load side of the low-side shunt.

### Channel B

- `GND_B` — internal power/control reference for Channel B.
- `OUT_B_P` — positive output terminal.
- `OUT_B_N` — negative output terminal on the load side of the low-side shunt.

### System/chassis

- `PE` — protective earth, if exposed/used.
- `CHASSIS` — mechanical chassis node if separately defined.

The following are **not aliases**:

```text
GND_A != OUT_A_N
GND_B != OUT_B_N
GND_A != GND_B
PE    != GND_A
PE    != GND_B
```

Any deliberate connection among them must be represented by an actual component, relay contact, link, connector, or external wiring instruction.

## Per-channel power path

The intended high-level path for each channel is:

```text
isolated commercial PSU
        │
external passive fuse
        │
input connector
        │
TVS / polarity and transient protection
        │
optional eFuse or populated bypass
        │
VIN_PROT
        │
XL4016-class buck preregulator
        │
VPRE
        │
TIP36C linear pass device
        │
output-current shunt
        │
K_OUT relay
        │
output terminal
```

The exact ordering of the shunt and K_OUT must remain consistent with the final sense/reference architecture, but the current baseline is low-side sensing with Kelvin connections.

## Preregulator + linear-stage philosophy

The preregulator should provide only enough voltage above the actual output requirement for the linear stage to retain regulation authority.

Conceptually:

```text
VPRE_TARGET ≈ VOUT + required linear-stage headroom
```

This minimizes TIP36C dissipation in normal DC operation.

The preregulator is deliberately slow relative to the analog CV/CC loop. It does not attempt to follow fast waveform content.

## Single TIP36C baseline

The current prototype baseline uses **one TIP36C per channel**.

The earlier two-transistor concept was removed to reduce cost, driver current, thermal-sharing complexity, and PCB area.

This makes short-circuit SOA validation more important: the complete `VCE(t) × IC(t)` trajectory from the instant of a short until VPRE falls or the output disconnects must be checked against the SOA of one device at realistic case temperature.

A second pass transistor is not part of the current baseline and should not be added simply as unmeasured safety margin.

## Series connection

Series/symmetric operation must be implemented by real switching/wiring, never by merging domain net labels in the schematic.

The intended relationship is:

```text
OUT_A_N -- K_SERIES / series network -- OUT_B_P
```

When the series link is open, the channels remain independent.

When it is closed, the midpoint becomes the series COM node.

The series relay/contact network must be rated for the real DC voltage it can interrupt and must be sequenced only with the outputs in a safe state.

## Relay sequencing

A typical safe series-mode transition is:

```text
open K_OUT_A and K_OUT_B
        ↓
reduce/disable outputs and discharge internal/output capacitance
        ↓
verify safe low-voltage state using available telemetry/interlocks
        ↓
change K_SERIES / precharge state
        ↓
wait for contact bounce / settling
        ↓
re-enable the requested channel outputs
```

Exact thresholds and delays are TUNE values, but the sequencing principle is FROZEN.

## Shared heatsink concept

A single heatsink assembly per channel is preferred if mechanical layout permits.

Likely hot parts include:

- XL4016 package;
- asynchronous buck Schottky;
- TIP36C.

The heatsink may be shared mechanically, but semiconductor tabs that belong to different electrical nodes must be insulated from the aluminum and from one another.

The XL4016 tab is associated with the switching node; it must never be assumed to be ground.

## External buck fallback

The integrated XL4016 power stage is the preferred architecture because it gives the project control over layout, thermals, feedback, instrumentation, and component choice.

However, XL4016E1 sourcing as a loose IC is a known risk. The PCB should preserve a low-cost fallback path that allows the integrated buck to be bypassed/omitted and an external buck module to provide `VPRE` without redesigning the entire linear/output section.

This fallback is an intentional risk-management feature, not the preferred production topology.
