 24-Hour Digital Clock — Pure Discrete Logic

**A 2-day hackathon project by Team Zero Hour.**

## Team Zero Hour

- [Ponmudi M R](https://github.com/ponmudimr)
- [Renga](https://github.com/Renga37)
- [Amrin Hidha](https://github.com/amrinhidha)

A full 24-hour digital clock (HH:MM:SS on six 7-segment displays) built entirely from discrete logic — one IC 555 timer and six CD4026 decade counter/display drivers. No microcontroller, no firmware, no BCD decoders. Every rollover is decoded straight off the segment lines.

## How It Works

### Timebase — 555 astable (~1 Hz)

The clock source is a 555 timer in astable mode:

| Component | Value |
|-----------|-------|
| R1        | 10 kΩ |
| R2        | 68 kΩ |
| C         | 10 µF |

```
f = 1.44 / ((R1 + 2·R2) · C)
  = 1.44 / ((10k + 136k) · 10µF)
  ≈ 0.99 Hz
```

This ~1 Hz square wave clocks the SECONDS-1 counter; each counter's carry/decoded reset cascades into the next stage.

### Counting chain

Six CD4026 ICs drive six common-cathode 7-segment displays directly:

```
555 (1 Hz) → SEC-1 → SEC-10 → MIN-1 → MIN-10 → HOUR-1 → HOUR-10
             (0-9)   (0-5)    (0-9)   (0-5)    (0-9)    (0-2)
```

### Rollover via segment-pattern decoding

The CD4026 has **no BCD outputs** — only 7-segment drive lines. So every rollover is detected by recognizing the segment pattern of the first "illegal" digit and firing RESET.

**Seconds-10 / Minutes-10 (reset at 6):**

```
RESET = (NOT b) AND e
```

Digit **6** is the only digit in range where segment *b* is off and segment *e* is on. The seemingly simpler `b̄·c` was **rejected** because it also matches digit **5** — the tens digit would never get past 4.

**Hours (reset at 24):**

A 3-input AND (74HC11) combines:

- **HG** — segment *g* of HOUR-10, which turns on at digit **2** (g is off for 0 and 1)
- **(NOT a) · f · g** on HOUR-1 — the unique signature of digit **4**

When HOUR-10 shows 2 and HOUR-1 hits 4 (i.e., 24:00:00), both hour counters reset and the clock rolls to 00:00:00.

## Build Log

### Day 1 — Schematic (Altium)

- Drew the full schematic in Altium Designer.
- Debugged the 555 astable wiring (rewired the threshold/trigger/discharge network).
- Caught and corrected an error in the hour-rollover decode logic.

### Day 2 — Breadboard build

Built and debugged the physical clock. Hard-won lessons:

- **Series 330 Ω resistors on every segment line are mandatory** — not just for LED current limiting. Without them, LED forward-voltage clamping pulled the decode lines below the CMOS logic threshold and the reset detection never fired.
- **Decode taps must be on the IC side of the segment resistor**, so the gates see full CMOS levels instead of the LED-clamped voltage.
- Gate package power: **pin 14 = VCC, pin 7 = GND** on the 74HC04/08/11 — easy to forget on a breadboard.
- **All unused CMOS inputs tied to GND.** Floating CMOS inputs oscillate and cause phantom resets.

## Hardware

| Qty | Part | Role |
|-----|------|------|
| 1 | NE555 | 1 Hz astable timebase |
| 6 | CD4026 | Decade counter + 7-segment driver |
| 6 | Common-cathode 7-segment display | HH:MM:SS digits |
| 1 | 74HC04 | Hex inverter (NOT for decode terms) |
| 1 | 74HC08 | Quad 2-input AND (seconds/minutes reset) |
| 1 | 74HC11 | Triple 3-input AND (hour reset) |
| — | Resistors (330 Ω segment series, 10 k / 68 k timing) | |
| — | Capacitors (10 µF timing, decoupling) | |

## Repository Layout

```
.
├── README.md
├── LICENSE
├── datasheets/                          # Component datasheets (PDF)
│   ├── CD4026-Datasheet.pdf
│   ├── SN74LS11-Datasheet.pdf
│   └── Seven-Segment-Display-Datasheet.pdf
└── schematic/                           # Altium Designer project
    ├── Digital-Clock-Schematic.PrjPcb   # Altium project file
    ├── Schematic-1.SchDoc               # Schematic sheet 1
    ├── Schematic-2.SchDoc               # Schematic sheet 2
    ├── Digital-Clock-Schematic.OutJob   # Output job
    └── Digital-Clock-Schematic.pdf      # Rendered schematic (view this)
```

Open `schematic/Digital-Clock-Schematic.PrjPcb` in Altium Designer, or just view `schematic/Digital-Clock-Schematic.pdf`.

## Datasheets

- [CD4026 — Decade counter with 7-segment display driver](datasheets/CD4026-Datasheet.pdf)
- [SN74LS11 — Triple 3-input AND gate](datasheets/SN74LS11-Datasheet.pdf)
- [Common-cathode 7-segment display](datasheets/Seven-Segment-Display-Datasheet.pdf)

## License

[MIT](LICENSE)
