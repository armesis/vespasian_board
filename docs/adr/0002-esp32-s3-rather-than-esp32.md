---
status: accepted
date: 2026-09-03
amended: 2026-09-15
---

# ESP32-S3 rather than the classic ESP32

ESP-FC lists both as recommended targets, so the choice was ours. We picked the `ESP32-S3-WROOM-1U-N16` because its native USB Serial/JTAG peripheral removes an entire subcircuit — no CH340 or CP2102N bridge, no DTR/RTS auto-reset transistors — on a board the team hand-assembles, and it hands us hardware debugging over the same cable. The no-PSRAM variant is deliberate: the octal-PSRAM parts consume GPIO35–37, and a flight controller has no use for the RAM.

## Considered options

The classic `ESP32-WROOM-32E` is ESP-FC's most-trodden target and has 8 freely-configurable RMT channels against the S3's hardwired 4 TX + 4 RX. That difference matters only for bidirectional DShot — and our ESCs are analog-PWM only, so RMT goes unused entirely. With that gone, the classic part's remaining advantages were cost and familiarity, weighed against a bridge IC and roughly seven extra components.

## Consequences

- If the ESCs are ever replaced with DShot-capable units, four-motor bidirectional DShot consumes every RMT channel. Rev.0.0 carries no WS2812 — status indication is the mast LED on `GPIO47` ([ADR-0005](0005-compass-on-the-gps-mast.md)) — so nothing competes today, but an addressable LED could not be added on RMT afterwards.
- ESP-FC's S3 target is less exercised than its ESP32 target. Validating the build, MSP-over-USB-CDC and the resource map is a prerequisite, not an assumption.
- Because native USB carries the console, UART0 on `GPIO43`/`GPIO44` is not needed for debug output. It is the third UART that [ADR-0011](0011-blackbox-to-onboard-flash.md) reserves for the telemetry radio.
