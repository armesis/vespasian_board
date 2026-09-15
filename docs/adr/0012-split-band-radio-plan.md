---
status: accepted
date: 2026-09-03
amended: 2026-09-15
---

# Three radios, three purposes, two bands

Control is FlySky SBUS at 2.4 GHz, on one pin: UART2 receive on `GPIO16`. Ground-station telemetry is a 433 MHz LoRa module on a UART. Drone-to-drone swarm messaging is ESP-NOW, on the ESP32's own 2.4 GHz radio and therefore free.

SBUS replaced the originally planned IBUS on 2026-09-09. SBUS is an inverted UART at 100 000 baud, 8E2, which on most flight controllers costs an inverter transistor. The ESP32-S3 inverts in the UART peripheral itself (`uart_set_line_inverse`), so the change added no hardware. UART2 has no IO_MUX pins on the S3 and reaches `GPIO16` through the GPIO matrix, which is irrelevant at this baud rate.

The LoRa link is deliberately reserved for the ground station alone rather than being loaded with swarm traffic as well, which is why the swarm link stays on ESP-NOW.

## Consequences — an accepted risk

ESP-NOW and the FlySky link share 2.4 GHz, and AFHDS-2A frequency-hops across the whole band, so a clear channel cannot simply be chosen. A transmitter radiating within centimetres of the flight-critical control receiver will periodically desense it, and this gets **worse as more aircraft join** — precisely when it matters most.

Mitigations, in order of effect:

1. Set ESP-NOW TX power to about **11 dBm** rather than the 20 dBm default. That is 9 dB less energy hitting our own receiver and still ample between aircraft. This is the single highest-value line of code on the problem.
2. Separate and cross-polarise the two antennas. [ADR-0003](0003-external-antenna-module.md) is what makes this possible — a PCB trace antenna could not be moved.
3. Cap swarm traffic near 50-byte packets at 5 Hz, roughly 0.25 % airtime per aircraft. Resist streaming anything.
4. Consider ESP-NOW long-range mode: lower rate, better link margin.

Separately, 433 MHz PA noise is a documented cause of GNSS desense. Keep that antenna at least 15 cm from the GPS, which is on a mast, so mount the LoRa antenna low and on the opposite side.

If this proves untenable in multi-aircraft flight testing, the exit is to move the *control* link out of band — 900 MHz ELRS — rather than to move the swarm link. The receiver connector carries only the receive line, which is enough for ELRS control over CRSF. Sending telemetry back up the control link would also need `GPIO15`, the free pin beside `GPIO16`, routed to the connector.
