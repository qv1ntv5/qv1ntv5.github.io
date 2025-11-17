---
layout: post
title: Basic Instructions.
subtitle: Introduction to NOP, PUSH and POP instructions.
tags: [assem]
---

### 1. Introduction. Assembly instruction definition.

An assembly instruction is a human-readable representation of a machine code operation that a computer's processor can execute. It's a low-level programming command that directly corresponds to the binary instructions that the CPU understands.

Basically they are mnemonic codes (like MOV, ADD, JMP) that represent specific operations the processor can perform. Each instruction typically consists of:

- An opcode (operation code) - what operation to perform.
- Operands - the data or memory locations the operation acts upon.

They serve as an intermediate language between high-level programming languages (like C, Python) and raw machine code (binary), making it easier for programmers to write and understand low-level code.

In this section, as long as we present some instructions such as PUSH or POP, we will introduce some important structures or actions related with those instructions, for example the Stack. So in this section we will cover more

<br>

### 2. No-Operation (NOP)

One singular operation is the NOP operation:

 - NOP - No operation (do nothing for one cycle)

It servers to pad/align bytes or delay time, often used for a block of code (such exploits) to execute in a more reliable way.

As trivial note, NOP is nothing but an alias for the operation: *XCHG EAX EAX*, this will be covered later and will get full sense.

<br>

### 3. The Stack. RSP pointer. Push & Pop operations.

#### 3.1. Introduction to the stack.

A Stack is a LIFO data structure where data is pushed in or popped out from the top. Is a conceptual area of main memory (RAM) designated by the OS when a program start to run, (conceptual since is just a normal chunk of memory we treat as stack structure).

```less
RAM (Physical Memory):
┌─────────────────┐ 0x00000000
│   OS Kernel     │
├─────────────────┤
│   Program Code  │ ← Your program's instructions
├─────────────────┤
│   Data/Heap     │ ← Global variables, malloc'd memory
├─────────────────┤
│   STACK         │ ← "Conceptual area designated by OS"
│   (grows down)  │    This is just RAM, but used as a stack!
└─────────────────┘ 0xFFFFFFFF
```

Stack and Heap tends to overlap the same memory region both starting opposite sides and groeing opposites directions. By convention, stack grows toward lower directions (memory address nearest to 0x00000000).

<br>

##### 3.1.2. RSP Pointer.

Recovering what we where saying on "Computer Registers" section, RSP or R4 is an x86-64 pointer which points to the top of the stack, this means, points to the last element pushed onto the stack or, what is the same, to the lowest currently-used address in the stack.

It is worth to mention that data exists in all memory address but those being unused is considered undefined.

<br>

##### 3.1.3. Elements stored in the stack.

This are some of the elements we can find in the stack; (Some of the elements presented next will be explained in deep later.)


- *Return Address* so a called function can return to the function that call it.
- Local variables.
- Arguments passed between functions.
- Space for registers.
- Dynamically allocated memory via alloca() (old allocation way, heap-allocation via malloc is way more common).

<br>

##### 3.1.4. Nested stacksframes.

At first, lets stablish a difference between stack, and stackframe. For convention, stack refers to the whole memory region or memory address that can hold elements pushed on the stack of a program.

Inside the program stack, there exist reserved memory regions called stack frames, one for each function call, which hold the function's context (parameters, local variables, return address, etc.). We can think about the stack frames as the context of the function being called pushed as once to the program stack.

To ilustrate this idea lets consider the following code:

```c
#include stdio.h

int bar(int y) {
    int a = 3*y;
    printf("bar returned %d", a);
    return a;
}

int foo(int x){
    int b = 5*x;
    printf("foo passed %d", b);
    return bar(b);
}

int main() {
    int c = foo(7);
    printf("main passed %d", c);
    return 0;
}
```

The stack representation ggets over the following steps:

- First, the program run and start by executing main() function, so main's stackframes is pushed on the top of the program stack:

    ```less
    Stack (grows downward):
    High Address
        ┌──────────────────────┐
        │   main's Stack Frame │
        ├──────────────────────┤
        │ Return addr (to OS)  │
        ├──────────────────────┤
        │ Saved RBP            │
        ├──────────────────────┤ ← RBP
        │ c (uninitialized)    │
        └──────────────────────┘ ← RSP
    Low Address
    ```

- Then, main() calls foo() and then foo's stackframe gets pushed onto the top of the stackfram of main:

    ```less
    Stack:
    High Address
        ┌──────────────────────┐
        │   main's Stack Frame │
        ├──────────────────────┤
        │ Return addr (to OS)  │
        ├──────────────────────┤
        │ Saved RBP (old)      │
        ├──────────────────────┤
        │ c (uninitialized)    │
        ├──────────────────────┤
        │   foo's Stack Frame  │ ← NEW FRAME!
        ├──────────────────────┤
        │ Return addr (main)   │ ← Where to return after foo
        ├──────────────────────┤
        │ Saved RBP (main's)   │
        ├──────────────────────┤
        │ x = 7 (parameter)    │
        ├──────────────────────┤ ← RBP (foo's base)
        │ b = 35 (5*7)         │
        └──────────────────────┘ ← RSP
    Low Address
    ```

    And the same happens when foo() calls bar():

    ```less
    Stack:
    High Address
        ┌──────────────────────┐
        │   main's Stack Frame │
        ├──────────────────────┤
        │ Return addr (to OS)  │
        ├──────────────────────┤
        │ Saved RBP            │
        ├──────────────────────┤
        │ c (uninitialized)    │
        ├──────────────────────┤
        │   foo's Stack Frame  │
        ├──────────────────────┤
        │ Return addr (main)   │
        ├──────────────────────┤
        │ Saved RBP (main's)   │
        ├──────────────────────┤
        │ x = 7                │
        ├──────────────────────┤
        │ b = 35               │
        ├──────────────────────┤
        │   bar's Stack Frame  │ ← NEWEST FRAME!
        ├──────────────────────┤
        │ Return addr (foo)    │ ← Where to return after bar
        ├──────────────────────┤
        │ Saved RBP (foo's)    │
        ├──────────────────────┤
        │ y = 35 (parameter)   │
        ├──────────────────────┤ ← RBP (bar's base)
        │ a = 105 (3*35)       │
        └──────────────────────┘ ← RSP (stack pointer at deepest)
    Low Address
    ```

- When bar() execution finish then, the return value returns on the RAX register and the bar() stackframe gets popped out from foo() stackframe, the flow of execution returns over foo() (we say that bar() returns over foo()):

    ```less
    Stack (bar's frame is popped):
    High Address
        ┌──────────────────────┐
        │   main's Stack Frame │
        ├──────────────────────┤
        │ Return addr (to OS)  │
        ├──────────────────────┤
        │ Saved RBP            │
        ├──────────────────────┤
        │ c (uninitialized)    │
        ├──────────────────────┤
        │   foo's Stack Frame  │
        ├──────────────────────┤
        │ Return addr (main)   │
        ├──────────────────────┤
        │ Saved RBP (main's)   │
        ├──────────────────────┤
        │ x = 7                │
        ├──────────────────────┤ ← RBP
        │ b = 35               │
        │ RAX = 105 (return)   │ ← Return value in register
        └──────────────────────┘ ← RSP
    Low Address
    ```

    And foo() returns over main() with the return value on RAX:

    ```less
    Stack (foo's frame is popped):
    High Address
        ┌──────────────────────┐
        │   main's Stack Frame │
        ├──────────────────────┤
        │ Return addr (to OS)  │
        ├──────────────────────┤
        │ Saved RBP            │
        ├──────────────────────┤ ← RBP
        │ c = 105              │ ← Gets value from RAX
        │ RAX = 105            │ ← Return value
        └──────────────────────┘ ← RSP
    Low Address
    ```

    Then main returns and program ends poping out what is left.

<br>

#### 3.2. Push & Pop operations.

Push and Pop are two main instructions in assembly, Push and Pop, mainly used to introduce or get things out from the stack.         

<br>

##### 3.2.1. Push.

###### 3.2.1.1. Definition.

PUSH places data onto the stack, specifically, it push Word, Doubleword or Quadword onto top the stack. The sintax of the instruction is:

```
PUSH source
```

The effects of PUSH some source into the stack are:

- The RSP automatically gets decrement an amount of bytes 2, 4 or 8 bytes, this depends on what is being pushed (for example, if you push AX, then you would push 16 bits or 2 bytes, if you push RAX then RSP would decrement 8 bytes), note that what is being pushed is also dependant of the architecture of the system.

- Copies the content of the source operand to the location pointed to by the new stack pointer (RSP).

For example, lets assume that the register of a CPU contains the following values:

```less
| Register | Value |
| --- | --- |
| RAX | 00000000`00000004 |
| RSP | 00000000`0014FE00 | 
```

And the stack contains for example:

```less
| address | stack |
| --- | --- |
| 00000000`0014FE10 | 00000000`00000001 |
| 00000000`0014FE08 | 00000000`00000002 |
| 00000000`0014FE00 | 00000000`00000003 | (RSP)
| 00000000`0014FDF8 |       undef       |
```

Then, lets suppose the folowing instruction is performed:

```c
push RAX
```

Then, stack would grow; the RSP register would point to a lower quadword and the RAX content would be copied on that address making all lok as follows:

```less
| Register | Value |
| --- | --- |
| RAX | 00000000`00000004 |
| RSP | 00000000`0014FDF8 | 


| address | stack |
| --- | --- |
| 00000000`0014FE10 | 00000000`00000001 |
| 00000000`0014FE08 | 00000000`00000002 |
| 00000000`0014FE00 | 00000000`00000003 | 
| 00000000`0014FDF8 | 00000000`00000004 | (RSP)
```

We can observe that RSP value change as well as the content of the address pointed by RSP.

<br>

Let's talk now about what 'source' can be. The source or operand of a PUSH instruction can be either:

- A register.
- A value from memory given in *r/mX* form (r/m16, r/32, r/64), we will se this later.

<br>

###### 3.2.1.2. "r/mX" Address Form. 1st and 2nd forms.

The "r/mX" notation is just shorthand that instruction manuals use to say "this operand can be either a register OR a memory address" - and the X indicates the bit size. So what *r/mX* means:

- r = register (like rax, rbx, etc.)
- m = memory address
- X = bit size (8, 16, 32, or 64)

So "r/m64" means "this can be either a 64-bit register OR a 64-bit memory address."

In Intel syntax, most of the square brackets [] means to treat the value within as a memory address and then fetch the value at that address (like deferencing a pointer).

Thus, an "r/mX" form can take 4 forms:

**1st.** Register -> rbx; Use the register's content.

**2nd.** Memory, base only -> \[rbx\]; As we say above, \[\] means, treat what is inside as memory address and derefence it. So, while the operation above treats *rbx* as a normal variable (use the contents of rbx), this nomenclature treats *rbx* as a pointer, this is; as if *rbx* contains a memory address. Then \[rbx\] means use the contents stored in the memory address holded by *rbx*.

<br>

###### 3.2.1.3. "r/mX" Address Form. 3th and 4th forms.

The forms 3 and 4 have been designed to identify items within an array, thus they need some further explanation.

Let's think that we have an array of integers in C:

```c
int arr[10];
int value = arr[5];
```

In order to access the 5th item of the array, the CPU needs to calculate the position in memory. So, what we know is that we have a pointer to the base of the array which is *arr* and also, we have the position of the item within the array and since the array is an array of integers, every slot have an integer's width (4 bytes), so applicating poiner arithmetics teh CPU calculates the address position of the item we want to reach:

```c
arr + 5
```

In C this simple fórmula works because the compiler already knows that arr is an int type array or, saying in other words, arr is a pointer to integer so arr + 5 is moving 5 times forward in the array but this is not accurate in assembly because at lowlevel data type are dismissed and all is understanded in terms of memory address. One address store 1 byte of data so we have to remake the formula above in a form that works with 1 byte step:

```c
(void *) arr + 5 * sizeof(int);
```

If we pass this formula to assembly, we get the 3th form of memory address representation:

**3th.** Memory, base + index -> \[rbx + rcx\*X\]

This formula maps directly between each other:

```c
(void *) arr +  5 * sizeof(int)
          ↓     ↓      ↓
[        rbx + rcx  *  X     ]
```

According to our Computer Registers section:

- RBX; Base pointer to memory access.
- RCX; Counter or index pointer.
- X; Scale (to measuer the width of each step)

<br>

From this formula, we could add a *displacement* (Y) in form of an offset to for example reach structs fields in array items:

**4th.** Memory, base + index\*scale + displacement -> \[rbx + rcx\*X + Y\]

<br>

###### 3.2.1.4. Note about the ` address convention.

When writing 64 bits numbers, it can be easy to lose track of whether you have the right number of digits. WinDbg allow you to write 64 bit numbers with a ` between the two 32 bit halves and it is useful to note at a glance whether if a number is bigger than 32 bits or not, so it will often write that numbers like that but it is only a convention of WinDbg (which we will not be using).

<br>

##### 3.2.2. Pop.

Now that we have explained PUSH operation, we can define POP as the reverse.

POP removes data from the stack, specifically, it pops a Word, Doubleword, or Quadword from the stack into a destination. The syntax of the instruction is:

```less
POP destination
```

The effects of POP some destination from the stack are:

- Copies the content from the location pointed to by the current stack pointer (RSP) into the destination operand.

- The RSP automatically gets incremented by an amount of bytes depending on the architecture of the system.

Retrieving the example from PUSH:

If we have the following state of the stack and the CPU:

```less
| Register | Value |
| --- | --- |
| RAX | 00000000`00000016 |
| RSP | 00000000`0014FDF8 | 


| address | stack |
| --- | --- |
| 00000000`0014FE10 | 00000000`00000001 |
| 00000000`0014FE08 | 00000000`00000002 |
| 00000000`0014FE00 | 00000000`00000003 | 
| 00000000`0014FDF8 | 00000000`00000004 | (RSP)
```

And the instruction pop RAX is executed (note that this means that the system is x86-64 and POP/PUSH would move quadwords), then RSP would increment in a quadword and the RAX register would get filled with the contents of the address pointed by RSP:

```less
| Register | Value |
| --- | --- |
| RAX | 00000000`00000004 |
| RSP | 00000000`0014FE00 | 


| address | stack |
| --- | --- |
| 00000000`0014FE10 | 00000000`00000001 |
| 00000000`0014FE08 | 00000000`00000002 |
| 00000000`0014FE00 | 00000000`00000003 | (RSP)
| 00000000`0014FDF8 | 00000000`00000004 | 
```

Note that the contents of 0014FDF8 are not erased but the address it self is *unused* and the contents are considered *undefined*.

<br>

#### 3.3. Practical exercises.

In order to reinforce the concepts lets do some practical exercises:

```
HIGH ADDRESSES
================
BA771E70AD5 <- RBP
================
7e1eca57
================
cabba1a
================
deadfa11
================
b100d <- RSP
================
LOW ADDRESSES
What is the offset to cabba1a ?
(Enter answer in the form of "rsp{+,-}0x??" or "rbp{+,-}0x??",
where ?? must always be 2 digits, e.g. rsp-0x00 or rbp+0x08)
```

The answer must be, rbp-0x10 or rsp+0x10 

Another try:

```
HIGH ADDRESSES
================
d00dad <- RBP
================
b1ade
================
c0bb1e
================
d0771e
================
c01055a1 <- RSP
================
LOW ADDRESSES

What is the offset to d0771e ?
(Enter answer in the form of "rsp{+,-}0x??" or "rbp{+,-}0x??",
where ?? must always be 2 digits, e.g. rsp-0x00 or rbp+0x08)
```

The answer must be either rbp-0x18 or rsp+0x08. 

Other one:

```
HIGH ADDRESSES
================
1ead <- RSP
================
FacedFab1edF0e
================
deadfa11
================
ba1dd00d
================
7e1eca57 <- RBP
================
f007ba11
================
babb1e
================
5e1ec7ee
================
LOW ADDRESSES
What is the offset to babb1e ?
(Enter answer in the form of "rsp{+,-}0x??" or "rbp{+,-}0x??",
where ?? must always be 2 digits, e.g. rsp-0x00 or rbp+0x08)
```

The answer must be: rsp-0x30 or rbp-0x10

As you can see, each step is 8 bytes wide and always add towards higher address and substract towards lower address.

One more:

```
LOW ADDRESSES |----------------|----------------|----------------|----------------| HIGH ADDRESSES
LOW ADDRESSES |..........5e1ec7|......de7ec7ab1e|........ca55e77e|.........effab1e| HIGH ADDRESSES
LOW ADDRESSES |^^^^^^^^^^^^^^^^|----------------|----------------|----------------| HIGH ADDRESSES
LOW ADDRESSES |____RBP_&_RSP___|________________|________________|________________| HIGH ADDRESSES
What is the offset to effab1e ?
(Enter answer in the form of "rsp{+,-}0x??" or "rbp{+,-}0x??",
where ?? must always be 2 digits, e.g. rsp-0x00 or rbp+0x08)
```

The answer is rsp/rbp+0x18.

<br>