---
status: deprecated
date: 2026-09-03
amended: 2026-09-18
---

# Level-shift buffer on the motor outputs, with its /OE as a hardware interlock

> **Dropped on 2026-09-18.** The ESCs in use work from the ESP32-S3's 3.3 V output directly, so the level shifter — this record's first reason — is not needed. Rev.0.0 carries no `74AHCT125`: `GPIO39`–`GPIO42` drive the ESC headers straight from the module.
>
> Dropping the buffer drops the second reason with it. Rev.0.0 has **no hardware interlock**: arming is gated in firmware alone, and that includes the mast's `SAFETY_SWITCH` ([ADR-0005](0005-compass-on-the-gps-mast.md)). The original decision is kept below for context.

## What dropping it costs

- A firmware hang is no longer backstopped in hardware ([ADR-0001](0001-esp-fc-on-a-custom-carrier-board.md)). Treat a connected battery as armed: props off on the bench, and nobody's hands near the motors while firmware is being flashed or debugged.
- Nothing holds the ESC signal lines low while the GPIOs are high-impedance during reset and boot. If an ESC twitches or beeps oddly at power-up, pull-downs on `M0`–`M3` are the cheap fix.
- Getting the interlock back later needs a buffer or gate in the motor path and one free GPIO for its enable.

## Original decision (2026-09-03)

All four ESC signal lines pass through a `74AHCT125` instead of being driven straight from the MCU. SimonK-era ESCs run an ATmega at 5 V, where V<sub>IH</sub> is about 3.0 V, and the ESP32-S3 outputs 3.3 V. That 0.3 V of margin across temperature, cable length and part variation is the kind that works on the bench and then intermittently fails to arm one motor. HCT inputs are TTL-threshold, so 3.3 V drives them solidly, and the part runs from 5 V so the ESCs see clean 5 V pulses. Add 33–100 Ω series resistors on the outputs for edge damping.

The second reason is safety. The buffer's four active-low output enables are tied together to `GPIO9` with a pull-up to 3.3 V and an external safety switch in series. On power-up and throughout reset the MCU's GPIOs are high-impedance inputs, so the pull-up holds `/OE` high, all four outputs go high-Z, and the ESCs cannot arm. Firmware must *actively* drive that line low to enable motors.

### Consequences, as originally recorded

- Firmware drives `GPIO9` low only after a full sensor and receiver health check.
- The interlock is independent of firmware, so the aircraft can be handled with a battery connected and props fitted without trusting code — which matters on a team build where several people handle it.
- Pull-downs on the ESC side of the buffer keep the lines defined while the outputs are high-Z.
