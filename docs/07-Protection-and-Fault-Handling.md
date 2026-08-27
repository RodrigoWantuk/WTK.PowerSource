# Protection and fault handling

## Protection layers

WTK.PowerSource deliberately uses several protection layers with different jobs.

```text
commercial PSU internal protection
        ↓
external passive fuse
        ↓
optional eFuse near legitimate input-current limit
        ↓
preregulator hardware limits
        ↓
analog CC loop
        ↓
analog OCP / OVP / thermal protection
        ↓
K_OUT physical disconnect
```

No single layer is expected to replace all others.

## External passive fuse

A conventional fuse is always expected in the final assembly, mounted externally in the harness/panel or another serviceable location.

Its function is:

- catastrophic fault-energy protection;
- wiring protection;
- protection in builds where the eFuse is DNP;
- protection after a severe semiconductor failure that bypasses active control.

It is **not** the normal operating current limiter.

Because the commercial PSU will often enter current limit/hiccup before a slow fuse sees enough current/time to open, the passive fuse intentionally has more margin than the eFuse.

Exact fuse rating is TUNE after PSU, inrush, and maximum normal input current are measured.

## eFuse philosophy

The eFuse is an optional active protection layer placed on the DC input of each channel.

Its current threshold should be configured **close to the maximum legitimate input current**, taking the device's current-limit tolerance into account.

The design intent is:

```text
maximum normal input current
        ↓ small margin
eFuse current limit / trip region
        ↓ larger margin
passive fuse region
```

This ensures the eFuse reacts to an abnormal load demand before the commercial PSU or passive fuse becomes the only limiting mechanism.

The eFuse must remain DNP-capable so sourcing cannot block the entire prototype.

## MP5046 candidate

MP5046-family devices are technically attractive because of high voltage range, integrated back-to-back MOSFET behavior, current limiting, reverse blocking, enable/status functions, and suitability for the present 24 V / future ≤36 V input envelope.

The specific MP5046 PN remains **PROVISIONAL / DNP-capable / sourcing-dependent**. It must not become a single-source architectural dependency.

## Analog CC

CC is a normal control mode, not a fault by itself.

The CC loop should limit requested load current quickly and continuously. It remains active without MCU intervention.

`CC_ACTIVE` is exported for firmware/VPRE/telemetry purposes.

## Hardware OCP

A separate hard overcurrent threshold is required above the normal programmable CC range.

Purpose:

- detect gross failure or loop saturation;
- force a fault state;
- disable preregulator and/or open K_OUT;
- provide a hardware response even if firmware is stalled.

Exact threshold and hysteresis are TUNE after current-loop behavior is measured.

## OVP

The output has an analog overvoltage protection path independent of the normal CV command.

OVP must be designed so a fault in the control reference cannot move both the programmed output and its protection threshold in the same unsafe direction.

Where practical, control reference and protection reference should be physically/logically independent.

OVP behavior should remove output authority through the hardware kill chain and K_OUT.

## Thermal protection

Thermal supervision exists at two levels:

1. firmware telemetry/fan management;
2. hardware emergency shutdown for critical temperature.

The hardware trip remains TUNE. An initial conservative design region around **65–70 °C heatsink/case measurement** is more appropriate than waiting for 80 °C before emergency action, but final limits depend on sensor placement, thermal interface, and junction-to-case rise.

An NTC is too slow to protect the TIP36C from instantaneous second-breakdown stress during a short. Thermal protection complements, but does not replace, SOA-safe electrical foldback.

## K_OUT fail-safe chain

K_OUT is normally open with the coil de-energized.

Conceptually, coil drive requires:

```text
MCU_OUTPUT_ENABLE
AND
LOCAL_HW_OK
```

Any critical hardware fault must be able to remove `LOCAL_HW_OK` without firmware cooperation.

Possible K_OUT-opening causes include:

- user OUTPUT OFF;
- power-up/reset;
- watchdog/reset cause;
- OVP;
- hard OCP;
- critical temperature;
- eFuse fault;
- backfeed/reverse-energy detection;
- unrecoverable Channel-B communication failure;
- series-mode switching sequence;
- invalid setpoint/calibration state.

## Backfeed and reverse-energy protection

The output must tolerate realistic bench scenarios such as:

- battery connected to the output;
- another power supply forcing the output;
- inductive load returning energy;
- one series channel remaining energized while the other changes state.

Two distinct conditions matter:

### Negative output forcing

A clamp path should prevent destructive reverse voltage across the pass stage if the output is driven negative relative to its local return.

### Positive external backfeed

If an external source drives the output above the internal command, current may try to propagate through the linear stage toward VPRE and the buck.

K_OUT is the intended physical isolation mechanism, but its mechanical opening delay means a fast clamp/blocking strategy may still be needed.

Final architecture is OPEN pending backfeed tests.

## Backfeed validation instrumentation

Backfeed tests must record more than the output voltage.

Measure simultaneously or with sufficient synchronized instrumentation:

- `VOUT`;
- `VPRE`;
- buck VIN / protected input;
- input-branch current where possible;
- K_OUT coil/contact timing;
- clamp current/temperature where relevant.

The goal is to identify exactly how far reverse energy travels before K_OUT opens.

## Fault classes

Firmware should distinguish at least:

- `FAULT_HW_A/B` — local electrical protection event;
- `COMM_B_FAULT` — persistent isolated-bus/control failure;
- `THERMAL_WARNING` — temperature high but not yet emergency trip;
- `EFUSE_FAULT` — input protection event;
- `BACKFEED_FAULT` — external reverse-power condition where detected;
- `SENSOR_FAULT` — implausible/missing telemetry;
- `WATCHDOG_RESET` / reset-cause diagnostic.

Different faults may share the same safe action while remaining distinct in the UI/log for diagnosis.
