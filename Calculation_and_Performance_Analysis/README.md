# 4-Bit LFSR — Calculation and Performance Analysis

This document records the waveform-based observations and the calculation methods for the 4-bit LFSR project. It separates values that can be read from the supplied transient plot from values that require Cadence simulation or post-layout extraction data.

## 1. Supplied transient waveform

The graph below is the waveform image already present in the repository as `waveform.jpeg`. It shows `/Xor`, `/CLK`, and the four outputs `/Q0` to `/Q3` over approximately 0–150 ns.

![Transient response waveform showing XOR, CLK, Q0, Q1, Q2 and Q3](../waveform.jpeg)

## 2. Waveform observations

The clock repeats at approximately 10 ns intervals. Its high and low intervals are each approximately 5 ns.

| Parameter | Approximate value | Calculation / observation |
|---|---:|---|
| Displayed time interval | 0–150 ns | Read from horizontal axis |
| Clock period, Tclk | 10 ns | Time between successive rising edges |
| Clock frequency, fclk | 100 MHz | 1 / 10 ns |
| Clock HIGH time | 5 ns | Read from CLK trace |
| Clock LOW time | 5 ns | Read from CLK trace |
| Clock duty cycle | 50% | (5 ns / 10 ns) × 100 |

These are estimates read from the plotted screenshot. They are not precision measurements from raw transient data.

## 3. Propagation delay analysis

Propagation delay must be measured between the 50% voltage crossings of the relevant input and output. For a combinational XOR path, use the actual XOR input transition that causes the output transition and measure the XOR output. Do not use the clock period as propagation delay.

### Definitions and equations

- **tPHL:** input-to-output delay for an output transition from HIGH to LOW.
- **tPLH:** input-to-output delay for an output transition from LOW to HIGH.

```text
tPHL = t(output falling at 50%) - t(input triggering at 50%)
tPLH = t(output rising at 50%)  - t(input triggering at 50%)

tp(avg) = (tPHL + tPLH) / 2
```

### Results from the supplied screenshot

| Timing parameter | Result | Reason |
|---|---|---|
| XOR tPHL | Not numerically resolvable | Exact input/output 50% crossing times are not available |
| XOR tPLH | Not numerically resolvable | Exact input/output 50% crossing times are not available |
| Average propagation delay | Cannot be calculated | Requires numerical tPHL and tPLH |
| Clock-to-Q delay | Not numerically resolvable | Plot resolution is insufficient for precise clock/Q crossing measurement |
| Setup time | Not measurable from this plot | D-input transition and setup boundary are not shown |
| Hold time | Not measurable from this plot | D-input transition and hold boundary are not shown |
| Critical-path delay | Not determined | Requires path timing data/netlist and extracted parasitics |
| Clock period | Approximately 10 ns | Read from CLK waveform |
| Clock frequency | Approximately 100 MHz | Calculated from clock period |

**Important:** The apparent alignment of CLK and Q transitions does not prove zero clock-to-Q delay. The 10 ns clock period is not a propagation delay.

### How to measure in Cadence

1. Open the transient waveform in ViVA.
2. Select the relevant XOR input and XOR output traces.
3. Identify a clean input transition and its corresponding output transition.
4. Measure both crossings at 50% of their respective voltage swings.
5. Calculate the time difference for tPHL or tPLH.
6. Repeat for multiple transitions and report the measured values and the averaging method.
7. For sequential paths, separately measure clock-to-Q and evaluate setup/hold at the receiving flip-flop.

## 4. RC parasitic analysis

Actual interconnect resistance and capacitance must come from the technology-aware extraction flow (for example, Quantus/QRC) using the project PDK and selected extraction corner.

### First-order equations

For a uniform rectangular conductor:

```text
R = rho x L / (W x T)
```

Where rho is resistivity, L is wire length, W is wire width and T is metal thickness.

A simplified parallel-plate capacitance estimate is:

```text
C = epsilon0 x epsilonr x A / d
Ctotal = Cground + Ccoupling
```

These equations explain the dependencies; they do not provide actual project values without geometry and process data.

| RC parameter | Project result | Required source |
|---|---|---|
| Interconnect resistance | Not available from waveform | Extracted DSPF/SPEF or Quantus report |
| Ground capacitance | Not available from waveform | Extracted parasitic report |
| Coupling capacitance | Not available from waveform | Extracted parasitic report |
| Total parasitic capacitance | Not available from waveform | Extracted parasitic report |
| Extraction corner | Not specified | Quantus/QRC setup |

### Cadence post-layout extraction flow

1. Complete layout and run DRC.
2. Run LVS and confirm layout/schematic consistency.
3. Run Quantus/QRC with the correct PDK, extraction corner and RC options.
4. Generate the parasitic view or DSPF/SPEF file.
5. Run post-layout transient simulation with extracted parasitics.
6. Compare pre-layout and post-layout delay and power.

## 5. Elmore delay analysis

Elmore delay is a first-order estimate for an RC tree. For every resistor on the source-to-observation path, multiply that resistance by the capacitance downstream of it and sum the products.

```text
tElmore = sum(Ri x Cdownstream,i)
```

For a two-section RC ladder:

```text
tElmore = R1(C1 + C2) + R2(C2)
```

For a simple dominant-pole response, an approximate 50% delay may be estimated as:

```text
t50 approximately 0.69 x tElmore
```

The 0.69 factor is an approximation and is not universal for every RC network.

| Elmore parameter | Project result |
|---|---|
| R1, R2, ... | Requires extracted RC network |
| C1, C2, ... | Requires extracted RC network |
| Elmore delay | Cannot be calculated without RC values |
| Approximate 50% delay | Cannot be calculated without Elmore delay |

## 6. Power analysis

The supplied screenshot contains logic voltage waveforms only. It does not contain supply current, supply voltage metadata, switched capacitance or leakage-current measurements. Therefore, numerical dynamic, static and average power cannot be derived from this graph alone.

### Dynamic power

Dynamic power is associated with charging and discharging circuit capacitances.

```text
Pdynamic = alpha x Csw x VDD^2 x f
```

- alpha = switching activity factor (state its convention)
- Csw = effective switched capacitance (F)
- VDD = supply voltage (V)
- f = operating frequency (Hz)

A numerical result requires VDD, switching activity and effective capacitance, or suitable supply-current measurements with a defined measurement method.

### Static / leakage power

Measure leakage current while the circuit is held in a stable state:

```text
Pstatic = VDD x Ileak
```

For multiple supply rails:

```text
Pstatic,total = sum(VDDk x Ileak,k)
```

Leakage can depend on the stored state, input condition and process/voltage/temperature corner.

### Average power

For time-varying supply voltage and current over a measurement window T:

```text
Paverage = (1/T) x integral from 0 to T of [VDD(t) x Isupply(t)] dt
```

For a constant supply voltage:

```text
Paverage = VDD x Iaverage
```

If average power is measured directly from total supply current during switching, it already includes leakage. Do not add static power again to that total.

### Power results from the screenshot

| Power parameter | Result | Data needed |
|---|---|---|
| Dynamic power | Not calculable from screenshot | VDD, Csw and activity factor, or measured switching supply current |
| Static/leakage current | Not shown | Stable-state supply current |
| Static/leakage power | Not calculable from screenshot | VDD and leakage current |
| Average supply current | Not shown | Supply-current transient data |
| Average power | Not calculable from screenshot | Supply voltage/current over a stated time interval |

## 7. Performance summary

| Performance metric | Waveform-based result |
|---|---|
| Clock period | Approximately 10 ns |
| Clock frequency | Approximately 100 MHz |
| Clock HIGH time | Approximately 5 ns |
| Clock LOW time | Approximately 5 ns |
| Duty cycle | Approximately 50% |
| XOR tPHL | Requires 50% crossing measurements |
| XOR tPLH | Requires 50% crossing measurements |
| Average propagation delay | Requires tPHL and tPLH |
| Clock-to-Q delay | Requires precise clock/Q measurements |
| Setup / hold time | Requires D-input and clock timing analysis |
| Critical-path delay | Requires timing path analysis |
| Interconnect R and C | Requires post-layout extraction |
| Elmore delay | Requires extracted RC tree |
| Dynamic power | Requires switching power data |
| Static power | Requires leakage measurement |
| Average power | Requires supply-current data over a defined interval |

## 8. Data required to complete numerical calculations

Please export or provide the following from Cadence:

1. Transient CSV/PSF containing CLK, XOR input(s), XOR output, Q0–Q3 and supply current.
2. Supply voltage (VDD), process/voltage/temperature corner and the time window for average-power measurement.
3. Stable-state supply current for leakage measurement.
4. Quantus/QRC DSPF, SPEF or extracted RC report for interconnect and Elmore calculations.
5. Relevant path endpoints and timing constraints for setup/hold and critical-path analysis.

## 9. Conclusion

The supplied waveform supports an approximate 10 ns clock period, 100 MHz clock frequency, 5 ns HIGH time, 5 ns LOW time and 50% duty cycle. It also displays the XOR and four LFSR output waveforms over approximately 150 ns. Exact propagation delays, clock-to-Q, setup/hold, RC parasitics, Elmore delay and numerical power values cannot be reliably extracted from the screenshot alone. These values are intentionally left as not determined until the corresponding Cadence transient, current and post-layout extraction data are available.

---
**Project:** 4-Bit Linear Feedback Shift Register (LFSR)  
**Author:** Rathla Deepika  
**Repository:** [Deepika8520/4_LFSR](https://github.com/Deepika8520/4_LFSR)
