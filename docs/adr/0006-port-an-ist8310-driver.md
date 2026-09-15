---
status: accepted
date: 2026-09-03
---

# Port an IST8310 driver into ESP-FC rather than change the GPS module

The chosen mast module carries an IST8310 — the ArduPilot and PX4 standard part, and therefore what Pixhawk-compatible GPS modules ship with. ESP-FC comes from the Betaflight lineage and has never needed it: `lib/Espfc/src/Device/Mag/` contains drivers for AK8963, HMC5883L, QMC5883L and QMC5883P only. We port the driver rather than swap to a QMC5883-based module.

The work is small. ESP-FC already has a `MagDevice` base class and a `BusI2C`, so this is one subclass with the register sequence lifted from PX4's `src/drivers/magnetometer/isentek/ist8310/`. Use `MagQMC5883L` as the structural template and PX4 only for the register sequence: I2C-only, address `0x0E` (some vendors strap `0x0C`, `0x0D` or `0x0F`), `WHO_AM_I` at `0x00` returns `0x10`, single-measurement mode via `CNTL1`, 0.3 µT per LSB.

## Consequences

- PX4 is BSD-3-Clause. Keep the copyright header on anything lifted; it is compatible, but do not strip it.
- Until this driver works there is no heading source at all, because the AK8963 is disabled by [ADR-0005](0005-compass-on-the-gps-mast.md).
