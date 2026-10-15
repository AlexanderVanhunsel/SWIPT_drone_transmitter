# SWIPT Drone – Wireless Power Transmitter

Transmitter side of **Team SWIPT** (*Simultaneous Wireless Information and Power Transfer*), the drone project of the
Bachelor in Electrical Engineering at **KU Leuven**.

The goal of the project is to power a small drone wirelessly through an inductive link, while also exchanging data
between the base station and the drone over that same link. This repository contains my part of the work:
**the power transfer (transmitter) and the bidirectional data communication**. It holds the LTspice simulations and
the final Altium Designer PCB of the transmitter. The receiver (drone) side was built by other team members and is not included here.

---

## How it works

```
 12 V DC ──► Full H-bridge (4× N-MOSFET) ──► Series-compensated ──►  ~ ~ ~  ──► Receiver coil (drone)
               ▲            ▲                transmit coil            magnetic
               │            │                (L_Tx + C_Tx)            coupling
          IR2110       IR2110                                          k ≈ 0.17
       (half-bridge  (half-bridge
          driver)       driver)
               ▲            ▲
               └──── complementary PWM (3.3 V logic, ~80 kHz) ── frequency tracking / data
```

### 1. N-MOSFET H-bridge with IR2110 gate drivers
A full H-bridge of four N-channel MOSFETs turns the 12 V DC supply into a square-wave AC voltage across the transmit coil.
Each leg is driven by an **IR2110** high-/low-side driver. Because the high-side switches are also N-MOSFETs, their gate
voltage is generated with a **bootstrap circuit** (bootstrap diode + capacitor per leg). The two legs are driven with
complementary 3.3 V logic signals, so the bridge output alternates between +12 V and −12 V.

### 2. Resonant inductive link
The transmit coil (≈ 8.4 µH) is placed in series with a capacitor (470 nF), forming a series-resonant tank at

$$f_0 = \frac{1}{2\pi\sqrt{L C}} \approx 80\ \text{kHz}$$

At resonance the reactance of the coil is cancelled, so the coil current – and with it the magnetic field that couples
into the receiver coil – is maximal. The simulations use a coupling factor of k ≈ 0.17 between transmit and receive coil.

### 3. Frequency tracking
The real resonance frequency shifts with component tolerances, coil distance/alignment and the load on the drone side.
Instead of driving the bridge at a fixed frequency, the transmitter uses **frequency tracking**: the switching frequency
is swept/adjusted around the nominal resonance and the frequency that gives the best power transfer is kept as the
operating point.

### 4. Bidirectional data communication
Data is sent over the same inductive link:
- **Downlink (base station → drone):** the H-bridge drive signal is modulated by a data signal (the simulation switches
  between two drive patterns with a control signal), which changes the field seen by the receiver.
- **Uplink (drone → base station):** the receiver switches its load (*load modulation*). This changes the impedance
  reflected into the transmit coil, which is detected on the transmitter side as a change in coil current/voltage.

---

## Repository structure

```
SWIPT-drone-transmitter/
├── ltspice/
│   ├── 01_coupled-coils/        First models of the inductive link
│   │   ├── Draft1.asc               first RLC resonance model of the Tx/Rx tanks
│   │   ├── powertransfer.asc        coupled coils (k = 0.17), power transfer to a load
│   │   └── 2-coil.asc               link + rectifier, 12 V / 5 V regulators on the receive side
│   ├── 02_h-bridge-inverter/    H-bridge development
│   │   ├── inverter4.asc            first IR2110 + MOSFET H-bridge with bootstrap
│   │   ├── inverter(fast).asc       H-bridge driving the resonant transmit coil
│   │   ├── inverter(fast)2.asc      + receiver coil, rectifier and comparator
│   │   └── eigentest.asc            own test variant of the complete link
│   ├── 03_full-transmitter/     Complete system simulations
│   │   ├── Integrated Circuit.asc                       full power-transfer chain
│   │   ├── Integrated Circuit + bidirectionaldata.asc   + uplink (load modulation)
│   │   └── Circuit_8dec.asc                             + downlink modulation of the bridge drive
│   └── models/                  Custom symbols (.asy) and SPICE models (.spi)
│                                    IR2110, IRFS7440, diodes, LM7805/7812, ideal comparator
└── altium/                      Final transmitter PCB (Altium Designer)
    ├── Transmitter_PCB3.PrjPcb      project file
    ├── Transmitter_PCB3.SchDoc      schematic
    ├── Transmitter_PCB3.PcbDoc      PCB layout
    └── outputs/                     design rule check report
```

## Opening the files

### LTspice
1. Install [LTspice](https://www.analog.com/en/resources/design-notes/ltspice-simulator.html).
2. The schematics use custom symbols and models stored in `ltspice/models/`. Make LTspice find them in one of two ways:
   - **Option A:** in LTspice go to *Settings → Search Paths* and add the `ltspice/models` folder to both the
     *Symbol Search Path* and the *Library Search Path*, **or**
   - **Option B:** copy the contents of `ltspice/models/` into the folder of the schematic you want to open.
3. Open an `.asc` file and press *Run*. The full-transmitter simulations are long transient runs and can take a while.

### Altium Designer
Open `altium/Transmitter_PCB3.PrjPcb` in Altium Designer. The project contains the schematic and the PCB layout of
the final transmitter board (IR2110 drivers, 4× IRF540N MOSFET H-bridge, bootstrap circuitry, bulk decoupling and screw
terminals for supply, coil and control signals).

## Notes
- Simulations use the IRFS7440 MOSFET model; the final PCB uses IRF540N MOSFETs.
- The SPICE models in `ltspice/models/` are vendor/course-supplied models (e.g. the IR2110 model © International
  Rectifier) and are included only so the simulations run out of the box.
- This is student work, made as part of the KU Leuven Bachelor in Electrical Engineering.

## Author
**Alexander Vanhunsel** – Team SWIPT, responsible for the wireless power transfer and bidirectional data communication
(transmitter side).
