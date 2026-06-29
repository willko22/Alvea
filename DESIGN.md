# Alvea — nRF52840 Wireless Split Keyboard Design Summary

## Overview

- **MCU module:** Ebyte **E73-2G4M08S1C** (U2, `C356849`) — pre-certified nRF52840 module with **integrated PCB antenna**, 32 MHz crystal, RF matching, and DC-DC inductors all internal. Chosen on KiCad-community advice to avoid hand-designing an RF/antenna section.
- **Type:** Wireless split keyboard — both halves are identical component-wise, each half fully independent.
- **Firmware:** ZMK
- **Bootloader:** Adafruit nRF52 UF2 bootloader (flashed via SWD once)
- **Power:** LiPo 3.5 Ah (3500 mAh) single cell
- **Key switches:** Ambient Silent Twilight (Choc low-profile, silent linear), HEX footprint
- **Keycaps:** fkcaps HEX — hexagonal, translucent ABS (RGB-friendly); requires HEX-footprint PCB
- **Keys:** 20 switches per half = **19 functional matrix keys + 1 dedicated power switch (S20, outside the matrix)**
- **Matrix:** 5 columns × 4 rows (19 of 20 positions populated with diodes D2–D20)
- **PCB:** Designed in KiCad, manufactured by JLCPCB
- **Assembly:** JLCPCB SMT for small components; hotswap sockets, per-key LEDs, and battery lead hand-soldered. Switches snap into the hotswap sockets (not soldered). Foam Choc spacers sit under the switches to lower typing volume.

---

## What Changed vs. Previous Revision (discrete-nRF design)

- **Bare nRF52840-QIAA → E73-2G4M08S1C module.** Removed from the board (now internal to module): 32 MHz HF crystal X1, RF matching L1/C3/C4, PCB trace antenna, DCDC inductors L2/L4, and the bulk of MCU decoupling.
- **Battery monitoring: resistor-divider ADC → dedicated fuel gauge.** Added **MAX17048 (U8, `C2682616`)** on the I2C bus with an alert line. Removed the old P0.29/AIN5 resistor divider (the previous revision's battery-sense pair) and all ZMK ADC-gain notes. *(Note: the R2/R3 designators are reused in this revision for unrelated parts — see BOM.)*
- **Added USB ESD protection: USBLC6-2SC6 (U9)** on D+/D−.
- **Power-circuit ICs renumbered & re-specified** (Schmitt, D-FF, charger, protection — see below).
- **LED driver unchanged in concept: still LP5024 over I²C** (not I2S — the hardware is I²C constant-current; "I2S" was a misnomer).
- **LDO and all discrete RF parts: dropped / absorbed into module.**

---

## PCB Architecture — Two Board Design

- **Module board:** E73 module, LP5024, MAX17048, BL8573 charger, DW01A protection, power-switch latch circuit, USB-C, USB ESD diode, battery header, all passives
- **Matrix board:** Key switches (DNP), hotswap sockets (DNP), diodes (JLCPCB populated), per-key LEDs (DNP, hand soldered)
- **Connection:** FPC flat flexible cable (pin count per layout)

---

## Power Architecture

- **VBUS_RAW:** Raw USB-C 5 V → polyfuse F1 → VBUS net
- **VBUS:** Feeds charger (U5) and module USB/VBUS-sense (U2 pad 27)
- **BAT+:** LiPo positive, switched via P-MOS Q1
- **PWR net:** Switched BAT+ output (drain of Q1) — powers the module (Vin), LP5024 VCC, and all LED anodes
- **VDD_NRF:** Regulated rail **out of the module** (module pad 19); used as the local logic reference / I²C pull-up rail
- **GND:** BAT− to GND plane always

---

## Power Consumption (per half)

All figures are typical design-reference values, **not measured** — nothing has been fabricated or profiled yet. Actual draw depends on ZMK configuration, BLE connection interval, and LED usage.

### Battery
- **Cell:** single LiPo, 3500 mAh (3.5 Ah) nominal, ~3.7 V

### Per-LED
- **Per LED:** ~1 mA (OSO5PAS1C1A-1MA, driven by LP5024 constant-current sink)
- **All 19–20 LEDs at full brightness:** ~20 mA total
- LED current is set by the LP5024 (RIREF = R6, 75.3 kΩ), independent of MCU draw
- LEDs are off when LED_EN (module pad 15) is LOW; LP5024 shutdown draw ~1 µA

### MCU (nRF52840 module, E73-2G4M08S1C)
- **Deep sleep (System OFF / ZMK deep sleep):** ~single-digit µA for a cleanly configured board (peripherals disabled, no leaky external circuits)
- **Active, BLE connected (idle-to-typing):** typically ~hundreds of µA up to low single-digit mA, varying with BLE connection interval and scan rate
- **BLE TX bursts:** transient peaks higher than the average active figure

### Always-on support circuitry
- **MAX17048 fuel gauge:** ~3 µA (hibernate) typical — confirm against datasheet
- **LP5024 (enabled, outputs off):** quiescent draw per datasheet; ~1 µA in shutdown when LED_EN is LOW
- **Power-latch logic (Schmitt + D-FF) and protection/charger ICs:** small quiescent draws contributing to standby current

### Dominant draw
- The **LEDs dominate** active consumption by a wide margin: ~20 mA with all LEDs on vs. single-digit µA asleep and sub-mA for the MCU when typing. LED duty cycle is therefore the single largest factor in overall draw.

---

## Power Switch / Latch Circuit (same topology, new refs)

- **P-MOS:** Q1 — `C389353` (Nexperia PMF170XP,115, SOT-323)
  - Gate = PMOS_GATE (from D-flip-flop), Source = BAT+, Drain = PWR
- **Power button:** SW (S20) — BAT+ → PWR_BTN net (separate from the key matrix)
- **Schmitt trigger:** U1 — **SN74LVC1G17 (`C7836`)**, SOT-23-5 (was 74AHC1G14)
  - RC long-press timing on PWR_BTN
- **D-flip-flop:** U3 — **SN74LVC1G74 (`C2870716`)**, X2SON-8 (was 74HC dual)
  - Toggles PMOS_GATE on a valid long press
- **Power-button sense:** **PWR_BTN_PRESS** net → module pad 6, via R1/R2 divider — lets the MCU detect button presses for other functions while powered on.
- **Discharge/steering diode:** D1 — `C94366` (onsemi NSR05F20, Schottky 0402)
- **Pull-down:** R3 — `C25744` (10 kΩ)

---

## Battery Charger — BL8573

- **IC:** U5 — `C2999147` (BL8573CS8TR, ESOP-8)
- **VCC:** VBUS (5 V), **BAT:** BAT+
- **Charge current: 1.47 A** (R11 = 680 Ω). Per BL8573 datasheet V1.2: ICHG = (VPROG / RPROG) × 1000, VPROG = 1 V → 1000 / 680 ≈ 1.47 A (~0.42C into the 3.5 Ah cell).
- **RPROG:** R11 — `C137948` (680 Ω).
- Requires a ≥1.5 A USB source for full current; a 500 mA port current-limits lower.
- Dissipation at 1.47 A: P_D ≈ (5 V − 3.75 V) × 1.47 A ≈ 1.84 W in the ESOP-8. U5 uses a wide GND pour + thermal vias; thermal regulation throttles ICHG above ~120 °C die temp.
- RPROG sets battery charge current only — independent of LED current (LP5024 controls that, ~1 mA/LED).
- **CHRG:** open-drain → STDBY/CHRG nets → module (CHRG = pad 4, STDBY = pad 3) as GPIO status inputs
- **Polyfuse:** F1 — `C2760275` (SMD1206-150C-16V) between VBUS_RAW and VBUS
- Logic: CHRG LOW = charging, STDBY LOW = charge complete

---

## Battery Protection — DW01A + Dual MOSFET

- **IC:** U6 — `C42380662` (DW01A, SOT-23-6)
- **MOSFET:** Q2 — `C2830320` (FS8205A dual N-MOS, SOT-23-6)
- Standard 1-cell Li-ion protection (overcharge / overdischarge / overcurrent)
- Nets: U6 OD/OC → Q2 G2/G1; sense node U6-VM; pack negative to GND

---

## Battery Fuel Gauge — MAX17048 (NEW)

- **IC:** U8 — `C2682616` (MAX17048, DFN-8 2×2 mm)
- **Function:** I²C state-of-charge / cell-voltage gauge — replaces the old resistor-divider + nRF ADC scheme
- **VCC/CELL:** BAT+ (pins 2/3), **GND:** pins 1/4/6/9
- **SCL:** pin 7 → SCL bus, **SDA:** pin 8 → SDA bus
- **ALRT (BAT_ALTR):** pin 5 → module pad 9, pulled via R4 — wake/alert on low charge
- ZMK: use a fuel-gauge battery driver over I²C (not the ADC voltage-divider sensor)

---

## I²C Bus (shared)

- **Members:** Module U2 (SDA pad 7 / SCL pad 8), LP5024 U7 (SDA pin 28 / SCL pin 29), MAX17048 U8 (SDA pin 8 / SCL pin 7)
- **Pull-ups:** SDA → R14, SCL → R16 (both `C25905`, 5.1 kΩ) to VDD_NRF

---

## Battery Connector

- **2-pin header, hand-soldered** (not in BOM by design). Pin 1 = BAT+, Pin 2 = GND. Mates with the LiPo JST lead.
- A 3-pin TH header **H1 (`C22373927`)** is the **debug header: SWDIO, SWDCLK, GND** — for one-time bootloader flashing via the Pico programmer. Distinct from the hand-soldered 2-pin battery lead.

---

## USB-C + ESD

- **Connector:** USBC1 — `C165948` (TYPE-C-31-M-12, right-angle SMD)
- **VBUS_RAW** (A4/B9 + B4/A9) → F1 → VBUS; **GND** to GND
- **D+/D−:** through **USBLC6-2SC6 (U9)** ESD array → module D+/D− (pads 31 / 29)
- **CC1/CC2:** 5.1 kΩ to GND (sink/UFP)
- **Decoupling:** C5 (`C703694`, 4.7 µF) on VBUS

---

## E73 Module Core

- nRF52840 + 32 MHz crystal + RF matching + integrated antenna, all internal — no external RF parts on the board.
- **LF crystal (32.768 kHz):** X1 — `C99010` (FC-135 9 pF ±20 ppm) external, for accurate low-power RTC/sleep timing. Load caps C1/C3 (`C1547`, 12 pF C0G).
- **VDD_NRF:** module-regulated logic rail out (pad 19).
- **PWR (Vin):** pad 23, from switched BAT+.
- **Module decoupling:** C2/C7/C8 (`C52923`, 1 µF) and C4/C6/C9 (`C307331`, 100 nF) on PWR/VDD_NRF/RESET.

---

## LP5024 LED Driver

- **IC:** U7 — `C427525` (TI LP5024, VQFN-32 4×4 mm, constant-current sink) over **I²C**
- **VCC (pin 27):** PWR (switched BAT+)
- **VCAP (pin 32):** 1 µF to GND; **VLED:** LED anodes tied directly to PWR
- **SCL/SDA:** shared I²C bus (pins 29 / 28)
- **EN (LED_EN, pin 30):** module pad 15 + pull-down R17 (`C26083`, 1 MΩ) — GPIO HIGH = on, LOW/deep-sleep = ~1 µA shutdown
- **RIREF:** R6 (`C491105`, 75.3 kΩ ERA2AED753X) → sets per-channel max current
- **OUT0–OUT19 → LED1…LED20** (19 per-key LEDs used + spare); OUT20–23 unused
- **Flow:** PWR → LED anode → cathode → LP5024 OUTx → GND
- **Brightness:** I²C BANK_BRIGHTNESS (single write drives all keys)

---

## Per-Key LEDs

- **Part:** 20× OPTOSUPPLY **OSO5PAS1C1A-1MA** (Orange 3528/PLCC2, Vf 1.5–2.2 V, 1 mA, 120° viewing)
- **Source:** Local supplier (not LCSC) — **hand soldered** after PCB arrives
- **Schematic:** DNP placeholder, 3528 footprint
- **Connection:** Anode → PWR, Cathode → LP5024 OUTx
- **Total current:** ~20 mA

---

## Key Matrix

- **Layout:** 5 columns × 4 rows; **19 functional keys** (diodes D2–D20) + S20 power switch outside the matrix
- **Switches:** Ambient Silent Twilight Choc low-profile (HEX footprint), hotswap (DNP)
- **Diodes:** D2–D20 — `C6341459` (CDSQC4148-HF, DFN1006) one per switch
- **Orientation / scan:** row2col — `diode-direction = "row2col"`; rows driven, columns read
- **Anti-ghosting:** diode on every switch

### Matrix nets (from netlist)
- **ROWs → module:** ROW1 pad 10, ROW2 pad 12, ROW3 pad 28, ROW4 pad 43
- **COLs → module:** COL1 pad 40, COL2 pad 38, COL3 pad 36, COL4 pad 34, COL5 pad 32
- Each ROW drives the anodes of 5 (ROW1–3) or 4 (ROW4) diodes → switch → COL.

---

## GPIO / Module-Pad Assignments

Net → E73 module pad. Raw nRF pins are shown where the layout data embeds them.

| Net | Module pad | nRF (if known) |
|-----|-----------|----------------|
| ROW1 | 10 | — |
| ROW2 | 12 | — |
| ROW3 | 28 | — |
| ROW4 | 43 | — |
| COL1 | 40 | — |
| COL2 | 38 | — |
| COL3 | 36 | — |
| COL4 | 34 | — |
| COL5 | 32 | — |
| SDA (I²C) | 7 | — |
| SCL (I²C) | 8 | — |
| LED_EN | 15 | — |
| PWR_BTN_PRESS | 6 | — |
| BAT_ALTR (gauge ALRT) | 9 | — |
| CHRG (charger status) | 4 | — |
| STDBY (charger status) | 3 | — |
| RESET | 26 | P0.18/RESET |
| USB D+ | 31 | — |
| USB D− | 29 | — |
| VBUS sense | 27 | — |
| SWDIO | 37 | — |
| SWDCLK | 39 | — |
| VDD_NRF (out) | 19 | — |
| PWR (Vin) | 23 | — |

> Note: the layout file also exposes several module pads carrying raw nRF labels (P1.11, P1.10, P0.06, P0.08, P1.09, P0.07, P0.13, P0.24, P1.06) that are routed to internal/spare nets — available for future use.

---

## SWD

- Debug header **H1** (3-pin TH, `C22373927`): SWDIO (pad 37), SWDCLK (pad 39), GND
- Programmer: Raspberry Pi Pico + picoprobe
- Use: flash Adafruit nRF52 UF2 bootloader once; thereafter ZMK via USB drag-and-drop

---

## Reset

- **SW1** — `C720477` (TS-1088-AR02016 tactile) → RESET net (module pad 26)
- Debounce: C6 (`C307331`, 100 nF) + R8 (`C25744`, 10 kΩ) pull-up

---

## Components Dropped / Absorbed

- **Bare nRF52840-QIAA, 32 MHz crystal, RF matching (L1/C3/C4), trace antenna, DCDC inductors L2/L4** — now inside the E73 module
- **LDO (TPS7A0233)** — no 3V3 loads
- **Resistor-divider battery sense (old AIN5 divider + ADC)** — replaced by MAX17048 *(designators R2/R3 are reused for other parts this revision)*
- **IS31FL3731** — replaced by LP5024 (legacy)
- Series LED resistor — LP5024 sets current

---

## BOM (left half, as built)

| Ref | Value / MPN | LCSC | Notes |
|-----|-------------|------|-------|
| U2 | E73-2G4M08S1C | C356849 | nRF52840 module w/ antenna |
| U7 | LP5024RSMR | C427525 | 24-ch LED driver, I²C |
| U8 | MAX17048 | C2682616 | Fuel gauge, I²C |
| U5 | BL8573CS8TR | C2999147 | Li-ion charger |
| U6 | DW01A | C42380662 | Battery protection |
| U1 | SN74LVC1G17 | C7836 | Schmitt trigger (power latch) |
| U3 | SN74LVC1G74 | C2870716 | D-flip-flop (power latch) |
| U9 | USBLC6-2SC6 | C7519 | USB ESD on D+/D− |
| Q1 | PMF170XP,115 | C389353 | Power-switch P-MOS |
| Q2 | FS8205A | C2830320 | Protection dual N-MOS |
| USBC1 | TYPE-C-31-M-12 | C165948 | USB-C |
| X1 | FC-135 32.768 kHz 9 pF | C99010 | LF crystal |
| L1 | BRC2016 10 µH | C223154 | (board inductor) |
| F1 | SMD1206-150C-16V | C2760275 | Polyfuse |
| H1 | HC-PM254 3P header | C22373927 | debug: SWDIO/SWDCLK/GND |
| D1 | NSR05F20NXT5G | C94366 | Schottky (power ckt) |
| D2–D20 (×19) | CDSQC4148-HF | C6341459 | Matrix diodes |
| C1, C3 | 12 pF C0G | C1547 | LF crystal load |
| C2, C7, C8 | 1 µF | C52923 | decoupling |
| C4, C6, C9 | 100 nF | C307331 | decoupling |
| C5 | 4.7 µF | C703694 | VBUS bulk |
| R1, R17 | 1 MΩ | C26083 | PWR_BTN_PRESS / LED_EN |
| R9, R10, R14, R16 | 5.1 kΩ | C25905 | CC + I²C pull-ups |
| R11 | 680 Ω | C137948 | RPROG → 1.47 A charge current |
| R3, R4, R7, R8, R12, R13 | 10 kΩ | C25744 | R3 = power-btn pull-down; rest = pull-ups / status |
| R2 | 2.49 MΩ | C368359 | timing |
| R5 | 2.61 MΩ | C166493 | RC timing (0402) |
| R6 | 75.3 kΩ | C491105 | LP5024 IREF |
| SW1 | TS-1088-AR02016 | C720477 | Reset button |
| S20 | power switch | — | separate, on BAT+/PWR_BTN |
| LED ×20 | OSO5PAS1C1A-1MA orange | DNP local | hand soldered |
| BATTERY | 3500 mAh LiPo | — | 2-pin lead hand-soldered |
| FI1–3 | fiducials | — | — |

---

## Key Design Decisions

- **Module over discrete RF:** E73-2G4M08S1C — integrated antenna removes RF/antenna design risk (KiCad-community recommendation)
- **Fuel gauge over ADC divider:** MAX17048 for accurate SoC reporting
- **LED:** 19–20× orange 3528 @ ~1 mA driven by LP5024 over I²C, anodes to PWR
- **I²C bus:** module + LP5024 + MAX17048, 5.1 kΩ pull-ups to VDD_NRF
- **Power latch:** long-press Schmitt + D-FF toggles P-MOS; MCU also senses press via PWR_BTN_PRESS
- **Matrix:** row2col, 5×4, 19 functional + 1 power switch
- **Split:** wireless BLE, identical halves, each independent
- **SWD:** one-time bootloader flash via Pico, then USB UF2

---

## Notes

- **Charge current** is 1.47 A (R11 = 680 Ω), verified against BL8573 datasheet V1.2. Use a ≥1.5 A USB source for full current.
- **GPIO table** lists net → E73 module pad. Raw nRF `Px.yy` names are only needed if a ZMK config requires them; the module's pad / Pro-Micro-style mapping is normally sufficient.

---

## Sourcing

| Source | Parts |
|--------|-------|
| JLCPCB | PCB fabrication + SMT assembly of the small components |
| Keeb Supply | Twilight low-profile switches, HEX keycaps, foam Choc spacers, hotswap Choc sockets, battery connector |
| TME | Per-key LEDs, battery, and a Raspberry Pi Pico (firmware burner + SWD debugger) |

Switches are not soldered — only the hotswap Choc sockets are, and the switches snap into those.
