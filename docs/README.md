# WTK.PowerSource technical documentation

This directory is the **canonical engineering documentation** for WTK.PowerSource. The project no longer depends on a PDF as the active design baseline; technical decisions are maintained as version-controlled Markdown so schematic, firmware, sourcing, and validation changes can evolve together.

## Documentation index

1. [`01-Project-Scope-and-Requirements.md`](01-Project-Scope-and-Requirements.md) — project goals, electrical envelope, current-source mode, series operation, and status vocabulary.
2. [`02-Electrical-Architecture.md`](02-Electrical-Architecture.md) — isolated domains, per-channel power path, grounding rules, series topology, pass stage, and buck fallback.
3. [`03-Input-Preregulator-and-Power-Stage.md`](03-Input-Preregulator-and-Power-Stage.md) — PSU, fuse/eFuse, XL4016, Schottky, inductor, DAC-controlled VPRE, SOA/foldback, and reverse-power considerations.
4. [`04-CV-CC-and-Output-Control.md`](04-CV-CC-and-Output-Control.md) — analog CV/CC loops, TIP36C, current-source compliance, setpoints, K_OUT, waveform considerations, and output capacitance.
5. [`05-Measurement-Telemetry-and-Calibration.md`](05-Measurement-Telemetry-and-Calibration.md) — 50 mΩ shunt, ADS1115, CD4051/NTCs, update rates, calibration, and Flash storage.
6. [`06-Isolation-and-Communication.md`](06-Isolation-and-Communication.md) — one-MCU rule, 6N137 paths, isolated I2C, local Channel-B safety, STM32F103 I2C recovery, and `COMM_B_FAULT`.
7. [`07-Protection-and-Fault-Handling.md`](07-Protection-and-Fault-Handling.md) — protection hierarchy, eFuse/fuse coordination, OVP/OCP/thermal, K_OUT, backfeed, and fault classification.
8. [`08-Thermal-Mechanical-and-PCB.md`](08-Thermal-Mechanical-and-PCB.md) — 1 oz PCB strategy, power-wire jumpers, Kelvin routing, shared heatsink, NTC/fan approach, isolation layout, and test points.
9. [`09-Firmware-Architecture.md`](09-Firmware-Architecture.md) — firmware responsibilities, source layout, VPRE management, current-source behavior, telemetry, I2C recovery, startup, and testing.
10. [`10-Schematic-Organization-and-Net-Naming.md`](10-Schematic-Organization-and-Net-Naming.md) — eight EasyEDA sheets, domain-safe net naming, isolation-barrier rules, grounds, and inter-sheet connectivity.
11. [`11-Bring-Up-and-Validation-Plan.md`](11-Bring-Up-and-Validation-Plan.md) — staged prototype qualification and release gates.
12. [`12-BOM-and-Sourcing.md`](12-BOM-and-Sourcing.md) — component direction, Brazil sourcing philosophy, XL4016/eFuse risks, DNP strategy, and package preferences.
13. [`13-Open-Items-and-Decision-Log.md`](13-Open-Items-and-Decision-Log.md) — unresolved items, frozen decisions, and explicitly removed/archived approaches.

## Status vocabulary

All technical documents use the same terms:

- **FROZEN** — architectural decision; do not change silently.
- **PROVISIONAL** — current choice, still replaceable within the architecture.
- **TUNE** — must be finalized on the bench.
- **OPEN** — intentionally unresolved.
- **DNP** — footprint/function may exist but may remain unpopulated.

## Documentation rules

- Do not promote a target into a guaranteed specification before bench qualification.
- Keep component sourcing reality separate from datasheet suitability.
- Record safety, isolation, SOA, thermal, and series-mode changes explicitly.
- A released PCB revision must reference documentation that describes the same electrical state.
- `PCB/` contains actual EDA/fabrication artifacts; architecture diagrams and rationale belong here.
- Firmware behavior that is safety-relevant must be reflected both here and in `Firmware/README.md` when appropriate.

## Historical PDF

Earlier Rev.B.x PDF documents were useful during architecture exploration, but they are no longer the canonical baseline. If retained locally for history, they should be treated as superseded snapshots rather than authoritative design input.
