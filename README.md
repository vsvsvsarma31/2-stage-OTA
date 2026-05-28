# 2 Stage OTA — TSMC 180nm

Design and simulation of a Miller-compensated two-stage operational transconductance amplifier (OTA) in TSMC 0.18 µm CMOS technology. The amplifier is characterised in open-loop (AC) and verified in a closed-loop non-inverting gain-of-2 configuration (transient).

---

## Specifications

| Parameter | Target | Technology |
|---|---|---|
| Technology | TSMC 0.18 µm | V<sub>DD</sub> = 1.8 V |
| DC Gain | ≥ 40 dB | — |
| Closed-loop gain | 2 V/V | Non-inverting |
| Load capacitance | 10 pF | C1 |
| Input step | 0.2 V (0.9 → 1.1 V) | 0.1 ns edges |
| V<sub>Tn</sub> / V<sub>Tp</sub> | 0.37 V / 0.39 V | — |
| µ<sub>n</sub>C<sub>ox</sub> / µ<sub>p</sub>C<sub>ox</sub> | 230 / 100 µA/V² | — |

---

## Circuit Architecture

### First Stage — Folded Differential Pair with Active Load

- **M1, M2** (NMOS): Input differential pair. Both sized W/L = 6.5 µm / 1 µm. Biased at I<sub>D</sub> = 30 µA each (half of 60 µA tail).
- **M3, M4** (PMOS): Current-mirror active load, W/L = 3.57 µm / 1 µm. Convert the differential current to a single-ended output at the drain of M2/M3.
- **M5** (NMOS, W/L = 60 µm / 2 µm): Tail current source. Mirrors M6 with a **10:1 ratio** to source 60 µA.
- **M6** (NMOS, W/L = 6 µm / 2 µm): Diode-connected bias reference, driven by the 6 µA ideal reference source I1.

> **Bias generation:** A single ideal current source I1 = 6 µA is used (= 1/10 of the tail current, as required). All other currents are derived by mirroring.

### Second Stage — Common-Source Amplifier

- **M8** (NMOS, W/L = 60 µm / 2 µm): Common-source gain stage. Provides additional voltage gain and drives the load.
- **M7** (PMOS, W/L = 7.14 µm / 1 µm): PMOS current-source load for the second stage.

### Miller Compensation Network

| Component | Value | Purpose |
|---|---|---|
| C2 | 2.2 pF | Miller compensation capacitor (C<sub>c</sub>) — pole splitting |
| R1 | 15 kΩ | Nulling resistor — eliminates the RHP zero introduced by C2 |
| C1 | 10 pF | Output load capacitor |

The nulling resistor R1 shifts the RHP zero to either infinity or the LHP by satisfying:

$$R_z = \frac{1}{g_{m8}} \quad \text{(to cancel zero)} \quad \Rightarrow \quad R_1 > \frac{1}{g_{m8}}$$

---

## Transistor Sizing Summary

| Ref (Assignment) | Schematic Name | Type | W (µm) | L (µm) | Role |
|---|---|---|---|---|---|
| M0 | M5 | NMOS | 60 | 2 | Tail current source (10× mirror) |
| M1 | M1 | NMOS | 6.5 | 1 | Diff pair — non-inverting input |
| M2 | M2 | NMOS | 6.5 | 1 | Diff pair — inverting input |
| M3 | M3 | PMOS | 3.57 | 1 | Active load (drain-side) |
| M4 | M4 | PMOS | 3.57 | 1 | Active load (diode-connected) |
| M5 | M8 | NMOS | 60 | 2 | 2nd stage amplifier |
| M6 | M7 | PMOS | 7.14 | 1 | 2nd stage current source |
| Bias ref | M6 | NMOS | 6 | 2 | Mirror reference (diode-connected) |

> **Note:** Assignment naming (M0–M6) differs from schematic instance names (M1–M8). The table above maps them explicitly.

---

## Hand-Calculated Operating Points

Assuming V<sub>ov</sub> = 200 mV, I<sub>tail</sub> = 60 µA → I<sub>D1,2</sub> = 30 µA.

| Transistor | g<sub>m</sub> (µA/V) | V<sub>GS</sub>−V<sub>T</sub> (V) | I<sub>D</sub> (µA) |
|---|---|---|---|
| M1, M2 | ≈ 300 | 0.20 | 30 |
| M3, M4 | ≈ 146 | ~0.41 | 30 |
| M5 (tail) | — | — | 60 |

> g<sub>m1,2</sub> = √(2 · µ<sub>n</sub>C<sub>ox</sub> · (W/L) · I<sub>D</sub>) = √(2 × 230 × 6.5 × 30×10⁻⁶) ≈ 300 µA/V

*Simulated values (g<sub>m</sub>, r<sub>o</sub>, V<sub>GS</sub>−V<sub>T</sub>) from `.op` should be filled in after running the DC operating point analysis.*

---

## File Structure

```
.
├── Open_loop_240102259.asc      # Open-loop OTA — AC gain & Bode plot
├── Closed_loop_240102259.asc    # Closed-loop (gain = 2) — transient step response
└── tsmc018.lib                  # TSMC 0.18 µm MOSFET model file (required)
```

---

## Simulation Setup

### Open Loop — AC Analysis (`Open_loop_240102259.asc`)

| Setting | Value |
|---|---|
| Analysis | `.ac dec 10000 1k 10G` |
| Input (V2) | AC = 0.5 V, 0° (non-inv) |
| Input (V3) | AC = 0.5 V, 180° (inv) |
| DC bias | V<sub>CM</sub> = 0.9 V |
| Load | C1 = 10 pF |

**Purpose:** Measure DC gain, unity-gain bandwidth (UGB), phase margin.

### Closed Loop — Transient (`Closed_loop_240102259.asc`)

| Setting | Value |
|---|---|
| Analysis | `.tran 1m` |
| Input (V2) | `PULSE(0.9, 1.1, 0.1m, 0.1n, 0.1n, 1m)` |
| Feedback | R2 = R3 = 10 kΩ → A<sub>v</sub> = 1 + R2/R3 = 2 |
| Bias (V3) | 0.9 V at inverting node |

**Purpose:** Verify gain-of-2, measure slew rate, settling time on a 0.2 V step.

---

## How to Run in LTspice

1. Clone or download the repository.
2. Place `tsmc018.lib` in the **same directory** as the `.asc` files (the `.INCLUDE` path is relative).
3. Open `Open_loop_240102259.asc` → **Run** → probe `V(out)` in the AC plot.
   - Check DC gain magnitude at 1 kHz (should be ≥ 40 dB).
   - Read phase at 0 dB crossing for phase margin.
4. Open `Closed_loop_240102259.asc` → **Run** → probe `V(out)` and `V(in)`.
   - Confirm output swings from ≈ 1.8 V to ≈ 2.2 V (gain = 2 around 0.9 V CM).
   - Observe settling behaviour and overshoot.
5. For operating point data: add `.op` to either schematic and run → **View → SPICE Error Log** for g<sub>m</sub>, r<sub>o</sub>, I<sub>D</sub>.

---

## Expected Results

| Parameter | Expected | Notes |
|---|---|---|
| DC Open-loop gain | ≥ 40 dB | Primary design constraint |
| Closed-loop gain | 2 V/V (6 dB) | Verified by R2 = R3 |
| Output swing (transient) | 0.9 V → 1.8 V | On 0.2 V input step with gain = 2 |
| Phase margin | > 45° | Ensured by C2 = 2.2 pF, R1 = 15 kΩ |
| Power consumption | ≈ 1.8 V × I<sub>total</sub> | I<sub>total</sub> ≈ I<sub>tail</sub> + I<sub>2nd stage</sub> |

*Populate this table with simulated values after running `.op` and `.ac`.*

---

## Design Decisions & Trade-offs

- **Long-channel devices (L = 1–2 µm):** Chosen over L<sub>min</sub> = 0.18 µm to increase r<sub>o</sub> and hence DC gain. Penalty: larger area and slower speed.
- **10:1 bias mirror ratio:** Allows a 6 µA reference (low quiescent power) while delivering 60 µA to the tail — satisfying the single-ideal-source constraint.
- **Nulling resistor R1 = 15 kΩ:** Eliminates the RHP zero that would degrade phase margin. Sized above 1/g<sub>m8</sub>.
- **PMOS active load (M3/M4):** Provides higher output impedance than resistive load, improving first-stage gain. Also enables rail-to-rail input common-mode range extension.
- **M3/M4 at W = 3.57 µm:** Set by the V<sub>ov</sub> = 200 mV design target with µ<sub>p</sub>C<sub>ox</sub> = 100 µA/V², resulting in a non-integer width — expected in hand design.

---

## Applications

- Switched-capacitor filters and ADC front-ends
- Sensor interface amplifiers (low-power CMOS)
- Sample-and-hold circuits
- Building block for bandgap references and LDO error amplifiers

---

## Technology

| Parameter | Value |
|---|---|
| Process | TSMC 0.18 µm CMOS |
| Supply | 1.8 V |
| Simulator | LTspice XVII |
| Model file | `tsmc018.lib` |
