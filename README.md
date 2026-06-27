# 🎛️ 5-Band Analog Parametric Audio Equalizer

> A fully analog, State Variable Filter (SVF) based parametric equalizer covering the full audio spectrum (20 Hz – 20 kHz), designed as Minor Project II at NIT Andhra Pradesh.

---

## 📌 Project Overview

This project implements a **5-band analog parametric equalizer** using the **KHN (Kerwin-Huelsman-Newcomb) State Variable Filter** topology. It features independent control of center frequency, gain (±9 dB), and Q (bandwidth) for each band — without any cross-coupling between parameters.

The complete design flow covers:
- Transfer function derivation and circuit analysis
- LTspice behavioral and SPICE-level simulation
- Proteus verification with exact TL074CDR models
- KiCad schematic capture (EESchema) and 4-layer PCB layout (PCBnew)

---

## 🗂️ Repository Structure

```
analog-parametric-equalizer/
│
├── ltspice/                  # LTspice simulation files (.asc)
│   ├── full_circuit.asc      # Complete cascaded equalizer
│   ├── low_shelf.asc
│   ├── band1_250Hz.asc
│   ├── band2_800Hz.asc
│   ├── band3_2500Hz.asc
│   └── high_shelf.asc
│
├── kicad/                    # KiCad project files
│   ├── Mini_project.kicad_pro
│   ├── Mini_project.kicad_sch
│   ├── Mini_project.kicad_pcb
│   └── Mini_project.kicad_prl
│
├── docs/                     # Documentation
│   ├── Project_report.pdf    # Full 27-page project report
│   └── ppt_EQ.pptx           # Presentation slides
│
├── images/                   # Screenshots and renders
│   ├── ltspice_full_circuit.png
│   ├── kicad_schematic.png
│   ├── pcb_top_view.png
│   ├── pcb_3d_isometric.png
│   └── sim_smile_curve.png
│
└── README.md
```

---

## ⚙️ Filter Architecture

The equalizer uses a **cascaded (series)** signal chain — each filter block feeds the next, which simplifies gain staging and inter-band tuning.

```
Audio In (3.5mm TRRS)
    │
    ▼
[DC Block + Impedance Match]  →  [ESD Protection — TVS1400DRV]
    │
    ▼
[Low Shelf Filter]        fc: 120–500 Hz,    Gain: ±9 dB
    │
    ▼
[SVF Band 1 — Bass]       f₀: 120–500 Hz,   Gain: ±9 dB,  Q: adjustable
    │
    ▼
[SVF Band 2 — Midrange]   f₀: 400–1600 Hz,  Gain: ±9 dB,  Q: adjustable
    │
    ▼
[SVF Band 3 — Upper Mid]  f₀: 1.2–5 kHz,    Gain: ±9 dB,  Q: adjustable
    │
    ▼
[High Shelf Filter]        fc: 1.2–5 kHz,   Gain: ±9 dB
    │
    ▼
[Output Buffer + ESD]  →  Audio Out (3.5mm TRRS)
```

### Band Summary

| Band         | Center Freq | Tuning Range | Integrator Caps | Frequency Pot     |
|--------------|-------------|--------------|-----------------|-------------------|
| Low Shelf    | 120–500 Hz  | Adjustable   | C11 = 80 nF     | RV2 (100kΩ dual)  |
| Band 1 Bass  | 250 Hz      | 120–500 Hz   | C3, C4 = 33 nF  | RV3/RV4 (10kΩ dual)|
| Band 2 Mid   | 800 Hz      | 400–1600 Hz  | C7, C8 = 10 nF  | RV7/RV9 (10kΩ dual)|
| Band 3 Hi-Mid| 2.5 kHz     | 1.2–5 kHz    | C9, C10 = 3.3 nF| RV10/RV12 (10kΩ dual)|
| High Shelf   | 1.2–5 kHz   | Adjustable   | C2 = 6.8 nF     | RV15 (100kΩ dual) |

---

## 🔬 Key Design Achievements

### Q–Gain Decoupling
In a standard SVF, bandpass peak = Q × Vin, meaning tuning Q inadvertently changes the boost/cut magnitude. This was solved by relocating the Q-control resistor to the BP output feedback path of the summing amplifier — peak amplitude is now constant at Vin, fully independent of Q.

### Dual-Gang Frequency Control
Center frequency ω₀ = 1/(RC) requires both integrators to track the same R simultaneously. Dual-gang potentiometers ensure both integrators always see equal resistance, maintaining SVF symmetry at any frequency setting.

### SVF Bandpass Transfer Function

```
              (ω₀/Q) · s
H(s) = ─────────────────────────────
         s² + (ω₀/Q)·s + ω₀²
```

---

## 🧰 Components

**Core ICs:**
- `TL074CDR` — Quad JFET-input op-amp, SOIC-14 (×5, giving 20 op-amp stages)
- `TVS1400DRV` — TVS ESD protection array (×2, at input and output)

**Op-Amp Key Specs (TL074CDR):**

| Parameter       | Value       |
|-----------------|-------------|
| Supply voltage  | ±15 V       |
| Slew rate       | 13 V/µs     |
| GBW product     | 3 MHz       |
| Input bias      | 65 pA       |
| Noise voltage   | 18 nV/√Hz   |

**Total BOM:** 89 components, 56 fixed resistors, 11 capacitors, 16 potentiometers, 2 audio connectors.

---

## 📊 Simulation Results

All five filter blocks were simulated in **LTspice XVII** (AC sweep, 20 Hz – 20 kHz, 1000 pts/decade) and verified in **Proteus** using exact TL074CDR SPICE models.

| Block          | Target f₀/fc   | Result                  | Gain Range   | Status  |
|----------------|----------------|-------------------------|--------------|---------|
| Low Shelf      | 250 Hz nominal | ~250 Hz, tunable 120–500 Hz | ±9 dB   | ✅ Pass |
| Band 1 (Bass)  | 250 Hz         | ~250 Hz, tunable 120–500 Hz | ±9 dB   | ✅ Pass |
| Band 2 (Mid)   | 800 Hz         | ~800 Hz, tunable 400–1600 Hz| ±9 dB   | ✅ Pass |
| Band 3 (Hi-Mid)| 2.5 kHz        | ~2.5 kHz, tunable 1.2–5 kHz | ±9 dB  | ✅ Pass |
| High Shelf     | 2.3 kHz nominal| ~2.3 kHz, tunable 1.2–5 kHz | ±9 dB  | ✅ Pass |

**Preset EQ responses verified:** V-Shape (Smile Curve), Vocal Boost

---

## 🖥️ PCB Design

Designed in **KiCad PCBnew v9.0** with a 4-layer FR-4 stackup:

| Layer      | Role                                  |
|------------|---------------------------------------|
| F.Cu (Top) | Signal routing, component placement   |
| In1.Cu     | Solid GND plane                       |
| In2.Cu     | ±15 V power distribution              |
| B.Cu (Bot) | Secondary signal routing              |

**PCB Stats:** 1,468 track segments · 150 vias · A3 drawing sheet

Layout follows signal-flow order (left → right), with decoupling caps adjacent to op-amp supply pins and all potentiometers along one edge for front-panel mounting.

---

## 🛠️ Tools Used

| Tool              | Version | Purpose                       |
|-------------------|---------|-------------------------------|
| LTspice XVII      | —       | AC sweep simulation           |
| Proteus Design Suite | —    | SPICE verification            |
| KiCad EESchema    | 9.0     | Schematic capture             |
| KiCad PCBnew      | 9.0     | PCB layout                    |

---

## 👥 Team

| Name              | Roll No. |
|-------------------|----------|
| Mohit Kumar Gupta | 623148   |
| Satyam Kumar      | 623172   |
| Vishal Ray        | 622271   |

**Guide:** Dr. M. C. Raju, Assistant Professor, DECE, NIT Andhra Pradesh
**Course:** Minor Project II (EC399) · Academic Year 2025–26

---

## 📄 License

This project is shared for educational and reference purposes. Feel free to study, fork, and build upon it — attribution appreciated.

---

## 🔮 Future Work

- [ ] PCB fabrication and hardware assembly
- [ ] Audio characterisation using REW (THD+N, noise floor, dynamic range)
- [ ] Enclosure and front-panel design for all 16 potentiometers
- [ ] Expand to 7 or 10 bands with additional SVF stages
- [ ] PCB noise optimisation based on measured results
