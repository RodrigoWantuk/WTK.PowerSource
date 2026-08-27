# Input, preregulator, and power stage

## Input supplies

Rev.1 assumes two independent commercial isolated DC supplies, initially nominally **24 VDC**.

The exact PSU model is OPEN. Selection should consider:

- real continuous power;
- output-current limiting behavior;
- thermal derating;
- output sag near rated load;
- mechanical dimensions;
- price and local availability;
- whether voltage adjustment is available, without requiring adjustment above 24 V for Rev.1.

The project deliberately does not add a boost converter simply to guarantee a specific top-end output voltage.

## Passive fuse

A conventional passive fuse is expected **outside the PCB** on each channel input.

Its role is catastrophic/fault-energy protection and wiring protection, not normal current regulation.

Important coordination rule:

- the eFuse, if populated, should be configured close to the legitimate maximum input-current envelope;
- the passive fuse should have more margin and generally be time-delay/slow-blow;
- the commercial PSU itself may current-limit before the passive fuse ever receives enough I²t to open.

The final fuse rating is TUNE and must be chosen only after the PSU and inrush behavior are measured.

## eFuse

An eFuse function is **PROVISIONAL but strongly desired** per channel.

The eFuse is placed at the DC input, ahead of the buck stage. It is not placed in the variable output because a typical 4.5 V+ eFuse would not cover near-zero output voltage and input current is not identical to output current after the buck.

Desired behavior:

- electronic current limit close to normal maximum input current;
- enable/shutdown from firmware;
- autonomous hardware overcurrent/short response;
- status/fault output to the MCU where practical;
- reverse-current blocking where the chosen device supports it;
- voltage rating compatible with the 36 V future input ceiling.

### MP5046 family

MP5046 is a technical candidate because its voltage/current/reverse-blocking concept aligns well with the project. It must remain **DNP-capable** and is not yet a production-frozen PN.

The PCB must be able to operate correctly with the eFuse unpopulated by using a deliberate high-current bypass link.

The eFuse is an extra protection layer; the circuit must not depend on it for CV, CC, OVP, output OCP, thermal shutdown, or K_OUT operation.

## Buck preregulator

The current preferred power converter is an **XL4016-based asynchronous buck integrated onto the WTK.PowerSource PCB**.

Reasons:

- removes dependence on an unknown clone-module layout/revision;
- allows controlled thermal placement;
- permits a shared channel heatsink strategy;
- exposes the feedback network for a known DAC/control interface;
- improves test-point access;
- permits intentional high-current routing and wire reinforcement;
- allows selection of inductor, Schottky, and capacitors for the real operating envelope.

## XL4016 electrical constraints

The XL4016 is suitable for the current 24 V design and is still plausible near the future 36 V ceiling, but 36 V should be treated as the **maximum intended system input**, not a comfortable nominal XL4016 operating point.

The project should therefore avoid making the rest of the board dependent on XL4016 forever. A later 30 V/channel revision may replace the buck while retaining the same VPRE/linear/output interfaces.

## Buck Schottky

Because the XL4016 is asynchronous, the freewheel Schottky can dissipate significant power at low output voltage and high current, where the off-time is large.

A 45 V Schottky is acceptable for the current 24 V-only case but gives insufficient transient margin if VIN is later allowed to reach 36 V.

The preferred design direction is therefore **60 V Schottky class** for the integrated PCB.

A part such as STPS20L60CT-class dual common-cathode Schottky is representative; the final production PN remains subject to sourcing.

## Inductor

The current electrical starting point is approximately **47 µH** with current ratings comfortably above the 5 A output requirement and with low DCR.

The exact inductor PN and footprint are OPEN because sourcing and mechanical size are major constraints.

Bench qualification must include the low-output/high-current region, e.g. 1.5–3 V at high current, because the asynchronous diode duty and converter thermals may be worse there than at a conventional 12 V output point.

## VPRE control

The preregulator output `VPRE` is slow-controlled from firmware through a DAC-oriented architecture, currently MCP4725-class.

The DAC must not be allowed to create an unsafe output solely through software configuration. The feedback network must have physical bounds such that:

- power-up defaults are safe;
- loss of DAC command does not force the buck to maximum;
- hardware can disable the buck independent of normal firmware control;
- a software bug cannot exceed the hardware VPRE ceiling.

`VPRE_HARD_MAX` or its equivalent is therefore a hardware requirement, even if exact implementation values remain OPEN/TUNE.

## VPRE behavior during CC and short circuit

A critical design problem is the interval immediately after an output short.

If VPRE remains near the previous high-voltage command while the analog CC loop limits output current, the TIP36C can momentarily see very high `VCE × IC` dissipation.

The design therefore requires:

1. fast analog current limiting independent of the MCU;
2. hardware authority to force VPRE down and/or disable the preregulator during severe CC/fault conditions;
3. firmware VPRE optimization after the immediate protection has acted;
4. bench validation of the complete transient against **one TIP36C** SOA.

The exact foldback response time is not to be guessed. It is a validation result.

## Power capacitors and voltage rating

The future input ceiling is 36 V. Power-path electrolytics should therefore normally use at least **50 V** ratings where they can see the input or VPRE domain. 63 V parts are acceptable when cost/size are similar but are not a blanket requirement.

## Reverse-power concern

The buck path must be tested for energy returning from the output through the linear stage toward VPRE and the preregulator.

During backfeed testing, instrumentation must observe at least:

- `VOUT`;
- `VPRE`;
- buck input/VIN;
- input-branch current where practical.

The goal is to determine whether the output relay plus clamp network is sufficient or whether an additional reverse-blocking element is needed ahead of the buck.
