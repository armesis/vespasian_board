---
status: accepted
date: 2026-09-03
---

# Level-shift buffer on the motor outputs, with its /OE as a hardware interlock

All four ESC signal lines pass through a `74AHCT125` instead of being driven straight from the MCU. SimonK-era ESCs run an ATmega at 5 V, where V<sub>IH</sub> is about 3.0 V, and the ESP32-S3 outputs 3.3 V. That 0.3 V of margin across temperature, cable length and part variation is the kind that works on the bench and then intermittently fails to arm one motor. HCT inputs are TTL-threshold, so 3.3 V drives them solidly, and the part runs from 5 V so the ESCs see clean 5 V pulses. Add 33–100 Ω series resistors on the outputs for edge damping.

The second reason is safety. The buffer's four active-low output enables are tied together to `GPIO9` with a pull-up to 3.3 V and an external safety switch in series. On power-up and throughout reset the MCU's GPIOs are high-impedance inputs, so the pull-up holds `/OE` high, all four outputs go high-Z, and the ESCs cannot arm. Firmware must *actively* drive that line low to enable motors.

## Consequences

- Firmware drives `GPIO9` low only after a full sensor and receiver health check.
- The interlock is independent of firmware, so the aircraft can be handled with a battery connected and props fitted without trusting code — which matters on a team build where several people handle it.
- Pull-downs on the ESC side of the buffer keep the lines defined while the outputs are high-Z.
