# 🚀 5-Stage Pipelined RISC-V (RV32I) Processor with Forwarding Unit

A complete implementation of a **32-bit 5-stage pipelined RISC-V (RV32I)** processor designed and verified in **Logisim**, featuring a hardware **Forwarding Unit** for resolving data hazards without unnecessary pipeline stalls.

---

# Overview

This project implements a **32-bit RISC-V processor** based on the **RV32I Base Integer Instruction Set**. The processor was first developed as a **single-cycle CPU** and then upgraded to a **5-stage pipelined architecture**.

To improve performance, a dedicated **Forwarding Unit** was implemented to eliminate common data hazards by forwarding recently computed values directly to the ALU instead of waiting for them to be written back into the register file.

Special care was also taken to correctly handle the **x0 register**, ensuring full compliance with the RISC-V specification.

---

# Features

* RV32I Base Integer ISA
* 32-bit datapath
* 5-stage pipeline architecture
* Pipeline registers (IF/ID, ID/EX, EX/MEM, MEM/WB)
* Forwarding Unit for data hazard resolution
* Zero register (`x0`) protection
* Register File with separate read/write clock phases
* Logisim implementation
* Assembly and hexadecimal test programs
* Verified forwarding behavior with dedicated test cases

---

# Pipeline Architecture

```
       +------+      +------+      +------+      +------+      +------+
PC --> |  IF  | ---> |  ID  | ---> |  EX  | ---> | MEM  | ---> |  WB  |
       +------+      +------+      +------+      +------+      +------+
          |              |              |              |              |
       IF/ID          ID/EX         EX/MEM         MEM/WB       Register File
```

The processor follows the standard 5-stage RISC-V pipeline.

### 1. Instruction Fetch (IF)

* Fetches a 32-bit instruction from instruction memory (ROM)
* Computes `PC + 4`

### 2. Instruction Decode (ID)

* Decodes instruction fields
* Reads operands from the Register File
* Generates control signals

### 3. Execute (EX)

* Performs arithmetic and logical operations
* Executes forwarding logic before ALU execution

### 4. Memory Access (MEM)

* Reads from or writes to data memory (RAM)
* Used by load and store instructions

### 5. Write Back (WB)

* Selects the final execution result
* Writes the result back into the Register File

---

# Forwarding Unit

To eliminate unnecessary pipeline stalls caused by **data hazards**, a hardware forwarding unit was implemented.

Instead of waiting until the Write-Back stage, the processor forwards newly generated results directly to the ALU whenever possible.

## Supported Forwarding Paths

### EX → EX Forwarding

Forwards the ALU result from the **EX/MEM** pipeline register to the next instruction's ALU inputs.

Used when the dependent instruction immediately follows the producer instruction.

---

### MEM → EX Forwarding

Forwards data from the **MEM/WB** pipeline register.

Used when there is one instruction between the producer and consumer.

---

## Zero Register Protection

According to the RISC-V specification, register **x0** must always contain zero.

The forwarding logic includes the condition:

```
rd != 0
```

to prevent invalid forwarding from instructions that attempt to write into `x0`.

---

# Clock Synchronization

To avoid race conditions and structural hazards, different clock edges are used for pipeline registers and the register file.

### Pipeline Registers

* IF/ID
* ID/EX
* EX/MEM
* MEM/WB

All pipeline registers operate on the **rising edge** of the clock.

### Register File

The Register File performs write operations on the **falling edge**.

This timing ensures that:

* results are written back before the next decode stage,
* registers can be safely read within the same cycle,
* read-after-write conflicts are avoided.

---

# Debugging and Design Challenges

During development, several hardware issues were identified and resolved.

### ALU Timing Issues

The PC update path was separated from the ALU datapath, and all pipeline registers were synchronized to eliminate propagation delays and unstable execution timing.

### Undefined Signals

Several undefined (`?`) signals were traced to:

* inconsistent tunnel naming,
* incorrect multiplexer select inputs,
* instruction width mismatches.

All instruction memories were verified to contain **32-bit RV32I instructions**, avoiding accidental generation of compressed (RVC) instructions.

---

# Test Cases

Three dedicated programs were created to verify the forwarding unit.

---

## Test Case 1 — EX → EX Forwarding

This test verifies forwarding from the EX/MEM stage.

### Assembly

```assembly
addi x1, x0, 5
addi x2, x0, 10
add  x3, x1, x2
add  x4, x3, x1
```

### ROM Contents

```text
v2.0 raw
00500093
00a00113
002081b3
00118233
```

### Expected Result

```
x4 = 20
0x00000014
```

---

## Test Case 2 — MEM → EX Forwarding

This test verifies forwarding from the MEM/WB stage.

### Assembly

```assembly
addi x1, x0, 8
addi x2, x0, 3
add  x3, x1, x2
addi x5, x0, 0
add  x4, x3, x1
```

### ROM Contents

```text
v2.0 raw
00800093
00300113
002081b3
00000293
00118233
```

### Expected Result

```
x4 = 19
0x00000013
```

---

## Test Case 3 — x0 Protection

This test ensures that writes to `x0` neither modify the register nor trigger forwarding.

### Assembly

```assembly
addi x1, x0, 7
add  x0, x1, x1
add  x4, x0, x1
```

### ROM Contents

```text
v2.0 raw
00700093
00108033
00100233
```

### Expected Result

```
x4 = 7
0x00000007
```

If the result becomes **21**, the forwarding unit is incorrectly forwarding values written to `x0`.

---

# Running the Project

1. Open the `.circ` file in **Logisim**.
2. Right-click the instruction **ROM** and select **Edit Contents**.
3. Paste one of the provided hexadecimal programs.
4. Run the processor using:

   * **Ctrl + K** for continuous clock execution.
   * **Ctrl + T** for single-step execution.
5. Observe the pipeline registers and Register File to verify execution.

---

# Project Structure

```
.
├── cpu.circ                 # Logisim CPU
├── README.md
├── testcases/
│   ├── ex_forwarding.asm
│   ├── mem_forwarding.asm
│   ├── zero_register.asm
│   ├── ex_forwarding.hex
│   ├── mem_forwarding.hex
│   └── zero_register.hex
```

---

# Future Improvements

Planned enhancements include:

* Hazard Detection Unit for automatic load-use stall insertion
* Branch Control Unit
* Pipeline flushing
* Branch prediction
* Support for additional RV32I instructions
* Performance evaluation (CPI analysis)
* Automated testbench generation

---

# Technologies

* **Logisim**
* **RISC-V RV32I**
* Digital Logic Design
* Computer Architecture
* Pipeline Processing
* Hazard Resolution
* Forwarding Logic

---

# License

This project was developed for educational purposes as part of a Computer Architecture course and is free to use for learning and academic reference.
