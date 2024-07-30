# Logisim 16-Bit processor with unified cache memory

## Overview
This is a college work developed together with *[Luigi Eliabe](https://github.com/luigieli)*, and other classmates, that aimed to develop a 16-bit data processor with a unified cache memory.

![motherboard](https://github.com/Felkng/CPU_CACHE/blob/main/images-cpu/main_circuit.png)

## Cache archtecture

### Structure
The cache memory is organized into a two-way associative set structure, consisting of:

- 4 Sets: Each set contains 2 lines.
- Cache Line: Each cache line comprises 4 words, with each word being 16 bits.
- Total Cache Size: 64 Bytes.

### Details
#### Two-Way Associative Set Structure
A two-way associative set structure allows each block of memory to map to any of two cache lines within a set, providing a good balance between the complexity of fully associative caches and the inflexibility of direct-mapped caches.

#### Cache Write Policy: Write-Back
The write-back policy is employed in this cache system. In a write-back cache, modifications to data in the cache are not immediately written to the main memory. Instead, the data is written back to the main memory only when the cache line is replaced. This reduces the number of write operations to the main memory, improving performance. However, it requires the cache to keep track of which cache lines have been modified (dirty lines).

#### Replacement Algorithm: Least Recently Used (LRU) with Modification
The Least Recently Used (LRU) algorithm is used for cache replacement. LRU keeps track of the usage of each cache line and replaces the least recently used line when a new line needs to be loaded into the cache.

#### Modification to LRU
In this implementation, the line to be overridden is always the older data holder line, regardless of its last usage. This means that the algorithm prioritizes replacing the data that has been in the cache the longest, rather than strictly following the least recently used policy.

## Data Flow

### Instruction Format

Each instruction in this processor consists of 16 bits of data. The instruction format varies based on the type of instruction, which determines how the bits are split between the opcode, registers, immediate values, and labels.

### Instruction Types

| Instruction Type | OpCode (bits) | RD (bits) | RT (bits) | RS/IMM (bits) | Label (bits) |
|------------------|---------------|-----------|-----------|---------------|--------------|
| Type I           | 3             | 3         | 4         | 6             | -            |
| Type R           | 3             | 3         | 4         | 6             | -            |
| Type J           | 3             | -         | -         | -             | 13           |
| BEQ              | 3             | 3         | 3         | -             | 7            |

#### BEQ Instruction Exception

For BEQ-type instructions, the register bit split is modified. This instruction only requires 8 registers, allowing the label field to be extended to 7 bits. This modification provides more address space for jumps.

### Opcode Division

The opcode is a 3-bit field that defines the operation to be performed. The following table lists the opcode values and their corresponding operations:

| Opcode | Operation |
|--------|-----------|
| 000    | ADD       |
| 001    | AND       |
| 010    | ADDI      |
| 011    | ANDI      |
| 100    | LW        |
| 101    | SW        |
| 110    | BEQ       |
| 111    | JMP       |

## Cache Memory Components

The cache associative by set uses two mapping methods: direct mapping at the set level and associative mapping at the lines within the set. Direct mapping uses the SET field of the block address, which can be decoded into 4 sets.

#### LRU Algorithm Structure
![lru](https://github.com/Felkng/CPU_CACHE/blob/main/images-cpu/lru.png)
- **V1 and V2 Fields**: Indicate whether the lines are valid.
- **LRU 1 and LRU 2 Fields**: Indicate which line was the last to be stored.
- **Miss Field**: Indicates a cache failure, triggering line replacement only on failures.
- **Bloco Salvo Field**: Auxiliary bit for write-back operations, ensuring modified lines are written back to main memory before replacement.
- **WAY 1 and WAY 2 Outputs**: Indicate which line the next block will replace. Initially, when both lines are empty, line 1 (WAY 1) is chosen.

#### Cache Line Structure

![cache_line](https://github.com/Felkng/CPU_CACHE/blob/main/images-cpu/cache_line.png)

The data entry structure in the cache line includes:

![cache_line_bits](https://github.com/Felkng/CPU_CACHE/blob/main/images-cpu/line_bits.png)

- **Data/Instruction Storage**: Stores data or instructions.
- **Address Offset Field**: Determines the word to be overwritten in store-word operations.
- **Registers**: Manage the output and data storage of the line.
  - **TAG**: References the block in memory.
  - **V**: Validity bit.
  - **M**: Modification bit.
  - **LRU**: Indicates whether the line will be replaced.
  - **INSTRUÇÃO**: Indicates whether the block stored is an instruction or data.

Input structure for selection bits:

![input_bits](https://github.com/Felkng/CPU_CACHE/blob/main/images-cpu/selection_bits.png)

- **Offset**: References the word within the block.
- **Tag**: References the entire block in memory.
- **SW**: Enables new block replacement on cache failure.
- **CLK**: Circuit clock.
- **EN**: Enables writing in write-word and record block operations.
- **LRU**: Updates the LRU value.
- **CLEAR**: Clears each component when high.
- **Block Saved**: Informs the cache that the block was saved in main memory during write-back operations, clearing the modification bit.
- **Modified**: Enables the modification bit in store-word operations.

#### Write Policy Structure
![write_back](https://github.com/Felkng/CPU_CACHE/blob/main/images-cpu/write_back.png)

The write-back policy updates the data in main memory only when a modified cache line is overwritten. It evaluates whether the line has been modified and if it is about to be replaced, enabling data output for replacement. It filters each set to find the row to be replaced and checks for cache failure, enabling the write-back method to write data to main memory before reading a new block.

#### Cache Hit
![cache_hit](https://github.com/Felkng/CPU_CACHE/blob/main/images-cpu/cache_hit.png)

A cache hit occurs when the requested data from the CPU is found in the cache. The set-associative cache allows simultaneous searches in all lines of the set. A hit is confirmed if the saved tag matches the searched tag and the line is valid, selecting the data for cache output. If none of the sets meet these conditions, a cache miss occurs, requiring data retrieval from main memory.

#### Cache Controller Structure

![cache_controller](https://github.com/Felkng/CPU_CACHE/blob/main/images-cpu/cache_controller.png)

The cache controller manages the cache states and actions:

| State       | Actions                   |
|-------------|----------------------------|
| Stall       | BLOCK SAVED and MISS      |
| MemRead     | MISS                      |
| MemWrite    | WRITE BACK (WB)           |
| Cache Read  | HIT and LW                |
| Cache Write | HIT and SW                |

The cache controller ensures better control and visualization of circuit operations.

## Processor Components

### Registers Bank

![registers_bank](https://github.com/Felkng/CPU_CACHE/blob/main/images-cpu/registers_bank.png)

- **Registers**: 8 registers to hold data and perform operations.
- **RB Register**: Points to address 0 of the memory.
- **Outputs**: 3 outputs for RT, RS, and RD.

### Stall Unit

![stall_unity](https://github.com/Felkng/CPU_CACHE/blob/main/images-cpu/stall_unity.png)

Handles cache failures with the following controller bits:

- **stall**: Indicates a cache failure; turns off PC enable, stopping instruction execution.
- **writing**: Indicates data in the SW operation is still being written to the cache; PC enable is turned off.
- **stall 2**: Restarts an instruction if there is a cache miss.
- **start**: Auxiliary bit to start the PC at the main instruction address of the memory.

## Main Memory

![main_memory](https://github.com/Felkng/CPU_CACHE/blob/main/images-cpu/main_memory_cell.png)

The main memory has 1 K cells with two bytes each. The structure is presented as 10 bits of addressing, and 16 bits for words (2 bytes).

### Addressing

The address inside the main memory initially ignores the last 2 bits, because they are words inside the block. These 8 bits are concatenated with their respective words and targeted to the entrance of data of each cell of the main memory.

### In and Out Operations

- **SW**: Save the word.
- **LW**: Read the word.

### Storage Cells

- **Blocks**: 4 words per block.
- **BLOCK SAVED Bit**: Controller bit that indicates when data is correctly saved into the main memory.

## Results

The program execution given was this:

| Address | Instruction | Operation              | Label          |
|---------|-------------|------------------------|----------------|
| 32      | 5600        | add R6, RB, $0         | Label1: BeginFunc |
| 33      | 5544        | add R6, R6, $4         |                |
| 34      | 6cc0        | and R4, R4, $0         |                |
| 35      | 7100        | and R5, R5, $0         |                |
| 36      | 8140        | lw R1, 0(R6)           | ForLabel1:     |
| 37      | 1004        | add R5, R1, R5         |                |
| 38      | 4cc1        | add R4, R4, $1         |                |
| 39      | 5541        | add R6, R6, $1         |                |
| 40      | 5e00        | add R8, RB, $0         |                |
| 41      | 5dd8        | add R8, R8, $24        | soma 1         |
| 42      | b1c0        | sw R5, 0(R8)           |                |
| 43      | 6440        | and R2, R2, $0         |                |
| 44      | 444a        | add R2, R2, $10        |                |
| 45      | ccf7        | beq R4, R2, Label7     |                |
| 46      | e024        | jump ForLabel1         | EndFunc        |
| 47      | 560e        | add R6, RB, $14        | Label2: BeginFunc |
| 48      | 6cc0        | and R4, R4, $0         |                |
| 49      | 7100        | and R5, R5, $0         |                |
| 50      | 8140        | lw R1, 0(R6)           | ForLabel2:     |
| 51      | 1004        | add R5, R1, R5         |                |
| 52      | 4cc1        | add R4, R4, $1         |                |
| 53      | 5541        | add R6, R6, $1         |                |
| 54      | 5e00        | add R8, RB, $0         |                |
| 55      | 5dd9        | add R8, R8, $25        | soma 2         |
| 56      | b1c0        | sw R5, 0(R8)           |                |
| 57      | 6440        | and R2, R2, $0         |                |
| 58      | 444a        | add R2, R2, $10        |                |
| 59      | ccf8        | beq R4, R2, Label8     |                |
| 60      | e032        | jump Label2For         | EndFunc        |
| 61      | 5600        | add R6, RB, $0         | Label3: BeginFunc |
| 62      | 5a00        | add R7, RB, $0         |                |
| 63      | 5544        | add R6, R6, $4         |                |
| 64      | 598e        | add R7, R7, $14        |                |
| 65      | 6cc0        | and R4, R4, $0         |                |
| 66      | 7100        | and R5, R5, $0         |                |
| 67      | 8140        | lw R1, 0(R6)           | ForLabel3:     |
| 68      | 8580        | lw R2, 0(R7)           |                |
| 69      | 0c03        | add R4, R1, R4         |                |
| 70      | 0c43        | add R4, R2, R4         |                |
| 71      | 5541        | add R6, R6, $1         |                |
| 72      | 5981        | add R7, R7, $1         |                |
| 73      | 5101        | add R5, R5, $1         |                |
| 74      | 5e00        | add R8, RB, $0         |                |
| 75      | 5dda        | add R8, R8, $26        | soma 3         |
| 76      | adc0        | sw R4, 0(R8)           |                |
| 77      | 6880        | and R3, R3, $0         |                |
| 78      | 488a        | add R3, R3, $10        |                |
| 79      | d179        | beq R5, R3, Label9     |                |
| 80      | e043        | jump ForLabel3         | EndFunc        |
| 81      | 5600        | add R6, RB, $0         | Label4: BeginFunc |
| 82      | 5544        | add R6, R6, $4         |                |
| 83      | 6cc0        | and R4, R4, $0         |                |
| 84      | 7100        | and R5, R5, $0         |                |
| 85      | 8140        | lw R1, 0(R6)           | ForLabel4:     |
| 86      | 1004        | add R5, R1, R5         |                |
| 87      | 6cc2        | add R4, R4, $2         |                |
| 88      | 5542        | add R6, R6, $2         |                |
| 89      | 5e00        | add R8, RB, $0         |                |
| 90      | 5ddb        | add R8, R8, $27        | soma 4         |
| 91      | b1c0        | sw R5, 0(R8)           |                |
| 92      | 6880        | and R3, R3, $0         |                |
| 93      | 488a        | add R3, R3, $10        |                |
| 94      | cd7a        | beq R4, R3, Label10    |                |
| 95      | e055        | jump ForLabel4         | EndFunc        |
| 96      | 560e        | add R6, RB, $14        | Label5: BeginFunc |
| 97      | 5544        | add R6, R6, $4         |                |
| 98      | 6cc0        | and R4, R4, $0         |                |
| 99      | 7100        | and R5, R5, $0         |                |
| 100     | 8140        | lw R1, 0(R6)           | ForLabel5:     |
| 101     | 1004        | add R5, R1, R5         |                |
| 102     | 4cc2        | add R4, R4, $2         |                |
| 103     | 5542        | add R6, R6, $2         |                |
| 104     | 5e00        | add R8, RB, $0         |                |
| 105     | 5ddc        | add R8, R8, $28        | soma 5         |
| 106     | b1c0        | sw R5, 0(R8)           |                |
| 107     | 6880        | and R3, R3, $0         |                |
| 108     | 488a        | add R3, R3, $10        |                |
| 109     | cd7b        | beq R4, R3, Label11    |                |
| 110     | e064        | jump ForLabel5         | EndFunc        |
| 111     | 8200        | lw R1, 0(RB)           | --MAIN--       |
| 112     | 8601        | lw R2, 1(RB)           |                |
| 113     | 0801        | add R3, R1, R2         |                |
| 114     | 2c01        | and R4, R1, R2         |                |
| 115     | 8202        | lw R1, 2(RB)           |                |
| 116     | 8603        | lw R2, 3(RB)           |                |
| 117     | 0000        | -                      | -              |

As it is a unified memory, the first 32 lines (0 - 31) are used for data storage, while lines 32 to 139 are the actual operations carried out in the program. The code aims to perform summations within functions in which the results obtained were:

- **Sum 1:** 155
- **Sum 2:** 155
- **Sum 3:** 310
- **Sum 4:** 75
- **Sum 5:** 516

Regarding the results obtained for cache misses and hits and the total number of memory accesses during program execution were:

- **Failures:** 59
- **Hits:** 585
- **Hypothetical total memory accesses:** 642

With a total of 59 cache misses, this means there are a total of 59 accesses to main memory. Based on the hypothesis that there would be 642 memory accesses if there was no cache on the processor, we can say that there was a reduction of approximately 90% in accesses to main memory.
