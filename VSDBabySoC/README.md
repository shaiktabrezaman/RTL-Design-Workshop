# VSDBabySoC — RTL Simulation, Synthesis, and Gate-Level Simulation

## Overview

This folder documents the functional verification flow carried out on **VSDBabySoC**, a small SoC built around the `rvmyth` RISC-V core along with an analog PLL (`avsdpll`) and DAC (`avsddac`). The flow followed here is: clone the design, run RTL simulation, synthesize it in Yosys against the SKY130 standard-cell library, inspect the resulting netlist, run gate-level simulation (GLS) on that netlist, and finally compare the RTL and GLS waveforms to confirm they match.

<p align="center">
  <img width="601" height="332" alt="VSDBabySoC" src="https://github.com/user-attachments/assets/21bffbe6-56b3-4b3c-a8b4-f26b8e32911c" />
</p>

<p align="center">
  <sub>Source: VLSI System Design (VSD)</sub>
</p>

---

## Contents

- [1. Setting Up the Project Folder](#1-setting-up-the-project-folder)
- [2. RTL and Testbench](#2-rtl-and-testbench)
- [3. RTL Simulation](#3-rtl-simulation)
- [4. Synthesis Using Yosys](#4-synthesis-using-yosys)
- [5. Netlist Views](#5-netlist-views)
- [6. Gate-Level Simulation (GLS)](#6-gate-level-simulation-gls)
- [7. RTL vs GLS Comparison](#7-rtl-vs-gls-comparison)
- [8. Conclusion](#8-conclusion)

---

## 1. Setting Up the Project Folder

A dedicated folder was created and the BabySoC simulation repository was cloned into it.

```bash
mkdir baby_soc
cd baby_soc/
git clone https://github.com/Subhasis-Sahu/BabySoC_Simulation
cd BabySoC_Simulation
```

<img width="1536" height="319" alt="image" src="https://github.com/user-attachments/assets/64279b89-a20b-4455-8715-b34ca584a5b1" />

---

## 2. RTL and Testbench

The design consists of the top-level `vsdbabysoc` module instantiating the `rvmyth` core, along with the PLL and DAC models. A dedicated testbench drives `CLK` and `reset` and observes the DAC output path.

<img width="1536" height="787" alt="VirtualBox_vsdworkshop_30_08_2026_18_34_12" src="https://github.com/user-attachments/assets/21aa5837-7f28-49ee-9637-770f1f8a9d18" />

---

## 3. RTL Simulation

RTL simulation was run first, using a `PRE_SYNTH_SIM` define so the testbench selects the pre-synthesis simulation path.

```bash
iverilog -o ./pre_synth_sim.out -DPRE_SYNTH_SIM src/module/testbench.v -I src/include/ -I src/module/

./pre_synth_sim.out

gtkwave pre_synth_sim.vcd
```

This produces `pre_synth_sim.vcd`, opened in GTKWave to observe `CLK`, `reset`, `OUT`, and the `RV_TO_DAC[9:0]` bus.

<img width="1536" height="787" alt="9" src="https://github.com/user-attachments/assets/e1010ee3-bb93-4f25-b81a-c16e6b7d9955" />

---

## 4. Synthesis Using Yosys

The following commands were used to synthesize the design in Yosys, mapping it onto the SKY130 standard-cell library along with the PLL and DAC liberty models.
 
### Reading the Top-Level Design
 
```tcl
read_verilog src/module/vsdbabysoc.v
```
 
This reads the main top-level Verilog file into Yosys. The `vsdbabysoc` module connects the major blocks of the design.
 
---
 
### Reading the RVMYTH Module
 
```tcl
read_verilog -I src/include/ src/module/rvmyth.v
```
 
This reads the RVMYTH processor core. The `-I src/include/` option lets Yosys locate the include files that `rvmyth.v` depends on.
 
---
 
### Reading the Clock Gate Module
 
```tcl
read_verilog -I src/include/ src/module/clk_gate.v
```
 
This reads the clock gate module, again with the include directory provided. Once all three RTL files are read, Yosys has the full design hierarchy.
 
---
 
### Reading the PLL Library
 
```tcl
read_liberty -lib src/lib/avsdpll.lib
```
 
This loads the Liberty file describing the PLL block, so Yosys knows its pins and timing behavior during mapping.
 
---
 
### Reading the DAC Library
 
```tcl
read_liberty -lib src/lib/avsddac.lib
```
 
This loads the Liberty file describing the DAC block, for the same reason as above.
 
---
 
### Reading the SKY130 Standard Cell Library
 
```tcl
read_liberty -lib src/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```
 
This loads the main SKY130 standard-cell library, containing the logic gates, buffers, inverters, and flip-flops that the design will eventually be mapped onto.
 
---
 
### Selecting and Synthesizing the Top Module
 
```tcl
synth -top vsdbabysoc
```
 
This tells Yosys that `vsdbabysoc` is the top module of the design and runs Yosys's generic synthesis flow on it: elaboration, coarse-grain optimization, and technology-independent mapping into an internal logic representation.
 
---
 
### Mapping Flip-Flops
 
```tcl
dfflibmap -liberty src/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```
 
This maps the generic flip-flops produced by `synth` onto actual SKY130 flip-flop cells.
 
```text
RTL Flip-Flop
      ↓
dfflibmap
      ↓
Sky130 Flip-Flop Cell
```
 
---
 
### Generic Logic Optimization
 
```tcl
opt
```
 
This runs a round of technology-independent optimization, constant propagation and redundant logic removal, cleaning up the design before it goes into technology mapping.
 
---
 
### Technology Mapping Using ABC
 
```tcl
abc -liberty src/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```
 
The `abc` command maps the remaining combinational logic (AND, OR, NAND, NOR, inverters, and so on) onto actual SKY130 cells. This is the step that turns generic logic into technology-specific logic.
 
---
 
### Replacing Undefined Values
 
```tcl
setundef -zero
```
 
Any undefined (`x`) signals left in the design are replaced with a defined logic `0`, so the netlist doesn't carry unknown values into simulation.
 
---
 
### Cleaning the Design
 
```tcl
clean -purge
```
 
This removes unused wires, cells, and dangling nets left over after optimization and mapping.
 
---
 
### Renaming Generated Objects
 
```tcl
rename -enumerate
```
 
Yosys often generates hard-to-read internal names during synthesis. This command renames those objects with a simple enumerated scheme, making the resulting netlist easier to read.
 
---
 
### Reviewing the Synthesis Summary
 
```tcl
stat
```
 
This prints a summary of the final design (cell counts by type, wire counts, and estimated area), used here to confirm the design mapped cleanly onto the SKY130 cells.
 
---
 
### Writing Out the Netlist
 
```tcl
write_verilog -noattr baby_soc_net.v
```
 
This writes the final synthesized netlist out as a Verilog file, `baby_soc_net.v`. The `-noattr` flag strips Yosys's internal attributes so the file stays clean for gate-level simulation.

The `stat` command was used at the end to review the final cell count and confirm the design mapped cleanly onto the SKY130 cells.

<div align="center">

<table>
  <tr>
    <td align="center">
      <img width="317" height="690" alt="stat1" src="https://github.com/user-attachments/assets/4f59dca9-661f-4297-82ac-c938d4bc5253" />
    </td>
    <td align="center">
      <img width="391" height="715" alt="stat2" src="https://github.com/user-attachments/assets/9b1a4b23-23d7-413f-be06-8990e89d291c" />
    </td>
    <td align="center">
      <img width="347" height="714" alt="stat3" src="https://github.com/user-attachments/assets/2ecd0f6d-c212-4f5e-937a-a1a8cbab287b" />
    </td>
  </tr>
</table>

</div>

<div align="center">

<table>
  <tr>
    <td align="center">
      <img width="324" height="714" alt="stat4" src="https://github.com/user-attachments/assets/e408bae6-27bc-4965-8382-514fd3beec96" />
    </td>
    <td align="center">
      <img width="389" height="712" alt="stat5" src="https://github.com/user-attachments/assets/8d3f90f7-d2ff-49b5-a5e8-93aad80c888d" />
    </td>
  </tr>
</table>

</div>
  
---
## 5. Netlist Views

The synthesized netlist was viewed using the Yosys show command (rendered via the Dot Viewer). Two levels of the same netlist were captured — a top-level block view and the fully gate-mapped internal view.

### Top-Level / Block-Level Netlist

This view shows the vsdbabysoc hierarchy at the block level — the avsdpll instance driving CLK, feeding into the rvmyth core, which in turn drives the avsddac instance through the RV_TO_DAC bus, out to OUT. Port-level connections between the three sub-blocks are clearly visible here.

<img width="1536" height="787" alt="netlist" src="https://github.com/user-attachments/assets/9094fc42-f46b-44b7-868f-af4af9751cd2" />

### Gate-Level Netlist (Post-abc Mapping)

This is the same design after abc has mapped everything down to SKY130 standard cells, the dense mesh of wires reflects the actual gate-level implementation rather than the clean block hierarchy above. It's not meant to be read cell-by-cell. it's included to show the scale/density of the mapped logic compared to the block-level view.

<img width="1536" height="787" alt="6" src="https://github.com/user-attachments/assets/29a5324f-1d99-42e6-b8f1-095c919b31ec" />

---

## 6. Gate-Level Simulation (GLS)

The synthesized netlist was then simulated with the standard-cell Verilog models included, using `POST_SYNTH_SIM` and `FUNCTIONAL` defines so the testbench exercises the post-synthesis functional path.

```bash
iverilog -DPOST_SYNTH_SIM -DFUNCTIONAL -I src/include/ -I ../../sky130RTLDesignAndSynthesisWorkshop/my_lib/verilog_model/ -I src/module/ src/module/testbench.v

./a.out

gtkwave post_synth_sim.vcd
```

<img width="1536" height="787" alt="11" src="https://github.com/user-attachments/assets/470432d5-ea09-468f-b290-6108a5f523fc" />

---

## 7. RTL vs GLS Comparison

The `RV_TO_DAC[9:0]` bus, `CLK`, `reset`, and `OUT` were compared between `pre_synth_sim.vcd` (RTL) and `post_synth_sim.vcd` (GLS). Both waveforms line up bit-for-bit across the full simulation window, confirming the synthesized netlist behaves identically to the original RTL.

<img width="1536" height="787" alt="10" src="https://github.com/user-attachments/assets/00c9cf4c-5340-4532-894f-4915272c6425" />

---

## 8. Conclusion

The BabySoC design was taken through the complete verification loop, covering RTL simulation, Yosys synthesis onto the SKY130 library, netlist inspection, and gate-level simulation. The RTL and GLS waveforms matched exactly on the `RV_TO_DAC` bus and other observed signals. This confirms that synthesis preserved the intended functional behavior of the design with no mismatch introduced during the RTL-to-gate-level translation.
