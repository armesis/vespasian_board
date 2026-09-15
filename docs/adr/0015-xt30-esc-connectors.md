---
status: accepted
date: 2026-09-08
---

# Each ESC plugs in by an XT30, re-terminated from its XT60

The four ESCs connect to the carrier board through board-mounted `XT30PW-F` connectors, one per ESC, rather than through solder pads or the XT60s the ESCs shipped with. Each ESC carries a quarter of pack current, so a pack-sized connector on every output spends mass and board edge on capacity nothing uses. One connector per ESC keeps each ESC individually removable and needs no external splitter harness.

## Considered options

- **Board-mount XT60 on every output.** Mates with the ESCs as delivered, but puts five XT60s on a board that [ADR-0009](0009-escs-mounted-centrally.md) needs compact.
- **Solder pads.** Lightest and lowest-resistance, but no ESC can then be removed without a soldering iron.

## Consequences

- **All four ESCs are re-terminated from XT60 to XT30.** Check each lead's polarity with a meter afterwards; a swapped lead mates happily ([ADR-0014](0014-no-reverse-polarity-protection.md)).
- **The margin is thinner than the connector's name suggests.** AMASS rates the XT30PW at 15 A continuous, permitting 30 A for up to 30 minutes at under 80 °C rise — "XT30" is the short-term figure, not the rating. A quarter of the 60 A burst is 15 A, exactly at the continuous rating, and a quarter of the 66 A design ceiling is 16.5 A, just over it. This holds because burst is never sustained and hover is about 5 A per ESC, but it is not the 2× headroom the name implies. A sustained high-throttle flight profile would need re-checking against the 30-minute figure.
- Each ESC feed gains about 1.2 mΩ of contact resistance.
