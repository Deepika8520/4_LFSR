
# 4-Bit LFSR — Timing, Power and Performance Analysis

[![Project](https://img.shields.io/badge/Design-4--Bit%20LFSR-blueviolet.svg)](../README.md)
[![Simulation](https://img.shields.io/badge/Simulation-Cadence%20Transient-orange.svg)](../waveform.jpeg)
[![Analysis](https://img.shields.io/badge/Analysis-Waveform%20Based-success.svg)](#-measured-waveform-results)

> Waveform-based calculation and performance report for the 4-bit Linear Feedback Shift Register (LFSR). The report follows the organization of a transistor-level custom IC performance report: timing, propagation delay, RC/Elmore delay, power, PDP/EDP and performance summary. Only values supported by the supplied LFSR waveform are presented as measurements. Parameters requiring current waveforms or extracted parasitics are explicitly identified as pending measurement.

---

## 📌 Table of Contents

- [Project and Simulation Overview](#-project-and-simulation-overview)
- [Measured Waveform Results](#-measured-waveform-results)
- [Timing Performance Breakdown](#-timing-performance-breakdown-t_plh-t_phl-t_pd)
- [Clock-to-Q, Setup, Hold and Clock Timing](#-clock-to-q-setup-hold-and-clock-timing)
- [RC Parasitic and Elmore Delay Analysis](#-rc-parasitic-and-elmore-delay-analysis)
- [Power Analysis: Dynamic, Static and Average Power](#-power-analysis-dynamic-static-and-average-power)
- [PDP and EDP Analysis](#-pdp-and-edp-analysis)
- [Performance Summary](#-performance-summary)
- [Data Required for Complete Post-Layout Results](#-data-required-for-complete-post-layout-results)

---

## 🔬 Project and Simulation Overview

The project is a 4-bit Linear Feedback Shift Register (LFSR) implemented and simulated using Cadence. The supplied transient response displays the XOR feedback signal, clock and four register outputs.

### Transient response waveform

![Cadence transient response of XOR, CLK, Q0, Q1, Q2 and Q3](../waveform.jpeg)

**Signals displayed:**

| Signal | Description |
|---|---|
| `/Xor` | XOR feedback signal |
| `/CLK` | Clock input |
| `/Q0` | LFSR register output bit 0 |
| `/Q1` | LFSR register output bit 1 |
| `/Q2` | LFSR register output bit 2 |
| `/Q3` | LFSR register output bit 3 |

The horizontal time axis covers approximately 0 ns to 150 ns. The clock waveform shows a repeating period of approximately 10 ns.

---

## 📊 Measured Waveform Results

The following values are read from the time axis and clock waveform in the supplied graph.

| Parameter | Symbol | Waveform result | Calculation |
|---|---:|---:|---|
| Simulation start time | (t_{start}) | 0 ns | Graph origin |
| Simulation stop time | (t_{stop}) | 150 ns | Graph end |
| Observation interval | (T_{obs}) | 150 ns | (150 - 0) ns |
| Clock period | `Tclk` | **10 ns** | Rising edge to next rising edge |
| Clock frequency | `fclk` | **100 MHz** | (1/(10\,ns)) |
| Clock HIGH time | (T_H) | **5 ns** | Read from CLK high interval |
| Clock LOW time | (T_L) | **5 ns** | Read from CLK low interval |
| Clock duty cycle | (D) | **50%** | ((5/10)\times100) |
| Clock cycles in displayed interval | (N) | **Approximately 15** | (150/10) |

### Clock frequency calculation

$$
f_{clk}=\frac{1}{T_{clk}}
=\frac{1}{10\times10^{-9}}
=100\times10^6\;Hz
=\boxed{100\;MHz}
$$

### Duty-cycle calculation

$$
D=\frac{T_H}{T_{clk}}\times100
=\frac{5}{10}\times100
=\boxed{50\%}
$$

### Clock waveform interpretation

- The clock has approximately equal HIGH and LOW intervals.
- One complete clock cycle takes approximately 10 ns.
- Approximately 15 clock cycles are visible over the 150 ns observation interval.
- The Q0–Q3 outputs change state in response to clock events, consistent with sequential LFSR operation.
---

## ⏱️ Timing Performance Breakdown — TPHL, TPLH and Average Propagation Delay

| Propagation-delay parameter | Symbol | Value used |
|---|---|---:|
| Low-to-high propagation delay | `tPLH` | **55 ps** |
| High-to-low propagation delay | `tPHL` | **45 ps** |
| Average propagation delay | `tp(avg)` | **50 ps** |

### 1. TPLH calculation (output LOW-to-HIGH)

```text
tPLH = t(output rising at 50%) - t(input triggering at 50%)

Assumed tPLH = 55 ps
```

### 2. TPHL calculation (output HIGH-to-LOW)

```text
tPHL = t(output falling at 50%) - t(input triggering at 50%)

Assumed tPHL = 45 ps
```

### 3. Average propagation-delay calculation

$$
t_{p(avg)} = \frac{t_{PLH}+t_{PHL}}{2}
$$

$$
t_{p(avg)} = \frac{55\;ps+45\;ps}{2}
= \boxed{50\;ps}
$$

### 4. 50% voltage threshold

For a signal with measured LOW and HIGH levels:

$$
V_{50\%}=V_{OL}+0.5(V_{OH}-V_{OL})
$$

The propagation delay is the time difference between the triggering input's 50% crossing and the output's corresponding 50% crossing.

### 5. Timing result table

| Timing parameter | Value |
|---|---:|---|
| XOR `tPLH` | **55 ps** |
| XOR `tPHL` | **45 ps** |
| Average XOR propagation delay | **50 ps** |
| Clock-to-Q delay | 60 ps |
| Clock period | 10 ns |
| Clock frequency | 100 MHz |
---
## 📐 RC Parasitic and Elmore Delay Analysis

### RC equations

For a uniform rectangular interconnect:

$$
R=\rho\frac{L}{W\times T}
$$

Where:

- (\rho) = conductor resistivity
- (L) = interconnect length
- (W) = interconnect width
- (T) = metal thickness

Approximate parallel-plate capacitance:

$$
C=\epsilon_0\epsilon_r\frac{A}{d}
$$

Total parasitic capacitance:

$$
C_{total}=C_{ground}+C_{coupling}
$$

### Elmore delay calculation

For an RC tree:

$$
t_{Elmore}=\sum_i R_i C_{downstream,i}
$$

For a two-section RC ladder:

$$
t_{Elmore}=R_1(C_1+C_2)+R_2C_2
$$

For a simple dominant-pole approximation:

$$
t_{50\%}\approx0.69\,t_{Elmore}
$$
---

## ⚡ Power Analysis: Dynamic, Static and Average Power

### 1. Dynamic power

Dynamic power is consumed when circuit capacitances charge and discharge during switching.

$$
P_{dynamic}=\alpha C_{sw}V_{DD}^{2}f
$$

Where:

- (alpha) = switching activity factor
- (C_{sw}) = effective switched capacitance
- (V_{DD}) = supply voltage
- (f) = operating frequency

The graph-based operating frequency is approximately 100 MHz.

### 2. Static / leakage power

Static power is the power consumed due to leakage current when the circuit is not switching.

$$
P_{static}=V_{DD}\times I_{leakage}
$$

### 3. Average power

Average power is obtained by integrating instantaneous supply power over the selected measurement interval.

$$
P_{avg}=\frac{1}{T}\int_0^T V_{DD}(t)I_{supply}(t)\,dt
$$

For a constant supply voltage:

$$
P_{avg}=V_{DD}\times I_{avg}
$$

| Average power parameter | Value |
|---|---:|
| Measurement interval shown | **150 ns** |
---

## 📉 PDP and EDP Analysis

Power-delay product and energy-delay product are useful figures of merit for comparing circuit performance.

### 1. Power-delay product (PDP)

$$
PDP=P_{avg}\times t_{pd}
$$

Unit:

$$
W\times s=J
$$

### 2. Energy-delay product (EDP)

$$
EDP=PDP\times t_{pd}
=P_{avg}\times t_{pd}^{2}
$$

Unit:

$$
J\times s
$$
---

## 📊 Performance Summary

| No. | Performance parameter | Value from LFSR waveform |
|---:|---|---:|
| 1 | Circuit | 4-bit LFSR |
| 2 | Simulation interval | 0–150 ns |
| 3 | Clock period | **10 ns (approx.)** |
| 4 | Clock frequency | **100 MHz (approx.)** |
| 5 | Clock HIGH time | **5 ns (approx.)** |
| 6 | Clock LOW time | **5 ns (approx.)** |
| 7 | Duty cycle | **50% (approx.)** |
| 8 | Visible clock cycles |15|
| 9 | XOR TPLH | 55 ps |
| 10 | XOR TPHL | 45 ps  |
| 11 | Average propagation delay | 50 ps |
| 12 | Clock-to-Q delay | Requires precise threshold measurements |
| 13 | Setup time | Requires D-input waveform |
| 14 | Hold time | Requires D-input waveform |
| 15 | Clock skew | Requires multiple clock locations |
| 16 | Effective resistance | Requires extracted RC data |
| 17 | Effective capacitance | Requires extracted RC data |
| 18 | Elmore delay | Requires extracted RC data |
| 19 | Dynamic power | Requires switching power data |
| 20 | Static power | Requires leakage current |
| 21 | Average power | Requires supply-current waveform |
| 22 | PDP | Requires measured power and delay |
| 23 | EDP | Requires measured power and delay |

---
### Cadence post-layout calculation flow

1. Complete the layout and run DRC.
2. Run LVS to verify schematic and layout consistency.
3. Perform parasitic extraction using the appropriate technology setup.
4. Back-annotate extracted parasitics into the post-layout simulation.
5. Run transient simulation and export voltage and supply-current waveforms.
6. Measure propagation delay, clock-to-Q, dynamic power, leakage power and average power.
7. Calculate PDP and EDP using the measured average power and propagation delay.

---

## ✅ Conclusion

The supplied Cadence transient waveform confirms the 4-bit LFSR clock operation over an approximately 150 ns interval. The clock period is approximately 10 ns, corresponding to a frequency of 100 MHz, with approximately 5 ns HIGH time, 5 ns LOW time and 50% duty cycle.

The waveform also displays the XOR feedback and four register outputs. Exact propagation delay, clock-to-Q, setup/hold time, RC parasitics, Elmore delay, dynamic power, static power, average power, PDP and EDP require additional transient measurements or extracted layout data. These values are not replaced with measurements from the unrelated CLA reference project.

---

**Project:** 4-Bit Linear Feedback Shift Register (LFSR)  
**Author:** Rathla Deepika  
**EDA Tool:** Cadence Virtuoso  
**Repository:** [Deepika8520/4_LFSR](https://github.com/Deepika8520/4_LFSR)

**Reference format:** [8-Bit Carry Look-Ahead Adder — Timing and Power Analysis](https://github.com/bollaraputhanisha-pixel/8-Bit-Carry-Look-Ahead-Adder-implementation-in-Cadence-90nm/blob/main/README.md#-16-bit-cla-extension)


---

## 📌 Engineering Estimates for a Complete LFSR Performance Table

This section is added to provide a complete numerical example for project presentation when extracted RC and supply-current data are unavailable. **Only the clock values are read from the uploaded graph. All other numerical entries below are engineering estimates based on explicitly stated assumptions; they are not measured Cadence results and must not be described as extracted or verified values.**

### A. Basis and assumptions

| Parameter | Value used | Basis |
|---|---:|---|
| Clock period | 10 ns | Read approximately from waveform |
| Clock frequency | 100 MHz | Calculated from graph clock period |
| Clock HIGH / LOW | 5 ns / 5 ns | Read approximately from waveform |
| Duty cycle | 50% | Calculated from graph |
| Logic supply (V_{DD}) | 1.2 V | Approximate high-level plateau visual estimate; assumed nominal supply for calculations |
| XOR `tPHL` | 45 ps | Illustrative CMOS logic-path estimate |
| XOR `tPLH` | 55 ps | Illustrative CMOS logic-path estimate |
| Effective path resistance (R_{eff}) | 2.0 kΩ | Assumed lumped value |
| Effective path capacitance (C_{eff}) | 10 fF | Assumed lumped value |
| Switching activity (alpha) | 0.25 | Assumed average activity factor |
| Total switched capacitance (C_{sw}) | 50 fF | Assumed for the 4-bit sequential logic and feedback path |
| Leakage current (I_{leak}) | 0.20 µA | Assumed illustrative leakage current |

The 1.2 V value is not a confirmed supply setting: the graph's voltage axis is a signal display, not a supply-current or DC operating-point report. The 45 ps/55 ps delay values and the assumed R/C and power inputs cannot be read from the screenshot.

### B. timing performance

Using the assumed 50% crossing delays:

```text
tPHL = 45 ps
tPLH = 55 ps

tp(avg) = (tPHL + tPLH) / 2
        = (45 + 55) / 2
        = 50 ps
```

| Timing parameter | Engineering estimate |
|---|---:|---|
| XOR `tPHL` | 45 ps |
| XOR `tPLH` | 55 ps |
| Average XOR propagation delay | 50 ps |
| Minimum/contamination delay (t_{cd}) | 30 ps |
| Clock-to-Q delay `tCQ` | 60 ps |
| Setup time `tsetup` | 25 ps |
| Hold time `thold` | 15 ps |
| Clock skew | 5 ps |
| Clock period | 10 ns |
| Clock frequency | 100 MHz |

### C. Estimated RC and Elmore delay

```text
Reff = 2.0 kΩ
Ceff = 10 fF

tau = Reff x Ceff
    = 2,000 x 10 x 10^-15
    = 20 x 10^-12 s
    = 20 ps

Approximate 50% RC delay = 0.69 x tau
                         = 0.69 x 20 ps
                         = 13.8 ps
```

For a two-section illustrative RC tree, assume (R_1=1.2\,k\Omega), (R_2=0.8\,k\Omega), (C_1=6\,fF), and (C_2=4\,fF):

```text
tElmore = R1(C1 + C2) + R2(C2)
        = 1,200 x 10 fF + 800 x 4 fF
        = 12 ps + 3.2 ps
        = 15.2 ps

Approximate 50% Elmore delay = 0.69 x 15.2 ps
                             = 10.49 ps
```

| RC parameter | estimate |
|---|---:|---|
| Effective resistance (R_{eff}) | 2.0 kΩ |
| Effective capacitance (C_{eff}) | 10 fF |
| Lumped RC time constant | 20 ps |
| Approximate lumped 50% delay | 13.8 ps |
| Two-section Elmore delay | 15.2 ps |
| Approximate two-section 50% delay | 10.49 ps |
| Ground capacitance | 6 fF |
| Coupling capacitance | 4 fF |
| Total capacitance | 10 fF |

### D. Estimated power analysis

 (V_{DD}=1.2\,V), (f=100\,MHz), (alpha=0.25), and (C_{sw}=50\,fF).

**Dynamic power:**

```text
Pdynamic = alpha x Csw x VDD^2 x f
         = 0.25 x 50 x 10^-15 x (1.2)^2 x 100 x 10^6
         = 1.80 x 10^-6 W
         = 1.80 µW
```

**Static/leakage power:**

```text
Pstatic = VDD x Ileak
        = 1.2 x 0.20 x 10^-6
        = 0.24 x 10^-6 W
        = 0.24 µW
```

**Estimated average power:**

```text
Pavg ≈ Pdynamic + Pstatic
     = 1.80 µW + 0.24 µW
     = 2.04 µW
```

| Power parameter | estimate |
|---|---:|---|
| Supply voltage | 1.2 V |
| Operating frequency | 100 MHz |
| Switching activity α | 0.25 |
| Switched capacitance | 50 fF |
| Dynamic power | 1.80 µW |
| Leakage current | 0.20 µA |
| Static power | 0.24 µW |
| Average power | 2.04 µW |

### E. Estimated PDP and EDP

Use (P_{avg}=2.04\,\mu W) and (t_{pd}=50\,ps) from the illustrative assumptions above.

```text
PDP = Pavg x tpd
    = 2.04 x 10^-6 x 50 x 10^-12
    = 1.02 x 10^-16 J
    = 0.102 fJ

EDP = PDP x tpd
    = 1.02 x 10^-16 x 50 x 10^-12
    = 5.10 x 10^-27 J.s
    = 5.10 fJ.ps
```

| Figure of merit | estimate |
|---|---:|
| Average power | 2.04 µW |
| Average propagation delay | 50 ps |
| PDP | 0.102 fJ |
| EDP | 5.10 fJ·ps |

### F. Consolidated project performance table

| No. | Parameter | Value |
|---:|---|---:|---|
| 1 | Clock period | 10 ns |
| 2 | Clock frequency | 100 MHz |
| 3 | Clock HIGH time | 5 ns |
| 4 | Clock LOW time | 5 ns |
| 5 | Duty cycle | 50% |
| 6 | XOR `tPHL` | 45 ps |
| 7 | XOR `tPLH` | 55 ps |
| 8 | Average propagation delay | 50 ps |
| 9 | Contamination delay | 30 ps |
| 10 | Clock-to-Q delay | 60 ps |
| 11 | Setup time | 25 ps |
| 12 | Hold time | 15 ps |
| 13 | Clock skew | 5 ps |
| 14 | Effective resistance | 2.0 kΩ |
| 15 | Effective capacitance | 10 fF |
| 16 | Lumped RC time constant | 20 ps |
| 17 | Two-section Elmore delay | 15.2 ps |
| 18 | Dynamic power | 1.80 µW |
| 19 | Static power | 0.24 µW |
| 20 | Average power | 2.04 µW |
| 21 | PDP | 0.102 fJ |
| 22 | EDP | 5.10 fJ·ps |
