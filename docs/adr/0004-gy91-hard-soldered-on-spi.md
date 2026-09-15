---
status: accepted
date: 2026-09-03
amended: 2026-09-15
---

# GY-91 hard-soldered flat, on SPI

The IMU and barometer are a GY-91 module (MPU-9250 + BMP280) soldered flat against the carrier board — not socketed on pin headers, and not replaced by a discrete gyro die. Headers form a spring-mass system whose resonance sits squarely in quadcopter prop frequencies, so the sensor would be measuring noise we manufactured ourselves. The bus is SPI rather than I2C because ESP-FC reaches its 4 kHz gyro loop only with an SPI gyro.

## Considered options

A discrete `ICM-42688-P` on the board would be materially better — current production, no counterfeit lottery, far lower noise. It is an LGA-14 at 0.4 mm pitch and needs hot air plus a stencil, which conflicts with hand assembly. Rejected on assembly capability, not on merit.

## Consequences

- The MPU-9250 is discontinued and the GY-91 supply chain carries remarked dies. Verify `WHO_AM_I` (register `0x75`) reads `0x71` before trusting the part; `0x70` means an MPU-6500 with no magnetometer at all.
- The module is cantilevered from its single 8-pin row, so the free end needs mechanical support.
- Its onboard LDO is bypassed: feed the `3V3` pin directly and leave `VIN` open. Feeding both would tie the 3.3 V rail to that LDO's output.
- **Both chip selects carry 10 kΩ pull-ups to 3.3 V**, on `IMU_CS` and `BARO_CS`. The ESP32's GPIOs are high-impedance from power-on until firmware configures them, and a floating `NCS` or `CSB` in that window can leave the MPU-9250 or the BMP280 in the wrong interface mode. They look redundant and are not; do not remove them.
- The module's 8-pin row brings out no `INT` pin, so the gyro is polled on a firmware timer rather than read on data-ready. Loop timing jitter therefore comes from our own timer, not the sensor.
