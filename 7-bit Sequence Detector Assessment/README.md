# Sequence Detector – RTL to Gate-Level Simulation

## Overview

This repository documents the complete RTL-to-Gate-Level Simulation flow of a Verilog sequence detector for the target sequence **1101011**.

The same testbench and input sequence are used for both RTL simulation and Gate-Level Simulation (GLS). The final comparison shows that the synthesized implementation preserves the functional behavior of the RTL design, with **5 detections** in both simulations and the first detection occurring at exactly **188 ns**.

## Design Flow

```text
RTL
 ↓
RTL Simulation (Icarus Verilog)
 ↓
GTKWave
 ↓
Yosys Synthesis (SKY130 std-cell library)
 ↓
det_netlist.v / Synthesized Netlist
 ↓
Post-synthesis Gate-Level Simulation (GLS)
 ↓
GTKWave
 ↓
Same Functional Behavior
```

---

## 1. RTL Design

The sequence detector is implemented as a finite state machine using a 3-bit state register.

- **Target sequence:** `1101011`
- **Number of states:** 7
- **State width:** 3 bits
- **Input:** `din`
- **Output:** `detected`
- **Reset:** synchronous, active high
- **Detection:** `detected` is asserted when the complete target sequence is recognized (Mealy-style — depends on current state **and** input, not state alone)
- **Overlap handling:** on a successful match (state 6, `din=1`), the FSM returns to state 2 rather than state 0, preserving the trailing `"11"` so overlapping matches are still caught

---

## 2. Testbench

The testbench generates the clock, applies reset, drives the input sequence bit-by-bit, records the detection pulses, and prints the final detection count.

### Important Testbench Parameters

| Parameter | Value |
|---|---|
| Clock | `#4 clk = ~clk` (period = 8 ns, 125 MHz) |
| Target sequence | `1101011` |
| Reset | Active high, held 4 clock cycles |
| Testbench style | Same input sequence for RTL and GLS |
| Assessment instance | `24eg104c52` |
| Detection count observed | `5` |


> The complete, un-truncated testbench (all ~180 `drive_bit` calls) is checked in at [`tb/tb.v`](tb/tb.v).

---

## 3. Pre-Synthesis / RTL Simulation

The RTL testbench was compiled and simulated with Icarus Verilog, and the generated VCD waveform was viewed in GTKWave.

```bash
iverilog sequence_detector.v ../tb/tb.v
./a.out
gtkwave dump.vcd
```

The waveform shows:

- `clk` running continuously at an 8 ns period.
- `reset` asserted initially for 4 cycles, released at ~32 ns, and asserted again at the end.
- `din` changing according to the testbench sequence.
- `state[2:0]` and `next_state[2:0]` stepping through the FSM (S0→S1→S2→S3→S4→S5→S6).
- `detected` pulsing when the target sequence `1101011` is recognized.
- `detection_count` reaching **5**.

### RTL GTKWave Evidence

<img width="1536" height="787" alt="RTL_SIM" src="https://github.com/user-attachments/assets/42763ece-10bf-47b9-bf76-d72d947466c2" />

The waveform shows the signals `clk`, `reset`, `din`, `detected`, `next_detected`, `state[2:0]`, and `next_state[2:0]`. The marker sits at **188 ns**, which is exactly where the first detection pulse occurs — matching the hand-derived prediction from the RTL (first match completing at bit 19 of the driven stream).

---

## 4. Yosys Synthesis

Yosys was used to read and synthesize the RTL into a gate-level representation, mapped onto the SKY130 `sky130_fd_sc_hd` standard-cell library.

```tcl
read_liberty -lib ../../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog sequence_detector.v
synth -top sequence_detector
dfflibmap -liberty ../../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
abc -liberty ../../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
show
stat
write_verilog det_netlist.v
```

### Synthesized Design Statistics

| Item | Count |
|---|---:|
| Wires | 44 |
| Wire bits | 50 |
| Public wires | 5 |
| Public wire bits | 11 |
| Ports | 4 |
| Port bits | 4 |
| Memories | 0 |
| Memory bits | 0 |
| Processes | 0 |
| Cells | 20 |

### Cell Breakdown

| Cell | Count |
|---|---:|
| `$_DFF_P_` | 7 |
| `$_SDFF_PP0_` | 1 |
| `sky130_fd_sc_hd__a21boi_0` | 1 |
| `sky130_fd_sc_hd__a21oi_1` | 1 |
| `sky130_fd_sc_hd__and2_0` | 3 |
| `sky130_fd_sc_hd__nand2b_1` | 1 |
| `sky130_fd_sc_hd__nor2_1` | 2 |
| `sky130_fd_sc_hd__nor2b_1` | 1 |
| `sky130_fd_sc_hd__nor3b_1` | 2 |
| `sky130_fd_sc_hd__nor4_1` | 1 |
| **Total** | **20** |

> The cell names above are taken directly from the Yosys `stat` output shown in the synthesis screenshot below.

### Yosys Statistics Evidence

<img width="1536" height="787" alt="stat" src="https://github.com/user-attachments/assets/e7acdd71-3c5c-4162-8995-ac18caf6894a" />

The screenshot shows the synthesized `sequence_detector` module: **7 flip-flops** for the 3-bit state register plus **1 special reset-capable flop** (`$_SDFF_PP0_`) for the registered `detected` output, and **13 combinational SKY130 cells** implementing the next-state and detection logic.

---

## 5. Synthesized Netlist / Logic Representation

After synthesis, the RTL is represented using SKY130 flip-flops and combinational logic cells.

The generated logic diagram (via Yosys `show`) shows:

- `din`, `reset`, and `clk` fanning into a small cluster of gate-level logic.
- `nor2b`, `and2`, `nor3b`, `nand2b`, `a21oi`, and `a21boi` cells implementing the combinational next-state and detection equations.
- A bank of `$DFF_P_` flops for the state bits, plus the `$SDFF_PP0_` flop driving `detected`.
- Clock and reset connectivity fanning across the whole register bank.

### Synthesized Logic Diagram (Full View)

<img width="1536" height="787" alt="netlist_sequence_detector" src="https://github.com/user-attachments/assets/fc12368f-72b0-4382-8749-a02d90c03ce2" />

### Synthesized Logic Diagram (Zoomed — Register/Output Cluster)

<img width="1536" height="787" alt="zoom view" src="https://github.com/user-attachments/assets/6d9637be-4f49-46c1-a27d-05df9ea238a1" />

This graph represents the synthesized gate-level structure produced by Yosys. The zoomed view makes the `R` (reset) pin on the `$SDFF_PP0_` flop for `detected` clearly visible, distinguishing it from the plain `$DFF_P_` cells used for the state bits.

### Written-Out Netlist (`det_netlist.v`)

<img width="1536" height="787" alt="detnetlidt v" src="https://github.com/user-attachments/assets/1c2a8268-6746-4387-b2d5-b49c8c865960" />

The register section of the netlist confirms the flop mapping one-to-one with the `stat` output:

```verilog
always @(posedge clk)
  if (_07_) detected <= 1'h0;
  else detected <= din;
always @(posedge clk) state[0] <= _06_;
always @(posedge clk) state[1] <= _00_;
always @(posedge clk) state[2] <= _01_;
always @(posedge clk) state[3] <= _02_;
always @(posedge clk) state[4] <= _03_;
always @(posedge clk) state[5] <= _04_;
always @(posedge clk) state[6] <= _05_;
```

Note that the synthesized `state` register is 7 bits wide here rather than the 3 bits declared in the RTL (`STATE_W = 3`) — Yosys/`abc` chose a one-hot-style encoding during optimization instead of keeping the binary encoding from the source. This does not change functional behavior; it's purely a different physical state encoding chosen at synthesis time.

---

## 6. Post-Synthesis Gate-Level Simulation (GLS)

The synthesized netlist was simulated with the same testbench and input sequence used for RTL simulation, using the SKY130 primitive and standard-cell Verilog models.

```bash
cd rtl
iverilog ../../my_lib/verilog_model/primitives.v ../../my_lib/verilog_model/sky130_fd_sc_hd.v det_netlist.v ../tb/tb.v
./a.out
gtkwave dump.vcd
```

### GLS Evidence

<img width="1536" height="787" alt="GLS_" src="https://github.com/user-attachments/assets/e354952b-d8c2-4dee-baf6-90eeadacee6b" />

The GLS run produced the same `TIME=... DIN=... DETECTED=...` console trace as the RTL simulation, starting with `TIME=45000 NS DIN=1 DETECTED=0`, and the same detection behavior on the waveform.

---

## 7. RTL vs GLS Comparison

The comparison is based on the same testbench and the same input sequence.

| Parameter | RTL Simulation | GLS |
|---|---:|---:|
| Target sequence | `1101011` | `1101011` |
| First detection | 188 ns | 188 ns |
| Total detections | 5 | 5 |
| Final detection count | 5 | 5 |

Both simulations successfully detect the target sequence `1101011`, with the detection count reaching **5** in both cases and the first detection landing at the same timestamp.

### Full Detection Timing Table (matches in both RTL and GLS)

| Detection # | Bit index | Time (ns) |
|---|---|---|
| 1 | 19  | 188  |
| 2 | 30  | 276  |
| 3 | 49  | 428  |
| 4 | 111 | 924  |
| 5 | 145 | 1196 |

---

## 8. Functional Verification

The functional behavior can be summarized as:

```text
Input sequence
      ↓
Sequence detector FSM
      ↓
Target = 1101011
      ↓
Detection pulses
      ↓
5 successful detections
      ↓
RTL result = GLS result
```

The GTKWave results show matching logical behavior between the RTL and gate-level simulations. The synthesized implementation therefore preserves the intended sequence-detection functionality for the given testbench.

---

## 9. Final Conclusion

The synthesized implementation **preserves the functional behavior of the RTL design for the given testbench**. Both RTL and Gate-Level Simulation detect the target sequence `1101011` and produce **five detection pulses**, as observed in the GTKWave waveforms and console output. The GLS implementation can introduce small gate-level propagation delays, but the logical detection behavior remains unchanged.

---

## 10. Result

```text
Target sequence        : 1101011
RTL detections         : 5
GLS detections         : 5
RTL first detection    : 188 ns
GLS first detection    : 188 ns
Functional match       : YES            
```
