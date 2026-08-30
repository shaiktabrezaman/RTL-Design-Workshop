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

The RTL was synthesized in Yosys, mapping the design onto the SKY130 standard-cell library along with the PLL and DAC liberty models.

```text
read_verilog src/module/vsdbabysoc.v
read_verilog -I src/include/ src/module/rvmyth.v
read_verilog -I src/include/ src/module/clk_gate.v

read_liberty -lib src/lib/avsdpll.lib
read_liberty -lib src/lib/avsddac.lib
read_liberty -lib src/lib/sky130_fd_sc_hd__tt_025C_1v80.lib

synth -top vsdbabysoc

dfflibmap -liberty src/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
opt
abc -liberty src/lib/sky130_fd_sc_hd__tt_025C_1v80.lib

setundef -zero
clean -purge
rename -enumerate

stat
write_verilog -noattr baby_soc_net.v
```

The `stat` command was used at the end to review the final cell count and confirm the design mapped cleanly onto the SKY130 cells.



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
iverilog -DPOST_SYNTH_SIM -DFUNCTIONAL \
-I src/include/ \
-I ../../sky130RTLDesignAndSynthesisWorkshop/my_lib/verilog_model/ \
-I src/module/ \
src/module/testbench.v

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
