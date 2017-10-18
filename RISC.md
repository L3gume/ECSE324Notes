# Lecture 4

## Intro to computer organisation

**Components of a computing system**

* Main Mem (RAM)
* Interconnection network
* I/O
* Processor
    * Datapath
    * Control

### How does C implement variables?

> int z = 42; // 32 bits (ARMv7-A)  
> short s = 11; // 16 bits (ARMv7-A)  
> char c = 30; // 8 bits in C, not 16-bits as in Java  
> z = s + c; // answer in z will be 32 bits (4 bytes)  

*There is RAM space allocated for each variable.*

### Memory locations and adresses

* The memory is a linear array of bytes
* Each bytes in the memory has it's own unique address ("byte-addressable")
* The processor can read the value of the byte given its address
* It can also write a byte to a particular address

*Both data and instructions are stored in memory, there are ways of telling them apart*

**Address Space**

* represented using k bits (address size)
* There are 2^k addressable locations in the address space of the computer, numbered from: 0 to 2^k-1
* eg: a 24-bit address has: 2^24 addresses
* a 32-bit address has: 2^32 addresses
* Since each address corresponds to a location that stores a byte, the **capacity** of the memory is 4 Gb (32 Gbits)

### Accessing multiple bytes

* Most computers process data in chunks of several bytes, called a word
    * A typical word size or length is 32 bits (4 bytes)
    * The word size and address size of a computer are often, *but not always* equal
* Since words are accessed so often, the memory has a mechanism to read and write multiple consecutive bytes with a single request instead of having to acess bytes individually multiple times.
* Data can be accessed in chunks of bytes by giving address of starting bytes + word size

### Byte Ordering

**Big Endian**

* Logical ordering (left to right)

**Little Endian**

* Reversed ordering (right to left)

### Memory Alignment

* Some computers requires the memory access must start on an address that is a multiple of the chunk size in bytes.
    * 32-bit words can only be accessed at address 0, 4, 8, ...
    * 16-bit half words can only be accessed at address 0, 2, 4, 6, ...
    * Bytes can be accessed at any address

### Processor

*diagrams in slides*

Memory contains data & program instructions

* Coordinates are stored in the control, which coordinates the execution of intructions in the datapath
* data is manipulated in the datapath

**Datapath:**

There are a couple general-purpose registers that are used as fast, temp data storage for the ALU

* In a 32-bit machine, the registers are 1 word long (4 bytes)

**Other not general-purpose registers:**

The *Program Counter (PC)* and the *Instruction Register (IR)*

### RISC Architecture - reduced instruction set architecture

* The ALU can only get its input data and store its results from/to registers
* Since data is stored in mem, we need special load and store instructions for transfers between registers and mem
* Load and Store are the only instructions that are allowed to access memory

**Typical RISC instruction: load**

> Load R2, LOC

Reads (copies) 4 consecutive bytes from the memory starting at the memory address LOC and writes them as a word into register R2.

* We can precisely describe the operation of this statement with register transfer notation (RTN)
    * [Rn] : contents of register Rn
    * [A] : means the contents of the memory starting at address A
    * Mem[A] : means the memory at address A
    * (left arrow) : transfer (copy)
    * nb of bits on both sides should be equal

**Typical RISC instruction: store**

> Store R4, LOC

Writes (copies) 4 consecutive bytes from the memory starting at the memory address LOC and writes them as a word into register R4.

**Typical RISC instruction: add**

> Add R4, R2, R3

Add R2 and R3 and put into R4

