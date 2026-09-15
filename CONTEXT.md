# VESPASIAN Electronics

The flight electronics for the VESPASIAN quadrotor: one carrier board that is simultaneously the flight controller, the power distribution board, and the radio hub. This glossary fixes the words the electronics, firmware and mechanical sub-teams use in common, so that a term means one thing in a schematic review, a commit message and a CAD file alike.

Design decisions live in [docs/adr/](docs/adr/README.md), not here. This file is a glossary and nothing else.

## Language

### The aircraft

**Boom**:
A structural limb running from the centre plate to a motor.
_Avoid_: Arm — reserved for the verb, see **Arm**.

**Nose**:
The forward reference direction of the airframe. Every sensor orientation, heading and mixer sign is expressed relative to it.
_Avoid_: Front, head, forward edge.

**Centre plate**:
The flat structural panel the carrier board and ESCs mount to.
_Avoid_: Base plate, main plate, chassis.

**All-up weight**:
The mass of the complete aircraft as flown, including pack and payload.
_Avoid_: Takeoff weight, gross weight. Abbreviate as AUW after first use.

### Arming and safety

**Arm**:
To place the aircraft in the state where motor commands are honoured. Always a verb about state, never a part of the airframe.
_Avoid_: Enable, activate, go live.

**Disarm**:
To leave the armed state deliberately, by pilot command or firmware decision.
_Avoid_: Kill, cut, shut down.

**Interlock**:
The hardware gate that prevents motor signals reaching the ESCs regardless of what firmware does. Distinct from disarming, which is a firmware state.
_Avoid_: Safety switch, kill switch, arming switch.

**Failsafe**:
The behaviour firmware falls back to when the control link is lost.
_Avoid_: Fallback, emergency mode, panic.

### The three radio links

Never say "telemetry" or "the link" unqualified — this aircraft carries three radios and they fail in different ways.

**Control link**:
The pilot-to-aircraft radio carrying stick commands.
_Avoid_: RC link, receiver link, radio.

**Telemetry link**:
The aircraft-to-ground radio carrying status down to the ground station.
_Avoid_: Downlink, GCS link, LoRa link.

**Swarm link**:
The aircraft-to-aircraft radio carrying position and state between members of a swarm.
_Avoid_: Mesh, peer link, inter-drone link.

**Ground station**:
The operator's laptop and its software, at the other end of the telemetry link.
_Avoid_: GCS on first use, base station, controller — "controller" means the flight controller.

### The board

**Carrier board**:
The physical PCB. Used when the subject is copper, parts or mechanical fit.
_Avoid_: Mainboard, FC board, PDB — the board is more than its power distribution.

**Flight controller**:
The board and its firmware together, as the system that flies the aircraft. Used when the subject is behaviour rather than hardware.
_Avoid_: Autopilot, controller alone.

**Power domain**:
The region of the board carrying pack current, and everything in it.
_Avoid_: Dirty side, high side — "high side" already means something specific in current sensing.

**Digital domain**:
The region of the board carrying only logic and sensor signals.
_Avoid_: Clean side, quiet side, logic side.

**Rail**:
A regulated supply voltage distributed across the board. Named by its voltage.
_Avoid_: Bus, supply, line.

### Mounting

**Soft mount**:
The compliant stage between the carrier board and the centre plate that attenuates airframe vibration.
_Avoid_: Isolation, damping, anti-vib — see **Galvanic isolation** for why "isolation" alone is ambiguous here.

**Galvanic isolation**:
The absence of a conductive path between two circuits. An electrical property only; never used about vibration.
_Avoid_: Isolation alone.

**Mast**:
The standoff that lifts the GPS and compass above the airframe, away from power wiring and structure.
_Avoid_: Pole, tower, stalk.

**Cover**:
The removable enclosure over the carrier board. It attaches to the centre plate, not to the board.
_Avoid_: Case, lid, canopy, shell.

### Rates

Four distinct rates get called "the rate" in conversation. Name them.

**Loop rate**:
How often the flight controller completes one full sense-and-command cycle.
_Avoid_: Update rate, PID rate, refresh rate.

**Gyro sample rate**:
How often angular rate is read from the sensor. Bounds the loop rate but is not the same number.
_Avoid_: ODR, sensor rate, sampling frequency.

**Motor update rate**:
How often a fresh command is delivered to the ESCs. Bounded by what the ESCs accept, not by what firmware can produce.
_Avoid_: PWM rate, ESC rate, output rate.

**Telemetry rate**:
How often a status message is sent down the telemetry link or across the swarm link. Always say which.
_Avoid_: Report rate, update rate.

### Operating conditions

**Hover current**:
Pack current with the aircraft holding altitude at its design weight. The condition that governs endurance and steady-state heat.
_Avoid_: Nominal current, cruise current.

**Burst current**:
Pack current with all four motors at full throttle. The condition that governs conductor sizing, connector choice and pack selection — never sustained.
_Avoid_: Peak current, max current, stall current.

### Sensing

**Compass**:
The device and its reading that provide magnetic heading. Refers to the function regardless of which chip implements it.
_Avoid_: Magnetometer, mag — use these only when the subject is a specific part.

**Fix**:
A GNSS position solution of usable quality. "No fix" is a distinct state from "no GPS".
_Avoid_: Lock, satellite lock, position.

**Blackbox**:
The onboard recording of flight data for post-flight analysis.
_Avoid_: Log, logger, flight recorder — "log" alone also means firmware console output.
