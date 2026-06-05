RTL Design and Synthesis using SKY130 Technology

About the Workshop

This repository documents my learning journey, hands-on exercises, Verilog designs, synthesis outputs, and simulation results from the RTL Design and Synthesis using SKY130 Technology Workshop conducted by VLSI System Design (VSD).

The workshop provided practical exposure to RTL design using Verilog, simulation using open-source EDA tools, synthesis using Yosys, and analysis of synthesized netlists using the SKY130 standard cell library.
```
Tools Used

* iVerilog – Open-source Verilog compiler and simulator
* GTKWave – Waveform viewer for simulation analysis
* Yosys – Open-source synthesis suite
* SKY130 Standard Cell Library – Open-source PDK library for synthesis
```

Workshop Overview

Day 1 – Introduction to RTL Design and Synthesis

Topics Covered:

* Introduction to RTL Design Flow
* Verilog RTL Coding Basics
* Testbench Development
* Simulation using iVerilog
* Waveform Analysis using GTKWave
* Introduction to Logic Synthesis
* Synthesizing a 2:1 Multiplexer using Yosys
* Understanding Setup Time and Hold Time

Learning Outcomes:

* Understood the complete RTL-to-GDSII design flow.
* Performed RTL simulation and waveform verification.
* Generated synthesized netlists using SKY130 libraries.
``

Day 2 – Timing Libraries and Synthesis Techniques

Topics Covered:

* SKY130 Standard Cell Library (.lib)
* Process, Voltage, and Temperature (PVT) Corners
* Hierarchical Synthesis
* Flat Synthesis
* Sub-module Synthesis
* Various Flip-Flop Coding Styles
    * Asynchronous Reset D Flip-Flop
    * Asynchronous Set D Flip-Flop
    * Synchronous Reset D Flip-Flop

Learning Outcomes:

* Explored standard cell timing libraries.
* Compared hierarchical and flat synthesis approaches.
* Analyzed synthesized hardware generated from different flip-flop coding styles.

``

Day 3 – Combinational and Sequential Logic Optimization

Topics Covered:

* Logic Optimization Techniques
* Constant Propagation
* Boolean Simplification
* Removal of Redundant Logic
* Sequential Logic Optimization
* Optimization of Unused Flip-Flops and Outputs

Learning Outcomes:

* Learned how synthesis tools simplify logic automatically.
* Observed reduction in gate count through optimization.
* Understood how synthesis can eliminate unnecessary hardware.
``

Day 4 – Gate Level Simulation and Synthesis-Simulation Mismatch

Topics Covered:

* Gate Level Simulation (GLS)
* Generated Netlist Verification
* Blocking vs Non-Blocking Assignments
* Sensitivity List Issues
* Common RTL Coding Mistakes
* Synthesis and Simulation Mismatch Analysis

Learning Outcomes:

* Verified synthesized netlists using GLS.
* Identified coding styles that lead to mismatches.
* Understood best practices for RTL coding.

``

Day 5 – Verilog Control Statements and Hardware Inference

Topics Covered:

* If Statements and Latch Inference
* Case Statements and Default Conditions
* For Loops in RTL Design
* Generate Statements
* Ripple Carry Adder Design using Generate Blocks
* Multiplexer and Demultiplexer Design

Learning Outcomes:

* Learned how Verilog constructs map to hardware.
* Understood latch inference and how to avoid it.
* Implemented scalable hardware using generate blocks.

``

Key Takeaways

* RTL simulation is essential before synthesis.
* Gate Level Simulation helps verify synthesized designs.
* Proper sensitivity lists prevent simulation mismatches.
* Incomplete if/case statements may infer unintended latches.
* Blocking and non-blocking assignments must be used appropriately.
* Synthesis tools perform powerful logic optimizations automatically.
* Coding style significantly affects synthesized hardware.

``
References

* VSD RTL Design and Synthesis Workshop
* SKY130 Open PDK Documentation
* Yosys Open Synthesis Suite Documentation
* GTKWave Documentation
* Icarus Verilog Documentation

``

Author

Lokashsri M

Electronics and Communication Engineering

Hands-on learning repository created as part of the RTL Design and Synthesis using SKY130 Technology Workshop.
