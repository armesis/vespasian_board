---
status: accepted
date: 2026-09-08
---

# No reverse-polarity protection; the keyed XT60 is the only guard

The carrier board has no diode or MOSFET against a reversed pack. The XT60 housing is keyed, so a correctly built pigtail cannot mate backwards, and that removes the in-field mistake reverse-polarity protection usually exists for. Protecting the 60 A pass-through would need a large MOSFET adding resistance to the highest-current path on the board, and the only cheap option — protecting the regulator branch alone — saves the flight controller but not the aircraft.

## Considered options

- **Series Schottky on the buck input only** (an `SS26`, say). About 0.4 V and a tenth of a watt at a few hundred milliamps: harmless, and one part. It protects the MCU, regulators and sensors, but the ESCs and the polarised bulk capacitors on the pass-through stay exposed.
- **High-side P-MOSFET with a gate zener**, on the same branch. Removes a diode drop this branch never needed, at the cost of three parts and an orientation — drain toward the pack — that fails silently if reversed.
- **Low-side N-MOSFET in the ground return.** Defeated outright: USB-C ground gives the pack a return path around it whenever USB is connected.

## Consequences — an accepted risk

The keying does not cover three cases, and all three are most likely during build and bring-up:

1. **The XT60 soldered to the board rotated 180°.** It then mates perfectly and delivers reversed power every time. The XT60 footprint must carry `+` and `−` on silkscreen for exactly this reason.
2. **A bench supply on clip leads**, which have no keying at all.
3. **A pigtail or ESC lead built with its wires swapped** — including the ESC re-terminations of [ADR-0015](0015-xt30-esc-connectors.md).

A reversed 3S pack drives the TPS54336A's input and the INA180's inputs far below their −0.3 V absolute minimum, reverse-biases the 470 µF electrolytics, which vent, and destroys the ESCs. Check polarity with a meter before the first power-up of every newly assembled board and every newly made lead, and bring boards up on a current-limited supply.
