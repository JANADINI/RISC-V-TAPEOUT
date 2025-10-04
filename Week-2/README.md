#  Week-2 Task:BabySoC Fundamentals & Functional Modelling

### Objective  
The aim of this task is to:  
1. Build a solid understanding of **System-on-Chip (SoC) fundamentals**.  
2. Practice **functional modelling** of BabySoC using Verilog simulation tools.  
3. Analyze simulation results to verify BabySoC behaviour.  

### Tools used
- **Icarus Verilog (iverilog)** → Verilog compilation & simulation  
- **GTKWave** → waveform analysis  

---

## Part 1 – Theory (Conceptual Understanding)  :

###  What is SoC?  
A **System-on-Chip (SoC)** integrates CPU, memory, peripherals, and interconnect onto a **single chip**.  
This reduces power, cost, and board size while improving performance and reliability.  

### Components of a SoC  
- **CPU** → Executes instructions and processes data  
- **Memory** → Stores instructions (ROM) and temporary data (RAM)  
- **Peripherals** → Input/output controllers, timers, communication modules  
- **Interconnect** → Buses / interconnect fabric that connects CPU, memory, and peripherals  

###  Why BabySoC?  
BabySoC is a **simplified learning model** of a real SoC.  
It helps learners:  
- Understand module interactions (CPU ↔ memory ↔ peripherals)  
- Practice simulation without complex hardware overhead  
- Build a foundation for RTL and physical design stages  

### Role of Functional Modelling  
- **Before RTL/Physical design**, functional modelling checks correctness of behaviour.  
- Ensures modules interact as expected (reset, clock, dataflow).  
- Saves design time by catching errors early.  

Detailed notes are in [`Theory/SoC_Fundamentals.md`](./Theory/SoC_Fundamentals.md)  

---

##  Part 2 – Labs (Hands-on Functional Modelling)  :

### Steps Performed  
1. Clone the BabySoC project repo:
   ```bash
   iverilog -o baby_soc_sim BabySoC.v Testbench.v


   
3. Compile Verilog modules using Icarus Verilog:

   ```bash
   iverilog -o baby_soc_sim BabySoC.v Testbench.v
   vvp baby_soc_sim
      
4. Generate .vcd file for waveform analysis.

5. Open .vcd in GTKWave:
   ```bash
   gtkwave dump.vcd
---



# Observe and analyze:

- Reset operation

- Clocking behaviour

- Dataflow between modules



   





