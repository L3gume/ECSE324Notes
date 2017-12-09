# Problem set (Final)

## Question 1

### Stuff we know:

* Cache size: 1K bytes -> 1024 bytes
* Block size: 128 bytes -> That means we can fit 32 instructions in a block
* Cache miss penalty: 80T (T is tau).

### Thing to know:

The address will use 3 bits for the block field, since we have 8 blocks:

* 1024 bytes / (128 bytes/block)
* 3 bits represent numbers 0 to 7

The address will use 7 bits for the byte field, to respresent what byte of the block it points to:

* 2^7 = 128

The rest will be the tag

### How to determine which instructions go into what block?

We know we can fit 128 bytes, or 32 instructions, into a block. So we have to count intervals of 128 bytes in the decimal mem locations we are given. We will be reading locaions in order

* The start of the program, 8-52, all fit into block 0 (52 - 8 = 44 bytes)
    * Just keep counting that way to determine what goes where!

| Addresses | Block(s) |
|-----------|----------|
|   8 - 52  |     0    |
|  56 - 136 |   0, 1*  |
| 140 - 240 |     1    |
|244 - 1200 |1-7, 0,1**|
|1204 - 1504|  1, 2, 3 |

\* *The instructions don't all fit in the same block, overflows into the next one*  
\** *There's a lot of instructions, so they wrap around back to the beginning of the cache*

### computing the total miss penalty:

We basically just have to count the number of times we have to load a block, knowing the number of iterations of each instruction

PENALTY = 80T

At the start of the program, we load the first instructions in block 0, We then have to load the inner loop for the first time, which all fits into block 1, so it is loaded only once for the 20 iterations. After that, we have to load the rest of the outer loop into 2 to 7 and also overwrite 0 and 1. At the very end, we load into 2 and 3. The sequence will look a bit like:

* 0, [1, 2, 3, 4, 5, 6, 7, 0, 1] [(0, 1, 0, 1)(x9)], [2, 3]

This means that our penalty will be:

```
penalty = (1 + 9 + (9 x 4) + 2 ) * 80T = 3840T
```

*Things to note:*

* Count the outer loop first and separate the sections of the execution to properly count

Now for the total execution time, we basically just have to count all the accesses to the cache:

```
Start:
    56 - 8 = 48T

Inner Loop:
    20 * 10 * (244 - 140) = 20800T

Outer Loop:
    ((1204 - 56) - (244 - 140)) * 10 = 10440T

End:
    1508 - 1204 = 304T

TOTAL: 35432T
```

*Things to note*:

* Use the start of the next section as the boundary since the last byte of a section has to be counted

## Question 2

### a)

#### Stuff we know:

* Cache is set associative
* 64 blocks, 4-block sets (16 sets)
* Main memory has 4096 blocks, each 32 words
* Address space is 32 bit

#### Find the fields

##### Set field

We have 16 sets, we need 4 bits to represent 0 - 15

* Set field = 4 bits

##### Word field

Assuming 32-bit words. Each block containing 32 words means that each block contains (32 * 4) bytes

* Block size: 128 bytes
* we need 2^7 bits to represent 0 to 127
* Word field: 7 bits

##### Tag field

Since we have 4 bits for the set field and 7 bits for the word field, that means we have 21 bits left for the tag

* Tag field: 21 bits

### b)

*Can't really plot graphs in vim*

## Question 3

### Stuff we know:

* Usage statistics:

|Instruction|usage|
|-|-|
|Branch|20%|
|Load|20%|
|Store|10%|
|Computational instructions|50%|

* CPU clock speed: 1Ghz

### a) every mem access is one clock cycle

Assume that each step takes one clock cycle, and that branch runs in 5 steps even though it only does stuff in the 3 first steps:

```
rate = 1E9 / 5 = 200 000 000 instr/sec
```

### b) different mem access times

#### What we know:

* 90% of instr fetch takes 1 clock cycle, the rest takes 4 cycles
* access to operands of load and store takes 3 cycles

#### find the rate:

We will again assume that branch runs in 5 steps.

```
Average fetch time:
    fetch_time = 0.9 * 1 + 0.1 * 4 = 1.3 cycle

Branch:
    branch_time = 1.3 + 4

Load/store:
    load_store_time = 1.3 + 3 + 3

computational instructions:
    comp_time = 1.3 + 4

average excution time:
    avg_time = branch_time * 0.2 + load_store_time * 0.3 + comp_time * 0.5
    avg_time = 5.9

rate = 1E9 / 5.9 = ~169 491 525 instr/sec
```

*To note*:

* Each step runs in one clock cycle, unless it involves a memory access of instruction fetching
* To find the average execution time, break down each step of the instruction, determining if it involves memory access or not.
