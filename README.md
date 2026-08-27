# WTK.PowerSource

WTK.PowerSource is a dual-channel programmable bench power supply focused on **low-cost, repairable hardware**, analog safety loops, digital supervision, and practical component sourcing.

The project combines two isolated DC power domains with a single **STM32F103C8T6** controller. Each output channel uses a switching preregulator followed by a linear pass stage so that the switching converter carries most of the power conversion while the linear stage provides the final CV/CC regulation, transient control, and output quality.

> **Current status — August 2026:** architecture and protection strategy are being consolidated before the first formal schematic and PCB revision. Values marked as provisional in the technical specification still require bench characterization. The repository must not present unqualified targets as guaranteed product specifications.

## Project goals

- Two electrically isolated and independently controlled output channels.
- Initial operation from **24 VDC isolated supplies per channel**.
- No artificial top-end voltage limit in Rev.1: the maximum regulated output is whatever remains after the real headroom and losses of the preregulator, linear stage, shunt, relay, wiring, and source sag.
- Future electrical envelope planned for **up to 36 VDC input per channel** and approximately **30 V output per channel**, without requiring that capability in the first build.
- Up to **5 A output current per channel**, subject to the channel power and thermal envelope.
- Approximately **60 W per channel** as the initial power-class target.
- Series operation for higher differential voltage and symmetric `+V / COM / -V` operation.
- Analog CV and CC loops that remain functional independently of firmware timing.
- Hardware OVP/OCP/thermal shutdown paths and fail-safe output relays.
- Current-source operation using the same CV/CC hardware: the user sets target current and a voltage-compliance limit while the analog CC loop performs the fast regulation.
- Output ripple/noise acceptance target of approximately **25 mVpp or better** in normal DC CV operation, subject to bench qualification.
- Current measurement resolution target in the **5–10 mA** range.
- Two-layer, **1 oz copper** PCB as the fabrication baseline, using wide pours/traces and explicit power-wire jumpers where routing 5 A exclusively through PCB copper is impractical.

## High-level architecture

```text
                one STM32F103C8T6
                       │
              control / UI / safety
                       │
             isolation to Channel B
                       │
       ┌───────────────┴────────────────┐
       │                                │
   CHANNEL A                        CHANNEL B
   isolated PSU                     isolated PSU
       │                                │
 external fuse                     external fuse
       │                                │
 optional eFuse                    optional eFuse
       │                                │
 switching preregulator            switching preregulator
       │                                │
      VPRE_A                           VPRE_B
       │                                │
    TIP36C                          TIP36C
 linear pass stage                linear pass stage
       │                                │
  current shunt                    current shunt
       │                                │
  K_OUT_A relay                    K_OUT_B relay
       │                                │
     OUT A                            OUT B
```

The two power domains remain isolated so the outputs can be used independently or reconfigured in series. The normal product configuration has **no USB, PC, or debugger connection during operation**; only the isolated AC/DC supplies power the instrument.

## Current hardware baseline

| Block | Current direction |
|---|---|
| Main MCU | 1 × STM32F103C8T6 / Blue Pill class |
| Input supplies | 2 × isolated 24 VDC supplies initially |
| Future input ceiling | up to 36 VDC per channel |
| Preregulator | XL4016-based buck integrated on the project PCB; external-module fallback should remain possible |
| Linear pass transistor | 1 × TIP36C per channel initially |
| Output disconnect | fail-safe relay `K_OUT` per channel |
| Current shunt | ~50 mΩ low-side, initial implementation using 2 × 0.10 Ω / 5 W in parallel |
| Precision telemetry ADC | ADS1115 per isolated domain |
| Thermal multiplexing | CD4051-class analog mux for multiple NTC channels |
| VPRE command | DAC-based, MCP4725-class architecture currently preferred |
| Channel-B digital isolation | fast optocouplers for critical PWM paths + isolated I²C for ADC/DAC telemetry/control |
| eFuse | optional/DNP-capable; MP5046 family is a technical candidate but not a production-frozen PN |
| Passive fuse | conventional external fuse per channel, always expected in the finished assembly |
| PCB | 2 layers, 1 oz copper, wide power pours and optional soldered power-wire reinforcement |
| Power semiconductors | mounted with thermal strategy suitable for one heatsink assembly per channel where practical |

## Protection philosophy

WTK.PowerSource intentionally uses multiple independent protection layers.

```text
isolated PSU internal protection
        ↓
external conventional fuse
        ↓
optional eFuse near normal input-current limit
        ↓
preregulator fixed hardware limits
        ↓
analog CC loop
        ↓
analog OCP / OVP / thermal protection
        ↓
K_OUT physical output disconnect
```

The **eFuse is intended to sit close to the maximum legitimate input-current envelope**. The passive fuse deliberately has more margin and exists mainly for catastrophic faults, wiring protection, eFuse-DNP builds, or failures where the active protection no longer controls the current. Because the commercial PSU itself will generally current-limit, the passive fuse is not treated as the normal operating limiter.

Critical shutdown functions must not depend on the MCU noticing a fault first. Firmware may supervise, log, command, and re-arm protections, but hardware protection paths retain authority to remove preregulator enable and/or open the output relay.

## CV and current-source operation

The same analog control hardware supports two user-facing operating styles.

**Voltage-oriented operation:**

```text
VSET = desired output voltage
ISET = current limit
```

**Current-source operation:**

```text
ISET = desired output current
VSET = compliance-voltage ceiling
```

No firmware PID is required for the fast current loop. The firmware configures setpoints and slowly optimizes `VPRE`; the analog CV/CC loops determine which limit is active.

## Measurement and thermal supervision

The measurement architecture is designed around calibrated low-side current shunts and ADS1115-class 16-bit ADCs. Current and voltage are priority telemetry signals; slower channels are multiplexed for thermal and rail supervision.

Planned temperature points include, as useful for the final mechanical design:

- commercial PSU / incoming power section;
- preregulator / buck hot spot;
- TIP36C pass transistor;
- shared channel heatsink;
- additional spare thermal point.

The exact NTC count and placement remain subject to PCB and heatsink layout.

## Isolation

Only **one MCU** is used. Channel B remains a separate electrical domain.

- Fast setpoint/PWM signals cross the barrier through fast optocouplers such as 6N137-class devices.
- Telemetry and slow digital control can use an isolated I²C path.
- Channel-B hardware protections remain local to Channel B.
- A persistent Channel-B communication failure is a fail-safe condition: firmware should disable the affected output rather than continue operating without valid telemetry/control.
- STM32F103 I²C recovery must include transaction timeout and bus-recovery handling independent of the global watchdog.

## PCB philosophy

The board is intentionally designed around common fabrication constraints rather than requiring 2 oz copper.

- 2-layer FR-4.
- 1 oz copper baseline.
- Wide traces and copper pours for high-current paths.
- Top/bottom copper sharing and via stitching where useful.
- Dedicated PTH points for short power-wire jumpers when this is cleaner than very large PCB traces.
- Kelvin routing for shunts and other low-level sense points must remain separate from power-current paths and jumper voltage drops.
- Manual assembly and component replacement are design considerations.

## Repository layout

```text
WTK.PowerSource/
├── README.md
├── LICENSE.md
├── PCB/
│   ├── README.md
│   ├── source/
│   ├── fabrication/
│   ├── renders/
│   └── revisions/
├── Firmware/
│   ├── README.md
│   ├── config/
│   ├── src/
│   │   ├── app/
│   │   ├── bsp/
│   │   ├── control/
│   │   ├── drivers/
│   │   ├── hardware/
│   │   ├── measurement/
│   │   ├── protection/
│   │   └── ui/
│   ├── tests/
│   ├── tools/
│   └── third_party/
└── docs/
    ├── README.md
    └── WTK_PowerSource_Especificacao_Tecnica_Rev_B4.pdf
```

- [`PCB/README.md`](PCB/README.md) describes schematic/PCB source and manufacturing-artifact conventions.
- [`Firmware/README.md`](Firmware/README.md) describes the planned firmware responsibilities and safety boundaries.
- [`docs/README.md`](docs/README.md) indexes the engineering documentation.

## Design status and open engineering gates

The first production PCB is **not yet released**. Before hardware is considered production-ready, the project still requires bench validation of at least:

- actual XL4016 efficiency, ripple, low-output/high-current thermal behavior, and external-feedback control;
- sourcing fallback for the XL4016 power stage;
- SOA trajectory of the single TIP36C during short-circuit transients and VPRE foldback;
- CV/CC stability and compensation;
- output ripple/noise under representative loads;
- current-measurement accuracy and effective 5–10 mA resolution;
- reverse-energy/backfeed behavior into VPRE and the buck stage;
- output-relay sequencing and series-mode interlocks;
- thermal limits, NTC placement, fan strategy, and heatsink performance;
- isolated-I²C recovery and Channel-B communication fault handling;
- waveform/setpoint fidelity between Channel A and Channel B where applicable.

The detailed Rev.B.4 engineering baseline is stored under [`docs/`](docs/).

## Out of scope for the current revision

The previously explored idea of using the supply as a bidirectional analog power amplifier / stereo or bridged signal amplifier is **archived for now**. It is not part of the current schematic baseline and should only be reconsidered if PCB area and thermal margin make the extra circuitry worthwhile.

## License

WTK.PowerSource is released under the **PolyForm Noncommercial License 1.0.0**, following the licensing model used by other WTK.* projects.

Personal use, study, research, evaluation, hobby use, and other noncommercial purposes are permitted under the license terms. Commercial use, resale, or integration into a paid product or service requires a separate commercial license from the copyright holder.

**Required Notice:** Copyright 2026 Rodrigo Wantuk.

See [`LICENSE.md`](LICENSE.md) for the complete terms.
