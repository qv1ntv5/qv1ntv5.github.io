---
layout: post
title: Calling Functions.
subtitle: Introduction to CALL, RET, MOV, ADD and SUB instructions.
tags: [assem]
---

### 1. Calling functions.

In order to introduce some more instructions, lets consider the following C code

```c
int func(){
    return 0xbeef;
}

int main() {
    func();
    return 0xf00d;
}
```

and the corresponding Assembly tranlation:

```assem
func:
0000000140001000 mov    eax, 0BEEFh
0000000140001005 ret

main:
0000000140001010 sub    rsp, 28h
0000000140001014 call   func (0140001000h)
0000000140001019 mov    eax, 0F00Dh
000000014000101E add    rsp, 28h
0000000140001022 ret
```

The assembly code above contains all the function we gonna review.

<br>

### 2. CALL and RET.

#### 2.1. Definition and Sintax.

##### 2.1.1. CALL.

The CALL instruction is one of the most important instructions in assembly language, used to implement function/procedure calls. CALL's job is to transfer the flow of execution to a different function with the appropiate measures to ensure the flow can be resumed later where it left off using RET.

The CALL instruction performs two essential operations:

- **Saves the return address** - Pushes the address of the next instruction (after CALL) onto the stack, this ensures the restauration of the flow of execution when the called function finish.
- **Jumps to the target** - Introduces in the RIP register (Instruction Pointer register; pointer to the next instruction to execute) the address of the first instruction of the called function. This transfers control to the specified function/procedure address.


It has the following sintax:

```
CALL memory_address    ; Direct call
CALL register          ; Indirect call through register
CALL [r/mX]            ; Indirect call through memory
```

Note that CALL is equal to:

```assem
push RIP + jmp target
```

<br>

##### 2.1.2. RET.

The RET instruction is the counterpart to CALL, used to return from a function/procedure back to the caller.

RET performs two essential operations:

- **Pops the return address** - Retrieves the return address from the top of the stack which was saved previously by CALL.
- **Jumps to that address** - Changes the RIP value with the address of the instruction popped out from the stack. This way it transfers the control back to the instruction after the original CALL.

It gets called simply with RET.

Is worth to mention that RET's functionally is equivalent to POP RIP (pop out the next instruction of the stack, the address previously saved by CALL, and insert it in the destination) though you can't actually write POP RIP directly in most assemblers because RIP/EIP is a special register (not a general-purpouse register).

```assem
pop RIP + jmp RIP
```

<br>

#### 2.2. Sinergy in an example.

Now that we know how CALL and RET formaly works. Let's see the behaviour described before in an example, some address are just cutdown for simplicity, the archicture however is x86-64:

```less
main:
    mov rax, 10          ; Address: 0x401000
    mov rbx, 20          ; Address: 0x401007
    call AddNumbers      ; Address: 0x40100E (pushes 0x401013 onto stack)
    mov rcx, rax         ; Address: 0x401013 - rax contains 30
    ret                  ; Address: 0x40101A

AddNumbers:              
    add rax, rbx         ; Address: 0x402000 - rax = 10 + 20 = 30
    ret                  ; Address: 0x402003 - pops 0x401013, jumps there
```

<br>

Then the stack status before CALL is something like follows:

```less
Step 1: Before CALL
┌─────────────────────────────────────┐
│ RIP: 0x40100E                       │
│ RSP: 0x7FFFFFFFE000 (stack pointer) │
│ RAX: 10                             │
│ RBX: 20                             │
└─────────────────────────────────────┘
```

When CALL function is processed, the address of the next instruction after call (0x401013) gets pushed into the stack and RIP contains the address of the first instruction of *AddNumbers*:

```less
Step 2: CALL executes
┌─────────────────────────────────────┐
│ Action 1: PUSH 0x401013             │  Push return address (8 bytes)
│ Action 2: RSP = 0x7FFFFFFFDF8       │  Stack grows down by 8 bytes QUADWORD
│ Action 3: RIP = 0x402000            │  Jump to AddNumbers
└─────────────────────────────────────┘

Stack after CALL (64-bit):
      High Memory
    ┌──────────────────┐
    │       ...        │
    ├──────────────────┤ 0x7FFFFFFFE000 
    │ 0x0000000401013  │ 0x7FFFFFFFDFF8  (RSP)
    ├──────────────────┤
    │       ...        │
      Low Memory
```

Now, the function *AddNumbers* execute normally, it perform a simple addition and then RET takes place, it pops out the next instruction of the stack (which result to be the return address saved by CALL) into RIP, restoring the execution flow of the program to the caller:

```less
Step 3: Inside AddNumbers
┌─────────────────────────────────────┐
│ RIP: 0x402000                       │
│ RSP: 0x7FFFFFFFDF8                  │
│ Execute: add rax, rbx               │
│ RAX: 30 (result)                    │
└─────────────────────────────────────┘

Step 4: RET executes
┌─────────────────────────────────────┐
│ Action 1: POP into RIP              │  RIP = 0x401013
│ Action 2: RSP = 0x7FFFFFFFE000      │  Stack restored (RSP + 8)
└─────────────────────────────────────┘
```

Now, with RET having restored the flow of execution, the system is ready to continue with the execution of the program where it leaved it:

```less
Step 5: Back in main
┌─────────────────────────────────────┐
│ RIP: 0x401013                       │  Continues after CALL
│ RSP: 0x7FFFFFFFE000                 │  Stack pointer restored
│ RAX: 30                             │  Has the result
└─────────────────────────────────────┘
```

<br>

#### 2.3. Brief warning.

Is worth to mention that, despite we warn about CALL and RET are equals to some instructions which manipulates RIP register:

```assem
CALL <==> push RIP + jmp target
RET <==> pop RIP + jmp RIP
```

This lines can't be found the example above, this is because RIP is a very critic register as we see in our section dedicated to [registers](https://qv1ntv5.github.io/2025-11-08-ComputerRegisters/) and the CPU performs the action for us. We will se in other procedures, like *stackframe creation*, lines which manipulates special-purpouses registers as RBP or RSP because this are optional procedures established by compiler's convention.

<br>

### 3. Two-Operand instructions sintax.

#### 3.1. Syntax.

There are two types of syntax in base of the flow of the assignation:

- **Intel Syntax**: Destination <- Source (like in algebra, *y = x + 1*).
- **AT&T Syntax**: Source -> Destination (the opposite, mlike when we calculate the result of an operation 1 + 2 = 3).

For now on, we gonna read in Intel syntax, but is worth to note that exists both of them.

<br>

#### 3.2. CISC vs RISC Architectures.

In the world of computers, "architecture" (specifically, Instruction Set Architecture or ISA) is like the basic vocabulary and grammar of a language that the CPU understands. When C code is compiled, the compiler translates this C code to a set of instructions that the CPU understand and is able to process, this is machine-code and the precise set of instructions or commands used to craft this byte-level code is the so called architecture.

Assembly is a human-readable translation of the machine-code.

In this context, this set of instructions can be:

- RISC (Reduced Instruction Set Computer); small set of simple, highly optimized instructions.

- CISC (Complex Instruction Set Computer); a large set of powerful, complex instructions 

Regarding to our interests, there is a major distinction between CISC and RISC architectures on how they handle different size-architectures commands.

- RISC; have different instructions for the same operation depending of the size being operated with.

    ```assem
    LDRB R1, [R2]    ; Load BYTE (8-bit) from memory address in R2
    LDRH R1, [R2]    ; Load HALFWORD (16-bit) from memory address in R2  
    LDR  R1, [R2]    ; Load WORD (32-bit) from memory address in R2
    LDRD R1, R2, [R3] ; Load DOUBLEWORD (64-bit) from memory address in R3
    ```

- CISC use the same set of instructions despite the size and use directives to specify the amount of bits of an address that is being used in an operation. It allows using the same instruction with different operand sizes, if a directive specifying the size of the data in the operation is used in:

    - Memory Operations with Immediate Values:


        ```assem
        MOV BYTE PTR [eax], 5     ; Write 1 byte
        MOV WORD PTR [eax], 5     ; Write 2 bytes  
        MOV DWORD PTR [eax], 5    ; Write 4 bytes
        MOV QWORD PTR [eax], 5    ; Write 8 bytes
        ```

    - Memory Operations where Register Size isn't Clear:

        ```assem
        MOV [ebx], 10     ; Ambiguous: size unknown
        MOV DWORD PTR [ebx], 10  ; Clear: 32-bit operation
        ```

        Size directives are only needed when the size cannot be inferred from the registers

There are a lot of cases but the important thing is that this serve to us to introduce the size directives:

    ```assem
    BYTE PTR [r/mX];  Use one 1 byte from the address r/mX.
    WORD PTR [r/mX]; Use 2 bytes from the address r/mX. 
    DWORD PTR [r/mX]; Use 4 bytes from the address r/mX.
    QWORD PTR [r/mX]; Use 8 bytes from the address r/mX.
    ```


### 4. MOV - Move.

#### 4.1. Definition.

The MOV (move) instruction is one of the most fundamental and frequently used instructions in assembly language. Despite its name, it doesn't actually "move" data it copies data from a source to a destination.

<br>

#### 4.2. Syntax and restrictions.

The basic sintax of the MOV instruction is:

```assem
MOV destination, source
```

And MOV can copy between: 

1. Register to Register:

```assem
MOV rax, rbx        ; Copy value from RBX to RAX
MOV eax, ebx        ; 32-bit registers
MOV ax, bx          ; 16-bit registers
MOV al, bl          ; 8-bit registers
```

2. Immediate Value to Register:

```assem
MOV rax, 42         ; Store the value 42 in RAX
MOV rcx, 0x1000     ; Store hexadecimal value in RCX
MOV edx, -5         ; Negative values work too
```

3. Memory to Register (Load):

```assem
MOV rax, [rbx]              ; Load value from memory address in RBX into RAX
MOV eax, [rsp+8]            ; Load from stack offset
MOV rcx, [rbp-16]           ; Access local variables
MOV rdx, QWORD PTR [rax]    ; Explicit size qualifier (64-bit)
```

4. Register to Memory (Store):

```assem
MOV [rax], rbx              ; Store RBX value to memory address in RAX
MOV [rsp+4], ecx            ; Store to stack
MOV DWORD PTR [rdi], 100    ; Store immediate to memory (32-bit)
```

5. Immediate Value to Memory:

```assem
MOV QWORD PTR [rax], 42     ; Store value 42 to memory (need size specifier)
MOV BYTE PTR [rbx+5], 0xFF  ; Store byte value
```

<br>

Also, there are a few restrictions:


- **No Memory-to-Memory**

    In x86-64 there are no memory-to-memory tranfer operations, they need a register as an intermediate:

    ```assem
    MOV [rax], [rbx]    ; INVALID - cannot move directly between memory locations
    
        
    ; Must use a register as intermediate:
    MOV rcx, [rbx]      ; Load from memory
    MOV [rax], rcx      ; Store to memory
    ```


- **Size Matching**

    ```assem
    MOV rax, ebx        ; INVALID - size mismatch (64-bit vs 32-bit)
    MOV eax, ebx        ; VALID - both 32-bit
    MOV rax, rbx        ; VALID - both 64-bit
    ```

- **Segment Registers (Special Case)**:

    ```assem
    MOV ds, ax          ; Load segment register from general register
    MOV ds, 0x10        ; INVALID - cannot load immediate into segment register
    ```

<br>

### 5. ADD & SUB.

This instructions are both kind of trivials, they just add or substract as expected except between memory address. Both instructions performs addition/substraction on two operands and stores the result in the destination operand.

```
ADD/SUB destination, source    ; destination = destination + source
```

Again, the destination and the source (this is, the operands) can be registers, immediates or memory address but since memory copy from memory to memory is not allowed both operands cannot be memory addressa at the same time.


```assem
add rsp, 8 -> (rsp = rsp + 8)
sub rax, [rbx*2] -> (rax = rax - memorypointedtoby(rbx*2))
```

<br>