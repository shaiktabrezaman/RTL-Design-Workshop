# Day 1 – Exploring Verilog RTL Design Through Simulation and Synthesis

## Overview

The objective of this experiment was to understand the fundamentals of **Register Transfer Level (RTL) design using Verilog**. The experiment focused on understanding the roles of a design, testbench, and simulator, followed by compiling and simulating a Verilog design using **Icarus Verilog (iverilog)**. The simulated output was verified using **GTKWave**, and the RTL design was subsequently synthesized using **Yosys** to observe its corresponding gate-level netlist.

A **2-to-1 Multiplexer** was implemented as the practical RTL design for this experiment. The complete process provided an introduction to the RTL-to-netlist workflow used in digital IC design.

---

## Contents

- [RTL Verification Fundamentals](#1️⃣-rtl-verification-fundamentals)
- [Simulation Workflow with Icarus Verilog](#2️⃣-simulation-workflow-with-icarus-verilog)
- [2:1 Multiplexer – Simulation](#2️⃣-21-multiplexer--simulation)
- [Multiplexer Design Explanation](#4️⃣-multiplexer-design-explanation)
- [RTL Synthesis Using Yosys](#5️⃣-rtl-synthesis-using-yosys)
- [Synthesized Netlist](#6️⃣-synthesized-netlist)
- [Observation](#7️⃣-observation)
- [What I Learned](#8️⃣-what-i-learned)
- [Conclusion](#9️⃣-conclusion)

---

# 1️⃣ RTL Verification Fundamentals

## Simulator

A **simulator** is a software application used to execute a digital design in a virtual environment and observe its behavior without physically implementing the circuit. It allows different input conditions to be applied to the design and helps verify whether the outputs behave as expected. In this experiment, **Icarus Verilog** was used as the simulator.

## Design

The **design** is the Verilog RTL module that describes the functionality of the digital circuit. It defines the inputs, outputs, and logical behavior of the circuit. In this experiment, the design represents a **2-to-1 Multiplexer**, where the output is selected from two input signals based on a select signal.

## Testbench

A **testbench** is a separate Verilog module used to verify the functionality of the design. It provides different combinations of input signals to the **Design Under Test (DUT)** and allows the resulting output to be observed. The testbench also generates the **Value Change Dump (.vcd)** file used for waveform analysis in GTKWave.

---

# 2️⃣ Simulation Workflow with Icarus Verilog

**Icarus Verilog (iverilog)** is an open-source Verilog compiler and simulator used to compile and execute Verilog designs. The design and its testbench are compiled together, after which the simulation is executed to generate a waveform file in **VCD (Value Change Dump)** format. This file can then be opened and analyzed using **GTKWave**.

## Simulation Flow

<img width="1318" height="530" alt="Screenshot 2026-08-09 000249" src="https://github.com/user-attachments/assets/b2d2a7f7-a4de-4199-962a-9a4af6b66067" />

---

# 3️⃣ 2:1 Multiplexer – Simulation

## Step 1 – Compile the Design

The Verilog design and its testbench were compiled using **Icarus Verilog**.

```text
iverilog good_mux.v tb_good_mux.v
```

The command compiles the RTL design and testbench and generates the simulation executable.

## Step 2 – Execute the Simulation

The compiled simulation was executed using:

```text
./a.out
```

The testbench applies different input combinations during simulation and generates the corresponding VCD waveform file.

## Step 3 – Open the Waveform

The generated waveform file was opened using **GTKWave**:

```text
gtkwave tb_good_mux.vcd

```

The input signals, select signal, and output signal were then observed to verify the functional behavior of the multiplexer.

### GTKWave Simulation Waveform

<img width="1146" height="1078" alt="image" src="https://github.com/user-attachments/assets/3aa5c0f5-4e80-438f-aa95-e55f4aa5e3f0" />

---

# 4️⃣ Multiplexer Design Explanation

## 2-to-1 Multiplexer

A **2-to-1 Multiplexer** is a combinational digital circuit that selects one of two input signals and connects the selected signal to the output. The selection is controlled by a single select signal.

### Inputs

- `i0` – First input
- `i1` – Second input
- `sel` – Select signal

### Output

- `y` – Multiplexer output

### Operation

- When `sel = 0`, the output `y` follows `i0`.
- When `sel = 1`, the output `y` follows `i1`.

## Verilog RTL Design

The 2-to-1 multiplexer was implemented using an `always @(*)` combinational block.

```verilog
module good_mux (
    input i0,
    input i1,
    input sel,
    output reg y
);

always @(*)
begin
    if (sel)
        y <= i1;
    else
        y <= i0;
end

endmodule
```

### RTL Design And Test Bench

<img width="1146" height="1079" alt="image" src="https://github.com/user-attachments/assets/aae11b8e-433a-4fb2-b2e1-9d7147d2dd7c" />

---

# 5️⃣ RTL Synthesis Using Yosys

After verifying the functionality of the RTL design through simulation, the `good_mux` design was synthesized using **Yosys**. Yosys reads the Verilog RTL, performs synthesis and technology mapping using the **SkyWater SKY130 standard-cell library**, and generates a gate-level netlist.

The synthesis process was performed using the following Yosys commands.

## Step 1 – Load the Standard Cell Library

The SKY130 standard-cell Liberty file was loaded using:

```text
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```

This command loads the timing and cell information from the SKY130 standard-cell library into Yosys for use during technology mapping.

## Step 2 – Read the Verilog RTL

The RTL design was loaded into Yosys using:

```text
read_verilog good_mux.v
```

Yosys parses the Verilog source and generates an internal RTL representation of the `good_mux` module.

## Step 3 – Perform RTL Synthesis

The synthesis process was performed with:

```text
synth -top good_mux
```

The `-top good_mux` option specifies `good_mux` as the top-level module for synthesis.

Yosys performs several synthesis passes to optimize and prepare the design for technology mapping.

## Step 4 – Visualize the Synthesized Design

The synthesized design was visualized using:

```text
show
```

This command generates a **Graphviz representation** of the current design and opens the resulting schematic using the available graphical viewer.


## Step 5 – Technology Mapping Using ABC

The synthesized logic was mapped to cells from the SKY130 standard-cell library using:

```text
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```

The `abc` command performs technology mapping using the specified Liberty library. This converts the synthesized logic into available standard cells from the target technology.

## Step 6 – Generate the Gate-Level Netlist

The synthesized design was written back into Verilog netlist format using:

```text
write_verilog good_mux_netlist.v
```

This generates the synthesized gate-level Verilog file:

```text
good_mux_netlist.v
```

A version without synthesis attributes was also generated using:

```text
write_verilog -noattr good_mux_netlist.v
```

The `-noattr` option removes additional synthesis attributes from the generated Verilog netlist.

---

# 6️⃣ Synthesized Netlist

The synthesized design was visualized as a **gate-level netlist**. The netlist represents the structural implementation of the original RTL design using cells from the target technology library.

For the 2-to-1 multiplexer, the synthesized netlist shows the input signals `i0`, `i1`, and `sel` connected to the corresponding multiplexer cell, with the resulting signal connected to the output `y`.

### Synthesized Netlist of 2:1 Multiplexer

<img width="1153" height="1079" alt="image" src="https://github.com/user-attachments/assets/26f9a497-34b4-4642-8af7-6bfb408458d7" />

---

# 7️⃣ Observation

The simulation confirmed the expected behavior of the 2-to-1 multiplexer. When the select signal was low, the output followed `i0`, and when the select signal was high, the output followed `i1`. The GTKWave waveform was used to verify these signal transitions.

The RTL design was then successfully synthesized using Yosys. The resulting netlist provided a structural representation of the multiplexer using a technology-specific standard cell.

---

# 8️⃣ What I Learned

Through this experiment, I learned:

- Fundamentals of **RTL design using Verilog**.
- The purpose of a **design module, testbench, and simulator**.
- How to compile Verilog designs using **Icarus Verilog**.
- How to execute a Verilog simulation.
- How to generate and analyze **VCD waveform files**.
- How to use **GTKWave** for functional verification.
- The basic concept of **RTL synthesis**.
- How **Yosys** is used for synthesizing Verilog RTL.
- How RTL can be converted into a **gate-level netlist**.
- The role of a **standard-cell library** during technology mapping.
- How **ABC** performs technology mapping using a Liberty library.
- The difference between **simulation/verification** and **synthesis**.

---

# 9️⃣ Conclusion

This experiment provided a practical introduction to the **RTL design, simulation, verification, and synthesis workflow**. A **2-to-1 multiplexer** was designed using Verilog and verified through simulation using **Icarus Verilog** and **GTKWave**. The same RTL design was subsequently synthesized using **Yosys**, technology mapped using the **SKY130 standard-cell library**, and converted into a gate-level netlist.

The experiment established a foundation for understanding how a Verilog RTL description progresses from functional simulation to a synthesized hardware representation.
