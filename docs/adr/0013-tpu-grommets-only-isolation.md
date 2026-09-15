---
status: accepted
date: 2026-09-03
---

# TPU grommets are the only vibration isolation

The board is soft-mounted on four printed TPU grommets at M3 holes on a 78 × 78 mm pattern, and the GY-91 gets no damping pad of its own. Because the IMU is hard-soldered to the board ([ADR-0004](0004-gy91-hard-soldered-on-spi.md)), those grommets are the *entire* isolation stage.

They must place the mount's natural frequency well below the prop fundamental — roughly 100 Hz at hover and 206 Hz flat out, for 980 KV motors on 3S. Target **40–50 Hz**, which by f<sub>n</sub> ≈ 15.76/√δ means 0.10–0.16 mm of static deflection under the ~77 g board.

## Consequences — an accepted risk

TPU is highly resilient and lightly damped: it stores energy rather than absorbing it, so transmissibility peaks sharply at resonance instead of rolling off gently. Print soft (85–95 A shore) and thin-walled, and expect a rough patch during spool-up as the rotor sweeps through f<sub>n</sub>.

The hedge is in the layout rather than the firmware. The GY-91 footprint uses short flexible wire links with 1.5–2 mm of clearance underneath, so a foam damping pad can be added later — without a board respin — if the blackbox shows gyro noise. Filtering our way out in firmware is the fallback, not the plan: filters add phase lag to the control loop, and a GY-91-class gyro may not have the signal-to-noise headroom to spare.
