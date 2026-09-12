# Analog Multiplier — Design Documentation

Part of the analog computer project (adder, subtractor, multiplier, integrator, differentiator built around NE5532D op-amps and a Gilbert-cell core). This document covers the multiplier subsystem, reconstructed from the Multisim schematic/netlist: circuit architecture, component list, and the calculations behind each stage.

## 1. Purpose

Perform four-quadrant analog multiplication of two input signals (V1, V2) using a Gilbert cell (translinear BJT multiplier core), preceded by input scaling/buffering stages and followed by a differential-to-single-ended output conversion (handled by a separate subtractor stage documented elsewhere in the project).

## 2. Circuit Architecture

```
Vin_1 (raw input) ──► ÷100 inverting attenuator (op-amp) ──► scaled Vin_1
                                                                  │
                                            ┌─────────────────────┴─────────────────────┐
                                            │                                             │
                                   unity-gain inverter                              AC-coupled directly
                                   produces −Vin_1                                        │
                                            │                                             │
                                       AC-coupled                                          │
                                            └──────────────► V1+ / V1− (Gilbert cell bases) ◄┘

(identical signal path for Vin_2 → V2+ / V2−, feeding the Gilbert cell's lower differential pair)

V1+/V1−, V2+/V2− ──► Gilbert cell (6× BC547C) ──► V_gil_out_1, V_gil_out_2 (differential product output)
                                            │
                                  (→ downstream subtractor stage converts this
                                     differential pair into a single-ended final answer)
```

## 3. Component List (Bill of Materials)

| Ref. Des. (netlist) | Component | Value | Role |
|---|---|---|---|
| Q4, Q5, Q6, Q7, U19, U20 | NPN transistor | BC547C | The 6 transistors forming the Gilbert cell core (lower differential pair + upper switching quad) |
| U11, R2 | Resistor | 2.2kΩ | Collector load resistors — convert the Gilbert cell's differential output current into voltage (V_gil_out_1, V_gil_out_2) |
| R8 | Resistor | 820Ω | Tail resistor for the lower differential pair (sets bias/tail current) |
| R3 | Resistor | 6.8kΩ | Top resistor of the bias divider (from VCC) |
| R9, U21 | Resistor | 3.9kΩ each | Remaining bias-divider resistors, setting DC bias points for the transistor bases |
| U15, U16, U17, U18 | Resistor | 4.7kΉ each | Couple the DC bias nodes onto the transistor bases (V1+, V1−, V2+, V2−) |
| U14, U13, U12, C3 | Tantalum capacitor | 470µF, 10V (TPSV477K010R0060) | AC-couple the buffered signal onto the biased Gilbert cell inputs without disturbing the DC bias point |
| U3 (section A), U1 (section B) | Op-amp | NE5532 | ÷100 input attenuator stage (one per input channel) |
| U4 (section A), U5 (section A) | Op-amp | NE5532 | Unity-gain inverter stage, producing the −Vin_1/−Vin_2 differential companion signals |
| R12, U6 | Resistor | 47Ω | Feedback resistor for each ÷100 attenuator stage |
| R21, U7 | Resistor | 4.7kΩ | Input resistor for each ÷100 attenuator stage |
| U10, U8 | Resistor | 4.7kΩ | Feedback resistor for each unity-gain inverter stage |
| R11, U9 | Resistor | 4.7kΩ | Input resistor for each unity-gain inverter stage |
| XFG1, XFG3 | Function generator (simulation only) | 2Vpk @ 3kHz, 5Vpk @ 2kHz | Test tones injected during simulation to verify true multiplication (see §6) — not part of the physical build |

## 4. Principal Calculations

### 4.1 Input ÷100 attenuator stage

Each input (Vin_1, Vin_2) first passes through an inverting op-amp stage with:
- Feedback resistor (Rf) = 47Ω
- Input resistor (Rin) = 4.7kΩ

```
Gain = −Rf / Rin = −47 / 4700 = −1/100
```

This confirms the intended ÷100 voltage-reduction stage — the output is an inverted, 100×-reduced copy of the raw input, keeping the signal within the Gilbert cell's safe operating range.

### 4.2 Unity-gain inverter (differential companion signal)

To drive the Gilbert cell differentially, each scaled input is also fed through a second op-amp stage with matched feedback and input resistors (both 4.7kΩ):

```
Gain = −Rf / Rin = −4700 / 4700 = −1
```

This produces −Vin_1 and −Vin_2 — exact inverted copies — so the Gilbert cell receives true differential pairs (Vin_1/−Vin_1 and Vin_2/−Vin_2) rather than single-ended signals referenced to ground.

### 4.3 Bias divider network

The Gilbert cell's transistor bases need a DC operating point in addition to the AC signal. This is set by a resistor divider from VCC (+15V) to ground:

```
VCC ── R3 (6.8kΩ) ── node15 ── R9 (3.9kΩ) ── node17 ── R21* (3.9kΩ) ── GND

(*this R21 in the bias network is a separate 3.9kΩ resistor from the
4.7kΩ R21 used in the attenuator feedback — same reference designator
reused for a different part in the exported netlist, worth renaming
if this causes confusion on your PCB silkscreen)
```

Approximate (unloaded) bias voltages:

```
V(node15) = VCC × (R9 + R21_bias) / (R3 + R9 + R21_bias)
          = 15V × (3900 + 3900) / (6800 + 3900 + 3900)
          = 15V × 7800 / 14600
          ≈ 8.0V

V(node17) = VCC × R21_bias / (R3 + R9 + R21_bias)
          = 15V × 3900 / 14600
          ≈ 4.0V
```

Node15 biases the V1+/V1− bases (via the two 4.7kΩ resistors), and node17 biases the V2+/V2− bases the same way.

**Note:** this calculation ignores the loading effect of the 4.7kΩ resistors feeding the transistor bases, so actual simulated values will differ slightly — treat this as a first-order estimate, and cross-check against the actual DC operating point Multisim reports at these nodes.

### 4.4 Tail current estimate

The lower differential pair's emitters share node10, tied to ground through the 820Ω tail resistor (R8). Using the node17 bias voltage as an approximation for the base voltage of that pair, minus a typical BJT base-emitter drop:

```
V(node10) ≈ V(node17) − V_BE ≈ 4.0V − 0.7V ≈ 3.3V

I_tail ≈ V(node10) / R8 = 3.3V / 820Ω ≈ 4.0mA
```

This tail current is what gets steered between the two branches of the lower pair (by V2), and subsequently between the four collector paths of the upper quad (by V1) — the core translinear multiplying action of the Gilbert cell.

### 4.5 On the overall gain constant

A Gilbert cell's exact output-to-input relationship depends on the tail current, the load resistance, and the transistor's thermal voltage (≈26mV at room temperature) in a way that's sensitive to the precise bias point — rather than assert a specific closed-form gain formula here, it's more reliable to **measure your actual simulated gain constant directly**: apply known DC or low-frequency values for V1 and V2, measure the resulting V_gil_out_1 − V_gil_out_2, and back out the effective scale factor empirically. This is more trustworthy than a hand-derived formula given how sensitive Gilbert cell linearity is to the exact operating point.

## 5. Downstream Processing

The Gilbert cell's differential output pair (V_gil_out_1, V_gil_out_2) feeds into a separate subtractor stage (NE5532-based) that converts this differential signal into a single, final-answer output — this stage is documented separately, since it's shared circuitry with the project's dedicated Adder/Subtractor sheet.

## 6. Verification Method Used in Simulation

Two independent test tones are injected via Multisim's function generator components (not part of the physical circuit):
- **XFG1:** 2V peak, 3kHz — feeds the V1 input path
- **XFG3:** 5V peak, 2kHz — feeds the V2 input path

This is the standard way to verify a circuit is truly *multiplying* rather than just amplifying or mixing: true multiplication of two sinusoids at frequencies f1 and f2 produces new frequency components at the **sum** (3kHz + 2kHz = 5kHz) and **difference** (3kHz − 2kHz = 1kHz) — neither of which is present in the original inputs. Checking the output spectrum (via Multisim's Fourier/spectrum analysis tools) for energy specifically at 1kHz and 5kHz, and the absence of significant energy remaining at the original 2kHz/3kHz tones, is the standard confirmation that the multiplier is working correctly.

## 7. Notes / Assumptions to Verify

- The bias-network calculations in §4.3–4.4 are first-order estimates ignoring resistor loading — confirm against Multisim's actual DC operating point results.
- Two different resistors in the exported netlist share the reference designator "R21" for unrelated parts (one in the attenuator feedback network, one in the bias divider) — worth renaming one before finalizing the PCB silkscreen to avoid confusion during assembly/debugging.
- Confirm the actual measured gain constant (§4.5) against your target multiplier scale factor for the overall analog computer's operation range.
