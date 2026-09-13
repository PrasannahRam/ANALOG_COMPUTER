# Differentiator

### Triangular Input 
| frequency     | C
| ------------- | ------------- |
1 - 10khz|100pF
10Hz - 1khz|10nF
5Hz - 10hz|100nF
1Hz - 5hz|1000nF

### Square Input
- Follow the above values.
- If unwanted spikes encountered increase the R2 value. 
- If the wave is distorted, reduce the R2 value.

---
---

# Component Justification for Active Differentiator Circuit

### 1. Active Element
* **Op-Amp (U1B - NE5532P):**
  * **Role:** Core high-speed, low-noise operational amplifier providing the required open-loop gain for differentiation.
  * **Value / Selection:** `NE5532P`
  * **Justification:** An ideal differentiator requires an op-amp with high bandwidth and slew rate to accurately process steep signal slopes ($\frac{dV}{dt}$) without slew-rate limiting or phase distortion. The NE5532 provides a high gain-bandwidth product ($10\text{ MHz}$), low noise voltage ($5\text{ nV}/\sqrt{\text{Hz}}$), and a slew rate of $9\text{ V}/\mu\text{s}$, making it suitable for processing signals up to $10\text{ kHz}$.

---

### 2. Switched Frequency-Band Capacitors ($C1, C2, C4, C5$)
* **Role:** Input differentiation capacitors determining the time constant $\tau = R_f C_{in}$ for each operating band.
* **Justifications:**
  * **$C1 = 1\ \mu\text{F}\ (1000\text{ nF})$ [$1\text{ Hz} - 5\text{ Hz}$]: At very low frequencies, the derivative ($\frac{dV}{dt}$) is tiny. A large capacitance ($1\ \mu\text{F}$) provides enough input current to yield a readable output voltage gain ($0.63\text{ V/V}$ to $3.14\text{ V/V}$) across $R1 = 100\text{ k}\Omega$.
  * **$C5 = 100\text{ nF}$ [$5\text{ Hz} - 10\text{ Hz}$]: Scaled down by $10\times$ relative to $C1$ to offset the higher input frequency range, keeping maximum band gain at $0.63\text{ V/V}$.
  * **$C4 = 10\text{ nF}$ [$10\text{ Hz} - 1\text{ kHz}$]: Covers two decades of frequency. A $10\text{ nF}$ value keeps the gain bounded between $0.063\text{ V/V}$ and $6.28\text{ V/V}$ at $1\text{ kHz}$, preventing op-amp output rail saturation.
  * **$C2 = 150\text{ pF}$ [$10\text{ kHz}$]: At high frequencies, $\frac{dV}{dt}$ is extremely large. Reducing $C_{in}$ to $150\text{ pF}$ maintains a gain near unity ($0.94\text{ V/V}$), preventing high-frequency noise amplification.

---

### 3. Stability & Input Resistance ($R2, R6$)
* **Role:** Input series limiting resistors.
* **Value / Selection:** $R2 = 10\text{ k}\Omega$, $R6 = 1\text{ k}\Omega$
* **Justification:** An ideal differentiator has zero input resistance, causing infinite high-frequency gain and unconditional instability (self-oscillation). Adding input series resistance caps the maximum high-frequency gain to $A_{max} = \frac{R1}{R_{in}}$ and forms a high-pass corner $f_c = \frac{1}{2\pi R_{in} C_{in}}$, stabilizing the feedback loop and dampening high-frequency noise.

---

### 4. Feedback Network ($R1, C6$)
* **Feedback Resistor ($R1 = 100\text{ k}\Omega$):**
  * **Role:** Sets the overall differentiation scaling factor ($V_{out} = -R1 \cdot C_{in} \cdot \frac{dV_{in}}{dt}$).
  * **Justification:** A high resistance ($100\text{ k}\Omega$) yields high transimpedance gain without requiring large, bulky input capacitors, keeping the load current within the NE5532 driving limit.
* **Feedback Capacitor ($C6 = 22\text{ pF}$):**
  * **Role:** High-frequency compensation capacitor (low-pass filter pole).
  * **Justification:** Placed in parallel with $R1$ to create a second-order pole at $f_p = \frac{1}{2\pi R1 C6} \approx 72.3\text{ kHz}$. This rolls off gain beyond the $10\text{ kHz}$ target bandwidth, suppressing high-frequency op-amp noise and ringing.

---

### 5. DC Compensation & Output Coupling ($R3, C3, R4$)
* **Bias Resistor ($R3 = 100\text{ k}\Omega$):**
  * **Role:** Non-inverting terminal DC bias balance.
  * **Justification:** Equalizes the DC source resistance seen by both inverting and non-inverting terminals ($R3 = R1 = 100\text{ k}\Omega$). This cancels out DC offset voltage shifts caused by the NE5532 input bias currents.
* **Output AC Coupling Capacitor ($C3 = 470\ \mu\text{F}$):**
  * **Role:** DC blocking capacitor at the output stage.
  * **Justification:** Prevents any residual op-amp DC offset voltage from reaching the measurement load/oscilloscope.
* **Output Load Resistor ($R4 = 100\text{ k}\Omega$):**
  * **Role:** Output ground reference and load resistor.
  * **Justification:** Combined with $C3$, it sets a high-pass corner frequency at $f_{HP} = \frac{1}{2\pi (470\mu\text{F})(100\text{k}\Omega)} \approx 0.003\text{ Hz}$. This ensures phase and amplitude linearity for input signals as low as $1\text{ Hz}$.


# Differentiator Transfer Function & Constant Derivation

### 1. General Transfer Function Derivation

For a practical active op-amp differentiator with input resistor $R_{in}$, input capacitor $C_{in}$, feedback resistor $R_f$, and parallel feedback capacitor $C_f$:

* **Input Impedance:** 
  $$Z_{in}(s) = R_{in} + \frac{1}{s C_{in}} = \frac{1 + s R_{in} C_{in}}{s C_{in}}$$

* **Feedback Impedance:** 
  $$Z_f(s) = R_f \parallel \frac{1}{s C_f} = \frac{R_f}{1 + s R_f C_f}$$

Using the inverting op-amp configuration gain formula $H(s) = -\frac{Z_f(s)}{Z_{in}(s)}$:

$$H_{diff}(s) = \frac{V_{out,stage}(s)}{V_{in}(s)} = - \left( \frac{s R_f C_{in}}{(1 + s R_{in} C_{in})(1 + s R_f C_f)} \right)$$

---

### 2. Time-Domain Differentiation Equation

In the operational differentiation sub-band ($f \ll \frac{1}{2\pi R_{in} C_{in}}$ and $f \ll \frac{1}{2\pi R_f C_f}$), the pole terms in the denominator evaluate to approximately $1$:

$$H_{diff}(s) \approx - s \cdot (R_f C_{in})$$

Taking the inverse Laplace transform gives the time-domain output equation:

$$v_{out}(t) = - K \cdot \frac{d v_{in}(t)}{dt}$$

Where the **differentiation scaling constant** $K$ is defined as:

$$K = R_f C_{in} \quad [\text{seconds}]$$

---

### 3. Frequency Band Specifications
* **Operating Range:** $0.003\text{ Hz}$ to $10\text{ kHz}$
* **Lower Cutoff Frequency ($f_L$):** Determined by the output AC-coupling network ($C3, R4$)
  $$f_L = \frac{1}{2\pi R_4 C_3} = \frac{1}{2\pi (100\text{ k}\Omega)(470\ \mu\text{F})} = \frac{1}{295.31} \approx 0.00339\text{ Hz} \approx 0.003\text{ Hz}$$
* **Upper Cutoff Frequency ($f_H$):** Determined by the feedback compensation network ($R1, C6$)
  $$f_H = \frac{1}{2\pi R1 C6} = \frac{1}{2\pi (100\text{ k}\Omega)(150\text{ pF})} = \frac{1}{9.425 \times 10^{-5}} \approx 10.61\text{ kHz} \approx 10\text{ kHz}$$

---

### 4. Output Filtering Stage ($C3, R4$)

The AC coupling network formed by $C3$ and $R4$ forms a high-pass filter:

$$H_{out}(s) = \frac{R_4}{R_4 + \frac{1}{s C_3}} = \frac{s R_4 C_3}{1 + s R_4 C_3} = \frac{1}{1 + \frac{1}{s R_4 C_3}}$$

Evaluating magnitude in the frequency domain ($s = j2\pi f$):

$$|H_{out}(f)| = \frac{1}{\sqrt{1 + \left( \frac{1}{2\pi f C_3 R_4} \right)^2}}$$

For operating frequencies within the passband ($f \ge 0.003\text{ Hz}$):

$$\frac{1}{2\pi f C_3 R_4} \ll 1 \implies H_{out}(s) \approx 1$$

---

### 5. Total System Transfer Function

Combining both the differentiator stage and the output filtering stage:

$$H_{total}(s) = H_{diff}(s) \cdot H_{out}(s) = - \left( \frac{s R_f C_{in}}{(1 + s R_{in} C_{in})(1 + s R_f C_f)} \right) \cdot \left( \frac{s R_4 C_3}{1 + s R_4 C_3} \right)$$

Within the operating passband ($0.003\text{ Hz} \le f \le 10\text{ kHz}$), all filter dynamics simplify:

$$H_{total}(s) \approx - s \cdot (R_f C_{in})$$

---

### 6. Switch-Wise Scaling Factor Calculations ($R_f = R1 = 100\text{ k}\Omega$)

| Switch Position | Selected Band | Active Capacitor ($C_{in}$) | Calculation ($K = R_f C_{in}$) | Value of $K$ |
| :--- | :--- | :--- | :--- | :--- |
| **Switch 4** | **1 Hz – 5 Hz** | $C1 = 1\ \mu\text{F}$ | $100\text{ k}\Omega \times 1\ \mu\text{F} = 10^5 \times 10^{-6}$ | $0.1\text{ s}\ (100\text{ ms})$ |
| **Switch 3** | **5 Hz – 10 Hz** | $C5 = 100\text{ nF}$ | $100\text{ k}\Omega \times 100\text{ nF} = 10^5 \times 10^{-7}$ | $0.01\text{ s}\ (10\text{ ms})$ |
| **Switch 2** | **10 Hz – 1 kHz** | $C4 = 10\text{ nF}$ | $100\text{ k}\Omega \times 10\text{ nF} = 10^5 \times 10^{-8}$ | $0.001\text{ s}\ (1\text{ ms})$ |
| **Switch 1** | **1 kHz – 10 kHz** | $C2 = 1\text{ nF}$ | $100\text{ k}\Omega \times 1\text{ nF} = 10^5 \times 10^{-9}$ | $0.0001\text{ s}\ (100\ \mu\text{s} / 0.1\text{ ms})$ |

---

### 7. Resulting Output Equations for Each Mode

* **Band 1 (1 Hz – 5 Hz, Switch 4 ON):**
  $$v_{out}(t) = -0.1 \cdot \frac{d v_{in}(t)}{dt}$$

* **Band 2 (5 Hz – 10 Hz, Switch 3 ON):**
  $$v_{out}(t) = -0.01 \cdot \frac{d v_{in}(t)}{dt}$$

* **Band 3 (10 Hz – 1 kHz, Switch 2 ON):**
  $$v_{out}(t) = -0.001 \cdot \frac{d v_{in}(t)}{dt}$$

* **Band 4 (1 kHz – 10 kHz, Switch 1 ON):**
  $$v_{out}(t) = -0.0001 \cdot \frac{d v_{in}(t)}{dt}$$