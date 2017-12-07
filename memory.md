# Memory

## Memory Technology

*image*

### RAM

* Memory access time
    * time from intiation to completion of a word or byte transfer
* Memory cycle time
    * minimum time delay between initiation of successice tranfers

*Random-access memory (RAM)* means that access time is the same, independent of location

#### Semiconductor RAM

* Organized as an array of *cells*, each storing one bit
* Each row of the arr stores on *word* (not processor word)

For example, consider 16x8 RAM with a 8-bit wordsize and 16 words

* How many bits does this memory store
    * 16 x 8
* how many bits are needed for the memory address
    * that many (?)

*image*

*image*

#### SRAM

* *Static RAM*: retains contents as long as power is applied, but *volatile* - if power is removed the contents are destroyed
* Fast (access time of few ns), but expensive - each cell require 6 transistors to store a bit
* SRAMs are limited to how large they can be - typically at most few Mbs
* Used in cache memory, but not the main

*images*

#### DRAM

* *Dynamic RAM*: retains contents for only a few tens of milliseconds and must periodically "refreshed" to maintain the contents for longer periods
* Slower than SRAM, but more dense (less expensive) - the cell is sompler than the SRAM cell
* Can implement DRAMs with large capacity (~Gb)
* Used for main memory

*image*

#### Reading DRAM

* *Read*: Sense amplifier connected to the bit line detects if the charge capacitor is above threshold
* If above threshold, the sense amplifier drives the bit line to full voltage ("1") and as a result the capacitor is recharged to full voltage ("1").
* If below threshold, sense amp pulls the bit line down to ground ("0") and the capacitor is discharged fully ("0")

-> **Reading a DRAM cell refreshes its contents**

To refresh the entire DRAM each row must be peridically read - done by an external memory controller.

#### Refresh Overhead

* Assume that each row needs to be refreshed every 64 ms, the minimum time between two row accesses is 50 ns and that all rows are refreshed in 8192 cycles
* Read/write operations have to be delayed until refresh is finished. What is the refresh overhead?

*image*

#### Fast Page Mode

* In last example, all 16384 cells in a row are accessed (and also refreshed as a result)
* only 8 bits of data are actually tranferred for each full row/column addressing sequence
* *latches* in sense amplifiers hold cell contents (more efficient)
* For consecutive data, just assert CAS signal and increment column address in same row
* *fast page mode* useful in block transfersi

#### Synchronous DRAMs

*image*

* Modern sync DRAM ( **SDRAM** ) uses clock to generate internal timing signals (CAS and RAS)
* Memory controller is integrated on-chip
* "dynamic" nature of chip is invisible to the user

#### Efficient Block Transfers

* Sync DRAM reduces delay by having CAS assertion *once* for intial column address
* SDRAM circuitry increments column counter and transfers consecutive data automatically
* Burst length determines number of transfers

#### Double-Data-Rate (DDR) SDRAM

* Modern SDRAMs use both rising and falling edges od the clock ("double data rate")

e.g. DDR4 has a clock of 2133 MHz and can support up to 2400 MTransfers / second

*image*

## Non-volatile memories

* Non-volatile memories retain their contents even when the power is removed
* Slower than volatile memories and special procedure for writes
* Suitable for implementing long-term storage
    * Solid-State Disk (SSD)

### Read-only-memory (ROM)

contents only written *once*, at the time of manufacture

*image*

#### PROM, EPROM, EEPROM

* Cells of a *programmable ROM (PROM)* chip may be  written after the time of manufacture
* A fuse is burned out with a high current pulse
* An *erasable programmable ROM (EPROM)* uses a special transistor instead of a fuse
* Injecting charge allows transistors to turn on
* Erasure requires UV light to remove all charge
* An *electrically erasable ROM (EEPROM)* can have individual cells erased with a chip in place

#### Flash Memory

* High density, low-power and low-cost
* For higher density, flash cells are designed to be erased in larger blocks, not individually
* writing individual cels requires reading block, erasing block, then writing block with changes.
* Flash cells can only be written a certain number of times - wear levelling distributes writes to avoid wearing out one part of the memory before others
* Widely used in cell phones, digital cameras, and solid-state-drives

## Direct Memory Access (DMA)

* CPU overhead for block transfers between I/O and memory is high because because each transfer involves only a single word or a single byte
* Solutions:
    * *A direct memory access (DMA)* controller manages the transfer of larger blocks of data between memory & I/O devices
    * CPU intiates transfer, which is managed by the DMU unit without further CPU involvement

*image*

### DMA Controller

*image*

* shared, or in each I/O device
* Keeps track of progress with address counter
* Processor initiates DMA controller activity after writing information to special registers
* Processor interrupt used to signal completion

## Caches

### The problem

We want very large and very fast memory

* DRAM is large, but slow
* SRAM is fast, but small

Solution:

* Use both DRAM and SRAM in a way that the processor thinks it has a single large memory that is fast
* Should be transparent to the programmer

### Unlimited amounts of fast mem?

*image*

* Keep a **copy** of frequently used data in the small cache mem so that if it is needed again it is quickly accessible without going to the large main mem
* Specialized hardware manages the data between main mem and cache
* Transparent
* Having knowledge of how cache works helps write faster programs

*image*

### So why does this work?

* This only works because humans write programs with structure
* If you look at a trace of memory addresses issued by the processor running typical programs, there's patterns
* This is called "locality" and is the reason caches work

### Cache Basics

* CPU requests an item
* if it is found in cache: **hit**
* *read hit*: deliver the desired item to the processor
* *write hit*:
    * **write-through**: write to both the cache and the main memory
    * **write-back**: only write to the cache. Update the main mem only when that cache block is removed from cache to make room. A **dirty bit** is set to indicated hte cache block has been changed and is no longer identical to the block in main memory.
* If it is not found in the cache: **miss**
* *read miss*: copy the block from main memory into the cache and then deliver the item to the processor
* *write miss*:
    * if write-through is being used, then write directly to the main mem on a write miss
    * if write-back is being used, first copy the block containing the addressed word into the cache, then write the new word in the cache block

### Hit and Miss rate

For a cache to make sense, most accesses to memory have to hit the cache. Not uncommon for caches to have a **hit rate** of > 95%

### Where to put blocks in the cache

Main mem is divided into *blocks* each consisting of several consecutive data elements (e.g. bytes)

Block in main mem must be transferred to the cache after a miss

* The *mapping function* determines the location
    * Some mapping functinos are simple, and some are more complex but have higher performance, i.e. result in a higher hit rate

### Direct Mapping

* Every mem block amps to a single cache block
* n = #blocks in cache

`memory block j -> cache block (j mod n)`

Simplest approach: uses a fixed mapping
Multiple blocks may contend for same location

* New block always overwrites previous block
* if have multiple frequently accessed blocks that kick eachother out of the cache, you will have many cache misses and suffer the penalty of having to go to main memory frequently

*image*

* Each cache block has some space to store the "tag" of the memory block that is currently stored in that block
* On an access, the tag of the requested address is compared with the stored tag.
* if they match -> cache hit!

*image*

### Associative Mapping

* The most flexible mapping: a main mem block can be placed into *any* cache block
* A block is only ejected from the cache if it is full
* the entire book address is the tag
* Check if a block is in the cache by doing an *associative search* of ALL the cache tags in parallel - complex!

### Set-Associative Mapping

* *k-way set-associative cache*: Group blocks of cache into *sets* of k blocks
* Direct mapping of a memory block to a specific - any block in the set can be used
* Associative seach involves only tags in a set (k=2, 4, 8)
* Direct-mapped = 1-way, associative = n-way

### Stale Data

* Each block has a valid bit, initialized to 0 upon startup to indicate the block is "empty" - set to 1 when a block is copied to the cache
* for a hit, valid bit must be 1
* e.g. DMA
* Disk -> Mem: cache may contain *stale* data from memory, so valid bits are cleared to 0 for those blocks
* Mem -> Disk: avoid stale data by *flushing* modified blocks from (WB) cache to mem

### Replacement policies

* Replacement is trivial for direct mapping, but need a method for associative mapping
* *least-recently-used (LRU)* algorithm
    * requires specialized hardware to track accesses to cache blocks in the set
* Another replacement policy is to remove the "oldest" block in the set
* Random replacement works surprisingly well
