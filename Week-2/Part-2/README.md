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








