---
status: accepted
date: 2026-09-03
amended: 2026-09-15
---

# The compass lives on the GPS mast, not on the carrier board

Heading comes from a magnetometer co-located with the GPS on a mast, and the GY-91's own AK8963 is disabled in firmware. Earth's field is about 50 µT; a conductor carrying 30 A produces a comparable field at roughly 12 cm and far more up close. A magnetometer sitting centimetres from four ESC power outputs is not a compass, it is an ammeter.

The carrier board therefore exposes one 10-pin JST-GH (`BM10B-GHS-TBT`) with the full Pixhawk GPS1 pinout, carrying the GPS UART, the compass I2C and the mast module's accessories: a piezo buzzer, the `SAFETY_SWITCH` button and its LED. Note that pin 1 is **5 V**, not 3.3 V — Pixhawk-compatible GPS modules regulate on board — while pin 8, `VDD_3V3`, is a rail the *carrier board* supplies to those accessories.

## Consequences

- Costs 7 GPIO and a cable up the mast: UART1 on `GPIO17`/`GPIO18`, I2C on `GPIO8`/`GPIO9`, the buzzer on `GPIO5`, `SAFETY_SWITCH` on `GPIO21` and its LED on `GPIO47`.
- I2C runs at 100 kHz over that cable. The board carries no pull-ups and no footprints for them: the module's own were confirmed by measurement on 2026-09-09, and doubling them would over-pull the bus.
- Sidesteps the risk that the GY-91's AK8963 is absent ([ADR-0004](0004-gy91-hard-soldered-on-spi.md)) or unreachable through the MPU-9250's aux-I2C master.
- GNSS reception is the second, independent reason the module stays off-board: L1 arrives at roughly −130 dBm, and a switching regulator alongside it raises the noise floor.
- **Verify pin 8 is an input before first power-up.** The Pixhawk standard defines `VDD_3V3` as supplied by the autopilot, and Rev.0.0 ties it to the 3.3 V rail on that basis. A module that instead brings its own regulator out on pin 8 would put two regulator outputs in contention. Power the module from pin 1 alone and measure pin 8: if it reads about 3.3 V, disconnect it.
- The buzzer is a piezo returned to pin 8, so `BUZZER−` is pulled low straight from `GPIO5` through 110 Ω — no transistor. A piezo draws well under a milliamp on average; the resistor caps the edge current into its capacitance and costs no volume. Loudness comes from driving the element at its mechanical resonance, not from drive current.
- `SAFETY_SWITCH` is read by firmware on `GPIO21`, with a 10 kΩ pull-up and 100 nF debounce, and its LED is pulled low from `GPIO47` through 330 Ω. The button gates arming in software only. It is **not** the hardware interlock of [ADR-0007](0007-motor-buffer-and-hardware-interlock.md).
