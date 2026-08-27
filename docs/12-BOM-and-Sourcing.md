# BOM and sourcing strategy

## Sourcing philosophy

WTK.PowerSource is deliberately designed around parts that can be purchased in small quantities with practical Brazilian shipping.

International distributors may be used for datasheets and technical reference, but should not automatically be treated as practical BOM sources when freight, minimum order, or import overhead dominates the component cost.

The project prefers:

- Brazilian electronics retailers;
- common through-hole/SOIC/SOT packages;
- modules when they genuinely reduce sourcing or assembly risk;
- 0805/1206 passives rather than unnecessarily small packages;
- replaceable power semiconductors;
- multiple compatible sources where practical.

## Current principal components

| Function | Current part/direction | Status | Notes |
|---|---|---|---|
| MCU | STM32F103C8T6 / Blue Pill class | FROZEN architecture | exactly one MCU |
| Buck preregulator | XL4016E1 | PROVISIONAL / sourcing risk | integrated PCB preferred; external module fallback required |
| Pass transistor | TIP36C | PROVISIONAL/FROZEN baseline | one per channel initially |
| Buck Schottky | 20 A / 60 V class, STPS20L60CT-class | PROVISIONAL | 60 V preferred for future 36 V input margin |
| Telemetry ADC | ADS1115 | PROVISIONAL strong preference | one per isolated domain |
| VPRE DAC | MCP4725-class | PROVISIONAL strong preference | slow preregulator command |
| Thermal mux | CD4051-class | PROVISIONAL | inexpensive multiple-sensor expansion |
| Fast optocoupler | 6N137-class | PROVISIONAL strong preference | VSET/ISET Channel-B paths |
| I2C isolator | ISO1540-class | PROVISIONAL | exact sourcing to verify |
| eFuse | MP5046 family candidate | OPEN / DNP allowed | function desired, PN not production-frozen |
| Current shunt | 2 × 0.10 Ω / 5 W parallel | PROVISIONAL | ~50 mΩ / 10 W nominal combined |
| NTC | 10 kΩ B3950-class | PROVISIONAL | exact part/placement OPEN |
| Analog op-amps | LM358B/LM358BA-class where suitable | PROVISIONAL | final loops/offset requirements may change choice |
| Comparator/reference | LM393/TL431-class where suitable | PROVISIONAL | protection independence is more important than exact PN |
| Output relay | DC-rated normally-open relay | OPEN | must satisfy current and series-mode DC voltage requirements |
| External fuse | 5×20 mm time-delay class | TUNE | current rating after PSU/inrush tests |

## XL4016 sourcing risk

The XL4016 module ecosystem is common, but loose XL4016E1 IC availability in Brazil is weaker.

This is classified as a **high sourcing risk** because integrating the buck directly into the PCB affects the layout.

Mitigation is architectural:

- integrated XL4016 footprint remains preferred;
- provide an external-buck interface;
- allow the internal buck section to be omitted/bypassed without redesigning the linear/output circuitry.

Do not wait until bring-up to discover there is no physical way to substitute an external module.

## eFuse sourcing

The eFuse is intentionally non-critical to basic operation.

Requirements for a future production-approved PN:

- input voltage compatible with 36 V maximum system source;
- useful current limit around the actual input-current envelope;
- enable/shutdown;
- autonomous protection;
- reverse-current behavior understood;
- realistic Brazil sourcing/cost;
- manageable package/assembly.

If no suitable device is available, the board operates with the eFuse DNP and its power bypass populated, while the external passive fuse and all downstream protections remain present.

## Module use

Modules are acceptable when they solve a real procurement or prototype problem.

Examples:

- ADS1115 module during early prototype;
- MCP4725 module during early prototype;
- external XL4016 module as fallback.

However, a module should not be retained blindly if integrating the components improves thermals, current routing, control, and repeatability enough to justify the PCB area.

## Passives

Preferred baseline:

- ordinary precision signal resistors: 0805;
- higher voltage/power or mechanically robust positions: 1206/2512/PTH as appropriate;
- shunt: high-power PTH or purpose-built low-ohm part with Kelvin-aware layout;
- power electrolytics: 50 V rating in nodes that may see future ≤36 V input/VPRE;
- 63 V parts allowed where cost/size are similar but not mandatory.

## DNP documentation

Every prototype/release BOM must explicitly list DNP state for at least:

- eFuse;
- eFuse bypass link;
- external-buck/internal-buck selection links;
- optional snubbers;
- optional filtering/capacitance used for tuning;
- optional sensor footprints.

Never rely on schematic comments alone to communicate a safety-critical DNP population state.
