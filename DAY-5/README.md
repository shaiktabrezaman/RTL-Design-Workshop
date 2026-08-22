# Day 5 — Optimization in Synthesis

## Overview

Module 5 looks at RTL coding patterns that lead synthesis tools to infer unwanted latches, the difference between overlapping and clean `case` statements, and the two ways repetition is handled in Verilog — a procedural `for` loop versus a structural `generate` block. All of this was verified hands-on: each design was simulated at RTL, synthesized in Yosys, and then re-checked with gate-level simulation (GLS).

---

## Contents

- [1. Incomplete `if` — Latch Inference](#1-incomplete-if--latch-inference)
- [2. Incomplete `case` — Latch Inference](#2-incomplete-case--latch-inference)
- [3. Overlapping `case` (bad_case)](#3-overlapping-case-bad_case)
- [4. Mux Using a `for` Loop](#4-mux-using-a-for-loop)
- [5. Demux — Case vs Generate](#5-demux--case-vs-generate)
- [6. Ripple Carry Adder Using `generate`](#6-ripple-carry-adder-using-generate)
- [7. Key Takeaways](#7-key-takeaways)

---

## 1. Incomplete `if` — Latch Inference

When a combinational `always` block has an `if` with no matching `else`, the output has no defined value for the condition that isn't covered. Since simulation (and synthesis) has to hold onto *something*, the tool infers a latch to retain the last known value.

Two versions of this were built and compared — one plain incomplete `if`, and one with an extra nested condition to see how the latch inference scales.

### incomp_if — Simulation

<img width="1536" height="787" alt="incomp_if_sim" src="https://github.com/user-attachments/assets/3acead0f-0e3b-424b-95b2-1105f04aa29a" />

### incomp_if — Netlist / RTL view

<img width="1536" height="787" alt="incomp_if_netlist with rtl" src="https://github.com/user-attachments/assets/b6b97063-11e1-4962-90fa-05d1586ea002" />

### incomp_if2 — Simulation

<img width="1536" height="787" alt="icomp_if2_sim" src="https://github.com/user-attachments/assets/d39a7485-9b79-4af7-9ee0-a4bb6efebc92" />

### incomp_if2 — Netlist / RTL view

<img width="1536" height="787" alt="incomp_if2_netlist with rtl" src="https://github.com/user-attachments/assets/8988569c-e230-4cf3-99a2-2cbb988934e3" />

---

## 2. Incomplete `case` — Latch Inference

The same latch problem shows up with `case` statements when one or more input combinations aren't handled. A 2-bit selector has 4 possible values — if only 3 are written out and there's no `default`, the missing case leaves the output undriven for that condition, and a latch gets inferred just like with `if`.

### RTL viewed before synthesis

<img width="1536" height="787" alt="rtl_" src="https://github.com/user-attachments/assets/29c06a06-575a-4a60-a1ab-163a591819e5" />

### incomp_case — Simulation

<img width="1536" height="787" alt="incomp_case_sim" src="https://github.com/user-attachments/assets/6f396668-ae85-4067-a6c4-0462bf726c43" />

### incomp_case — Netlist

<img width="1536" height="787" alt="incomp_case_net" src="https://github.com/user-attachments/assets/71a8cf66-ef8b-43ab-8ad2-dcc003bdf763" />

### Fixing it with a complete case / default

Once every selector value is handled (or a `default:` is added), the latch disappears and the design synthesizes as pure combinational logic.

### comp_case — Simulation

<img width="1536" height="787" alt="comp_case_sim" src="https://github.com/user-attachments/assets/21523b9b-7c04-44b4-921c-555cc0f232f5" />

### comp_case — Netlist

<img width="1536" height="787" alt="comp_case_net" src="https://github.com/user-attachments/assets/10c52d80-29b7-479b-8b09-2b2ae77cd994" />

### Partially-fixed case — Netlist

An in-between version (some but not all cases covered) was also checked to see the partial effect on the synthesized structure.

<img width="1536" height="787" alt="partail_case_netlist" src="https://github.com/user-attachments/assets/a58eea3d-70e9-47c8-b640-082bb71b86f5" />

---

## 3. Overlapping `case` (bad_case)

A different problem from *missing* cases is *overlapping* ones — e.g. using a wildcard pattern like `2'b1?` alongside an explicit `2'b10`. Both branches can match the same input, which creates ambiguous priority behavior that can differ between simulation and the synthesized netlist.

### bad_case — RTL and Netlist

<img width="1536" height="787" alt="bad-case_rtl and netlist" src="https://github.com/user-attachments/assets/df6f4105-9410-44f4-983b-3c8a2eed0de5" />

### bad_case — RTL Simulation

<img width="1536" height="787" alt="bad_case_sim" src="https://github.com/user-attachments/assets/1a066e7c-bd26-4d66-a461-a4eb0cd17108" />

### bad_case — Gate-Level Simulation

Running GLS on this one specifically highlights the mismatch caused by the overlapping conditions.

<img width="1536" height="787" alt="bad_case_sim_GLS" src="https://github.com/user-attachments/assets/241493d4-0c88-4dac-9b9e-a8c0f742d395" />

---

## 4. Mux Using a `for` Loop

A procedural `for` loop (used inside an `always` block) is well suited to indexed/repetitive logic — here it's used to build an 8:1 mux by looping over the selector value instead of writing out 8 `case` branches by hand. A default assignment before the loop keeps the output fully driven, avoiding the latch issue from sections 1–2.

### mux_generate — Simulation

<img width="1536" height="787" alt="mux_generate_sim" src="https://github.com/user-attachments/assets/7a5535a3-4fb7-471f-9478-75178d7422b4" />

### mux_generate — Netlist

<img width="1536" height="787" alt="mux_generate netlist" src="https://github.com/user-attachments/assets/b93cbf1a-e8ec-4218-8dc4-82ecf6413e5e" />

### mux_generate — Gate-Level Simulation

<img width="1536" height="787" alt="mux_generate_gls" src="https://github.com/user-attachments/assets/f3149f0e-9167-4b67-b525-ab87cbf7945b" />

---

## 5. Demux — Case vs Generate

The same 1:8 demux was implemented two different ways, to directly compare a `case`-based description against a loop-based one:

**a) Demux written with `case`** — one line per selector value.

**b) Demux written with a `for` loop** — a single indexed assignment inside the loop, functionally identical but far less repetitive in code.

### RTL — both versions

<img width="1536" height="787" alt="rtl_demux_case and demux_generate" src="https://github.com/user-attachments/assets/258938e7-c0f5-44bf-99f5-f67895bd7ca3" />

### demux_case — Simulation

<img width="1536" height="787" alt="demux_case_sim" src="https://github.com/user-attachments/assets/77eed412-4423-4882-86ae-34590e7e502e" />

### demux_case — Netlist

<img width="1536" height="787" alt="demux_case_netlist" src="https://github.com/user-attachments/assets/ad670da9-1e5e-47af-b2bd-ffe5e9de336f" />

### demux_case — Gate-Level Simulation

<img width="1536" height="787" alt="demux_case_gls" src="https://github.com/user-attachments/assets/1e51b3a3-1b77-4088-b0e3-f00017bfe0bc" />

### demux_generate — Simulation

<img width="1536" height="787" alt="demux_generate_sim" src="https://github.com/user-attachments/assets/ca559910-d5d4-49ee-9a90-b1fa8d9f0ed0" />

### demux_generate — Netlist

<img width="1536" height="787" alt="demux_generate_netlist" src="https://github.com/user-attachments/assets/e51cde48-3a55-4d84-b39c-4317f20dddec" />

### demux_generate — Gate-Level Simulation

<img width="1536" height="787" alt="demux_generate_gls" src="https://github.com/user-attachments/assets/860f3001-d818-4245-bc85-b804d055d4f4" />

---

## 6. Ripple Carry Adder Using `generate`

Unlike the procedural loops above, a `generate` block is structural — it's evaluated at elaboration time to create multiple hardware instances, not to repeat a calculation. This was used to build an 8-bit Ripple Carry Adder (RCA) out of a single 1-bit full-adder module, instantiated 8 times with the carry chained between stages.

```
num1[0..7], num2[0..7], cin
        ↓
   FA0 → FA1 → FA2 → ... → FA7
        ↓
      cout, sum[0..7]
```

Since the RCA depends on the full-adder module, both files had to be compiled together:

```bash
iverilog fa.v rca.v tb_rca.v
./a.out
gtkwave tb_rca.vcd
```

### rca — RTL

<img width="1536" height="787" alt="rca_rtl" src="https://github.com/user-attachments/assets/6f239451-9e4b-4666-8469-e7d081c63ed1" />

### rca — Simulation

<img width="1536" height="787" alt="rca_sim" src="https://github.com/user-attachments/assets/ec3c1d06-b4fd-4758-96dd-4adcc481924f" />

### rca — Netlist

<img width="1536" height="787" alt="rca_netlist" src="https://github.com/user-attachments/assets/265e4c62-76cd-43cc-a4bf-b76030549dcb" />

### rca — Gate-Level Simulation

<img width="1536" height="787" alt="rca_gls" src="https://github.com/user-attachments/assets/50f14570-a108-4eb1-a4c8-47bd9afb277e" />

---

## 7. Key Takeaways

- An `if` without an `else`, or a `case` without full coverage / a `default`, leaves the output undriven in some condition — synthesis fills that gap with a latch.
- Overlapping `case` conditions (e.g. wildcard patterns colliding with explicit ones) create priority ambiguity that can surface as an RTL-vs-GLS mismatch.
- A procedural `for` loop is for repeating operations inside `always`/`initial` blocks (mux, demux logic); a `generate` block is for structurally replicating hardware instances (the RCA's chain of full adders).
- Hierarchical designs (e.g. `rca.v` depending on `fa.v`) need every source file passed to the compiler together, or elaboration fails with an "unknown module" error.
- Comparing RTL sim → netlist → GLS at each step was the consistent way to catch and confirm each of the above issues.
