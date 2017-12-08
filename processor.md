# Processor Implementation

## The Processor

A processor is responsible for reading program instructions from the computer's memory and executing them

* It *fetches* one instr at a time
* It *decodes* (interprets) the instruction
* Then, carries out the actions specified by the instruction

### Building Blocks

* `PC` holds the address of the next instruction to be fetched and executed.
* Instr is fetched into `IR`
* Instruction address generator updates `PC`
* Control circuitry decodes the instruction and generates control signals that direct the datapath

## Datapath Design

*image*

* Combinational circuits can be divided into simple subcircuits that are cascaded into a multi-stage structure

*image*

### Instruction Execution

* In RISC machines, all instr are executed in the same number of steps.
* Each step is carried out in a separated hardware stage.
* RISC CPU design will be illustrated in 5 hardware stages.
* The CPU design will model the basic RISC instructions and addressing modes, but not every detail of an ARM ISA.

#### Load Instruction

```asm
LDR R5, [R7, R8]
```

1. **Fetch** the instr and increment the program counter `PC`
2. **Decode** the instr and read the content of registers `R7` and `R8` in the register file
3. **Compute** the effective address
4. Read the **memory** source operand
5. **Write** the operand into the **destination register**

How can we decode the instruction and read the registers at the same time?

* In a RISC ISA, register fileds are always in the same positions in the instruction. If the registers weren't need, they'd be ignored.

*image*

#### Arithmetic and Logical Instructions

```asm
ADD R3, R4, R5
```

1. **Fetch** the instruction and increment the program counter
2. **Decode** the instr and read registers `R4` and `R5` from the register file
3. **Compute** the sum
4. **NO ACTION**
5. **Write** the result into the **destination register**

*Stage 4 (memory access) is not involved in the instruction*

#### Immediate Operands

```asm
ADD R3, R4, #1000
```

The immediate operand is given in the instruction word and can be found in the IR.

1. **Fetch** the instruction and increment the program counter
2. **Decode** the instr and read registers `R4` from the register file
3. **Compute** the sum
4. **NO ACTION**
5. **Write** the result into the **destination register**

#### Load Instruction (immediate)

```asm
LDR R5, [R7, #X]
```

The immediate operand is given in the instruction word and can be found in the IR.

1. **Fetch** the instruction and increment the program counter
2. **Decode** the instr and read the content of register `R7` in the register file
3. **Compute** the effective address
4. Read the **memory** source operand
5. **Write** the result into the **destination register**

#### Store Instruction

```asm
STR R6, [R8, #X]
```

The immediate operand is given in the instruction word and can be found in the IR.

1. **Fetch** the instruction and increment the program counter
2. **Decode** the instr and read the content of register `R7` and `R8` in the register file
3. **Compute** the effective address
4. Store the contents of register `R6` into mem location `X + [R8]`
5. **NO ACTION**

### Summary - Actions to Implement an Instruction

1. **Fetch** an instr and increment the program counter
2. **Decode** the instruction and read registers from the register file
3. Perfrom an **ALU** operation
4. Read or write **memory** data if the instr involves a memory operand
5. **Write** the result into the **destination register**

## Hardware Components

### Register File

*image*

* A **2-port register file** is needed to read the two source registers at the same time
* It may be implemented using a 2-port memory

**Alternative implementation of 2-port register file:**

*image*

* Using two single-ported memory blocks each containing a copy of the register file

### ALU (Arithmetic Logic Unit)

*image*

* Both source operands and the destination location are in the register file
* Conceptual single-cycle view of an ALU with two source operands in registers

*image*

* One of the source operands is the immediate value in `IR`

### A 5-stage Implementation of a RISC processor

Instruction processing moves from stage to stage in every clock cycle, starting with fetch.

The instruction is decoded and source registers are read in stage 2.

Computation takes place in the ALU in stage 3.

If a mem operation is involved, takes place in stage 4.

Result of the instruction is stored in destination register in stage 5.

### Waiting for memory

* Assumed all mem accesses take one clock cycle.
    * Mostly realistic if we use cache
* In case of a cache miss, cpu must be stalled to wait for the memory access to complete (variable number of clock cycle)
* the cpu-memory interface generates a signal called *Memory Function Completed (MFC)*
* *Processor extends the duration of the memory step (in units of clock cycles) until MFC is asserted*

### The datapath - Stages 2 to 5

*image*

* Register file, used in stages 2 and 5
* Multicycle: Inter-stage registers `RA, RB, RZ, RY` needed to carry data from one stage to next
* ALU stage
* Memory stage
* Final stage to store result

#### Register File

*image*

* Address inputs are connected to the corresponding fields in `IR`
* Source registers are read in stage 2; their contents are stored in `RA` and `RB`
* In stage 5, the result of the instruction is stored in the destination register selected by `Address C`

#### ALU stage

*image*

* ALU performs calculation specified by the instruction
* Multiplexer `MuxB` selects either `RB` or the immediate field of `IR`
* Results stored in `RZ`
* Data to be written in the memory are transferred from `RB` to `RM`

#### Memory stage

*image*

* For a memory instruction, `RZ` provides mem address, and `MuxY` selects read data to be placed in `RY`
* `RM` provides data for write operation
* For a calculation instr, `MuxY` selects `[RZ]` to be placed in `RY`
* Input 2 of `MuxY` is used in subroutine calls, it is the return address


