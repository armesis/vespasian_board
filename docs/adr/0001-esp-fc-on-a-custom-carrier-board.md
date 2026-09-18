---
status: accepted
date: 2026-09-03
amended: 2026-09-18
---

# ESP-FC on a custom carrier board as the primary flight controller

The aircraft is flown by a modified build of [ESP-FC](https://github.com/rtlopez/esp-fc) running on our own board, not by a COTS autopilot such as a Pixhawk or a Betaflight flight controller. ESP-FC is Betaflight-lineage, already supports the sensors and protocols we need, and being ESP32-based it puts Wi-Fi and ESP-NOW on the same chip that closes the attitude loop — which the eventual swarm capability depends on. The cost is that we own the firmware: every sensor, protocol and safety behaviour is ours to verify.

## Consequences

- The control loop, the radio stacks and GPS parsing all share one MCU. Core pinning and loop timing are our responsibility, not a vendor's.
- A firmware hang is an uncommanded descent. [ADR-0007](0007-motor-buffer-and-hardware-interlock.md) proposed a hardware interlock against it and was dropped on 2026-09-18, so Rev.0.0 relies on firmware alone.
- Nothing in the design may depend on a capability ESP-FC does not already have unless someone writes it — see [ADR-0006](0006-port-an-ist8310-driver.md).
