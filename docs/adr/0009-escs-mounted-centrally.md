---
status: accepted
date: 2026-09-03
amended: 2026-09-15
---

# ESCs mount centrally, not on the arms

The four ESCs sit on the centre plate rather than out on the arms. This collapses the high-current path: with arm-mounted ESCs the two front motor feeds would run roughly 45 mm along the board edge to reach the chamfers, whereas central mounting clusters the XT60, bulk capacitors, shunt and all four ESC connectors ([ADR-0015](0015-xt30-esc-connectors.md)) into about 50 × 25 mm. That area was estimated around solder pads, so re-check it against the XT30 bodies at layout. The motor phase wires get longer instead, which is a far cheaper problem — the same current, and they can be twisted.

## Consequences

- The entire upper half of the board carries no significant current, which is what makes it defensible to put the gyro and the 2.4 GHz radio there.
- The chamfers of the octagonal outline are freed for mounting hardware rather than power pads.
- 60 A continuous would need roughly 40 mm of trace width, which is why the path is kept short instead of wide. Pour on L1, L3 and L4, stitch with via arrays, and open the solder mask over the pours so they can be flooded with solder.
- **The ground return carries pack current too.** Every amp that leaves through the ESC connectors comes back through the board to the XT60, so the ground copper between them is a power conductor, not a quiet reference. Star the four ESC grounds at the pack negative and give that return the same pour treatment as the positive path. Keep it confined to the power domain, place the digital domain's connection so that return current never flows beneath the INA180, the ADC dividers or the gyro, and route no digital signal across a gap in the reference plane.
