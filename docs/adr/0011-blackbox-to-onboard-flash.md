---
status: accepted
date: 2026-09-03
---

# Blackbox logs to onboard SPI flash

Flight logs go to a `W25Q128` on the board rather than to an OpenLog or OpenLager on a serial port. The ESP32-S3 has only three hardware UARTs, and four things wanted one — GPS, receiver, logger, and the ground-station telemetry radio. The logger was the only one of the four that could move off serial entirely.

It sits on SPI3, a separate host from the gyro's SPI2, so blackbox writes cannot inject jitter into gyro reads. The part is SOIC-8, one of the easiest things on the board to hand-solder.

## Considered options

An OpenLog logs to a removable microSD, which is far easier to get data off — pull the card versus dumping over MSP — and its capacity is effectively unlimited. It cost a UART, an external module, and a cable that can shake loose in a crash.

## Consequences

Logs come off over MSP, not by pulling a card. Budget time for that in the flight-test loop.
