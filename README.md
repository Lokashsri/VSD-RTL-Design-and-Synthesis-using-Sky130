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
---

## Day 1 – Introduction to Verilog RTL Design & Synthesis

### Key Concepts

- **Simulation** – Technique for applying different input stimulus to a design at different times to check if RTL code behaves as intended.
- **RTL Design** – Register Transfer Level abstraction of a digital circuit using combinational logic, registers, and modules.
- **Test Bench** – Setup to apply stimulus (test vectors) to the design to verify logical functionality. The test bench has no primary inputs or outputs.
- **Simulator (iVerilog)** – Continuously monitors input signal changes; evaluates output on any input change; dumps changes to a `.vcd` file.

### iVerilog Flow

```
RTL Design (.v) ──┐
                  ├──► iVerilog ──► a.out ──► .vcd file ──► GTKWave
Test Bench (.v) ──┘
```

### Lab Setup Commands

```bash
mkdir vlsi
cd vlsi
mkdir vsdflow
git clone https://github.com/kunalg123/sky130RTLDesignAndSynthesisWorkshop.git
cd sky130RTLDesignAndSynthesisWorkshop
ls -ltr
cd my_lib/lib        # Sky130 Standard Cell library
cd ../verilog_model  # Verilog models
cd ../verilog_files  # RTL and Testbench files
```

### Simulation of good_mux

```bash
iverilog good_mux.v tb_good_mux.v   # Step 1: Compile
./a.out                              # Step 2: Generate .vcd
gtkwave tb_good_mux.vcd             # Step 3: View waveform
```

### Yosys Synthesis of good_mux

```bash
cd verilog_files
yosys                                                          # Invoke Yosys
read_liberty -lib ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib  # Load library
read_verilog good_mux.v                                        # Read design
synth -top good_mux                                            # Synthesize
abc -liberty ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib  # Map to cells
show                                                           # View netlist graph
write_verilog -noattr good_mux_netlist.v                      # Write netlist
```

### good_mux Netlist (Generated)

```verilog
module good_mux(i0, i1, sel, y);
  input i0, i1, sel;
  output y;
  sky130_fd_sc_hd__clkinv_1 _6_ (.A(i0), .Y(_4_));
  sky130_fd_sc_hd__nand2_1  _7_ (.A(i1), .B(sel), .Y(_5_));
  sky130_fd_sc_hd__o21ai_0  _8_ (.A1(sel), .A2(_4_), .B1(_5_), .Y(y));
endmodule
```

### Why Different Flavours of Standard Cells?

- **Setup Time** → Needs cells with **least propagation delay** → Wide transistors → Higher power/area
- **Hold Time** → Needs cells with **more propagation delay** → Narrow transistors → Lower power/area
- This is why the library contains multiple versions of the same logic gate.

---

## Day 2 – Timing Libraries, Hierarchical vs Flat Synthesis & Flop Coding

### SKY130 Library

- **Technology:** SKY130 — a mature 180nm-130nm hybrid process by SkyWater Technology / Google
- **Library file:** `sky130_fd_sc_hd__tt_025C_1v80.lib` (tt = typical, 025C = temperature, 1v80 = voltage)

```bash
gvim sky130_fd_sc_hd__tt_025C_1v80.lib
:syn off   # Turn off syntax highlighting
:se nu     # Show line numbers
:/cell     # Search for cells
:g//       # List all matches
```

### Hierarchical Synthesis (Multiple Modules)

```verilog
module sub_module1 (input a, input b, output y);
   assign y = a & b;
endmodule

module sub_module2 (input a, input b, output y);
   assign y = a | b;
endmodule

module multiple_modules (input a, input b, input c, output y);
   wire net1;
   sub_module1 u1(.a(a), .b(b), .y(net1));
   sub_module2 u2(.a(net1), .b(c), .y(y));
endmodule
```

- In **hierarchical synthesis**, module hierarchy is preserved — sub_module1 and sub_module2 appear as instances inside multiple_modules.
- In **flat synthesis** (`flatten` command), all hierarchy is dissolved into a single netlist.

### Sub-Module Level Synthesis

Useful when:
- Same module is instantiated multiple times (synthesize once, replicate)
- Divide-and-conquer for large designs

```bash
synth -top sub_module1
```

### Flop Coding Styles

```bash
dfflibmap -liberty ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```

| Style | Behaviour |
|-------|-----------|
| Async Reset | Q goes to 0 immediately on reset, independent of clock |
| Async Set | Q goes to 1 immediately on set, independent of clock |
| Sync Reset | Q goes to 0 only on clock edge when reset is high |
| Sync Set | Q goes to 1 only on clock edge when set is high |

### Interesting Optimisation — mul2

```verilog
module mul2(input [2:0] a, output [3:0] y);
   assign y = a * 2;  // No cells needed! Just wire shift
endmodule
// Synthesized as: assign y = {a, 1'b0};
```

No standard cells are inferred — multiplication by 2 is just a **1-bit left shift** (wire connection).

---

## Day 3 – Combinational and Sequential Logic Optimisations

### Why Optimise?

- Squeeze logic to the most compact form
- Save area and power

### Combinational Optimisation Command

```bash
opt_clean -purge   # Remove unused wires and cells
```

### Examples

**opt_check.v** — Ternary mux simplifies to AND gate:
```verilog
module opt_check (input a, input b, output y);
   assign y = a ? b : 0;   // Optimised to: y = a & b
endmodule
```

**opt_check2.v** — Simplifies to OR gate

**opt_check3.v / opt_check4.v** — Further Boolean reductions

### Sequential Optimisation Examples

| File | Description | Result |
|------|-------------|--------|
| dff_const1.v | Reset to 0, else Q=1 | Flip-flop retained (follows clock) |
| dff_const2.v | Always Q=1 regardless of reset | Optimised to constant 1 (no FF) |
| dff_const3.v | Two FF chain | Both retained |
| dff_const4.v | Reset to 1, else Q1=1, Q=Q1 | Optimised — Q always 1 |
| dff_const5.v | Reset to 0, else Q1=1, Q=Q1 | Two FFs retained |

### Unused Output Optimisation — Counter

```verilog
module counter_opt (input clk, input reset, output q);
   reg [2:0] count;
   assign q = count[0];    // Only bit 0 used as output
   always @(posedge clk, posedge reset)
      if(reset) count <= 3'b000;
      else      count <= count + 1;
endmodule
// Synthesis result: Only 1 FF inferred (bits 1 and 2 optimised away)
```

---

## Day 4 – GLS, Blocking vs Non-Blocking & Synthesis Mismatch

### Gate Level Simulation (GLS)

```
Netlist (.v) ──┐
               ├──► iVerilog ──► Simulation Output
Test Bench ────┘
Gate Models ───┘
```

- GLS verifies **logical correctness** of the synthesized netlist
- Netlist is logically equivalent to RTL but uses actual standard cells
- The **same testbench** used for RTL can be reused for GLS

### Synthesis-Simulation Mismatch

**Cause 1: Missing Sensitivity List**

```verilog
// BAD MUX — only sensitive to sel
module bad_mux (input i0, i1, sel, output reg y);
   always @ (sel)          // ← WRONG: missing i0, i1
      if(sel) y <= i1; else y <= i0;
endmodule

// GOOD MUX — sensitive to all inputs
module good_mux (input i0, i1, sel, output reg y);
   always @ (*)            // ← CORRECT
      if(sel) y <= i1; else y <= i0;
endmodule
```

**Cause 2: Blocking vs Non-Blocking Statements**

```verilog
// BLOCKING CAVEAT — wrong ordering causes simulation mismatch
module blocking_caveat (input a, b, c, output reg d);
   reg x;
   always @ (*) begin
      d = x & c;   // Uses OLD value of x (evaluated first)
      x = a | b;   // x updated AFTER d is computed
   end
endmodule
```

| Statement | Behaviour |
|-----------|-----------|
| `=` (Blocking) | Executes sequentially; previous statement completes before next begins |
| `<=` (Non-Blocking) | All RHS evaluated first, then all LHS updated simultaneously |

> **Rule:** Always use `<=` (non-blocking) inside `always @(posedge clk)` sequential blocks.

---

## Day 5 – If, Case, For Loop and For Generate

### Incomplete IF — Infers Latch

```verilog
// INCOMPLETE IF — latch inferred for y when i0=0
module incomp_if (input i0, i1, i2, output reg y);
   always @ (*)
      if(i0) y <= i1;
      // No else — latch inferred!
endmodule
```

### Incomplete Case — Infers Latch

```verilog
// INCOMPLETE CASE — latch inferred for sel=2'b10, 2'b11
module incomp_case (input i0, i1, i2, input [1:0] sel, output reg y);
   always @ (*)
   case(sel)
      2'b00 : y = i0;
      2'b01 : y = i1;
      // Missing cases → latch!
   endcase
endmodule
```

### Complete Case — No Latch

```verilog
module comp_case (input i0, i1, i2, input [1:0] sel, output reg y);
   always @ (*)
   case(sel)
      2'b00 : y = i0;
      2'b01 : y = i1;
      default : y = i2;   // Covers all remaining cases
   endcase
endmodule
```

### For Loop — 4:1 MUX

```verilog
module mux_generate (input i0, i1, i2, i3, input [1:0] sel, output reg y);
   wire [3:0] i_int;
   assign i_int = {i3, i2, i1, i0};
   integer k;
   always @ (*)
      for(k = 0; k < 4; k = k+1)
         if(k == sel) y = i_int[k];
endmodule
```

### For Loop — 1:8 Demux

```verilog
module demux_generate (output o0,o1,o2,o3,o4,o5,o6,o7, input [2:0] sel, input i);
   reg [7:0] y_int;
   assign {o7,o6,o5,o4,o3,o2,o1,o0} = y_int;
   integer k;
   always @ (*) begin
      y_int = 8'b0;
      for(k = 0; k < 8; k++)
         if(k == sel) y_int[k] = i;
   end
endmodule
```

### Generate — Ripple Carry Adder (8-bit)

```verilog
module fa (input a, b, c, output co, sum);
   assign {co, sum} = a + b + c;
endmodule

module rca (input [7:0] num1, num2, output [8:0] sum);
   wire [7:0] int_sum, int_co;
   genvar i;
   generate
      for (i = 1; i < 8; i = i+1)
         fa u_fa_1(.a(num1[i]), .b(num2[i]), .c(int_co[i-1]),
                   .co(int_co[i]), .sum(int_sum[i]));
   endgenerate
   fa u_fa_0(.a(num1[0]), .b(num2[0]), .c(1'b0),
             .co(int_co[0]), .sum(int_sum[0]));
   assign sum[7:0] = int_sum;
   assign sum[8]   = int_co[7];
endmodule
```

| Construct | Use Case |
|-----------|----------|
| `for` loop (inside always) | Evaluate/select logic at runtime |
| `generate for` | Replicate hardware structures at elaboration time |

---

## 📚 Key Takeaways

1. **Not all Verilog is synthesizable** — always verify with simulation AND GLS
2. **Sensitivity list errors** cause simulation-synthesis mismatch — use `always @(*)`
3. **Incomplete if/case** infer unwanted latches — always add `else` or `default`
4. **Use non-blocking (`<=`)** in sequential blocks; blocking (`=`) in combinational
5. **Optimisations** can eliminate cells entirely (e.g., multiply-by-2, constant outputs)
6. **GLS is critical** — it catches bugs that RTL simulation may miss

---

## 🔗 References

- [VSD Workshop Repo by Kunal Ghosh](https://github.com/kunalg123/sky130RTLDesignAndSynthesisWorkshop)
- [SKY130 PDK Documentation](https://skywater-pdk.rtfd.io)
- [Yosys Documentation](http://www.clifford.at/yosys/)
- [iVerilog](http://iverilog.icarus.com/)

---
Author

Lokashsri M



