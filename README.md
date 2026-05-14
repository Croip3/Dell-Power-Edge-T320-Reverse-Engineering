# Dell-Power-Edge-T320-Reverse-Engineering
The goal is to reuse a Dell Power Edge T320 Case with the original power supple and power distribution as homeserver.

# Dell PowerEdge T320 / AC-108 PSU Reverse Engineering Report

## Hardware

System:
- Dell PowerEdge T320

Power subsystem:
- 2 redundant Dell hot-swap PSUs
- AC-108 Power Distribution Board (PDB)
- Dell motherboard originally expected
- PDB includes:
  - main power connectors
  - thin “data/control” harness
  - `PDB_I2C/P6` connector

---

# Objective

Goal:
- Use the Dell redundant PSU system independently from the original motherboard
- Understand startup logic and pinout
- Potentially adapt system to power a standard ATX motherboard

---

# Initial Findings

## PSU behavior

Observed:
- Plugging PSUs into 230V produced only standby power
- `12V_SB` rail became active
- Main power rails remained OFF

Result:
- PSUs enter standby correctly
- Main power stage is not enabled automatically

---

# Connector / Signal Investigation

## Presence pin

Observation:
- A `PRESENT` pin exists on the AC-108 PDB

Measured behavior:
- `PRESENT` idles at `0V`

Interpretation:
- PDB already considers PSU presence valid
- Presence detection is not the blocking issue

---

## PS_ON investigation

Observation:
- A signal labeled `PS_ON` exists
- `PS_ON` also idles at `0V`

Attempt:
- Bridged `PS_ON` to GND

Result:
- No change
- Main rails still OFF

Interpretation:
- `PS_ON` is likely:
  - not the real enable signal
  - status-only
  - or controlled internally by the PDB

---

# Discovery of Actual PSU Enable

## Thin control/data connector investigation

Observation:
- Thin-wire harness (~20 wires)
- Different wire colors
- Some lines measured:
  - `0V`
  - `3.3V`

Experiment:
- Bridged yellow wire(s) to GND

Result:
- PSU main stage activated
- Main `12V` rails turned ON

Additional finding:
- Required grounding yellow twice
- Likely one enable line per PSU module

Interpretation:
- Yellow wires are likely:
  - `ENABLE#`
  - remote-on
  - PSU startup request lines

Important:
- Real PSU enable was found on the control/data connector
- Not on the labeled `PS_ON` signal

---

# Main Rail Results

After successful PSU enable:

Available:
- `12V_SB`
- Main `12V` rails

Missing:
- `5V`
- `3.3V`

---

# AC-108 Rail Investigation

Observation:
- AC-108 PCB explicitly labels:
  - `5V`
  - `3.3V`
- Color-coded outputs exist for those rails

Interpretation:
- AC-108 likely contains onboard DC-DC converters
- System is NOT purely 12V-only

Likely architecture:

```text
PSU1 + PSU2
↓
12V bulk
↓
AC-108
    - OR-ing / redundancy logic
    - DC-DC converters
↓
Outputs:
    - 12V
    - 5V
    - 3.3V
```

---

# Reddit Discovery / Confirmation

Found Reddit post describing similar issue.

Critical information discovered:
- AC-108 uses:
  - `PDB_I2C/P6`
- Connection required for full normal startup
- P6 identified in Dell documentation as:
  - `PDB_I2C/P6`
  - “Power distribution board connector”

Interpretation:
- Motherboard and AC-108 communicate digitally
- Likely SMBus/I²C-based power sequencing

---

# Current Architecture Understanding

Most likely startup flow:

```text
PSUs connected
↓
12V standby active
↓
Yellow ENABLE# lines grounded
↓
Main PSU 12V bulk enabled
↓
Motherboard communicates with AC-108 via PDB_I2C/P6
↓
AC-108 enables:
    - 5V converters
    - 3.3V converters
↓
POWER_OK asserted
↓
Full system startup
```

---

# Current Conclusions

## Confirmed

- [x] PSUs functional
- [x] AC-108 functional at least partially
- [x] Main 12V bulk can be manually enabled
- [x] PSU enable occurs through thin control harness
- [x] `PS_ON` label is misleading or nonfunctional in isolation
- [x] `PRESENT` not blocking startup
- [x] AC-108 uses motherboard communication via I²C/SMBus
- [x] `5V` / `3.3V` rails are likely software/sequencing controlled

---

# Most Likely Remaining Blocker

Missing:
- I²C/SMBus handshake from motherboard over `PDB_I2C/P6`

Without this:
- AC-108 does not enable:
  - `5V` rail
  - `3.3V` rail

---

# Risks Identified

Potentially dangerous signals on thin control harness:
- `SMBCLK`
- `SMBDATA`
- current-share lines
- power-good/status lines

Risk:
- Random grounding may damage:
  - PSU controller
  - PDB microcontroller
  - logic circuitry

---

# Recommended Next Steps

## Option 1 — Use original Dell motherboard

Simplest approach.

Motherboard naturally performs:
- I²C handshake
- sequencing
- power-good logic

---

## Option 2 — Reverse engineer P6 I²C communication

Required tools:
- logic analyzer
- working Dell motherboard
- protocol capture

Possible emulation:
- Arduino
- RP2040
- ESP32

Complexity:
- High

---

## Option 3 — Bypass AC-108 secondary rails

Use existing `12V` output with:
- DC-ATX converter
- PicoPSU
- dedicated buck converters

Most practical for powering standard ATX systems.

---

# Current State Summary

| Rail | Status |
|---|---|
| `12V` standby | Working |
| Main `12V` | Working |
| `5V` | Not enabled |
| `3.3V` | Not enabled |

---

# Key Technical Insight

The Dell T320 redundant PSU system behaves as:
- an intelligent managed power subsystem
- not a standard ATX PSU

The AC-108 PDB performs:
- PSU control
- sequencing
- monitoring
- likely digital authentication/handshake
- secondary rail enable control via I²C/SMBus

---

# Power Distribution Board Data Cable Pinout (P6 Connector in the Manual)

| Comment | V/GND | Color(text) | Pin | Color || Color | Pin | Color(text) | VGND | Comment |
| --- | --- | ---: | ---: | ---: | - | :--- | :--- | :--- | --- | --- |
| PSU-2 (+12V) when pulled to GND | 3.3V | yellow | 1 | 🟨 || ⬜️ | 2 | white | 3.3V | |
|  | CONT-GND | black | 3 | ⬛️ || ⬜️ | 4 | white |  | |
|  | 0.17V | yellow | 5 | 🟨 || — | 6 | — |  | |
|  | 3.3V / CONT-GND | blue | 7 | 🟦 || 🟨 | 8 | yellow | 0.19V | |
|  |  | green | 9 | 🟩 || 🔲 | 10 | gray | 3.3V | |
| PSU-2 on when pulled to GND |  | red | 11 | 🟥 || 🟨 | 12 | yellow | 3.3V | |
| PSU1 ON signal? |  | purple | 13 | 🟪 || ⬜️ | 14 | white | 0.0–3.3V | Singal stuff going on, changing voltag periodically |
|  |  | purple | 15 | 🟪 || ⬜️ | 16 | white | 3.3V | |
|  | 3.3V / CONT-GND | gray | 17 | 🔲 || 🟦 | 18 | blue | 3.3V | |
|  |  | brown | 19 | 🟫 || 🟩 | 20 | green | 0V | |
| Signal → PSU5 off | GND | black | 21 | ⬛️ || 🟧 | 22 | orange | 0–3.3V / CONT-GND | Singal stuff going on, changing voltag periodically |
|  | CONT-GND | blue | 23 | 🟦 || 🟧 | 24 | orange |  | |
|  | 3.3V / CONT-GND | red | 25 | 🟥 || 🟧 | 26 | orange | CONT-GND | |
|  |  | red | 27 | 🟥 || ⬛️ | 28 | black | CONT-GND | |
|  | CONT-GND | red | 29 | 🟥 || 🟩 | 30 | green |  | |
|  |  | — | 31 | — || ⬜️ | 32 | white |  | |
|  |  | — | 33 | — || ⬜️ | 34 | white |  | |

PSU-1: middle position or left position when viewed from the rear

PSU-2: outer position or right position when viewed from the rear

CONT-GND: Continuity to GND
