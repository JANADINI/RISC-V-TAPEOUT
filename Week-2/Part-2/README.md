# BabySoC: Hands-on Functional Modeling

This guide provides step-by-step instructions to explore the functional modeling and simulation of the BabySoC design. It uses **Icarus Verilog** for simulation and **GTKWave** for waveform visualization, tying theoretical SoC principles to practical experiments.

---

## Table of Contents

1. [Clone the VSDBabySoC Repository](#clone-the-vsdbabysoc-repository)
2. [Explore Repository Structure](#explore-repository-structure)
3. [Verilog RTL Modules](#verilog-rtl-modules)
4. [Simulating RTL Modules](#simulating-individual-rtl-modules)
5. [Pre-synthesis Simulation](#pre-synthesis-simulation)
6. [Signal Analysis](#signal-analysis)
7. [Post-synthesis Simulation](#post-synthesis-simulation)
8. [Comparison: Pre- vs Post-synthesis](#comparison-pre--vs-post-synthesis)
9. [Summary & Special Features](#summary--special-features)

---

## Clone the VSDBabySoC Repository

Open a terminal and navigate to your desired working directory:
```bash
cd /home/janadinisk/vsd/VLSI/
```
Clone the BabySoC repository and navigate into it:
```bash
git clone https://github.com/manili/VSDBabySoC.git
cd VSDBabySoC
```

Confirm contents by listing files:
```bash
ls
```
![image](https://github.com/JANADINI/RISC-V-TAPEOUT/blob/main/Week-2/Part-2/Pictures/VSDBabySoC_repo_clone.png)
> [!Tip]
>  Keep your SoC projects organized in dedicated folders like `/home/janadinisk/vsd/VLSI/` to easily manage multiple designs and versions.

---
## Explore Repository Structure

Important files and folders inside `VSDBabySoC`:
```bash
VSDBabySoC/
├── LICENSE
├── Makefile
├── README.md
├── images/ # Visualization assets
└── src/
├── module/ # Core Verilog and TL-Verilog design files
│ ├── avsddac.v # 10-bit DAC module
│ ├── avsdpll.v # PLL (clock generator)
│ ├── rvmyth.tlv # RISC-V CPU core in TL-Verilog
│ └── vsdbabysoc.v # Top-level integration module
├── include/ # Header files for synthesis and compilation
└── gls_model/ # Standard cells and primitives
```
> [!Note]
> Separation of source code, headers, and libraries promotes maintainability and simplifies simulation & synthesis workflows.

---

## Verilog RTL Modules

- **`avsdpll.v`**: Contains the Phase-Locked Loop module that stabilizes and generates clock signals, critical for synchronized CPU and peripheral operation.
  
- **`rvmyth.tlv`**: The heart of BabySoC, a small RISC-V CPU core described in TL-Verilog, providing instruction execution and data generation.
  
- **`vsdbabysoc.v`**: The top-level module connecting CPU, PLL, and DAC modules, illustrating typical SoC integration.

---


## Simulating Individual RTL Modules

### Preparing the RISC-V Core

Before simulating the CPU core, translate TL-Verilog into Verilog using:

```bash
python3 -m sandpiper -i /home/janadinisk/vsd/VLSI/VSDBabySoC/src/module/rvmyth.tlv -o rvmyth.v --bestsv --noline -p verilog --outdir /home/janadinisk/vsd/VLSI/VSDBabySoC/src/module
```
![image](https://github.com/JANADINI/RISC-V-TAPEOUT/blob/main/Week-2/Part-2/Pictures/Sandpiper.png)
Example simulation command for `avsddac.v`:
```bash
iverilog -o /home/janadinisk/vsd/VLSI/avsddac.vvp /home/janadinisk/vsd/VLSI/VSDBabySoC/src/module/avsddac.v /home/janadinisk/vsd/VLSI/tb_avsddac.v
vvp /home/janadinisk/vsd/VLSI/avsddac.vvp
gtkwave /home/janadinisk/vsd/VLSI/tb_avsddac.vcd
```

> [!Tip]
>  Visualizing waveform outputs with GTKWave helps understand signal transitions and verify expected hardware behaviors.

Repeat similar steps for the PLL (`avsdpll.v`) and CPU core (`rvmyth.v`) modules with their respective testbenches.

---

## Pre-synthesis Simulation

Compile the full BabySoC design in pre-synthesis mode:

```bash
iverilog -o /home/janadinisk/vsd/VLSI/pre_synth_sim.vvp -DPRE_SYNTH_SIM
-I /home/janadinisk/vsd/VLSI/VSDBabySoC/src/include
-I /home/janadinisk/vsd/VLSI/VSDBabySoC/src/module
/home/janadinisk/vsd/VLSI/VSDBabySoC/src/module/testbench.v
```

Run simulation and view waveforms:
```bash
vvp /home/janadinisk/vsd/VLSI/pre_synth_sim.vvp
gtkwave /home/janadinisk/vsd/VLSI/pre_synth_sim.vcd
```
![image](https://github.com/JANADINI/RISC-V-TAPEOUT/blob/main/Week-2/Part-2/Pictures/pre_syn_sim.png)

Viewing DAC output in analog mode:
The Steps:
![image](https://github.com/JANADINI/RISC-V-TAPEOUT/blob/main/Week-2/Part-2/Pictures/Steps_for_Analog_signals.png)

Analog GTKWave pre_synth_sim.vcd:
![image](https://github.com/JANADINI/RISC-V-TAPEOUT/blob/main/Week-2/Part-2/Pictures/pre_syn_analog_sim.png)

---

## Signal Analysis

Key simulation signals to observe:
- **CLK (Clock signal):** Check for stable, periodic toggling that drives synchronous components.
- **reset:** Confirm it properly initializes or clears registers before operation.
- **OUT & RV_TO_DAC[9:0]:** Verify output correctness aligning with program logic.
- **Analog Signal (OUT real):** Note you simulate analog behavior using `real` datatype approximations, which helps bridge digital design with analog interfacing.

**Special Feature**:

<mark> The DAC modeled with real-number arithmetic is a unique educational aspect, allowing visualization of analog voltage levels directly computed from digital values.
</mark>

---

## Post-synthesis Simulation

### Prepare Synthesis

Copy required header files:

```bash
cp /home/janadinisk/vsd/VLSI/VSDBabySoC/src/include/sp_verilog.vh /home/janadinisk/vsd/VLSI/VSDBabySoC/
cp /home/janadinisk/vsd/VLSI/VSDBabySoC/src/include/sandpiper.vh /home/janadinisk/vsd/VLSI/VSDBabySoC/
cp /home/janadinisk/vsd/VLSI/VSDBabySoC/src/include/sandpiper_gen.vh /home/janadinisk/vsd/VLSI/VSDBabySoC/
cd /home/janadinisk/vsd/VLSI/VSDBabySoC
```

### Run synthesis commands inside `yosys` utility
Yosys:
![image](https://github.com/JANADINI/RISC-V-TAPEOUT/blob/main/Week-2/Part-2/Pictures/Yosys_tool.png)


```bash
read_verilog src/module/vsdbabysoc.v
read_verilog -I /home/janadinisk/vsd/VLSI/VSDBabySoC/src/include/ src/module/rvmyth.v
read_verilog -I /home/janadinisk/vsd/VLSI/VSDBabySoC/src/include/ src/module/clk_gate.v
read_liberty -lib src/lib/avsdpll.lib
read_liberty -lib src/lib/avsddac.lib
read_liberty -lib src/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
synth -top vsdbabysoc
dfflibmap -liberty src/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
opt
abc -liberty src/lib/sky130_fd_sc_hd__tt_025C_1v80.lib -script +strash;scorr;ifraig;retime;{D};strash;dch,-f;map,-M,1,{D}
flatten
setundef -zero
clean -purge
rename -enumerate
stat
write_verilog -noattr /home/janadinisk/vsd/VLSI/vsdbabysoc_synth.v
```

```bash
Printing statistics.

=== avsddac ===

   Number of wires:               5299
   Number of wire bits:           5411
   Number of public wires:           6
   Number of public wire bits:     118
   Number of memories:               0
   Number of memory bits:            0
   Number of processes:              0
   Number of cells:               5325
     $_ANDNOT_                    1189
     $_AND_                         84
     $_AOI3_                       335
     $_AOI4_                         3
     $_MUX_                        645
     $_NAND_                       154
     $_NOR_                        381
     $_NOT_                        307
     $_OAI3_                       256
     $_OAI4_                         5
     $_ORNOT_                      183
     $_OR_                         544
     $_XNOR_                       282
     $_XOR_                        957

=== avsdpll ===

   Number of wires:                  6
   Number of wire bits:              6
   Number of public wires:           5
   Number of public wire bits:       5
   Number of memories:               0
   Number of memory bits:            0
   Number of processes:              0
   Number of cells:                  2
     $_ANDNOT_                       1
     $_NAND_                         1

=== rvmyth ===

   Number of wires:                  4
   Number of wire bits:             66
   Number of public wires:           4
   Number of public wire bits:      66
   Number of memories:               0
   Number of memory bits:            0
   Number of processes:              0
   Number of cells:                  1
     rvmyth_gen                      1

=== rvmyth_gen ===

   Number of wires:                168
   Number of wire bits:            292
   Number of public wires:           5
   Number of public wire bits:      98
   Number of memories:               0
   Number of memory bits:            0
   Number of processes:              0
   Number of cells:                258
     $_ANDNOT_                      14
     $_AND_                          9
     $_AOI3_                        23
     $_DFF_PP0_                     64
     $_NAND_                        26
     $_NOR_                          4
     $_NOT_                         17
     $_OAI3_                        22
     $_ORNOT_                        7
     $_OR_                           9
     $_XNOR_                        21
     $_XOR_                         42

=== vsdbabysoc ===

   Number of wires:                 11
   Number of wire bits:             71
   Number of public wires:           9
   Number of public wire bits:      18
   Number of memories:               0
   Number of memory bits:            0
   Number of processes:              0
   Number of cells:                  3
     avsddac                         1
     avsdpll                         1
     rvmyth                          1

=== design hierarchy ===

   vsdbabysoc                        1
     avsddac                         1
     avsdpll                         1
     rvmyth                          1
       rvmyth_gen                    1

   Number of wires:               5488
   Number of wire bits:           5846
   Number of public wires:          29
   Number of public wire bits:     305
   Number of memories:               0
   Number of memory bits:            0
   Number of processes:              0
   Number of cells:               5585
     $_ANDNOT_                    1204
     $_AND_                         93
     $_AOI3_                       358
     $_AOI4_                         3
     $_DFF_PP0_                     64
     $_MUX_                        645
     $_NAND_                       181
     $_NOR_                        385
     $_NOT_                        324
     $_OAI3_                       278
     $_OAI4_                         5
     $_ORNOT_                      190
     $_OR_                         553
     $_XNOR_                       303
     $_XOR_                        999

10.27. Executing CHECK pass (checking for obvious problems).
checking module avsddac..
checking module avsdpll..
checking module rvmyth..
checking module rvmyth_gen..
checking module vsdbabysoc..
found and reported 0 problems.

yosys> 

```yosys> opt

13. Executing OPT pass (performing simple optimizations).

13.1. Executing OPT_EXPR pass (perform const folding).
Optimizing module avsddac.
<suppressed ~16 debug messages>
Optimizing module avsdpll.
Optimizing module rvmyth.
Optimizing module rvmyth_gen.
Optimizing module vsdbabysoc.

13.2. Executing OPT_MERGE pass (detect identical cells).
Finding identical cells in module `\avsddac'.
<suppressed ~12 debug messages>
Finding identical cells in module `\avsdpll'.
Finding identical cells in module `\rvmyth'.
Finding identical cells in module `\rvmyth_gen'.
<suppressed ~189 debug messages>
Finding identical cells in module `\vsdbabysoc'.
Removed a total of 67 cells.

13.3. Executing OPT_MUXTREE pass (detect dead branches in mux trees).
Running muxtree optimizer on module \avsddac..
  Creating internal representation of mux trees.
  No muxes found in this module.
Running muxtree optimizer on module \avsdpll..
  Creating internal representation of mux trees.
  No muxes found in this module.
Running muxtree optimizer on module \rvmyth..
  Creating internal representation of mux trees.
  No muxes found in this module.
Running muxtree optimizer on module \rvmyth_gen..
  Creating internal representation of mux trees.
  No muxes found in this module.
Running muxtree optimizer on module \vsdbabysoc..
  Creating internal representation of mux trees.
  No muxes found in this module.
Removed 0 multiplexer ports.

13.4. Executing OPT_REDUCE pass (consolidate $*mux and $reduce_* inputs).
  Optimizing cells in module \avsddac.
  Optimizing cells in module \avsdpll.
  Optimizing cells in module \rvmyth.
  Optimizing cells in module \rvmyth_gen.
  Optimizing cells in module \vsdbabysoc.
Performed a total of 0 changes.

13.5. Executing OPT_MERGE pass (detect identical cells).
Finding identical cells in module `\avsddac'.
Finding identical cells in module `\avsdpll'.
Finding identical cells in module `\rvmyth'.
Finding identical cells in module `\rvmyth_gen'.
Finding identical cells in module `\vsdbabysoc'.
Removed a total of 0 cells.

13.6. Executing OPT_RMDFF pass (remove dff with constant values).

13.7. Executing OPT_CLEAN pass (remove unused cells and wires).
Finding unused cells or wires in module \avsddac..
Finding unused cells or wires in module \avsdpll..
Finding unused cells or wires in module \rvmyth..
Finding unused cells or wires in module \rvmyth_gen..
Finding unused cells or wires in module \vsdbabysoc..
Removed 0 unused cells and 83 unused wires.
<suppressed ~2 debug messages>

13.8. Executing OPT_EXPR pass (perform const folding).
Optimizing module avsddac.
Optimizing module avsdpll.
Optimizing module rvmyth.
Optimizing module rvmyth_gen.
Optimizing module vsdbabysoc.

13.9. Finished OPT passes. (There is nothing left to do.)

yosys> 
```
```bash

---

### Post-synthesis Simulation

Prepare files:



Copy synthesized and dependency files:
```bash
cp /home/janadinisk/vsd/VLSI/VSDBabySoC/src/module/avsddac.v /home/janadinisk/vsd/VLSI
cp /home/janadinisk/vsd/VLSI/VSDBabySoC/src/module/avsdpll.v /home/janadinisk/vsd/VLSI
cp /home/janadinisk/vsd/VLSI/VSDBabySoC/src/gls_model/sky130_fd_sc_hd.v /home/janadinisk/vsd/VLSI
cp /home/janadinisk/vsd/VLSI/VSDBabySoC/src/gls_model/primitives.v /home/janadinisk/vsd/VLSI
```

Compile synthesized design with testbench:

```bash
iverilog -o /home/janadinisk/vsd/VLSI/vsdbabysoc_synth.vvp -DPOST_SYNTH_SIM -DFUNCTIONAL -DUNIT_DELAY=#1
-I /home/janadinisk/vsd/VLSI/VSDBabySoC/src/include
-I /home/janadinisk/vsd/VLSI/VSDBabySoC/src/module
-I /home/janadinisk/vsd/VLSI/VSDBabySoC/src/gls_model
/home/janadinisk/vsd/VLSI/VSDBabySoC/src/module/testbench.v
```

Run and view waveform:

```bash
vvp /home/janadinisk/vsd/VLSI/vsdbabysoc_synth.vvp
gtkwave /home/janadinisk/vsd/VLSI/post_synth_sim.vcd
```
> [!Tip]
> Post-synthesis simulation validates synthesis results and timing, ensuring your design behaves identically to the RTL model after optimization.
---

## Comparison: Pre- vs Post-synthesis

Both pre-synthesis and post-synthesis simulations align well, confirming that the design works correctly after synthesis.

**Highlight**:

<mark>
The close match between pre- and post-synthesis simulation results proves design integrity and correctness, fundamental for production-quality SoC development.</mark>

---

## Summary & Special Features

BabySoC integrates key SoC concepts in a simplified, student-friendly package:

- Minimalistic RISC-V CPU core in TL-Verilog enabling clean, understandable CPU design.
- Use of a real-valued DAC model to bridge digital-analog signal representation with behavioral simulation.
- PLL clock generator model simulating hardware timing control.
- Complete hands-on workflow covering source cloning, code compilation, simulation, synthesis, and waveform analysis.
- Clear modular design allowing focus on individual SoC blocks or full system integration.
- Open-source approach facilitating experimentation and learning.
- Encouragement to explore SoC modeling beyond theory, unlocking skills for real-world chip design.

> [!Tip]
> Regularly check signal waveforms and simulation logs to catch issues early. Use GTKWave’s measurement tools for timing analysis and signal verification.

> [!Note]
> his project enables beginners and enthusiasts to transition smoothly from SoC theory to practical implementation and verification, a crucial step in embedded and VLSI careers.

---








