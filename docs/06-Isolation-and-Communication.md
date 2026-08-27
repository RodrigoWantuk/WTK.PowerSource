# Isolation and communication

## One-MCU rule

WTK.PowerSource uses **one central STM32F103-class MCU**.

Adding a second MCU to simplify Channel-B isolation was evaluated and rejected because of cost, duplicated firmware, synchronization complexity, and BOM growth.

This is a FROZEN architectural rule for the current design.

## Domain arrangement

The MCU belongs to the Channel-A/control domain.

Channel B remains electrically isolated and contains its own local analog regulation, measurement, protection, and auxiliary supply rails derived from its isolated PSU.

The MCU crosses the barrier only through explicit isolators.

## Fast setpoint isolation

Signals such as Channel-B `VSET` and `ISET` PWM can cross through 6N137-class fast optocouplers.

The important naming rule is:

```text
PWM_VSET_B_MCU -> optocoupler -> PWM_VSET_B_ISO
PWM_ISET_B_MCU -> optocoupler -> PWM_ISET_B_ISO
```

The same net name must never be used on both sides of an isolation device.

Propagation-delay asymmetry is small relative to a ~70 kHz PWM period, but it can still produce duty-width error. Static calibration corrects gain/offset but does not prove waveform fidelity.

A/B dynamic comparison therefore remains a validation gate where waveform functionality is used.

## Isolated I2C

The preferred telemetry/control architecture uses an isolated I2C link to Channel B.

The Channel-B side can host devices such as:

- ADS1115_B;
- MCP4725_B or equivalent VPRE DAC;
- other slow digital peripherals if later required.

ISO1540-class I2C isolation is a current technical direction; exact PN remains subject to sourcing and schematic review.

Again, names must remain domain-specific:

```text
I2C_SCL_A -> isolator -> I2C_SCL_B
I2C_SDA_A -> isolator -> I2C_SDA_B
```

## Local Channel-B protection

Channel-B OVP, OCP, thermal kill, and output-disconnect hardware remain local to `GND_B`.

A hardware fault should remove local power/output authority before or independently of reporting the event to the MCU.

The MCU receives a separately isolated fault/status indication for logging/UI/recovery decisions.

## ENABLE_B fail-safe concept

Channel B should have an explicit enable path whose safe/default state is disabled.

Loss of the A-domain command, reset, broken isolator path, or uninitialized MCU should therefore not leave Channel B intentionally enabled.

Exact implementation may be optocoupler-based or incorporated into the local hardware-enable logic.

## STM32F103 I2C lock-up handling

The STM32F103 I2C peripheral has documented errata involving a stuck BUSY state. A global watchdog is not enough because the CPU can remain alive while only the I2C peripheral/bus becomes unusable.

Firmware requirements:

1. every isolated-I2C transaction has a bounded timeout;
2. on timeout, disable/release the peripheral as required;
3. switch SCL/SDA to open-drain GPIO for bus recovery;
4. generate recovery clocks/STOP where appropriate;
5. execute the STM32F1-specific recovery sequence needed for a stuck BUSY state;
6. reinitialize the I2C peripheral;
7. retry only a bounded number of times;
8. classify persistent failure as `COMM_B_FAULT`.

## COMM_B_FAULT behavior

Persistent loss of Channel-B communication is **not** the same as `FAULT_HW_B`, but it is still a fail-safe output condition.

Rationale: the isolated I2C bus carries both telemetry and slow VPRE control. Continuing to energize an external DUT while blind to Channel-B voltage/current/temperature and unable to command its DAC is not desirable.

After bounded recovery attempts fail, firmware should:

- open `K_OUT_B`;
- disable or force safe `VPRE_B` command/enable;
- show a distinct communication fault in the UI;
- retain local analog hardware protections;
- require explicit successful recovery before normal operation resumes.

## Normal-use isolation

During normal product use there is no USB/PC/debugger attached.

This keeps the channel isolation model straightforward and avoids an external PC earth connection from unintentionally bonding the A-domain reference.

Debug procedures are a service condition and must be written so that they do not invalidate low-side sensing or series-mode isolation assumptions.
