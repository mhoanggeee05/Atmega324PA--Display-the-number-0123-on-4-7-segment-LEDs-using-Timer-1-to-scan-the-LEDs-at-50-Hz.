# 7-Segment Display Counter – ATmega324PA (Assembly)

A multiplexed 4-digit 7-segment display driver written in AVR Assembly for the ATmega324PA. The program cycles through 4 digits displaying a static countdown (3, 2, 1, 0) using a latch-based multiplexing technique, with Timer1 in CTC mode providing the timing for each digit's display window.

---

## Core Idea

Driving 4 digits simultaneously would require 4 × 8 = 32 pins. Instead, this program uses **multiplexing** — only one digit is active at a time, switching between them fast enough that the human eye perceives all 4 as lit continuously.

Each digit is driven through two shared buses:

- **PORTA** — carries the 7-segment data (which segments to light up)
- **PORTC (PC0, PC1)** — controls two latches (LE0, LE1) that select which digit receives the data
```
PORTA (segment data) ──→ [Latch] ──→ Digit 0
                          [Latch] ──→ Digit 1
PORTC (LE0/LE1) ─────→ latch select
```

For each digit, the program pulses LE1 to latch the select pattern, writes the segment data, then pulses LE0 to latch the data — before moving on to the next digit.

---

## Features

-  **4-digit multiplexed display** — one shared data bus, latch-controlled digit selection
-  **Timer1 CTC mode** — ~5 ms delay per digit using hardware timer (OCR1A = 38, clk/1024)
-  **Segment lookup table** — supports digits 0–F stored in flash via `.DB`
-  **Continuous loop** — cycles 3 → 2 → 1 → 0 repeatedly

---

## How It Works

1. PORTA is configured as output for segment data; PC0 and PC1 as outputs for latch control.
2. Timer1 is set to CTC mode with a prescaler of 1024 and OCR1A = 38, producing ~5 ms per tick at 8 MHz.
3. The main loop calls `display_7seg` for each digit in sequence (3, 2, 1, 0):
   - Blanks PORTA first to prevent ghosting between digit switches
   - Pulses **LE1** to latch the digit select pattern from `select_led7`
   - Writes segment data from `data_led7` lookup table to PORTA
   - Pulses **LE0** to latch the segment data into the active digit
4. `delay_ms` polls the OCF1A flag and blocks until Timer1 fires, then clears the flag for the next cycle.

---

## Lookup Tables

| Table | Purpose |
|---|---|
| `data_led7` | Segment patterns for digits 0–F (active low, e.g. `0xC0` = '0') |
| `select_led7` | Digit select patterns for 4 digits (`0b00001110` = digit 0, etc.) |
