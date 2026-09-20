# ±15V Regulated Power Supply — Design Documentation

Part of the analog computer project (adder, subtractor, multiplier, integrator, differentiator built around NE5532D op-amps). This document covers the power supply subsystem: design goals, component selection, calculations, and simulation results.

## 1. Purpose

Provide a stable, low-noise ±15V DC supply to power the op-amp circuits and the CD4051 switching stage. Op-amps are sensitive to supply noise (it couples directly into the signal path), so filtering and regulation are prioritized alongside basic voltage accuracy.

## 2. Design Specifications

| Parameter | Value |
|---|---|
| Regulated output | +15V and −15V DC |
| Mains input | 230V RMS, 50Hz (Sri Lanka standard) |
| Regulator ICs | LM7815CT (positive), LM7915CT (negative) |
| Ripple target | As low as practical (achieved: sub-mV on final output, see §7) |

## 3. Circuit Architecture

```
Mains (230V, 50Hz)
      │
Center-tapped step-down transformer (230V primary : 15V-0-15V secondary)
      │
Full-wave rectifier (4× 1N4007, dual-rail configuration)
      │  (+ snubber caps across each diode)
Main filter capacitors (C1, C2 — bulk smoothing)
      │
Linear regulators (LM7815CT / LM7915CT)
      │  (+ input stability caps, output smoothing + HF caps)
Regulated ±15V output → op-amp circuit boards
```

The transformer's center tap is the single system ground reference (star ground) that all other grounds in the analog computer tie back to.

## 4. Component List (Bill of Materials)

| Ref. Des. | Component | Value / Part | Qty | Purpose |
|---|---|---|---|---|
| T1 | Center-tapped step-down transformer | 230V : 15V-0-15V | 1 | Steps mains down to a safe, rectifiable secondary voltage |
| D1–D4 | Rectifier diode | 1N4007 (1A, 1000V) | 4 | Full-wave rectification of both secondary halves into + and − rails |
| CSNUB1–CSNUB4 | Ceramic capacitor | 100nF | 4 | Suppress diode switching (reverse-recovery) noise |
| C1, C2 | Electrolytic capacitor | 2200µF, ≥35V | 2 | Main bulk ripple filtering after rectification |
| C3, C4 | Ceramic capacitor | 330nF | 2 | Regulator input stability (per datasheet) |
| U1 | Negative voltage regulator | LM7915CT | 1 | Regulates negative rail to −15V |
| U2 | Positive voltage regulator | LM7815CT | 1 | Regulates positive rail to +15V |
| C5, C6 | Electrolytic capacitor | 100µF | 2 | Regulator output bulk smoothing, transient response |
| C7, C8 | Ceramic capacitor | 100nF | 2 | High-frequency noise cleanup on regulated output |
| RL1, RL2 | Load resistor (simulation only) | 300Ω | 2 | Represents op-amp board current draw during simulation — not a real PCB part |
| F1 | Fuse (mains side) | Size per transformer VA rating | 1 | Safety — not simulated, required for physical build |

## 5. Principal Calculations

### 5.1 Transformer secondary voltage / turns ratio

The LM7815/LM7915 regulators need a minimum input voltage roughly 2–2.5V above their 15V output (dropout voltage), with a recommended practical minimum of about **17.5V** to regulate reliably.

Target secondary: **15V RMS per half** (30V RMS across the full secondary winding, tip to tip, since Multisim's turns-ratio field refers to the whole winding, not one half of a center-tapped winding).

```
Turns ratio = V_primary / V_secondary_full
            = 230V / 30V
            ≈ 7.65
```

This matches the corrected ratio used in simulation (previously set to ~15.3, which halved the intended per-side voltage — corrected to ~7.65 to get 15V RMS per half, confirmed in simulation).

### 5.2 Peak and rectified voltage

```
V_peak (per secondary half) = V_rms × √2
                             = 15V × 1.414
                             ≈ 21.2V

V_rectified ≈ V_peak − V_diode_drop
            ≈ 21.2V − 0.9V (typical 1N4007 forward drop)
            ≈ 20.3V
```

### 5.3 Ripple voltage across main filter caps (C1, C2)

```
V_ripple = I_load / (f_ripple × C)
```

Using an example load of 100mA per rail (**replace with your actual measured/estimated total op-amp + CD4051 current draw**) and C = 2200µF, with ripple frequency 100Hz (double the 50Hz mains, since full-wave rectification):

```
V_ripple = 0.1A / (100Hz × 2200µF)
         = 0.1 / 0.22
         ≈ 0.45V peak-to-peak
```

### 5.4 Regulator headroom check

```
Minimum voltage at ripple valley ≈ V_rectified − V_ripple
                                  ≈ 20.3V − 0.45V
                                  ≈ 19.85V
```

This sits comfortably above the ~17.5V minimum input requirement, giving roughly **2.3V of margin** even at the lowest point of the ripple waveform — this margin is what actually determines real-world stability under load, more so than the average/nominal voltage.

### 5.5 Regulator power dissipation (heatsinking check)

```
P_dissipated = (V_in_avg − V_out) × I_out
             = (20V − 15V) × 0.1A
             = 0.5W
```

At this dissipation level, the LM7815CT/LM7915CT can run without a heatsink, though adding a small clip-on heatsink is inexpensive insurance if the actual current draw ends up higher than the 100mA example used here.

**Note:** All calculations above use an assumed 100mA per rail as a placeholder. Recalculate §5.3–5.5 with your actual total current draw (sum of all NE5532D op-amp supply currents + CD4051 quiescent current) once known, for accurate final numbers.

## 6. Capacitor Placement Reasoning

| Capacitor(s) | Role | Why it's needed |
|---|---|---|
| CSNUB1–4 | Diode snubbers | Absorb reverse-recovery switching spikes from the rectifier diodes at the source, before they propagate as noise |
| C1, C2 | Bulk filtering | Smooth the pulsating rectifier output into near-steady DC before the regulator |
| C3, C4 | Regulator input stability | Prevent the regulator's internal feedback loop from oscillating due to trace/wire inductance |
| C5, C6 | Output bulk smoothing | Handle sudden current demand (e.g. op-amp transients) and mop up residual ripple |
| C7, C8 | Output HF cleanup | Ceramic caps handle high-frequency noise that electrolytics can't filter well, right before the rail reaches the op-amps |

**Overall pattern:** large electrolytics handle bulk/low-frequency smoothing, small ceramics handle high-frequency cleanup and stability, and snubber caps address noise at its source rather than filtering it out afterward.

## 7. Simulation Results (Multisim)

| Node | DC Value | Ripple (p-p) | Ripple Frequency |
|---|---|---|---|
| +15V rail (PR1, regulator output) | +14.9V | 198µV | 100Hz |
| −15V rail (PR2, regulator output) | −14.9V | 1.01mV | 100Hz |
| Secondary winding (PR3, pre-rectification) | — | 41.6V (raw AC) | 50Hz |

The 14.9V vs. 15.0V nominal is within the regulator's normal ±4% tolerance and is not a design issue. The sub-millivolt ripple on both regulated rails confirms the filtering/regulation stage is working as intended — well below what's needed for clean op-amp operation.

## 8. Build and Layout Recommendations

- **Star grounding:** all grounds (transformer center tap, regulator GND pins, op-amp board ground) should meet at a single point rather than daisy-chaining, to avoid ground loops.
- **Low-ESR electrolytics:** use low-ESR rated capacitors for C1, C2, C5, C6 where possible — they filter ripple more effectively and add less of their own high-frequency noise than generic electrolytics of the same value.
- **Physical placement:** keep all capacitors as close as possible to their respective pins (especially C3/C4 at the regulator input, and C7/C8 at the output) — lead/trace length reintroduces the inductance these capacitors are meant to cancel out.
- **Local decoupling at the op-amp boards:** in addition to this supply's own output capacitors, add local decoupling (e.g. ferrite bead + small ceramic/electrolytic pair) right where power enters each op-amp board, to guard against noise picked up along the wiring run.
- **Mains safety (not simulated):** include a fuse sized to the transformer's rated VA, and follow standard mains-wiring safety practice for the primary side.
- **Heatsinking:** add a small heatsink to U1/U2 if actual measured current draw pushes dissipation meaningfully above the ~0.5W estimate in §5.5.

## 9. Assumptions to Verify Before Finalizing PCB

- Actual total current draw of all op-amp stages + CD4051 switching stage (used to refine §5.3–5.5).
- Transformer VA/current rating sized with margin above that total.
- Confirm real 1N4007 forward voltage drop at your actual operating current (datasheet graph, not just the 0.7–0.9V typical figure used here).
