# BabySoC: Hands-on Functional Modeling

This guide provides step-by-step instructions to explore the functional modeling and simulation of the BabySoC design. It uses **Icarus Verilog** for simulation and **GTKWave** for waveform visualization, tying theoretical SoC principles to practical experiments.

---

## Table of Contents

1. [Clone the VSDBabySoC Repository](#clone-the-vsdbabysoc-repository)
2. [Explore Repository Structure](#explore-repository-structure)
3. [Verilog RTL Modules](#verilog-rtl-modules)
4. [Simulating RTL Modules](#simulating-rtl-modules)
5. [Pre-synthesis Simulation](#pre-synthesis-simulation)
6. [Signal Analysis](#signal-analysis)
7. [Post-synthesis Simulation](#post-synthesis-simulation)
8. [Comparison: Pre- vs Post-synthesis](#comparison-pre--vs-post-synthesis)
9. [Summary](#summary)

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
---
## Explore Repository Structure

Important files and folders inside `VSDBabySoC`:
```bash
VSDBabySoC/
├── LICENSE
├── Makefile
├── README.md
├── images/
└── src/
├── module/
│ ├── avsddac.v # DAC Module
│ ├── avsdpll.v # PLL Module
│ ├── rvmyth.tlv # RISC-V CPU Core source
│ └── vsdbabysoc.v # Top-level module
├── include/
│ ├── sandpiper.vh
│ ├── sandpiper_gen.vh
```

---

## Verilog RTL Modules

- **`avsddac.v`**: Digital-to-Analog Converter implementation.
- **`avsdpll.v`**: Phase-Locked Loop for clock generation.
- **`rvmyth.tlv`**: Behavioral RISC-V CPU core written in TL-Verilog.
- **`vsdbabysoc.v`**: Top-level integration module combining CPU, PLL, DAC.

---

## Simulating RTL Modules

### Compiling RTL Sources

Before compiling, convert the TL-Verilog source:

```bash
python3 -m sandpiper -i /home/janadinisk/vsd/VLSI/VSDBabySoC/src/module/rvmyth.tlv -o rvmyth.v --bestsv --noline -p verilog --outdir /home/janadinisk/vsd/VLSI/VSDBabySoC/src/module
```

Example simulation command for `avsddac.v`:
```bash
iverilog -o /home/janadinisk/vsd/VLSI/avsddac.vvp /home/janadinisk/vsd/VLSI/VSDBabySoC/src/module/avsddac.v /home/janadinisk/vsd/VLSI/tb_avsddac.v
vvp /home/janadinisk/vsd/VLSI/avsddac.vvp
gtkwave /home/janadinisk/vsd/VLSI/tb_avsddac.vcd
```

Similarly, compile and simulate other modules (`avsdpll.v`, `rvmyth.v`) with appropriate testbenches.

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

---

## Signal Analysis

Key simulation signals to observe:

- `CLK`: Clock from PLL feeding the CPU core.
- `reset`: External reset to initialize CPU.
- `OUT`: DAC analog output (digital simulation approximation).
- `RV_TO_DAC[9:0]`: 10-bit digital output from CPU to DAC.
- `OUT (real)`: Analog real-valued DAC output for simulation purposes.

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

### Launch `yosys` and run:

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

### Compile and simulate synthesized netlist:

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

---

## Comparison: Pre- vs Post-synthesis

Both pre-synthesis and post-synthesis simulations align well, confirming that the design works correctly after synthesis.

---

## Summary

This project demonstrates the full functional modeling and validation workflow of BabySoC—an educational system-on-chip integrating a RISC-V CPU, PLL clock generation, and DAC output. From high-level RTL simulation to gate-level post-synthesis verification, it helps bridge the gap between architectural theory and real hardware implementation with hands-on simulations in Icarus Verilog and GTKWave.

---








