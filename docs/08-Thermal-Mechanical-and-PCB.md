# Thermal, mechanical, and PCB design

## PCB baseline

The fabrication baseline is deliberately modest:

- 2-layer FR-4;
- 1 oz copper;
- ordinary prototype-compatible manufacturing;
- manual assembly and repair considered from the start.

The project does **not** require 2 oz copper simply to carry 5 A.

## High-current routing

Use a hierarchy of techniques:

1. wide top-layer copper where convenient;
2. parallel top/bottom pours;
3. via stitching between current-sharing planes;
4. very short high-current paths;
5. soldered copper-wire/cable jumpers where this is cleaner or more reliable than consuming large PCB area.

Power-wire jumpers are an allowed production/prototype technique, not an emergency hack.

The PCB should include explicit PTH pads for them where needed.

## Power-wire sizing

Exact wire gauge depends on length, temperature, assembly method, and enclosure, but short 1–1.5 mm² copper conductors are comfortably within the intended current class.

Final assembly drawings must identify:

- wire gauge;
- insulation rating;
- routing;
- solder length/strain relief;
- whether the wire is required or optional for a given hardware revision.

## Kelvin sense routing

The current-shunt power path and sense path must be independent.

Correct principle:

```text
high-current conductor -> shunt power terminal
                         shunt element
high-current conductor <- shunt power terminal

separate thin Kelvin trace from each shunt terminal
                    ↓
measurement/CC circuitry
```

Do not measure from a power jumper, via field, or remote copper region and assume it equals the resistor terminal voltage.

## Heatsink concept

A single heatsink/profile per channel is preferred where practical.

Potentially hot components:

- XL4016 package;
- buck Schottky;
- TIP36C.

They may share one piece of aluminum only if their metal tabs are insulated appropriately.

The shared heatsink concept reduces mechanical complexity and allows a compact thermal block, but it creates two engineering concerns:

- thermal coupling among devices;
- capacitive coupling from the buck switching node into the heatsink/linear stage.

Both must be measured.

## Electrical isolation to heatsink

Do not assume TO-220/TO-247 tabs are electrically equivalent.

The XL4016 tab belongs to its switching node. The buck Schottky tab may also belong to the switching/cathode node depending on PN. The TIP36C tab belongs to its collector.

If these parts share a heatsink, use appropriate insulating pads/mica, thermal compound as needed, and insulated mounting hardware.

The heatsink itself should not accidentally become a switching node or output node.

## Thermal sensors

Candidate NTC locations per channel:

- commercial PSU or incoming power section;
- XL4016/buck hotspot;
- Schottky or shared buck-heatsink region;
- TIP36C body/heatsink contact region;
- main shared heatsink;
- optional spare point.

The actual populated sensor count should be optimized after the PCB/mechanical layout is known.

## Fan strategy

The fan is firmware-managed using thermal telemetry.

Initial conceptual curve:

```text
below ~40 °C   fan off or minimum
40–60 °C       increasing airflow
above ~60 °C   100% fan
critical temp  output shutdown
```

These are TUNE values, not guaranteed thresholds.

PWM versus simple on/off drive remains OPEN until the fan and acoustic goals are selected.

## One TIP36C and SOA

The use of one pass transistor per channel reduces cost and layout area but requires careful thermal validation.

Normal operation should keep VPRE close to output voltage, limiting pass-device dissipation.

Worst stress occurs during:

- short-circuit onset before VPRE falls;
- large load transients;
- waveform operation with VPRE fixed above a varying output;
- cooling failure.

The transistor must be evaluated at realistic case temperature, not only at 25 °C datasheet conditions.

## Ripple versus layout

The current ripple/noise acceptance target is approximately ≤25 mVpp under defined DC test conditions.

Meeting this target depends on:

- buck hot-loop area;
- SW-node copper size and placement;
- Schottky/inductor/capacitor layout;
- ground-current return paths;
- separation of analog sense from switching currents;
- post-buck/linear-stage loop behavior;
- test-probe technique.

Do not add large filtering blindly before measuring; extra capacitance can alter CV/CC stability and waveform behavior.

## Isolation barrier on PCB

`GND_A` and `GND_B` must remain physically and electrically auditable.

The layout should provide an obvious isolation corridor around optocouplers/isolated-I2C devices and should avoid copper pours silently crossing the barrier.

Exact creepage/clearance is modest for the low-voltage DC difference involved, but manufacturing cleanliness and debugging clarity justify visibly conservative spacing.

## Test points

Provide accessible test points for at least:

- VIN raw/protected per channel;
- SW node with a nearby ground reference suitable for a short probe loop;
- VPRE;
- output-regulated internal node;
- output voltage sense;
- shunt Kelvin nodes;
- VSET/ISET;
- CC_ACTIVE / FAULT;
- isolated I2C sides;
- temperature mux output.

A power project without intentional test access creates avoidable bring-up risk.
