---
status: accepted
date: 2026-09-03
amended: 2026-09-15
---

# ESC header +5 V pins are unconnected by default

Each of the four ESC signal headers is 3-pin (`SIG` / `+5V` / `GND`) for cable compatibility, but the middle pin is **left unconnected**. In Rev.0.0 it has no jumper footprint and goes nowhere. Cheap 30–40 A ESCs each carry their own BEC; commoning four of them to the board's 5 V rail would put four regulators and our own buck converter in contention, with the strongest winning and dissipating the difference as heat.

This is deliberate. Do not "fix" the unconnected pin. If an ESC BEC is ever wanted as a backup supply, connect **one** header's middle pin to the 5 V rail — a jumper in a later revision, a wire on this one — and never more than one.
