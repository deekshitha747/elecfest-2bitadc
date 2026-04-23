# 2-Bit Flash ADC Design and Analysis

This repository contains the design, implementation, and analysis of a 2-bit Flash Analog-to-Digital Converter (ADC) using ideal comparator models in LTspice.

---

## 1. Overview
An ADC samples an analog waveform at uniform time intervals and assigns a digital value to each sample. The digital value appears on the converter’s output in a binary-coded format (specifically Thermometer Code in this Flash architecture).

### Flash ADC Architecture
Flash ADCs are made by cascading high-speed comparators. 
* For an **N-bit** converter, the circuit employs $2^N-1$ comparators.
* A resistive-divider with $2^N$ resistors provides the reference voltages.
* Each comparator produces a **1** when its analog input voltage is higher than the reference voltage applied to it. Otherwise, the output is **0**.
---


## 2. Behavior
A 2-bit ADC distinguishes between $2^2 = 4$ levels. Using a single **5V shared source** ($V_{DD}$) and four equal $1k\Omega$ resistors in series creates the following reference nodes:

* **Ref 3:** $3.75V$ ($\frac{3}{4} V_{DD}$)
* **Ref 2:** $2.50V$ ($\frac{2}{4} V_{DD}$)
* **Ref 1:** $1.25V$ ($\frac{1}{4} V_{DD}$)

### Truth Table (Standard Logic)
As $V_{in}$ rises, the comparators trigger sequentially:

| Analog Input ($V_{in}$) | Comp 1 (1.25V) | Comp 2 (2.50V) | Comp 3 (3.75V) | Digital State |
| :--- | :---: | :---: | :---: | :---: |
| $0V < V_{in} < 1.25V$ | 0 | 0 | 0 | `000` |
| $1.25V < V_{in} < 2.5V$ | **1** | 0 | 0 | `001` |
| $2.5V < V_{in} < 3.75V$ | 1 | **1** | 0 | `011` |
| $V_{in} > 3.75V$ | 1 | 1 | **1** | `111` |

---

## 3. Resolution
Resolution is the smallest analog change detectable by the ADC, representing the value of the Least Significant Bit (LSB).

$$Resolution = \frac{V_{ref(max)}}{2^n} = \frac{5V}{4} = 1.25V$$

This means every **1.25V** increase in $V_{in}$ causes the ADC to move to the next digital state.

---

## 4. Output Characteristics
The dynamic behavior of the ADC is characterized by how it processes different input signals:

### Ramp / DC Characteristics
For a ramp signal, the output is a **staircase signal** consisting of flat plateaus. The output only changes at specific threshold voltages ($1.25V, 2.5V, 3.75V$). Between these thresholds, the digital state remains constant despite small changes in analog input.

### Sinusoidal Characteristics
When a sine wave is applied:
* **Pulse Width Variation:** Because the sine wave spends different amounts of time at different voltage levels, the digital outputs have different pulse widths.
* **Lower Thresholds:** Result in wider digital pulses.
* **Higher Thresholds:** Result in narrower digital pulses (occurring only at the signal peaks).
  <img width="440" height="244" alt="image" src="https://github.com/user-attachments/assets/c1ac946b-3072-4e4b-9eb0-8e77e31b8a6a" />


---

## 5. Simulation Results
<img width="1917" height="420" alt="image" src="https://github.com/user-attachments/assets/9b5043ff-a45e-4241-9aae-639f502ba537" />


*Figure 1: 2-bit ADC Staircase output overlaid on the original Sinusoidal input signal.*

---

## 6. Signal Reconstruction & Visualization Logic

To visualize how the 2-bit ADC sees the analog signal, we reconstruct a single digital staircase from the individual comparator outputs. In the simulation, this is achieved using the following mathematical expression:

$$\text{Reconstructed Signal} = \frac{V(OUT_0) + V(OUT_1) + V(OUT_2)}{3}$$

### How the Reconstruction Works:
Because each comparator output is a binary signal (either $0V$ or $5V$), simply looking at three separate square waves can be difficult to interpret. By calculating the average of the active digital high states, we create a staircase waveform that tracks the input:

* **Level 0 (000):** When $V_{in} < 1.25V$, all outputs are $0V$. The reconstructed value is $0V$.
* **Level 1 (001):** When $V_{in}$ exceeds $1.25V$, $OUT_0$ goes High ($5V$). The expression calculates $(5+0+0)/3 = 1.66V$.
* **Level 2 (011):** When $V_{in}$ exceeds $2.50V$, both $OUT_0$ and $OUT_1$ are High. The expression calculates $(5+5+0)/3 = 3.33V$.
* **Level 3 (111):** When $V_{in}$ exceeds $3.75V$, all three are High. The expression calculates $(5+5+5)/3 = 5V$.

### Why Divide by 3?
The division by 3 is a scaling factor. Since our maximum possible digital sum is $15V$ ($3 \times 5V$) and our analog input max is $5V$, dividing by 3 brings the digital staircase into the same vertical scale as the analog sine wave.

