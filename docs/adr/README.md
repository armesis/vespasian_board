# Architecture decision records

Hardware and firmware-architecture decisions for the **VESPASIAN FC-1** carrier board — an ESP32-S3 flight controller and power distribution board for a 3S quadrotor.

Each record captures a decision that is hard to reverse, surprising without context, and the result of a genuine trade-off. Decisions that fail any of those three tests live in the design spec instead, not here.

| # | Decision | Note |
|---|---|---|
| [0001](0001-esp-fc-on-a-custom-carrier-board.md) | ESP-FC on a custom carrier board as the primary flight controller | Foundational |
| [0002](0002-esp32-s3-rather-than-esp32.md) | ESP32-S3 rather than the classic ESP32 | Amended 2026-09-15 |
| [0003](0003-external-antenna-module.md) | External antenna (WROOM-1U) rather than the PCB-antenna module | |
| [0004](0004-gy91-hard-soldered-on-spi.md) | GY-91 hard-soldered flat, on SPI | Amended 2026-09-15 |
| [0005](0005-compass-on-the-gps-mast.md) | The compass lives on the GPS mast, not on the carrier board | Amended 2026-09-15 |
| [0006](0006-port-an-ist8310-driver.md) | Port an IST8310 driver into ESP-FC rather than change the GPS module | Firmware work |
| [0007](0007-motor-buffer-and-hardware-interlock.md) | Level-shift buffer on the motor outputs, with its `/OE` as a hardware interlock | Safety · **Dropped 2026-09-18** |
| [0008](0008-esc-header-5v-pins-unconnected.md) | ESC header +5 V pins are unconnected by default | Do not "fix" · Amended 2026-09-15 |
| [0009](0009-escs-mounted-centrally.md) | ESCs mount centrally, not on the arms | Amended 2026-09-15 |
| [0010](0010-shunt-current-sensing.md) | Current sensing by shunt and INA180, not a hall sensor | Supersedes an ACS758 draft · Amended 2026-09-15 |
| [0011](0011-blackbox-to-onboard-flash.md) | Blackbox logs to onboard SPI flash | **Not in schematic** |
| [0012](0012-split-band-radio-plan.md) | Three radios, three purposes, two bands | Accepted risk · Amended 2026-09-15 |
| [0013](0013-tpu-grommets-only-isolation.md) | TPU grommets are the only vibration isolation | Accepted risk |
| [0014](0014-no-reverse-polarity-protection.md) | No reverse-polarity protection; the keyed XT60 is the only guard | Accepted risk |
| [0015](0015-xt30-esc-connectors.md) | Each ESC plugs in by an XT30, re-terminated from its XT60 | |

## Three records carry accepted risks

[ADR-0012](0012-split-band-radio-plan.md), [ADR-0013](0013-tpu-grommets-only-isolation.md) and [ADR-0014](0014-no-reverse-polarity-protection.md) document choices made with a known downside, along with the mitigations and, where one exists, the exit route. Read them before first power-up and flight testing, not after — they predict specific failure modes, and recognising one in the field is much faster than rediscovering it.

## Accepted, but not yet in the schematic

Compared against the saved Rev.0.0 schematic on 2026-09-15. These decisions stand, but the hardware they call for has not been drawn:

- **[ADR-0011](0011-blackbox-to-onboard-flash.md) — blackbox flash.** No `W25Q128`, and no SPI3 bus.
- **[ADR-0012](0012-split-band-radio-plan.md) — telemetry link.** No connector for the 433 MHz LoRa radio. UART0 on `GPIO43`/`GPIO44` is unused and free for it.

Eleven GPIO remain free — `GPIO4`, `6`, `7`, `15`, `35`–`38`, `43`, `44` and `48` — against the six these two need: four for SPI3 and two for the LoRa UART. `GPIO35`–`37` count as free only because the module is the no-PSRAM `N16` ([ADR-0002](0002-esp32-s3-rather-than-esp32.md)).

## Deliberately not recorded

The following were decided but do not meet the bar for an ADR, and live in the design spec:

- **Analog PWM ESC protocol** — dictated by the ESCs already owned. No alternative, so nothing to record.
- **Octagonal 112.4 mm outline, 4-layer 1.6 mm 2 oz stackup** — follows directly from the frame plate and the current requirement. Nobody will wonder why.
- **TPS54336A and AMS1117-3.3 regulator choices** — ordinary part selection, easily reversed. The buck copies TI's reference design in datasheet §8.2.4, Figure 38, exactly. One trap: pin 8 is soft-start on the TPS54336A but the frequency-setting resistor on the sibling TPS54335A, so Figure 22's 143 kΩ must never be copied onto it.
- **FlySky SBUS as the RC protocol** — folded into [ADR-0012](0012-split-band-radio-plan.md), where the band conflict it creates is the part that actually matters.
- **USB-C power path** — `VBUS` joins the 5 V rail through one Schottky, so the pack supply wins whenever both are present. Easily changed, but worth knowing: on USB power alone the AMS1117 input sits near 4.7 V, and a Wi-Fi transmit burst can pull the 3.3 V rail toward 3.1 V. Brownouts that happen only on USB point here, not at firmware.
- **GPIO allocation** — any digital peripheral can move through the S3's GPIO matrix, so pin choice carries no lock-in. Only the ADC channels, USB and the strapping pins are genuinely constrained.
