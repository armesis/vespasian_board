---
status: accepted
date: 2026-09-03
---

# External antenna (WROOM-1U) rather than the PCB-antenna module

We use the `ESP32-S3-WROOM-1U` with a U.FL connector and an off-board antenna instead of the WROOM-1's PCB trace antenna. A trace antenna buried in a drone stack — surrounded by a LiPo, four ESCs, carbon fibre and our own power pours — is detuned and shadowed, and inter-drone range is the entire point of the ESP-NOW link ([ADR-0012](0012-split-band-radio-plan.md)).

Two bonuses fell out of it. Per the Espressif datasheet (v1.8), the 1U "has no antenna keepout zone", which frees module placement on a congested board instead of pinning it to an edge. And it is 6.3 mm shorter than the WROOM-1.

## Consequences

- The U.FL connector is rated for roughly 30 mating cycles and tears off if the cable is pulled. The pigtail must be glued down and strain-relieved.
- If the cover is carbon fibre it is an RF shield, so the antenna has to exit outside it through a grommeted slot.
