# WTK.PowerSource

WTK.PowerSource is a dual-channel programmable bench power supply focused on **low-cost, repairable hardware**, analog safety loops, digital supervision, and practical component sourcing.

The project combines two isolated DC power domains with a single **STM32F103C8T6** controller. Each output channel uses a switching preregulator followed by a linear pass stage so the switching converter handles most of the power conversion while the linear stage performs the final CV/CC regulation and output control.

> **Current status — August 2026:** architecture and protection strategy are being converted into the first formal schematic. The project documentation is maintained as version-controlled Markdown under [`docs/`](docs/); targets still marked PROVISIONAL/TUNE are not guaranteed specifications.

## Project goals

- Two electrically isolated and independently controlled output channels.
- **One MCU only**: STM32F103C8T6/Blue Pill class.
- Initial operation from **24 VDC isolated supplies per channel**.
- No artificial top-end voltage limit in Rev.1: maximum regulated output is whatever remains after the real preregulator, pass-stage, shunt, relay, wiring, and PSU drops.
- Future design envelope of **up to 36 VDC input per channel** and approximately **30 V output per channel** when powered appropriately.
- Up to **5 A/channel**, subject to channel power and thermal limits.
- Approximately **60 W/channel** initial power class.
- Series operation for higher differential voltage and symmetric `+V / COM / -V` use.
- Analog CV/CC loops independent of firmware timing.
- Hardware OVP/OCP/thermal authority plus fail-safe output relays.
- Current-source user mode using target current plus a voltage-compliance ceiling.
- Output ripple/noise acceptance target of approximately **25 mVpp or better** in normal DC CV operation, subject to bench qualification.
- Current measurement target in the **5–10 mA usable-resolution** class.
- Two-layer, **1 oz copper** PCB with wide copper and explicit power-wire jumper reinforcement where useful.
- Components chosen with realistic low-quantity sourcing and repair in Brazil in mind.

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
 buck preregulator                 buck preregulator
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

The two power domains remain isolated so the outputs can be independent or connected in series. During normal operation the instrument has **no USB, PC, or debugger connection**; only the isolated AC/DC supplies power the system.

## Current hardware direction

| Block | Current direction |
|---|---|
| MCU | 1 × STM32F103C8T6 / Blue Pill class |
| Input supplies | 2 × isolated 24 VDC supplies initially |
| Future input ceiling | 36 VDC per channel maximum |
| Preregulator | XL4016-based buck integrated on PCB, with external-module fallback |
| Linear pass transistor | 1 × TIP36C/channel initially |
| Output disconnect | normally-open fail-safe `K_OUT` relay/channel |
| Current shunt | ~50 mΩ low-side, initial 2 × 0.10 Ω / 5 W parallel concept |
| Telemetry ADC | ADS1115/channel domain |
| Thermal expansion | CD4051-class mux + multiple NTCs |
| VPRE command | MCP4725-class DAC direction |
| Channel-B isolation | fast optocouplers for PWM/setpoints + isolated I2C |
| eFuse | optional/DNP-capable; MP5046 family is a candidate, not a frozen production PN |
| Passive fuse | conventional external fuse/channel, with more margin than eFuse |
| PCB | 2 layers, 1 oz copper, wide pours and optional soldered power-wire reinforcement |

## Protection philosophy

```text
commercial PSU internal protection
        ↓
external conventional fuse
        ↓
optional eFuse near normal input-current limit
        ↓
preregulator hardware limits
        ↓
analog CC loop
        ↓
analog OCP / OVP / thermal protection
        ↓
K_OUT physical disconnect
```

The eFuse, when used, is configured close to the legitimate maximum input-current envelope. The passive fuse intentionally has more margin and exists mainly for catastrophic faults, wiring protection, or eFuse-DNP/failure scenarios.

Critical shutdown functions do not rely on firmware noticing the event first.

## Voltage and current-source operation

The same analog hardware supports both workflows.

**Voltage-oriented:**

```text
VSET = requested voltage
ISET = current limit
```

**Current-source:**

```text
ISET = requested current
VSET = compliance-voltage ceiling
```

No fast firmware current PID is required. The analog loops select whichever limit is active; firmware configures setpoints, optimizes VPRE slowly, and presents the operating state.

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
│   ├── tests/
│   ├── tools/
│   └── third_party/
└── docs/
    ├── README.md
    ├── 01-Project-Scope-and-Requirements.md
    ├── 02-Electrical-Architecture.md
    ├── 03-Input-Preregulator-and-Power-Stage.md
    ├── 04-CV-CC-and-Output-Control.md
    ├── 05-Measurement-Telemetry-and-Calibration.md
    ├── 06-Isolation-and-Communication.md
    ├── 07-Protection-and-Fault-Handling.md
    ├── 08-Thermal-Mechanical-and-PCB.md
    ├── 09-Firmware-Architecture.md
    ├── 10-Schematic-Organization-and-Net-Naming.md
    ├── 11-Bring-Up-and-Validation-Plan.md
    ├── 12-BOM-and-Sourcing.md
    └── 13-Open-Items-and-Decision-Log.md
```

Start with [`docs/README.md`](docs/README.md) for the documentation index.

- [`PCB/README.md`](PCB/README.md) — EasyEDA/PCB organization and fabrication rules.
- [`Firmware/README.md`](Firmware/README.md) — planned firmware responsibilities and safety boundaries.
- [`docs/`](docs/) — canonical design documentation and validation plan.

## Schematic direction

The current EasyEDA organization uses eight **real electrical** sheets:

1. `CH_A_INPUT_BUCK`
2. `CH_A_LINEAR_CV_CC`
3. `CH_A_MEAS_THERMAL`
4. `CH_B_INPUT_BUCK`
5. `CH_B_LINEAR_CV_CC`
6. `CH_B_MEAS_THERMAL`
7. `MCU_CONTROL_ISOLATION`
8. `OUTPUT_SERIES_SYSTEM`

Documentation-only block diagrams must not be built from electrical wires/net labels inside the schematic because multi-sheet net naming can accidentally merge domains.

## Current design status

The first production PCB is **not released**. Major remaining qualification includes:

- XL4016 sourcing and integrated-buck characterization;
- one-TIP36C SOA/foldback validation;
- CV/CC compensation and stability;
- 5–10 mA-class current measurement verification;
- ≤25 mVpp ripple/noise validation;
- reverse-energy/backfeed behavior;
- thermal/heatsink/fan characterization;
- series-mode relay/interlock tests;
- isolated-I2C recovery and Channel-B fail-safe behavior.

See [`docs/11-Bring-Up-and-Validation-Plan.md`](docs/11-Bring-Up-and-Validation-Plan.md).

## Archived idea

A future external analog-input / bidirectional power-amplifier mode was explored but is **not part of the current schematic baseline**. It may be reconsidered only after the normal power-supply PCB layout is known and spare area/thermal margin can be evaluated.

## License

WTK.PowerSource is released under the **PolyForm Noncommercial License 1.0.0**, following the licensing model used by other WTK.* projects.

Personal use, study, research, evaluation, hobby use, and other noncommercial purposes are permitted under the license terms. Commercial use, resale, or integration into a paid product or service requires a separate commercial license from the copyright holder.

**Required Notice:** Copyright 2026 Rodrigo Wantuk.

See [`LICENSE.md`](LICENSE.md) for the complete terms.
