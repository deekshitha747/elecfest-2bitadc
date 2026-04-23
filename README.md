# 2-Bit Flash ADC with Combinational Priority Encoding

## 1. Overview
This project features a high-speed 2-bit Flash Analog-to-Digital Converter (ADC) designed in LTspice. While standard Flash architectures output raw thermometer code, this design integrates a **Priority Encoder** to provide standard 2-bit binary outputs ($Q_1, Q_0$). 

This conversion makes the ADC compatible with digital systems like microcontrollers and Digital Signal Processors (DSPs).

---

## 2. Circuit Architecture
The design consists of three primary stages:
1. **Resistor Ladder:** Four $1k\Omega$ resistors create reference voltages at $1.25V$, $2.5V$, and $3.75V$.
2. **Comparator Bank:** Three high-speed comparators ($D_1, D_2, D_3$) compare the analog $V_{in}$ against the reference levels.
3. **Priority Encoder:** A logic block (using AND, OR, and NOT gates) that maps the comparator states to binary bits.

### Logic Equations
To achieve binary output, the following Boolean logic was implemented:
* **MSB ($Q_1$):** Directly driven by the middle comparator ($D_2$).
* **LSB ($Q_0$):** Derived using the equation: $Q_0 = (D_1 \cdot \overline{D_2}) + D_3$

---

## 3. Truth Table (Binary Encoding)
The encoder translates the raw comparator data into a 2-bit binary word:

| Analog Input ($V_{in}$) | $D_3$ (3.75V) | $D_2$ (2.5V) | $D_1$ (1.25V) | **$Q_1$ (MSB)** | **$Q_0$ (LSB)** | Binary |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: |
| $0V < V_{in} < 1.25V$ | 0 | 0 | 0 | **0** | **0** | `00` |
| $1.25V < V_{in} < 2.5V$ | 0 | 0 | 1 | **0** | **1** | `01` |
| $2.5V < V_{in} < 3.75V$ | 0 | 1 | 1 | **1** | **0** | `10` |
| $V_{in} > 3.75V$ | 1 | 1 | 1 | **1** | **1** | `11` |

---

## 4. Signal Reconstruction & Visualization
To verify the ADC's accuracy, a reconstructed staircase is generated in the LTspice waveform viewer. Because the digital gates are set to `Vhigh=5V`, the following weighted sum expression is used to overlay the digital result on the 5V analog sine wave:

$$\text{Reconstructed Signal} = \frac{(2 \times V(Q_1)) + (1 \times V(Q_0))}{3}$$

* **Level 0 (00):** $0V$
* **Level 1 (01):** $1.66V$
* **Level 2 (10):** $3.33V$
* **Level 3 (11):** $5V$

---

## 5. Simulation Settings
* **Gate Logic:** All digital gates are configured as `Vhigh=5V Vlow=0V`.
* **Reference Grounding:** All logic gates have their reference pins (bottom pin) tied to the common ground to prevent floating nodes.
* **Unused Inputs:** To ensure logical stability in the 5-input gate models, unused **AND** inputs are tied to $V_{DD}$, while unused **OR** inputs are tied to **GND**.

---

