---
layout: post
title: Computer Registers.
subtitle: Brief introduction to CPU Registers.
tags: [assem]
---

### 1. Computer Memory Hierarchy.

Computer memory hierarchy is a structured arrangement of different types of storage in a computer system, organized in terms of efficiency between speed access, capacity and availability. 

1. At the top there are the fastest but smallest in storage: **CPU registers** (directly inside the processor, nanosecond access).

2. Just below are **cache memories** (L1, L2, L3), which are extremely fast SRAM that stores frequently accessed data - L1 is fastest but smallest, L3 is slower but larger.

3. Next comes **main memory (Random Access Memory)** - larger capacity (gigabytes) but slower than cache. This is where your running programs and their data actually live.

4. Below that are secondary storage devices like **SSDs and hard drives** - much larger (terabytes) but significantly slower, where data persists when device power is off.

5. Finally, at the bottom are tertiary storage systems like **tape drives or cloud storage** - massive capacity but very slow access. Data persist out of the device and can be accessed by several devices.

    <br>

    | Level | Type | Size | Speed | Volatile |
    |-------|------|------|-------|----------|
    | 1 | Registers | ~1 KB | <1 ns | Yes |
    | 2 | L1 Cache | 32-64 KB | 1-2 ns | Yes |
    | 3 | L2 Cache | 256 KB-1 MB | 3-10 ns | Yes |
    | 4 | L3 Cache | 8-32 MB | 10-20 ns | Yes |
    | 5 | RAM | 4-64 GB | ~100 ns | Yes |
    | 6 | SSD | 256 GB-4 TB | 0.1-1 ms | No |
    | 7 | HDD | 1-10 TB | 5-10 ms | No |
    | 8 | Tape/Cloud | Unlimited | sec-min | No |

<br>

The key principle: *as you go down the hierarchy, you get more capacity and availability but slower speed. As you go up, you get faster access but less space. The system tries to keep frequently-used data in faster levels through various caching mechanisms.*

<br>

### 2. x86-64 - General Purpouse Registers.

#### 2.1. Definition of a register.

Registers are small memory storage areas built into the processor. During program execution, the CPU cannot perform operations (like add, subtract, compare) directly on data in RAM. Data must first be loaded from RAM into a register. The operation is performed inside the CPU using the registers, and the result is then stored back to a register or written to RAM. On the registers essential data just about to be used by the processor at runtime is stored, operate with, and then returned back into RAM.

<br>

#### 2.2. Evolution of registers.

x86-64 is a 64-bit extension of the older 32-bit x86 architecture. This evolution is key to understanding the register names. The original 16-bit registers were extended to 32-bit, and then again to 64-bit.

So, for example lets take the register RAX. RAX is the 64-bits version of the "Accumulator Register", the lower 32 bits analog register is EAX ("E" for Extended, from the 32-bit era) which comes from the original lower 16 bits of that are AX (the original 16-bit registe: A eXtended). AX were split into AH (High 8 bits) and AL (Low 8 bits).

```less
┌────────────────────────────────────────────────────────────────┐
│                          RAX (64 bits)                         │
├───────────────────────────────────┬────────────────────────────┤
│          (upper 32)               │      EAX (32 bits)         │
│                                   ├────────────┬───────────────┤
│                                   │ (upper 16) │  AX (16 bits) │
│                                   │            ├───────┬───────┤
│                                   │            │  AH   │  AL   │
│                                   │            │ (8b)  │ (8b)  │
└───────────────────────────────────┴────────────┴───────┴───────┘
 63                              32             16   15  8 7     0
```

There also exists other names for this registers:

- RAX - R0
- EAX - R0D(ouble) Word
- AX  - R0W(ord)
- AL/AH - R0B(yte)

<br>

#### 2.3. Viewing the most important registers.

#### 2.3.1. General-Purpouse registers.

These registers are primarily used for arithmetic, logic, data movement, and memory addressing. While they have historical "specialized" roles, a modern compiler or programmer can use them for almost any purpose.

1. **RAX (Accumulator)**: The workhorse for arithmetic operations and function return values. While specialized, it's still considered general-purpose because you can store any value in it.

2. **RCX (Counter)**: Traditionally used for loops (LOOP instruction, rep prefixes), but can hold any data or address.

3. **RDX (Data)**: Used in extended arithmetic operations (e.g., with RAX for multiply/divide) and I/O, but is fully general-purpose.

4. **RBX (Base)**: A general-purpose data register. Its main distinction is that it's callee-saved, meaning functions must preserve its value.

5. **RSI (Source Index)**: Used as the source pointer for string operations (movs, lods, cmps), but is a general-purpose register for holding data or addresses.

6. **RDI (Destination Index)**: Used as the destination pointer for string operations, but is otherwise a general-purpose register.

<br>

#### 2.3.2. Special-Purpose Registers.

These registers have a dedicated, hardware-defined function critical to program execution. We cannot use them for general arithmetic or to hold arbitrary data in the same way.

1. **RSP (Stack Pointer)**

    Points to the top of the current stack frame. It's managed automatically by instructions like CALL, RET, PUSH, and POP.
    
    If RSP gets wrongly manipulated, the program will almost certainly crash. While it can be manipulated directly (e.g., sub rsp, 16 to allocate space), it cannot be used as a scratch register like RAX or RCX.

2. **RBP (Base Pointer)**

    Points to the base of the current stack frame to provide a stable reference for accessing local variables and parameters.

    Its role is defined by software convention and is crucial for stack unwinding and debugging. While modern optimizations (-fomit-frame-pointer) can free it for general use, by default, it is considered a special-purpose stack management register.

3. **RIP (Instruction Pointer)**

    Holds the memory address of the next instruction to be executed, it controls the flow of execution of a program.
    
    This register is critic, it cannot directly be manipulated (read/write) as with the others. Its value is controlled entirely by the CPU through instruction execution, jumps, and calls.

<br>