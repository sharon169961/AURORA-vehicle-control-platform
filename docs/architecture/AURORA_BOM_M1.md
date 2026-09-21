# AURORA Rev A — M1 Part Selection Lock

Every part below was verified live against TI/manufacturer data during this session.
Passive values (resistors, caps, inductors, MOSFETs, shunts) are NOT locked here —
those are calculated in M2 (power) and M5 (motor) from real design equations, not
picked in advance.

| Block | Part | Package | Key specs | Why |
|---|---|---|---|---|
| MCU | **TI MSPM0G3507SPMR** | 64-LQFP (PM) | 80MHz Cortex-M0+, 128KB flash, 32KB SRAM, 2×12-bit 4Msps ADC, 12-bit DAC, 2× zero-drift op-amp, 3× comparator, CAN-FD, hardware math accelerator | Largest package = easiest hand-solder, most GPIO/ADC breakout, native CAN-FD and analog peripherals eliminate external op-amp/ADC parts |
| CAN transceiver | **TI TCAN1042DR** | SOIC-8 | 5 Mbps, ±70V bus fault protection, 16kV HBM ESD | Catalog grade (not -Q1 automotive) — cheaper, easier to source in small qty, still robust |
| Buck stage 1 (36V→12V) | **TI LM5164DDAR** | SOIC-8 PowerPAD | 6V–100V in, 1A sync buck, constant on-time, ultra-low IQ, adjustable current limit, PGOOD, UVLO | Input range covers full 18–36V bus with margin; PowerPAD gives thermal path without needing a 4-layer thermal via array right under the IC |
| Buck stage 2 (12V→5V) | **TI TPS54331DR** | SOIC-8 | 3.5V–28V in, 3A, 570kHz current-mode, Eco-mode | 12V input is well inside its 28V max; current-mode control is simpler to compensate than COT for a first design |
| 3.3V rail | TBD in M2 | — | — | Likely a small LDO or third buck off the 5V rail — sized once the 3.3V load budget (MCU + logic) is totaled |
| Motor gate driver | **TI DRV8323SRTAR** | 40-WQFN | 6V–60V, 3× half-bridge (6 ch), integrated 3× current-shunt amps (5/10/20/40 V/V gain), SPI config, dead-time control, OTP/OVP/OCP | SPI variant chosen over PWM-only H variant so the MCU can digitally configure shunt gain and read fault status — this is the "embedded depth" the project needs |

## Deliberately deferred to later modules
- 3.3V regulator part (M2, after power budget is totaled)
- Reverse-polarity FET, TVS, fuse (M2 — sized to real fault-current calc)
- Op-amp for analog front end — **using the MSPM0's internal zero-drift op-amps** instead of an external part; simplifies BOM and is a stronger "we understood the part" story
- BLDC power MOSFETs and current-shunt resistor values (M5 — sized to real motor current)
- IMU / pressure sensor exact part numbers (M7, non-critical path)
- ESP32/LoRa module part number (M7)

## Rev A scope decision
Three-board split (Control/Power/Sensor) rejected — a single well-executed 4-layer
board is more credible in a one-day build than three partially-finished boards.
