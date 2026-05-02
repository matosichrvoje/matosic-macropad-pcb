# Matosic Macropad

Private development repo for the Matosic mechanical macropad PCB — a fork of [AnaviTechnology/anavi-macro-pad-10](https://github.com/AnaviTechnology/anavi-macro-pad-10).

## Status

🔒 **Private during development.** This repo will be made public when the design is ready to ship and at that point will fully comply with CC-BY-SA-4.0. Until then, the upstream license obligations don't apply because nothing has been distributed.

## Hardware

- 9-key Cherry MX + clickable rotary encoder (subset to be populated for v1)
- **Seeeduino XIAO module socket** — supports any XIAO variant (XIAO RP2040 recommended); USB-C and microcontroller live on the XIAO, not on this PCB
- Kailh hot-swap sockets for switches
- 9× per-key 1206 SMD LED backlight (single-color, driven via SOT-23 NPN — not WS2812B)
- 4× 1×3 expansion headers for external LED strips or other I/O
- Designed in KiCad 10 (migrated from upstream KiCad 5 source)

## Companion firmware

CircuitPython firmware: https://github.com/matosichrvoje/matosic-macropad

The firmware was originally written for a 6-key Pi Pico prototype and is being adapted to the Anavi 10 RP2040 pinout.

## Credits

Lead: **Hrvoje Matosic**
Contributors: Jorge Ochoa, Iván Aceves, Iván Estevez
Based on **ANAVI Macro Pad 10** by [anavi.technology](https://anavi.technology) — CC-BY-SA-4.0.

## Files

- `matosic-macropad.kicad_pro`, `matosic-macropad.kicad_pcb`, `matosic-macropad.sch` — KiCad project
- `components/` — bundled footprint/symbol libraries (keyswitches, LEDs, XIAO, logo)
- `case/` — OpenSCAD case sources + STL/STEP exports

**Open source programmable keypad with 9 mechanical switches and a rotary encoder.**
