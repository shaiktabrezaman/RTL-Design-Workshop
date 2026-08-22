# Day 4 – Gate-Level Simulation, Blocking/Non-Blocking Assignments, and Synthesis-Simulation Mismatch

## Overview

Day 4 focuses on validating RTL designs after synthesis and on identifying coding practices that cause a mismatch between RTL simulation and the actual synthesized hardware.

Topics covered:

- Gate-Level Simulation (GLS)
- RTL vs GLS comparison
- Synthesis-simulation mismatch
- Incomplete sensitivity lists
- Blocking vs non-blocking assignments
- Yosys synthesis flow
- SDF-based timing simulation

---

## 1. What is Gate-Level Simulation

Gate-Level Simulation (GLS) means simulating the **synthesized netlist** rather than the original RTL. The netlist is built from standard cells (muxes, gates, flip-flops), so GLS is used to confirm that synthesis preserved the intended functionality, and — when SDF is added — to check timing behavior as well.

**Flow:**

```
RTL → RTL Sim → Synthesis → Netlist → GLS → Compare Waveforms
```

---

## 2. RTL Simulation vs GLS

RTL simulation runs the behavioral Verilog (`always`, `if/case`, assignments), while GLS runs the post-synthesis gate/cell-level structure. RTL sim checks whether the *description* is correct; GLS checks whether the *hardware* Yosys generated behaves the same way.

---

## 3. Incomplete Sensitivity List Issue

A combinational `always` block that doesn't list every signal it reads can behave like a latch in simulation, even though the intent was pure combinational logic.

Example RTL used for this exercise: a 2:1 mux written with a ternary-style `if/else`, but with the sensitivity list containing only `sel`.

Expected behavior:

```
sel = 0 → y = i0
sel = 1 → y = i1
```

Problem: since `i0` and `i1` are not in the sensitivity list, changes to them don't re-trigger the block, so `y` can go stale.

**Fix options:**

```verilog
always @(i0 or i1 or sel)   // explicit list
always @(*)                 // preferred in Verilog
always_comb                 // SystemVerilog
```
### Ternary Operator MUX — RTL Simulation
<img width="1536" height="787" alt="rtl_sim_ternary_mux" src="https://github.com/user-attachments/assets/7bde752d-f1a3-4747-918b-019ba0fce06b" />

### Bad MUX — RTL Simulation
<img width="1536" height="787" alt="bad_mux_sim_rtl" src="https://github.com/user-attachments/assets/ecaa1d91-4d16-49b1-958a-4435733c718b" />

---

## 4. Sensitivity List Experiment

A simple testbench toggles `i0`, `sel`, and `i1` at fixed intervals and dumps a VCD for waveform inspection in GTKWave.

Steps performed:

1. Compile RTL + testbench with `iverilog`
2. Run the simulation executable
3. View the `.vcd` in `gtkwave`

---

## 5. Synthesis and GLS Flow

The mux RTL was synthesized in Yosys against the SKY130 liberty library:

```
read_liberty -lib <sky130 lib file>
read_verilog <design>.v
synth -top <module>
abc -liberty <sky130 lib file>
write_verilog -noattr <design>_netlist.v
```

The resulting netlist was then simulated using the SKY130 primitive/standard-cell models along with the same testbench, so the netlist behavior could be compared directly against the earlier RTL run.

```
iverilog primitives.v sky130_fd_sc_hd.v <netlist>.v <testbench>.v -o gls.out
./gls.out
gtkwave <dump>.vcd
```

### Ternary Operator MUX — NetList
<img width="1536" height="787" alt="prf_ternary" src="https://github.com/user-attachments/assets/8e4cf895-4975-404b-a939-e667d1984811" />

### Ternary Operator MUX — GLS
<img width="1536" height="787" alt="gls_sim_ternary_mux" src="https://github.com/user-attachments/assets/98eaf61f-fa07-4d2d-be9d-dbe8b99f594e" />

### Bad MUX — GLS
<img width="1536" height="787" alt="sim_gls_bad_mux" src="https://github.com/user-attachments/assets/914017af-4e58-4466-9d16-c75b8c043b87" />

---

## 6. Blocking Assignment Ordering Issue

Blocking assignments (`=`) execute strictly in the order they're written. If a signal is *used* before it's *updated* within the same block, the block reads a stale value — a classic simulation/synthesis mismatch source.

Example pattern used in this exercise:

```verilog
always @(*) begin
    d = x & c;   // uses old x
    x = a | b;   // x updated after
end
```

Here `d` is computed from the previous value of `x`, not the one being assigned in the same block — so simulated `d` doesn't match the intended `(a | b) & c` logic.

**Corrected ordering:**

```verilog
always @(*) begin
    x = a | b;
    d = x & c;
end
```
### Blocking Caveat — NetList
<img width="1536" height="787" alt="blocking_caveat_netlist" src="https://github.com/user-attachments/assets/d310cb80-ba36-4310-be1b-241dacf7b4b7" />

### Blocking Caveat — RTL
<img width="1536" height="787" alt="blocking_rtl_sim" src="https://github.com/user-attachments/assets/0165b118-c78c-4418-aa7b-e607bf32b67c" />

### Blocking Caveat — GLS
<img width="1536" height="787" alt="gls_blocking_sim" src="https://github.com/user-attachments/assets/8818f5a5-4697-4ba1-ac8c-707adda8d56e" />


---

## 7. Blocking vs Non-Blocking Assignments

| | Blocking (`=`) | Non-Blocking (`<=`) |
|---|---|---|
| Executes | Immediately, in program order | Scheduled, updates at end of time step |
| Typical use | Combinational logic | Sequential logic (clocked) |

Non-blocking is preferred for sequential logic because it models the parallel nature of flip-flops correctly — e.g., in a 2-stage shift register, both flops should sample their inputs "at the same instant" on the clock edge:

```verilog
always @(posedge clk) begin
    q1 <= d;
    q2 <= q1;   // gets the OLD q1, not the one just assigned
end
```

Using blocking assignments here would incorrectly let `q2` pick up the new `q1` value in the same cycle.

---

## 8. SDF Timing Simulation

GLS can additionally be run with an SDF (Standard Delay Format) file back-annotated, which adds real cell/interconnect delays to the simulation — useful for catching setup/hold-type timing issues that a purely functional GLS run wouldn't reveal.

```
RTL → Synthesis → Netlist → + SDF → Timing-aware GLS
```

---

## 9. Key Takeaways

- GLS validates the *synthesized* design, not just the RTL description.
- Missing signals in a sensitivity list can silently break combinational logic.
- Statement order matters with blocking assignments — read-before-write bugs are easy to introduce.
- Non-blocking assignments are the correct choice for clocked/sequential logic.
- Comparing RTL and GLS waveforms is the practical way to catch a synthesis-simulation mismatch.

---

## 10. Conclusion

Day 4 tied together gate-level simulation, sensitivity-list correctness, and blocking vs non-blocking assignment behavior — three of the most common places where RTL and synthesized hardware can diverge if coding guidelines aren't followed.
