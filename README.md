# ⬡ Alvea

**Wireless split keyboard · nRF52840 · ZMK · hexagonal keycaps**

A low-power, battery-powered wireless split keyboard. Each half is a fully independent BLE device with its own battery, per-key LEDs, and USB-C charging.

![MCU](https://img.shields.io/badge/MCU-nRF52840-blue)
![Firmware](https://img.shields.io/badge/firmware-ZMK-7c4dff)
![Hardware License](https://img.shields.io/badge/hardware-CC_BY--NC_4.0-lightgrey)
![Firmware License](https://img.shields.io/badge/firmware-MIT-green)

> [!WARNING]
> **Work in progress — nothing here has been fabricated or tested yet.**
> The hardware is fully designed, but the boards haven't been manufactured, assembled, or brought up. Any performance numbers are design targets, not measurements, and the firmware doesn't exist yet. This repo is documentation of my hobby build, not a guide for replicating it.

---

## 🗺️ Roadmap

| Stage | Status |
|-------|--------|
| Schematic design | ✅ Done |
| PCB layout | ✅ Done |
| BOM finalized | ✅ Done |
| **PCB polishing + panelization** | 🔄 **In progress** |
| PCB fabrication (JLCPCB) | ⬜ Planned |
| SMT assembly | ⬜ Planned |
| Hand-soldering (sockets, LEDs, battery lead) | ⬜ Planned |
| Bootloader flashing (SWD, one-time) | ⬜ Planned |
| ZMK firmware (keymap + board definition) | ⬜ Not started |
| Bring-up & testing | ⬜ Planned |
| 3D-printed case | ⬜ Planned (after electronics work) |
| Renders / photos | ⬜ Not yet available |

**Right now:** finishing the PCBs and panelizing for fabrication.
**After that:** fabrication → SMT assembly → hand-soldering → firmware → testing → case design.

Legend: ✅ done · 🔄 in progress · ⬜ planned / not started

---

## ✨ Highlights

- **Wireless BLE** — each half is fully independent; no TRRS cable between halves
- **nRF52840 module (Ebyte E73-2G4M08S1C)** — integrated PCB antenna, so no RF design needed
- **ZMK firmware** *(planned)* — flexible keymaps, deep-sleep power management, drag-and-drop USB updates
- **19 functional keys per half** in a 5×4 matrix, plus a dedicated power switch
- **Per-key LEDs** shining up through translucent hexagonal HEX keycaps
- **USB-C charging** with a dedicated fuel gauge for accurate battery reporting
- **Single 3.5 Ah LiPo per half** with ZMK deep sleep — see [`DESIGN.md`](DESIGN.md) for the per-component power breakdown

Full technical detail — every component, net, pin assignment, BOM line, the power/charging architecture, and the design rationale — lives in [`DESIGN.md`](DESIGN.md).

---

## 🛒 Sourcing

| Source | Parts |
|--------|-------|
| [JLCPCB](https://jlcpcb.com/) | PCB fabrication + SMT assembly of the small components |
| [Keeb Supply](https://keeb.supply/) | Twilight low-profile switches, HEX keycaps, foam Choc spacers, hotswap Choc sockets, battery connector |
| [TME](https://www.tme.eu/) | Per-key LEDs, battery, and a Raspberry Pi Pico (firmware burner + SWD debugger) |

The switches are **not soldered** — only the hotswap Choc sockets are, and the switches snap into those.

---

## 📄 License

This repository is **split-licensed**:

- **Hardware design files & documentation** — [CC BY-NC 4.0](https://creativecommons.org/licenses/by-nc/4.0/). Use, share, and modify freely with credit, but **not for commercial purposes** — no one may sell Alvea or its derivatives without my explicit consent. See [`LICENSE`](LICENSE).
- **Firmware** (ZMK keymap / board definition) — [MIT](https://opensource.org/licenses/MIT), matching [ZMK](https://zmk.dev/)'s own license so it stays compatible with the ZMK ecosystem. See [`LICENSE-FIRMWARE`](LICENSE-FIRMWARE).

The non-commercial restriction protects the hardware — the part someone would actually manufacture. The firmware is a thin config on top of MIT-licensed ZMK, so it matches ZMK.

---

## 🙏 Acknowledgements

- [ZMK Firmware](https://zmk.dev/)
- [Adafruit nRF52 Bootloader](https://github.com/adafruit/Adafruit_nRF52_Bootloader)
- [fkcaps HEX keycaps](https://fkcaps.com/keycaps/hex) by *s-ol*
- The KiCad community for guidance
