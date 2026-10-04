# AURORA — Autonomous Vehicle Control & Power Platform

**4-Layer Mixed-Signal PCB | TI MSPM0G3507 | DRV8323S BLDC FOC | CAN-FD | Power Management**

> Status: **Rev A** — schematic design complete, PCB layout in progress. Built in KiCad 10.

---

## Overview

AURORA is the electronics core for an autonomous ground/underwater vehicle — a single 4-layer PCB integrating power conversion, motor control, sensing, and vehicle networking. The board takes a wide-range 18–36V input and regulates it down through three cascaded stages to power an 80MHz Arm Cortex-M0+ MCU, a 3-phase BLDC gate driver with integrated current sensing, a CAN-FD vehicle network node, and a protected analog sensor front end.

This project was designed module-by-module with every component value backed by a real datasheet calculation — no placeholder values, no guessed passives.

## System Architecture

```
18–36V INPUT
      │
 [Reverse-Polarity + Transient Protection]   (LM74701-Q1, TVS-less ideal diode)
      │
      ▼
 [36V → 12V Buck]  (LM5164, synchronous, 300kHz)
      │
      ▼
 [12V → 5V Buck]   (TPS5430, 500kHz)
      │
      ▼
 [5V → 3.3V LDO]   (TLV75733)
      │
      ├──────────────┬──────────────┬──────────────┐
      ▼              ▼              ▼              ▼
   MCU CORE      ANALOG FE      CAN-FD NODE     MOTOR STAGE
 MSPM0G3507    3-ch protected   TCAN1043      DRV8323S +
 80MHz M0+     RC anti-alias   transceiver    3-phase gate
 CAN/SPI/I2C   ADC front end    120Ω term.    drive + Kelvin-
 /UART hub                                    sense shunts
```

## Key Design Decisions

- **Power tree sized from real calculations** — inductor values, feedback dividers, UVLO thresholds, and compensation networks are all calculated from TI's own design equations, not copied defaults
- **DRV8323S current sensing uses 4-pin Kelvin-sense shunts** (5mΩ, 1%) rather than 2-terminal resistors — eliminates trace/joint resistance error on the high-gain (up to 40V/V) current-sense path
- **Hardware-level motor kill switch** — a physical E-stop directly shorts the gate driver's ENABLE pin to ground, overriding any MCU/firmware state
- **TVS-less input protection** — LM74701-Q1's integrated VDS clamp meets automotive transient requirements without a discrete TVS diode
- **Reverse-polarity protection via ideal diode controller**, not a passive series diode — avoids the forward-voltage power loss of a traditional diode-OR input stage

## Tech Stack

| Domain | Parts |
|---|---|
| MCU | TI MSPM0G3507 (Arm Cortex-M0+, 80MHz, 128KB/32KB, CAN-FD, dual 12-bit ADC) |
| Motor control | TI DRV8323S (SPI-configured 3-phase gate driver, integrated current-shunt amps) |
| Power | TI LM5164, TPS5430, TLV75733, LM74701-Q1 |
| Networking | TI TCAN1043 (CAN-FD transceiver) |
| Tools | KiCad 10, Git |

## Repository Structure

```
hardware/aurora-ctrl/    KiCad project — schematic + PCB
hardware/lib/            Custom symbol/footprint libraries
hardware/outputs/        Gerbers, BOM, renders (post-fabrication)
docs/                    Architecture notes, calculations, design rationale
calcs/                   Component value derivations
```

## Status / Roadmap

- [x] Architecture + full component selection, every part datasheet-verified
- [x] Power tree (4 stages) — schematic complete, ERC clean
- [x] MCU core sheet — power, SWD, CAN, I2C/SPI/UART
- [x] Precision analog front end (3-channel, protected, anti-aliased)
- [x] 3-phase motor drive stage (DRV8323S, Kelvin-sense shunts)
- [x] CAN-FD network node
- [x] Hardware E-stop + fault indication
- [x] Whole-project schematic ERC clean, 6 sheets fully cross-wired
- [ ] PCB layout (4-layer, zoned power/analog/digital/motor)
- [ ] Gerber generation + fab
- [ ] Bring-up and validation (power rail testing, motor spin-up)

**Deferred to Rev B:** centralized PGOOD/fault-aggregation sheet, internal op-amp buffering on the analog front end (currently passive RC filtering), CAN sleep/wake modes.

---

*Designed as a demonstration of TI's embedded processing, power management, and motor control portfolio working together on a single board.*
