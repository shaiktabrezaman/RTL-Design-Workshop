# Module 3 – Combinational and Sequential Optimizations

## Overview

This module focuses on understanding how synthesis tools optimize **combinational and sequential logic** described using Verilog RTL.

The concepts covered include **constant propagation** and **Boolean logic optimization** for combinational circuits, followed by **sequential constant propagation** for sequential circuits. These concepts were then explored practically using Yosys through different RTL optimization and synthesis exercises.

The implementations completed in this module are:

- Optimization Check (`opt_check` to `opt_check4`)
- D Flip-Flop with Constant (`dff_const1` to `dff_const5`)
- Counter Optimization (`counter_opt` and `counter_opt2`)

---

## Contents

- [Combinational Logic Optimization](#1-combinational-logic-optimization)
- [Optimization Check](#2-optimization-check)
- [Sequential Logic Optimization](#3-sequential-logic-optimization)
- [D Flip-Flop with Constant](#4-d-flip-flop-with-constant)
- [Counter Optimization](#5-counter-optimization)
- [Key Observations](#6-key-observations)
- [Conclusion](#7-conclusion)

---

# 1. Combinational Logic Optimization

Combinational logic produces outputs based only on the present values of its inputs. During synthesis, the RTL description can often be simplified without changing its functional behavior. Two important optimization techniques covered in this module are **constant propagation** and **Boolean logic optimization**.

### Constant Propagation

**Constant propagation** is an optimization technique in which known constant values are propagated through the logic. This allows unnecessary logic to be removed or simplified during synthesis. For example, if a logic expression contains a signal whose value is known to be constant, Yosys can use that information to simplify the resulting circuit.

### Boolean Logic Optimization

**Boolean logic optimization** simplifies logical expressions while preserving their original functionality. Yosys analyzes the Boolean relationships in the RTL and can reduce unnecessary gates or combine logic into a simpler implementation.

---
# 2. Optimization Check

## Objective

The optimization check experiments were performed to observe how different combinational RTL descriptions are simplified during synthesis.

Four different RTL cases were implemented and analyzed using Yosys.

### `opt_check`

The RTL uses a conditional expression `a ? b : 0`, which is optimized to a **2-input AND gate**.

### Synthesized Result

<img width="1536" height="787" alt="opt_check1_prf" src="https://github.com/user-attachments/assets/38c52115-86d5-4b4a-871c-e009d870f8e7" />


### `opt_check2`

The expression `a ? 1 : b` is optimized by Yosys to a **2-input OR gate**.

### Synthesized Result

<img width="1536" height="787" alt="opt_check2_prf" src="https://github.com/user-attachments/assets/5bb2d712-7ed3-48ab-80f8-1020da302fb9" />


### `opt_check3`

The nested conditional expression `a ? (c ? b : 0) : 0` is simplified to a **3-input AND gate**.

### Synthesized Result

<img width="1536" height="787" alt="opt_check3_prf" src="https://github.com/user-attachments/assets/5797b4bd-0644-470b-b850-5b6ca4d333a4" />


### `opt_check4`

The complex conditional expression is optimized to an **XNOR gate between `a` and `c`**, while `b` becomes unused.

### Synthesized Result

<img width="1536" height="787" alt="opt_check4_prf" src="https://github.com/user-attachments/assets/3c8e1c20-3123-4a60-a815-3a8baa6866e0" />


### Observation

The four cases show how Yosys converts conditional RTL expressions into simpler equivalent gate-level logic.

---
# 3. Sequential Logic Optimization

Sequential circuits contain storage elements such as flip-flops and their behavior depends on clock and reset signals.

In this module, the main sequential optimization concept covered was **sequential constant propagation**.

### Sequential Constant Propagation

Sequential constant propagation identifies constant values associated with sequential logic and uses them during synthesis to simplify the resulting circuit.

This can allow redundant logic or unnecessary connections to be removed while maintaining the original behavior of the design.

Advanced sequential optimization topics such as **state optimization, retiming, and sequential logic cloning** were introduced but were outside the scope of the current implementation.

---
# 4. D Flip-Flop with Constant

## Objective

The `dff_const` experiments were performed to understand how Yosys handles constant values associated with sequential elements such as D flip-flops.

Five cases were implemented:

- `dff_const1`
- `dff_const2`
- `dff_const3`
- `dff_const4`
- `dff_const5`

---

## `dff_const1`

The first D flip-flop case uses a reset condition to assign `q = 0`, while the normal clocked operation assigns `q = 1`.

### Simulation

The waveform shows `q` remaining at logic `1` when reset is inactive.

<img width="1536" height="787" alt="dff1_sim" src="https://github.com/user-attachments/assets/5914395b-646f-4b85-b238-3943a01760e9" />

### Synthesized Netlist

Yosys reduces the logic to a D flip-flop with its D input tied to constant `1` and an active-low reset path.

<img width="1536" height="787" alt="dff1_prf" src="https://github.com/user-attachments/assets/031ed904-1297-4f81-b67c-6f81ff6cec03" />

---

## `dff_const2`

The second case assigns `q = 1` in both reset and normal conditions, making the output independent of the clock and reset.

### Simulation

The waveform shows `q` remaining constant at logic `1` throughout the simulation.

<img width="1536" height="787" alt="dff2_sim" src="https://github.com/user-attachments/assets/98c42dd1-cc95-41ef-9b17-d3822816c7c3" />

### Synthesized Netlist

Yosys optimizes the sequential logic completely and connects the output `q` directly to constant `1`.

<img width="1536" height="787" alt="dff2_prf" src="https://github.com/user-attachments/assets/3279a386-383a-4366-88cc-2182f303563a" />

---

## `dff_const3`

The third case uses two registers, where `q1` is assigned `1` and `q` follows the value of `q1`.

### Simulation

The waveform shows `q1` becoming `1` first, followed by `q` changing to `1` on the next clock edge.

<img width="1536" height="787" alt="dff3_sim" src="https://github.com/user-attachments/assets/dcc388e4-7a84-45ae-be12-7d081711153c" />

### Synthesized Netlist

The synthesized circuit contains two D flip-flops, with the first register receiving constant `1` and the second register driven by `q1`.

<img width="1536" height="787" alt="dff3_prf" src="https://github.com/user-attachments/assets/225f7d26-b48f-4541-a4a3-fd0c32afe246" />

---

## `dff_const4`

The fourth case initializes both `q` and `q1` to `1`, and `q` continues to follow `q1` during normal operation.

### Simulation

The waveform shows both `q1` and `q` remaining at logic `1` after reset.

<img width="1536" height="787" alt="dff4_sim" src="https://github.com/user-attachments/assets/0e583598-5a70-4289-8845-139d79b2db84" />

### Synthesized Netlist

Yosys removes the redundant sequential logic and connects both `q1` and `q` directly to constant `1`.

<img width="1536" height="787" alt="dff4_prf" src="https://github.com/user-attachments/assets/510dbe1e-7843-4bb6-b8ac-e5aa4b60d69e" />

---

## `dff_const5`

The fifth case resets both registers to `0`, then sets `q1` to `1` while `q` follows the previous value of `q1`.

### Simulation

The waveform shows `q1` changing to `1` first, while `q` changes to `1` on the following clock cycle.

<img width="1536" height="787" alt="dff5_sim" src="https://github.com/user-attachments/assets/5d7c7ced-9329-4c12-baee-d8c77530c6bc" />

### Synthesized Netlist

The synthesized circuit retains two D flip-flops, with the first D input tied to constant `1` and the second driven by `q1`.

<img width="1536" height="787" alt="dff5_prf" src="https://github.com/user-attachments/assets/f2cc3f0c-b7e7-494b-b6fc-925d0ed4927e" />

---

## Observation

The five cases demonstrate how Yosys applies constant propagation to sequential circuits. Depending on the RTL description, redundant flip-flops are either simplified to constants or retained when their clocked behavior affects the output.

---
# 5. Counter Optimization

## Objective

The counter implementations were used to observe how different RTL descriptions of sequential logic affect the resulting synthesized hardware.

A **3-bit counter** was implemented and synthesized using Yosys.

---

## `counter_opt.v`

The first counter implementation uses a 3-bit register that increments on every positive edge of the clock.

```verilog
module counter_opt (
    input clk,
    input reset,
    output q
);

reg [2:0] count;

assign q = count[0];

always @(posedge clk, posedge reset)
begin
    if (reset)
        count <= 3'b000;
    else
        count <= count + 1;
end

endmodule
````

### Working

The counter is reset to `000` when the reset signal is asserted. During normal operation, the counter increments by one on every positive clock edge.

The output is connected to the least significant bit:

```verilog
assign q = count[0];
```

Therefore, the counter progresses through:

```text
000 → 001 → 010 → 011 → 100 → 101 → 110 → 111 → ...
```

### Synthesized Netlist

<img width="1536" height="787" alt="counter_opt_prf" src="https://github.com/user-attachments/assets/cb313eba-feb3-43ee-b915-d9bd274a6dc3" />

---

## `counter_opt2.v`

A second version of the counter was implemented by changing the way the output is generated.

```verilog
module counter_opt (
    input clk,
    input reset,
    output q
);

reg [2:0] count;

assign q = (count[2:0] == 3'b100);

always @(posedge clk, posedge reset)
begin
    if (reset)
        count <= 3'b000;
    else
        count <= count + 1;
end

endmodule
```

### Working

The counter operation remains the same, but the output is now generated by comparing the counter value with `3'b100`.

```verilog
assign q = (count[2:0] == 3'b100);
```

As a result, `q` becomes HIGH when the counter reaches `100` and remains LOW for the other counter states.

### Synthesized Netlist

<img width="1536" height="787" alt="counter_opt2_prf" src="https://github.com/user-attachments/assets/26b2774c-fd8d-4462-bb1e-9056f39ad8ca" />

The counter itself remains the same in both implementations. The main difference is the output logic, which results in different combinational logic being present in the synthesized representation.

---

# 6. Key Observations

* Constant propagation can simplify RTL when signal values are known during synthesis.
* Boolean logic optimization can reduce a complex RTL expression into simpler equivalent logic.
* Sequential constant propagation can simplify logic associated with flip-flops.
* Different RTL descriptions can produce different synthesized structures even when they contain similar sequential behavior.
* Yosys performs optimization before generating the final synthesized representation.
* The synthesized netlist provides a hardware-level view of the RTL design.

---

# 7. Commands Used

The designs were processed using the **Yosys synthesis environment** with the SKY130 standard-cell library.

## Starting Yosys

```text
yosys
````

Yosys was started from the terminal to perform the synthesis and optimization operations.

## Loading the Standard-Cell Library

```text
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```

The SKY130 Liberty file was loaded to provide the standard-cell information required for technology mapping.

## Reading the RTL

```text
read_verilog <design_name>.v
```

The Verilog RTL design was loaded into Yosys.

## Selecting the Top Module

```text
synth -top <module_name>
```

The specified module was selected as the top-level design for synthesis.

## Optimization

```text
opt_clean -purge
```

This command removes unused and redundant logic from the synthesized design.

## Flip-Flop Mapping

```text
dfflibmap -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```

The generic flip-flops were mapped to suitable flip-flop cells from the SKY130 library.

## Technology Mapping

```text
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```

ABC performs technology mapping and maps the synthesized logic to available SKY130 standard cells.

## Viewing the Synthesized Design

```text
show
```

The synthesized design was displayed as a schematic to examine the optimized circuit.

---

# Conclusion

Module 3 provided practical exposure to **combinational and sequential logic optimization** using Verilog and Yosys.

The optimization-check experiments demonstrated how combinational logic can be simplified during synthesis. The `dff_const` experiments provided an understanding of constant propagation in sequential logic, while the counter implementations showed how different RTL descriptions can result in different synthesized logic.

Overall, the module helped in understanding how synthesis tools analyze RTL, apply optimization techniques, and convert the resulting design into a hardware-oriented netlist representation.

---
