
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
/home/janadinisk/vsd/VLSI/VSDBabySoC/
├── src/
│ ├── include/ # Contains header files (*.vh) with macros and reusable parameters
│ ├── module/ # Source files including Verilog and TL-Verilog for SoC modules
│ │ ├── avsddac.v # Digital-to-Analog Converter module
│ │ ├── avsdpll.v # PLL clock generator module
│ │ ├── rvmyth.tlv # RVMYTH CPU core written in TL-Verilog
│ │ ├── rvmyth.v # Verilog file generated from TLV conversion
│ │ ├── vsdbabysoc.v # Top module integrating all SoC functions
│ │ ├── testbench.v # Testbench for simulation runs
│ │ └── clk_gate.v # Clock gating control module
│ └── gls_model/ # Cell libraries and primitive gate definitions
├── images/ # Images and diagrams for documentation
├── Makefile # Makefile for build and simulation automation
├── LICENSE
├── README.md # This documentation file
├── output/              # Simulation outputs and compiled files
└── compiled_tlv/        # Intermediate TL-Verilog compilation directory

```
> [!Note]
> Separation of source code, headers, and libraries promotes maintainability and simplifies simulation & synthesis workflows.

---

## RTL Modules in VSDBabySoC

The following are the primary RTL modules that compose the VSDBabySoC design:

| Module Name      | Description                                               |
|------------------|-----------------------------------------------------------|
| `vsdbabysoc.v`   | Top-level SoC module integrating CPU, PLL, and DAC       |
| `rvmyth.tlv` / `rvmyth.v` | RISC-V CPU core implemented in TL-Verilog (source) and its converted Verilog version |
| `avsdpll.v`      | Phase-Locked Loop module responsible for clock generation |
| `avsddac.v`      | 10-bit Digital-to-Analog Converter for analog interfacing |
| `clk_gate.v`     | Clock gating module for controlling clock signals         |

Additional supporting files such as testbenches are also included but are outside the core RTL hierarchy.

All these modules are located under:

`/home/janadinisk/vsd/VLSI/VSDBabySoC/src/module/`

Together, these form the essential building blocks of the system, enabling the combined functionality of processing, clock management, and digital-to-analog conversion within the SoC.

---

## TLV to Verilog Conversion Steps

To convert the `rvmyth.tlv` TL-Verilog CPU core into Verilog in your workspace, follow these commands adjusted for your directory structure:
Step 1: Ensure python3-venv and pip are installed
```bash
sudo apt update
sudo apt install python3-venv python3-pip
```
Step 2: Create and activate a Python virtual environment in VSDBabySoC directory
```bash
cd /home/janadinisk/vsd/VLSI/VSDBabySoC/
python3 -m venv sp_env
source sp_env/bin/activate
```
Step 3: Install SandPiper-SaaS package inside the virtual environment
```bash
pip install pyyaml click sandpiper-saas
```
Step 4: Convert all TL-Verilog files (*.tlv) in src/module to Verilog
```bash
sandpiper-saas -i /home/janadinisk/vsd/VLSI/VSDBabySoC/src/module/*.tlv -o rvmyth.v --bestsv --noline -p verilog --outdir /home/janadinisk/vsd/VLSI/VSDBabySoC/src/module/
```
![image](https://github.com/JANADINI/RISC-V-TAPEOUT/blob/main/Week-2/Part-2/Pictures/Sandpiper.png)
> [!Note]
> Always activate the `sp_env` virtual environment before running the TL-Verilog conversion commands to ensure dependencies are resolved properly.
> to exit use`deactivate`

## Simulating Individual RTL Modules

### Preparing the RISC-V Core

Before simulating the CPU core, translate TL-Verilog into Verilog using:

```bash
python3 -m sandpiper -i /home/janadinisk/vsd/VLSI/VSDBabySoC/src/module/rvmyth.tlv -o rvmyth.v --bestsv --noline -p verilog --outdir /home/janadinisk/vsd/VLSI/VSDBabySoC/src/module
```
*Example simulation command for `avsddac.v`:*
<details>
    <summary>tb_avsddac.v:</summary>
    
```verilog

    
`timescale 1ns / 1ps

module tb_avsddac;

    // Inputs
    reg [9:0] D;
    reg VREFH;
    reg VREFL;

    // Output
    wire OUT;

    // Instantiate the DUT
    avsddac uut (
        .OUT(OUT),
        .D(D),
        .VREFH(VREFH),
        .VREFL(VREFL)
    );

    // Initialize signals
    initial begin
        D = 10'd0;
        VREFL = 1'b0;
        VREFH = 1'b1;
    end

    // Stimulus: simple counting for D
    initial begin
        repeat (20) begin
            #10 D = D + 1;   // increment input
        end
        $finish;
    end

    // VCD dump for waveform viewing
    initial begin
        $dumpfile("tb_avsddac.vcd");
        $dumpvars(0, tb_avsddac);
    end

endmodule
```
</details>

```bash

iverilog -o tb_avsddac.out -I src/include -I src/module src/module/avsddac.v tb_avsddac.v
vvp tb_avsddac.out
gtkwave tb_avsddac.vcd

```
**Waveform**:

![image](https://github.com/JANADINI/RISC-V-TAPEOUT/blob/main/Week-2/Part-2/Pictures/tb_avsddac.vcd.png)

> [!Tip]
>  Visualizing waveform outputs with GTKWave helps understand signal transitions and verify expected hardware behaviors.

## Repeat similar steps for the PLL (`avsdpll.v`) and CPU core (`rvmyth.v`) modules with their respective testbenches.
*Simulation command for`avsdpll.v`:*
<details>
<summary>tb_avsdpll.v</summary>
    
```verilog
    
`timescale 1ns / 1ps

module tb_avsdpll;

   // Testbench signals
   reg VCO_IN;
   reg ENb_CP;
   reg ENb_VCO;
   reg REF;
   wire CLK;

   // Instantiate DUT
   avsdpll uut (
      .CLK(CLK),
      .VCO_IN(VCO_IN),
      .ENb_CP(ENb_CP),
      .ENb_VCO(ENb_VCO),
      .REF(REF)
   );

   // Initial block to set/reset signals
   initial begin
      VCO_IN = 0;
      ENb_CP = 0;
      ENb_VCO = 0;
      REF = 0;

      #10 ENb_VCO = 1;       // Enable VCO
      #20 ENb_CP = 1;        // Enable Charge Pump

      #500 $finish;          // Stop simulation after 500ns
   end

   // Generate REF clock (example: 100MHz -> period 10ns)
   initial begin
      forever begin
         #5 REF = ~REF;       // Toggle every 5ns -> 100MHz
      end
   end

   // Generate VCO_IN signal (optional)
   initial begin
      forever begin
         #8.33 VCO_IN = ~VCO_IN; // 60MHz
      end
   end

   // Dump waveform
   initial begin
      $dumpfile("tb_avsdpll.vcd");
      $dumpvars(0, tb_avsdpll);
   end

endmodule
```
</details>

**Waveform**:

![image](https://github.com/JANADINI/RISC-V-TAPEOUT/blob/main/Week-2/Part-2/Pictures/tb_avsdpll.vcd.png)

*Simulation for`rvmyth.v`:*

<details>
<summary><b>tb_rvmyth.v</b></summary>

```verilog
`timescale 1ns/1ps

module tb_rvmyth;

    // Inputs
    reg CLK;
    reg reset;

    // 10-bit output
    wire [9:0] OUT;

    // Instantiate the rvmyth module
    rvmyth uut (
        .CLK(CLK),
        .reset(reset),
        .OUT(OUT)
    );

    // Clock generation: 10ns period -> 100 MHz
    initial CLK = 0;
    always #5 CLK = ~CLK;

    // Test sequence
    initial begin
        // Initialize dump for waveform
        $dumpfile("tb_rvmyth.vcd");
        $dumpvars(0, tb_rvmyth);

        // Reset sequence
        reset = 1;
        #20;
        reset = 0;

        // Wait some time for simulation
        #200;

        // Finish simulation
        $finish;
    end

    // Optional monitor
    initial begin
        $display("Time\tCLK\tRESET\tOUT");
        $monitor("%0t\t%b\t%b\t%0d", $time, CLK, reset, OUT);
    end

endmodule

```
</details>

**Waveform**:

![image]([https://github.com/JANADINI/RISC-V-TAPEOUT/blob/main/Week-2/Part%20-2/Pictures/tb_rvmyth.vcd.png](https://github.com/JANADINI/RISC-V-TAPEOUT/blob/main/Week-2/Part-2/Pictures/tb_rvmyth.vcd.png))


---

## Pre-synthesis Simulation


Compile the full BabySoC design in pre-synthesis mode:

```bash
cd ~/vsd/VLSI/VSDBabySoC/

mkdir -p output/pre_synth_sim


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
**Waveform**:

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
```
```bash
read_liberty -lib src/lib/avsdpll.lib
read_liberty -lib src/lib/avsddac.lib
read_liberty -lib src/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```
It will looks like:
![image]()
```bash
synth -top vsdbabysoc
```
It will looks like
<details>
    <summary>Printing Statistics</summary>
    ```shell
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
```
</details>
```bash
dfflibmap -liberty src/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```
![image](https://github.com/JANADINI/RISC-V-TAPEOUT/blob/main/Week-2/Part-2/Pictures/DFF_Cells.png)
```bash
opt
```
It will looks like:
<details>
    <summary>OPT</summary>
    ```shell
    
    yosys> opt

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
</details>
```bash

abc -liberty src/lib/sky130_fd_sc_hd__tt_025C_1v80.lib -script +strash;scorr;ifraig;retime;{D};strash;dch,-f;map,-M,1,{D}
```
![image](https://github.com/JANADINI/RISC-V-TAPEOUT/blob/main/Week-2/Part-2/Pictures/abc_results.png)
```bash
flatten
setundef -zero
clean -purge
rename -enumerate
```
```bash
stat
```
It will looks like:
<details>
    <summary>VSDBabySoC
    </summary>
    ```shell
17. Printing statistics.

=== vsdbabysoc ===

   Number of wires:               4278
   Number of wire bits:           4380
   Number of public wires:        4278
   Number of public wire bits:    4380
   Number of memories:               0
   Number of memory bits:            0
   Number of processes:              0
   Number of cells:               4298
     sky130_fd_sc_hd__a211oi_1      20
     sky130_fd_sc_hd__a21boi_0      59
     sky130_fd_sc_hd__a21o_2        27
     sky130_fd_sc_hd__a21oi_1      202
     sky130_fd_sc_hd__a22oi_1       36
     sky130_fd_sc_hd__a2bb2oi_1      1
     sky130_fd_sc_hd__a311oi_1       1
     sky130_fd_sc_hd__a31o_2         4
     sky130_fd_sc_hd__a31oi_1       49
     sky130_fd_sc_hd__a32oi_1        5
     sky130_fd_sc_hd__a41oi_1       27
     sky130_fd_sc_hd__and2_2        24
     sky130_fd_sc_hd__and2b_2        1
     sky130_fd_sc_hd__and3_2        16
     sky130_fd_sc_hd__and3b_2        1
     sky130_fd_sc_hd__clkinv_1     449
     sky130_fd_sc_hd__dfrtp_1       20
     sky130_fd_sc_hd__lpflow_inputiso0p_1      5
     sky130_fd_sc_hd__maj3_1         2
     sky130_fd_sc_hd__nand2_1      943
     sky130_fd_sc_hd__nand3_1      881
     sky130_fd_sc_hd__nand3b_1      30
     sky130_fd_sc_hd__nand4_1      220
     sky130_fd_sc_hd__nor2_1       502
     sky130_fd_sc_hd__nor3_1        55
     sky130_fd_sc_hd__nor3b_1        5
     sky130_fd_sc_hd__nor4_1        23
     sky130_fd_sc_hd__o2111a_1      29
     sky130_fd_sc_hd__o2111ai_1     17
     sky130_fd_sc_hd__o211a_1       21
     sky130_fd_sc_hd__o211ai_1      67
     sky130_fd_sc_hd__o21a_1        58
     sky130_fd_sc_hd__o21ai_0      248
     sky130_fd_sc_hd__o21bai_1      23
     sky130_fd_sc_hd__o221ai_1       1
     sky130_fd_sc_hd__o22a_2         1
     sky130_fd_sc_hd__o22ai_1       46
     sky130_fd_sc_hd__o2bb2ai_1     65
     sky130_fd_sc_hd__o31ai_1        5
     sky130_fd_sc_hd__o41ai_1        1
     sky130_fd_sc_hd__or2_2         11
     sky130_fd_sc_hd__or2b_2         3
     sky130_fd_sc_hd__xnor2_1       30
     sky130_fd_sc_hd__xnor3_4       23
     sky130_fd_sc_hd__xor2_1        38
     sky130_fd_sc_hd__xor3_4         3

18. Executing Verilog backend.
Dumping module `\vsdbabysoc'.

yosys> 
```
    
</details>
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








