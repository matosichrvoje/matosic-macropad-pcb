# Firmware pinout — v0.0.0.17

Derived from [matosic-macropad.kicad_pcb](matosic-macropad.kicad_pcb) on 2026-05-18 during the first board bring-up. Use this map when porting [code.py](https://github.com/matosichrvoje/matosic-macropad/blob/main/code.py) to XIAO RP2040.

## XIAO U1 pin map

The U1 footprint is `xiao:Seeeduino XIAO-MOUDLE14P-2.54-21X17.8MM`. Numbering matches the standard Seeed silkscreen — pin 1 at top-left, pin 14 at top-right.

| KiCad pad | XIAO silk | CircuitPython | Function |
|-----------|-----------|---------------|----------|
| 1  | D0  | `board.D0`  | LED enable (Q1 gate) — high turns all 9 LEDs on |
| 2  | D1  | `board.D1`  | ROW 3 — SW7/SW8/SW9 common |
| 3  | D2  | `board.D2`  | ROW 2 — SW4/SW5/SW6 common |
| 4  | D3  | `board.D3`  | ROW 1 — SW1/SW2/SW3 common |
| 5  | D4  | `board.D4`  | COL 1 — outputs of diodes D1/D4/D7 |
| 6  | D5  | `board.D5`  | COL 2 — outputs of diodes D2/D5/D8 |
| 7  | D6  | `board.D6`  | COL 3 — outputs of diodes D3/D6/D9 |
| 8  | D7  | `board.D7`  | Encoder phase B |
| 9  | D8  | `board.D8`  | Encoder phase A |
| 10 | D9  | `board.D9`  | Encoder push-switch |
| 11 | D10 | `board.D10` | Side-connector signal (R10 → J1 pad 2) |
| 12 | 3V3 | —           | Power |
| 13 | GND | —           | Power |
| 14 | 5V  | —           | Power (LED rail through Q1) |

## Key matrix

3×3 with per-key anti-ghosting diodes (D1–D9, 1N4148 SOD-123). Scan by driving each row LOW in turn and reading the column inputs — a pressed key pulls its column LOW through the diode.

|           | COL1 (D4)  | COL2 (D5)  | COL3 (D6)  |
|-----------|------------|------------|------------|
| ROW1 (D3) | SW1 / L1   | SW2 / L2   | SW3 / L3   |
| ROW2 (D2) | SW4 / L4   | SW5 / L5   | SW6 / L6   |
| ROW3 (D1) | SW7 / L7   | SW8 / L8   | SW9 / L9   |

`Lx` is the per-key indicator LED, not part of the switch wiring.

## LEDs

All 9 SMD LEDs (L1–L9) are in parallel, each through its own 68 Ω resistor (R1–R9). Common anode side ties to Q1's drain; Q1's source is +5V; gate is `board.D0`. Drive D0 HIGH → all 9 on, LOW → all off. No per-key control on this revision.

## Encoder (E1)

EC11 style. A = `board.D8`, B = `board.D7`, push-switch = `board.D9`. Common pin grounded. Use CircuitPython's `rotaryio` for the A/B phases and treat the push-switch as a 10th key.

## CircuitPython skeleton

```python
import time, board, digitalio, usb_hid
from adafruit_hid.keyboard import Keyboard
from adafruit_hid.keycode import Keycode

ROWS = [board.D3, board.D2, board.D1]   # ROW1, ROW2, ROW3
COLS = [board.D4, board.D5, board.D6]   # COL1, COL2, COL3
LED_ENABLE = board.D0

row_pins = []
for p in ROWS:
    pin = digitalio.DigitalInOut(p)
    pin.switch_to_output(value=True)    # idle high; pull low to scan
    row_pins.append(pin)

col_pins = []
for p in COLS:
    pin = digitalio.DigitalInOut(p)
    pin.switch_to_input(pull=digitalio.Pull.UP)
    col_pins.append(pin)

led = digitalio.DigitalInOut(LED_ENABLE)
led.switch_to_output(value=True)         # backlight on

kbd = Keyboard(usb_hid.devices)

# KEYMAP[row][col] -> Keycode or tuple of Keycodes for combos.
KEYMAP = [
    [(Keycode.CONTROL, Keycode.SHIFT, Keycode.COMMAND, Keycode.FOUR),  # SW1: screenshot region
     Keycode.X, Keycode.C],
    [Keycode.A, Keycode.S, Keycode.D],
    [Keycode.Q, Keycode.W, Keycode.E],
]
states = [[False]*3 for _ in range(3)]

while True:
    for r, row in enumerate(row_pins):
        row.value = False
        for c, col in enumerate(col_pins):
            pressed = not col.value
            if pressed and not states[r][c]:
                k = KEYMAP[r][c]
                kbd.press(*k) if isinstance(k, tuple) else kbd.press(k)
                states[r][c] = True
            elif not pressed and states[r][c]:
                k = KEYMAP[r][c]
                kbd.release(*k) if isinstance(k, tuple) else kbd.release(k)
                states[r][c] = False
        row.value = True
    time.sleep(0.005)
```

## Bring-up notes (2026-05-18)

- Soldered XIAO via castellated flush mount. Initial joints were balled — insufficient flux + dwell time. Cleaned up with a flux pen, longer iron-on-pad before feeding solder, and less solder per joint. Final joints are taller than ideal but pass continuity and power-up.
- Front-side SMD parts (diodes, resistors, LEDs, Q1) arrived pre-populated from JLCPCB PCBA.
- CircuitPython 10.2.1 boots cleanly; CIRCUITPY mounts over USB.
- A "discovery" firmware that polled D0–D10 as plain pull-up inputs did **not** detect switch presses. Root cause: the design uses a matrix, not pin-per-switch. With rows left floating-high as inputs, no GPIO is connected to ground when a key is shorted — both ends of the switch see the same potential. Must drive rows LOW and read columns.
- Side-connector net `R10 → J1-Pad2` runs to `board.D10`. The other expansion headers (J2/J3/J4) carry +5V/GND only — they're power taps, not signal extensions.
