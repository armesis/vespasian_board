---
status: accepted
date: 2026-09-03
amended: 2026-09-15
supersedes: an earlier draft specifying ACS758LCB-100U
---

# Current sensing by shunt and INA180, not a hall sensor

Pack current is measured with a 0.5 mΩ 2512 shunt in the positive rail and an `INA180A2` (gain 50). This replaces an `ACS758LCB-100U`, which cost about $10 against roughly $0.90 for the shunt and amplifier together.

Cost drove the change, but the analog output is what made it the right part rather than merely the cheap one: it drops into the ADC path already designed, so unlike an `INA226` it needs no new driver — only a scaling constant. Running the amplifier from **3.3 V rather than 5 V** means its output physically cannot overdrive the ADC, so the divider and clamp the hall sensor needed both disappear. 0.5 mΩ × 60 A × 50 = 1.5 V, in the middle of the ESP32's linear range, rising to 1.65 V at the 66 A design ceiling. The scaling constant firmware needs is 40 A per volt.

## Considered options

- `INA226` + shunt (~$2.20) — 16-bit and I2C, but another driver to write.
- `CC6920BSO-50A` (~$0.60) — hall, SOIC-8, but a 50 A ceiling under a 60 A burst.
- `ACS712-30A` — 30 A. Not enough.
- A PCB trace as the shunt — free, but copper drifts about 0.39 %/°C, so the reading changes as the board warms.

## Consequences

- **High-side, not low-side.** A shunt in the ground return would break the return path and inject noise straight into the reference plane.
- **The sense traces must Kelvin-connect** to the shunt's own pads as a tight differential pair. Tapping the surrounding pour means measuring our own copper alongside the shunt, which is the same tempco problem that ruled out a trace shunt. Correct and incorrect tap placement are drawn in [kelvin-sense-reference.png](../kelvin-sense-reference.png), with its KiCad source alongside.
- **The board is in-line, so everything downstream of the shunt is one node** — `VPACK` in the schematic: the four ESC connectors, the buck converter's input, and the bulk and ceramic capacitors. The buck taps the load side, so the flight controller's own draw is counted in pack current.
- **The bulk capacitors belong on the load side of the shunt, never the pack side.** There they supply the ESCs' switching pulses locally and the shunt sees near-DC. On the pack side every pulse crosses the shunt, and the INA180 output carries the full switching ripple. Rev.0.0 has two 470 µF electrolytics and a 4.7 µF ceramic at each ESC connector, all on `VPACK`.
- **Galvanic isolation is lost.** The INA180 inputs sit at pack potential — within its 26 V common-mode rating, which is independent of supply, but the part and its traces stay clear of the solder-flooded pour.
- **The shunt is a `CSS2H-2512R-L500F`: 0.5 mΩ, ±1 %, 6 W, TCR ±100 ppm/°C.** Sized against a 66 A ceiling — the 60 A burst plus 10 % — it dissipates 2.18 W, past what a 2 W 2512 can be argued into on burst duration alone. Hover is 0.2 W. The 6 W rating leaves about 3× margin, but it is quoted at 70 °C *terminal* temperature, so it assumes the pours genuinely pull heat out of the pads rather than merely touching them.
- **Two rejected shunts, recorded so they are not proposed again.** A `WSK2512` at 0.5 mΩ is 4-terminal, which is the right feature, but it is rated 1.0 W — 2.2× under the burst — and its TCR across 0.5 mΩ to 0.99 mΩ is ±350 ppm/°C rather than the ±20 ppm/°C on its features page, because at that value the copper terminals are 2.66 mm of a 6.35 mm part. Two 1 mΩ parts in parallel were the other candidate: they halve the per-element heat, but the split between them is set by each branch's feeding copper, so 50 μΩ of asymmetric pour misreads total current by 5 % — the trace-shunt tempco problem re-entering through the layout. Both lose to a single 6 W part.
- **To buy the Kelvin connection instead of laying it out**, a `CSS4J-4026R-L500F` is 4-terminal, 0.5 mΩ and 5 W in a larger 4026 footprint. Worth the swap if the sense-tap layout below proves awkward.
- **Neither ceiling binds at 0.5 mΩ, so the value is a choice rather than a limit.** The output clips at 131 A and leaves the window where gain error is specified at 112 A, so 66 A uses under half the range; output swing would allow 0.85 mΩ and a 6 W shunt would allow 1.38 mΩ. 0.5 mΩ is held deliberately, to keep burst dissipation at 2.18 W instead of the 3.27 W that 0.75 mΩ would put into a board already carrying four ESC paths. The price is paid at the low end, in the bullet below.
- **Hover sits on the edge of the specified accuracy window.** Gain error is specified only for V<sub>OUT</sub> between 0.5 V and V<sub>S</sub> − 0.5 V, and 20 A of hover current lands exactly on 0.5 V. The part still reads below that, but the ±500 μV input offset — the figure that applies at a 3S pack's common mode, not the ±150 μV quoted at V<sub>CM</sub> = 0 — is ±1 A referred to current: 1.5 % at burst, 5 % at hover, where it dominates every other error term. Null it at zero current on the ground before trusting a hover-current reading for endurance.
- Gain variant matters: `A2` = 50, which the 1.5 V figure assumes. An `A1` (gain 20) would read 0.6 V at 60 A and lose resolution.
