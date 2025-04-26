---
layout: post
title: SharedLibraries&FunctionHooking
subtitle: SharedLibraries and FunctionHooking theory with exampls in C
tags: [C]
---

## Binaries and Shared Libraries.

### Introduction to binaries.

In Unix-like system, the common standard file format for executable files is the **ELF** (Executable and Linked Format). Here we will break up a little how this binary is structured and what executes a file means.

<br>

#### Parts of ELF files (Disk and Memory).

An ELF binary is divided into three main parts:

1. **ELF Header**; which contains metadata about the file (pointers to other sections of the binary or structures necesary for the binary's execution).
2. **Program Header Table**; which defines segments used at runtime (for execution).
3. **Section Header Table**; which defines .text (code), .data (data), among other things (for linking and debugging).

Thus, we can have two representation of how a ELF file looks like:

- Disk (ELF Binary):

```pgsql
+-------------------------+
| ELF Header             |  <- Contains metadata, entry point, offsets, etc.
+-------------------------+
| Program Header Table   |  <- Describes segments (for execution).
+-------------------------+
| Sections (.text, .data)|  <- Code, data, symbols.
+-------------------------+
| Section Header Table   |  <- Describes sections (for linking/debugging).
+-------------------------+
```

And in Memory (Process Image):

```pgsql
+----------------+  <- Low Memory
| NULL Page     |  (to catch NULL pointer derefs)
+----------------+
| ELF Headers   |  (mapped from file)
+----------------+
| Code (.text)  |  (read-execute)
+----------------+
| Data (.data)  |  (read-write)
| BSS (.bss)    |  (read-write, uninitialized vars)
+----------------+
| Heap (malloc) |  (grows up)
+----------------+
| Stack         |  (grows down)
+----------------+  <- High Memory
```

Lets dive in how this structures are, but previously, lets explain what segments and sections are:

<br>

**Sections & Segments**

In the context of the ELF binary structure, a *section* is a logical grouping of data or code inside an ELF file. It's used during compilation and linking (not execution). Each section has a name and contains a specific type of content. For example;

- .text → compiled machine code (instructions).
- .data → initialized global/static variables.
- .bss → uninitialized global/static variables (zero-filled at runtime).
- .rodata → read-only data (e.g., string literals).
- .symtab, .strtab, .debug_* → symbol/debug info (used by linker/debugger).

Sections help tools like linkers, debuggers, and disassemblers know what's what inside the binary. 

<br>

In the other hand, a *segment* is a chunk of the ELF file that will actually be mapped into memory by the OS when the program is executed. Segments are defined in the Program Header Table, and each one tells the kernel how to map a mount of bytes in memory:

- PT_LOAD → loadable segment (code or data).
- PT_INTERP → tells the kernel which dynamic linker to use.
- PT_DYNAMIC → info used by the dynamic linker (symbols, relocations, etc.).
- PT_TLS, PT_NOTE, etc. → other specialized segments.

Segments are used by the OS loader — this is what ends up in RAM when you run the ELF file.

Having this clear, in simple terms; Sections defines how the ELF binary exists in disk whether Segments defines how the binary ELF gets mapped in memory during execution (process image; the complete in-memory representation of a program during its execution). When execution triggers, the sections (logically grouped amount of data) are grouped into segments and then gets mapped in memory when execution triggers. 

<br>

<div style="text-align:center">
<img src="{{ 'assets/img/C/Pasted image 20250316110636.png' | relative_url }}" text-align="center"/>
</div>

<br>

**ELF Header**

This is the main metadata structure that tells the kernel or linker:

- What kind of ELF file this is,
- How to interpret the rest of the file,
- Where to find the Program Headers (segments) and Section Headers.

| **Field** | **Size** | **Description** |  
| --- | --- | --- |
| e_ident[16] | 16 bytes | Magic number + ELF class (32/64-bit), endianness, version, OS ABI. |
| e_type | 2 bytes | File type (e.g., ET_EXEC, ET_DYN, ET_REL). |
| e_machine | 2 bytes | Target architecture (e.g., x86-64 = 0x3E). |
| e_version | 4 bytes | ELF version (usually 1). | 
| e_entry | 8 bytes | Entry point virtual address (where execution starts). |
| e_phoff | 8 bytes | Offset to the Program Header Table. |
| e_shoff | 8 bytes	| Offset to the Section Header Table. |
| e_flags	| 4 bytes | Architecture-specific flags. |
| e_ehsize | 2 bytes | Size of the ELF header (usually 64 bytes for 64-bit). |
| e_phentsize | 2 bytes | Size of each Program Header entry. |
| e_phnum | 2 bytes | Number of Program Headers. |
| e_shentsize | 2 bytes | Size of each Section Header entry. |
| e_shnum	| 2 bytes | Number of Section Headers. |
| e_shstrndx |	2 bytes |	Index of section header string table (names of sections). |

This are a set of value which tells the kernel some information (Description).

In Linux, we can check *ELF Headers* with the following command:

```sh
$ readelf -h /bin/bash
ELF Header:
Magic:   7f 45 4c 46 02 01 01 00 00 00 00 00 00 00 00 00 
Class:                             ELF64
Data:                              2's complement, little endian
Version:                           1 (current)
OS/ABI:                            UNIX - System V
ABI Version:                       0
Type:                              DYN (Position-Independent Executable file)
Machine:                           Advanced Micro Devices X86-64
Version:                           0x1
Entry point address:               0x31760
Start of program headers:          64 (bytes into file)
Start of section headers:          1300656 (bytes into file)
Flags:                             0x0
Size of this header:               64 (bytes)
Size of program headers:           56 (bytes)
Number of program headers:         14
Size of section headers:           64 (bytes)
Number of section headers:         29
Section header string table index: 28
```

<br>

**Program Header Table**

The *Program Header Table* (PHT) describes *segments*, which are the parts of the file mapped into memory at execution time (code, data, etc.).



Each entry of the table (segment descriptor) looks like follows:

| Field |	Size | Description | 
| --- | --- | --- |
| p_type | 4 bytes | Type of segment (e.g., PT_LOAD, PT_INTERP, PT_DYNAMIC, PT_PHDR) |
| p_flags | 4 bytes | Permissions (Read/Write/Execute) |
| p_offset | 8 bytes | Offset in the file where this segment starts |
| p_vaddr | 8 bytes | Virtual address where segment will be loaded in memory |
| p_paddr | 8 bytes | Physical address (usually ignored) |
| p_filesz | 8 bytes | Size of segment in file |
| p_memsz | 8 bytes	| Size of segment in memory (can be larger than filesz, e.g., for .bss) |
| p_align | 8 bytes | Alignment requirements in memory and file |



The loader uses this table, not the Section Header Table, to know what to load and how.
 We can check *ELF Segments* in Linux:

```sh
$ readelf -l /bin/bash

Elf file type is DYN (Position-Independent Executable file)
Entry point 0x31760
There are 14 program headers, starting at offset 64

Program Headers:
Type           Offset             VirtAddr           PhysAddr
                FileSiz            MemSiz              Flags  Align
PHDR           0x0000000000000040 0x0000000000000040 0x0000000000000040
                0x0000000000000310 0x0000000000000310  R      0x8
INTERP         0x0000000000000394 0x0000000000000394 0x0000000000000394
                0x000000000000001c 0x000000000000001c  R      0x1
    [Requesting program interpreter: /lib64/ld-linux-x86-64.so.2]
LOAD           0x0000000000000000 0x0000000000000000 0x0000000000000000
                0x000000000002ef70 0x000000000002ef70  R      0x1000
LOAD           0x000000000002f000 0x000000000002f000 0x000000000002f000
                0x00000000000c9161 0x00000000000c9161  R E    0x1000
LOAD           0x00000000000f9000 0x00000000000f9000 0x00000000000f9000
                0x0000000000037f6c 0x0000000000037f6c  R      0x1000
LOAD           0x0000000000131a50 0x0000000000131a50 0x0000000000131a50
                0x000000000000bd14 0x0000000000016d70  RW     0x1000
DYNAMIC        0x00000000001345c8 0x00000000001345c8 0x00000000001345c8
                0x0000000000000200 0x0000000000000200  RW     0x8
NOTE           0x0000000000000350 0x0000000000000350 0x0000000000000350
                0x0000000000000020 0x0000000000000020  R      0x8
NOTE           0x0000000000000370 0x0000000000000370 0x0000000000000370
                0x0000000000000024 0x0000000000000024  R      0x4
NOTE           0x0000000000130f4c 0x0000000000130f4c 0x0000000000130f4c
                0x0000000000000020 0x0000000000000020  R      0x4
GNU_PROPERTY   0x0000000000000350 0x0000000000000350 0x0000000000000350
                0x0000000000000020 0x0000000000000020  R      0x8
GNU_EH_FRAME   0x0000000000112f30 0x0000000000112f30 0x0000000000112f30
                0x000000000000470c 0x000000000000470c  R      0x4
GNU_STACK      0x0000000000000000 0x0000000000000000 0x0000000000000000
                0x0000000000000000 0x0000000000000000  RW     0x10
GNU_RELRO      0x0000000000131a50 0x0000000000131a50 0x0000000000131a50
                0x00000000000035b0 0x00000000000035b0  R      0x1

Section to Segment mapping:
Segment Sections...
00     
01     .interp 
02     .note.gnu.property .note.gnu.build-id .interp .gnu.hash .dynsym .dynstr .gnu.version .gnu.version_r .rela.dyn .rela.plt 
03     .init .plt .plt.got .text .fini 
04     .rodata .eh_frame_hdr .eh_frame .note.ABI-tag 
05     .init_array .fini_array .data.rel.ro .dynamic .got .data .bss 
06     .dynamic 
07     .note.gnu.property 
08     .note.gnu.build-id 
09     .note.ABI-tag 
10     .note.gnu.property 
11     .eh_frame_hdr 
12     
13     .init_array .fini_array .data.rel.ro .dynamic .got
```

<br>

**Section Header Table**

The Section Header Table (SHT) describes the sections in the ELF file: .text, .data, .bss, .rodata, .symtab, etc. Sections are used for compilation and linking (debug symbols, relocation info, source code references), but not necessarily for runtime execution.

An Entry of this table looks like follows:

| Field | Size | Description |
| --- | --- | --- |
| sh_name | 4 bytes | Offset into Section Header String Table (name of this section) |
| sh_type	| 4 bytes | Section type (SHT_PROGBITS, SHT_SYMTAB, SHT_NOBITS, etc.) |
| sh_flags | 8 bytes | Attributes (SHF_ALLOC, SHF_EXECINSTR, etc.) |
| sh_addr | 8 bytes | Virtual address in memory (if alloc'd) |
| sh_offset | 8 bytes | Offset in the file |
| sh_size	| 8 bytes	| Size of section |
| sh_link | 4 bytes | Depends on section type (e.g., symbol table links to string table) |
| sh_info | 4 bytes | Extra info (depends on section type) |
| sh_addralign | 8 bytes | Alignment in memory |
| sh_entsize | 8 bytes |	Entry size for tables (e.g., symbol table entries) |

We can check ELF Sections in Linux through the following command:

```sh
$ readelf -S /bin/bash
There are 29 section headers, starting at offset 0x13d8b0:

Section Headers:
  [Nr] Name              Type             Address           Offset
       Size              EntSize          Flags  Link  Info  Align
  [ 0]                   NULL             0000000000000000  00000000
       0000000000000000  0000000000000000           0     0     0
  [ 1] .note.gnu.pr[...] NOTE             0000000000000350  00000350
       0000000000000020  0000000000000000   A       0     0     8
  [ 2] .note.gnu.bu[...] NOTE             0000000000000370  00000370
       0000000000000024  0000000000000000   A       0     0     4
  [ 3] .interp           PROGBITS         0000000000000394  00000394
       000000000000001c  0000000000000000   A       0     0     1
  [ 4] .gnu.hash         GNU_HASH         00000000000003b0  000003b0
       0000000000004cd8  0000000000000000   A       5     0     8
  [ 5] .dynsym           DYNSYM           0000000000005088  00005088
       000000000000f180  0000000000000018   A       6     1     8
  [ 6] .dynstr           STRTAB           0000000000014208  00014208
       000000000000a181  0000000000000000   A       0     0     1
  [ 7] .gnu.version      VERSYM           000000000001e38a  0001e38a
       0000000000001420  0000000000000002   A       5     0     2
  [ 8] .gnu.version_r    VERNEED          000000000001f7b0  0001f7b0
       0000000000000100  0000000000000000   A       6     2     8
  [ 9] .rela.dyn         RELA             000000000001f8b0  0001f8b0
       000000000000e1f0  0000000000000018   A       5     0     8
  [10] .rela.plt         RELA             000000000002daa0  0002daa0
       00000000000014d0  0000000000000018  AI       5    24     8
  [11] .init             PROGBITS         000000000002f000  0002f000
       0000000000000017  0000000000000000  AX       0     0     4
  [12] .plt              PROGBITS         000000000002f020  0002f020
       0000000000000df0  0000000000000010  AX       0     0     16
  [13] .plt.got          PROGBITS         000000000002fe10  0002fe10
       0000000000000018  0000000000000008  AX       0     0     8
  [14] .text             PROGBITS         000000000002fe40  0002fe40
       00000000000c8315  0000000000000000  AX       0     0     64
  [15] .fini             PROGBITS         00000000000f8158  000f8158
       0000000000000009  0000000000000000  AX       0     0     4
  [16] .rodata           PROGBITS         00000000000f9000  000f9000
       0000000000019f30  0000000000000000   A       0     0     32
  [17] .eh_frame_hdr     PROGBITS         0000000000112f30  00112f30
       000000000000470c  0000000000000000   A       0     0     4
  [18] .eh_frame         PROGBITS         0000000000117640  00117640
       000000000001990c  0000000000000000   A       0     0     8
  [19] .note.ABI-tag     NOTE             0000000000130f4c  00130f4c
       0000000000000020  0000000000000000   A       0     0     4
  [20] .init_array       INIT_ARRAY       0000000000131a50  00131a50
       0000000000000008  0000000000000008  WA       0     0     8
  [21] .fini_array       FINI_ARRAY       0000000000131a58  00131a58
       0000000000000008  0000000000000008  WA       0     0     8
  [22] .data.rel.ro      PROGBITS         0000000000131a60  00131a60
       0000000000002b68  0000000000000000  WA       0     0     32
  [23] .dynamic          DYNAMIC          00000000001345c8  001345c8
       0000000000000200  0000000000000010  WA       6     0     8
  [24] .got              PROGBITS         00000000001347c8  001347c8
       0000000000000820  0000000000000008  WA       0     0     8
  [25] .data             PROGBITS         0000000000135000  00135000
       0000000000008764  0000000000000000  WA       0     0     32
  [26] .bss              NOBITS           000000000013d780  0013d764
       000000000000b040  0000000000000000  WA       0     0     32
  [27] .gnu_debuglink    PROGBITS         0000000000000000  0013d764
       0000000000000034  0000000000000000           0     0     4
  [28] .shstrtab         STRTAB           0000000000000000  0013d798
       0000000000000114  0000000000000000           0     0     1
Key to Flags:
  W (write), A (alloc), X (execute), M (merge), S (strings), I (info),
  L (link order), O (extra OS processing required), G (group), T (TLS),
  C (compressed), x (unknown), o (OS specific), E (exclude),
  D (mbind), l (large), p (processor specific)
```

<br>

#### Executing an ELF file. Memory mapping and permissions.

We speak about *execute* an ELF file when the Linux kernel loads the ELF file into memory, sets up the process environment (memory layout, stack, etc.), and starts executing it from its defined entry point (*e_entry* in the ELF header). Lets go deeper into this process, supose we execute "./program" in our terminal:

- First, the terminal executes *fork()* (spawn a child process) and then *execve()* syscall is invoked. This efectively replace the current child process image with the ELF binary's recently executed.
- The kernel loads the ELF binary. Then; the kernel reads the ELF Headers to verify if its a valid ELF and reads the Program Header Table which describes segments for load in memory.
- The kernel knows the segments to load (this is map to, put the contents into, a specific virtual memory region), based on the PT_LOAD entries type in the Program Header (those entries that must be loaded) and its permissions.

- Sets up virtual memory layout settings specific permissions (explained later): 
     - The kernel maps all segments at the right virtual addresses, 
     - It prepares the process memory: code, data, heap, stack 
     - Environment variables and arguments (argv, envp) are pushed onto the stack.

- When a binary is loaded, the kernel uses *mmap()* to map the sections of the ELF binary into virtual memory. Each mapping has specific flags that describes a specific permission managed by the Memory Management Unit;

     - **r** - Can read.
     - **w** - Can write.
     - **x** - Can execute (run as code).

     Let's go region by region describing its permissions.

     - **.text** (Code segment): r-x (read and execute permissions). This segment contains the code to be executed, so you need to read it and execute it.
     - **.data** (Global Variables): rw- (read and write permissions). This segment contains the global variables and other data needed by the main code so we need to acces it (read) and modifying permissions (write). 
     - **.rodata** (Read only data, like constants): r-- Read-only permissions.
     - **.bss** (Uninitialized Global Variables): rw-.
     - **GOT/PTL** (Address to functions): rw-/r-- (In the majority of cases read-only).
     - **Heap/Stack** (Manage dynamic memory, local vars or call frames): rw-. 

- Jumps to the entry point
     - The *e_entry* field in the ELF Header gives the virtual address of the entry point.
     - Usually, this is _start (not main()), which sets up things before calling main() (in dynamically linked ELF, this is managed by the dynamic loader first).

<br>

### From C Code to ELF Binary.

In the process of go through a C writed code to a complete runnabe ELF binary we difference 4 steps:

- **Preprocessing**.
- **Compilation**.
- **Assembly**
- **Linking**

<br>

#### Preprocessing.

We can only force the *preprocessing* stage of a C file with the following command in Linux:

```sh
gcc -E main.c -o main.i
```

This tells the compiler to only run the preprocesor stage, done by the C PreProcessor, **cpp**. This will transform the C source into a pure C file, where all preprocessor directives are evaluated and expanded. Lets see some examples:

- Processing *preprocessor directives*.

     For example, if we have the following code;

     ```c
     #include <stdio.h>
     int main() {
          printf("hello\n");
          return 0;
     }
     ```

     Then, the preprocessor will copy-paste the full contents of stdio.h into the source code. This will include all the stdio.h functions declarations (for example pritnf()), it is worth to mention that the declaration and the definition of a function are two differents things. The declaration of a function tells the compiler how is the function (signature, return value, etc) and the definition, located in another file in most of the cases, tells the compiler the actual implementation of the function; how the function works.

     In the case of printf(), its declaration is found in "<stdio.h>", which is included in the source code using the #include directive. This allows the compiler to recognize that printf() exists and can be used in the program. However, the actual definition of printf() is not in <stdio.h> but in the shared library libc.so.

     At runtime, the dynamic linker (ld.so) resolves the printf() symbol and loads its actual definition from libc.so, allowing the function to execute as intended.



- Macro definitions and Expansions.

     If we have the following macros on our code:

     ```c
     #define PI 3.14
     #define SQUARE(x) ((x)*(x))
     int main() {
          float r = 5;
          float a = PI * SQUARE(r);
     }
     ```

     The preprocessor replaces every usage with the defined expression expanding the macros, the resulting code would be:

     ```c
     int main() {
          float r = 5;
          float a = 3.14 * ((r)*(r));
     }
     ```

This are only a few examples of the cpp performance, once all includes, macros, and conditions are expanded, you get a giant .i file — pure C code.


<br>

#### Compilation.

When the preprocessor has finished and return a .i file of pure C, it starts the *compilation* stage, where the .i pur C file gets converted to Assembly code by the *compiler*. This phase waits a .i file and returns a .s file written in assembly. We can perform only the compulation stage through the following command in Linux:

```c
gcc -S main.i -o main.s
```
<br>

#### Assembly(as)

In this stage, the assembler transform the assembly .s file to machine code, returning an .o file. We can go through this step with one of the following commands:

```sh
gcc -c main.s -o main.o #Through the .s assembly file.

gcc -c main.c -o main.o #Through the .c source file.
```

The result file is a binary file taht contains several sections:

- Machine code in .text section
- Global/static variable definitions (.data, .bss)
- Symbol table (.symtab)
- Relocation entries (placeholders for unresolved addresses)

<br>

#### Linking.

In Unix-like systems, linking is the process of combining multiple object files and libraries into a single executable (or library). in this process different parts of a program (functions, variables) gets connected so they all work together in a single executable file. *It's the stage that comes after compilation*, and it's critical to produce a runnable binary.:

- Combines object files
- Resolves symbols
- Fixes memory addresses (relocations)
- And builds the final executable ELF binary

When you compile C code, each .c file becomes an object file (.o), but it doesn't yet know where functions really live (e.g., printf), where variables are or how to jump/call to other files' code. Linking resolves those gaps.

Let's break this up through an example, lets suppose we have two files:

```c
// main.c
#include <stdio.h>
void hello();  // declaration
int main() {
    hello();
    return 0;
}
```

```c
// hello.c
#include <stdio.h>
void hello() {
    printf("Hello from hello()\n");
}
```

If we compile this two codes we would obtain two binary files that have all the necesary to run:

```sh
gcc -c main.c   → main.o
gcc -c hello.c  → hello.o
```

Main code calls hello() function which is defined in other file. The .o file is a binary file wich have all the necesary parts to run, but this parts lack of organization to function in a coherent way.

When we link those files:

```sh
gcc main.o hello.o -o myprogram
```

The linker, (ld), invoked by gcc:

- Finds that main.o uses a symbol hello
- Finds the definition of hello in hello.o
- Patches the call instruction in main.o so it jumps to the address in hello.o

The result is myprogram, an ELF binary with proper machine code, fully connected function calls, all symbols resolved. 

<br>

##### Static Linking vs Dynamic Linking. Shared Libraries.

There are two mains forms of linking; static linking and dynamic linking.

<br>

**Static Linking**

The linker copies all needed code from libraries into your final binary. So your binary becomes self-contained and it doesn't depend on external .so files at runtime. It uses .a files (archive of .o files) during linking.

```sh
gcc main.c -static -o myprogram
```

<br>

**Dynamic Linking**

The linker does not copy the code from the libraries, but instead adds references (symbols) to the shared libraries and marks those symbols to be resolved at runtime. It uses .so files (shared objects).

A shared library (.so) in Linux is a compiled binary that contains functions and symbols which other programs can use at runtime. For example, libc.so.6 defines common functions like write(), read(), malloc(), etc. A symbol is simply a name that represents a function, variable, or data object an resolves during the compilation and linking process.

<br>

**Dynamic Linking Symbol Resolution flow**

Let's see deeper this process, there are a few importants elements which take a crucial parts in dynamic linking:

- PLT (Procedure Linkage Table): Code stubs used to call external functions.
- GOT (Global Offset Table): Holds actual addresses of external functions (resolved at runtime).
- .dynsym: Symbol table for dynamic linking.
- .dynstr: String table (names of symbols).
- .rel.plt / .rela.plt: Relocation entries for dynamic symbols.

Now, lets suppose we have a code that calls printf(). 

```c
#include <stdio.h>
int main() {

     printf("Hello, World!\n");
     return 0;
}
```

If we, at first, compile this program 
Then, the compiler turns this into a call in the PLT 

```
call printf@PLT
```

The PLT entry does not yet know where real main → PLT → GOT → real printf()printf() is. At runtime, the dynamic linker patches the GOT with the actual address of printf() from libc.so. Now, future calls to printf() jump directly to libc's real function via GOT.

So, the flow that the system follow in order to go to printf() is:

```text
main → PLT → GOT → real printf()
```

<br>

## Function Hooking.

### What is function hooking.

Function Hooking is a technique used to intercept and modify the behavior of functions at runtime. It allows a program to redirect function calls, inspect parameters, change return values, or execute additional code before or after a function runs. Hooking can be done in user space (e.g., hijacking dynamic libraries) or kernel space (e.g., modifying system call tables).
There are several methods to hook a function:

- **Inline Hooking (Code Overwriting)**: Modifies the first bytes of a function (machine code) to redirect execution elsewhere. 
- **Import Address Table (IAT) Hooking**: Modify the function pointer in order to replace it with a pointer to your custom function.
- **API/Shared Library Hooking**: Load custom function in application execution instead of default one.
- **System Call Hooking**: Modifying the System Call Table in the kernel.

In this case, we gonna see an example of *Shared Library hooking*, this kind tries to load a custom function instead of the defaul one through LD_PRELOAD method. 

Let's go back over the flow the program follows in order to call a function; main → PLT → GOT → function(). The goal of function hooking is to intercept and replace this flow. This can be done in several ways.

<br>

### Types of API Hooking.

In this module we are covering two types of function hooking.

A. LD_PRELOAD tecnique.
B. Overwriting GOT Entries (Static and Dynamic case)

<br>

### LD_PRELOAD Technique.

#### LD_PRELOAD explanation.

LD_PRELOAD is an enviroment variable used in Unix-like systems that instructs the linker (ld) to load a shared library before any other. This is very powerful since it allows you to override functions in existing programs without modifying their source code or binary. 

<br>

#### Basic internals. Hooking with LD_PRELOAD.

We know that the majority of C programs compiled in Linux systems used the dynamic linking to use shared libraries. This means that, when a program calls a function, it dosen't have the full implementation of the function but a symbol (a reference to the function in a shared library) which is resolved at runtime by the dynamic linker. 

We can intercep this process with LD_PRELOAD:

```sh
LD_PRELOAD=/path/to/mylib.so ./target_program
```

This command perform that the dynamic linker does this before loading libc or other libraries:

- Loads mylib.so

- Resolves symbols (functions like malloc, open, etc.) from your library first

- If a symbol is already resolved from your library, it won't continue to resolve the one in libc

So, your custom function is gonna be executed before any other function.

<br>

#### Building a Shared Library.

Our pretension is to write a custom function that replaces deafult *write()* system call. Before that, we will understand how write works and what 'replace' this functions means.

The *write()* function in C is a wrapper around a system call used to write data to a file descriptor and is defined in the *unistd.h* header. This means that when a program calls 'write()' function, it actually calls a function provided by *libc* (user-space function) which perform a system call to the kernel (sys_write) to execute the actual write operation.

The sintax of this function is as follows:

```c
ssize_t write(int fd, const void *buf, size_t count);
```

First, we have to say that *ssize_t* is an acronym of *signed size_t*, *size_t* in the other hand is an unsigned interger data type (named like that for cleaner code purposes) that can holds the maximum the maximum size of a theoretically possible array or an object. The *ssize_t* is used mainly for functions that return the number of bytes processed, with -1 indicating an error.

Said this, write() functions **return the number of bytes written to the file descriptor and -1 if there been any kind of error in the write process**.

It accepts three arguments:

- **int fd**: The file descriptor where data will be written.
- **const void \*buf**: A pointer to the buffer which contains the data to write.
- **size_t count**: The number of data to write.

<br>

##### Write functionality.

Now that we know how to call *write()*, then we can go deeper in how this functions works internally.

1. Checks file descriptor.

At first, the kernel checks if *fd* is valid. When you call write, at first the system performs the following steps in the kernel:

- System check if *fd* exists in the FDT of the process, if it not exists in the FDT, write returns an error of bad file descriptor (EBADF).
- If *fd* exists in FDT, then check if the file descriptor was opened to write checking the flag on the MODE field in the FDT, if it does not allow to write, the function returns EBADF. 

2. Copies data from *buf* to kernel space.

Initially, when you have to write a string to a file descriptor, the string exists in the *virtual address user-space* inside a buffer, this means that is a pointer to memory managed by the program. Since, for security reasons, user-space programs cannot overwrite memory kernel-space and file descriptors are system resources (handled by the kernel), the string have to be copied to an internal buffer managed by the kernel (sys_write()) in order to be copied to the file descriptor in next steps.

3. Perform actual write operation.

Once kernel have the data to be written on an internal buffer, it writes the contents on the desired file descriptor

4. Kernel update file offset,

The kernel updates the file offset by adding the number of bytes written (size), this means the next write operation will start from the new offset.

5. Returns number of bytes written

Sys_write detect how many bytes has been written to the fd and returns this information to write which also return this information.

<br>

##### Custom Write function. Dlsym call.

Know, consider the following code:

```c
#include <stdio.h>
#include <unistd.h>
#include <dlfcn.h>
#include <string.h>

ssize_t write(int fildes, const void *buf, size_t nbytes)
{
  ssize_t (*new_write)(int fildes, const void *buf, size_t nbytes);

  ssize_t result;

  new_write = dlsym(RTLD_NEXT, "write");

  if (strcmp(buf, "Hello, World!\n") == 0)
  {
      result = new_write(fildes, "Goodbye, cruel world!\n", 21);
  }

  else
  {
    result = new_write(fildes, buf, nbytes);
  }

  return result;
}
```

Lets break this up:

Now that what is function hooking and how the function we want to hook works

The code imports the following headers:

```c
#include <stdio.h> 
#include <unistd.h>
#include <dlfcn.h>
#include <string.h>
```

The first and the last one we already know it, is the standard inputo/output library and the tring manipulation functions header. But we will deep in the other two ones:

- **unistd.h**; Is the name of the libraru that the header file that provides access to the POSIX operating system API, Unix-systems system calls.
- **dlfcn.h**; Header file that provides functions to dynamically load shared libraries (.so files), retrieve function addresses, and unload them at runtime. This is useful to perform *Function hooking* (intercepting system calls).

Then, it accepts the same arguments:

- **int fildes**: The file descriptor to write. 
- **const void *buf**: The buffer with the contents to write
- **size_t**: Numbers of bytes to write.


Now,we declare a *function pointer* (this is a type of pointer that can hold the address of a function) with the same signature of write. The logic is to give this pointer to hold the real write function with the following code:

```c
ssize_t (*new_write)(int fildes, const void *buf, size_t nbytes);

new_write = dlsym(RTLD_NEXT, "write");
```

The second line uses **dlsym** call, which in brief terms, finds the next occurrence of write in the dynamic symbol resolution order allowing us to use the real *write()* function. Then we assign the dlsym() call to our function pointer.

Till this point, our function acts exacting like the *write()* function, accepts the same arguments and with *dlsym()* call it delegates to actual *write()* function. Now we are going to write the lines that differs from the normal write() functionality. If the string "Hello, World!" is detected in the buffer passed to the function, instead, the function writes "Goodbye, cruel world", else it just calls normal *write()* function.

```c
if (strcmp(buf, "Hello, World!") == 0)
{
     result = new_write(fildes, "Goodbye, cruel world!", 21);
}

else
{
     result = new_write(fildes, buf, nbytes);
}
```

<br>

##### Compiling custom code as a Shared Library.

We can compile this as a Shared Library with the following command:

```sh
gcc -shared -fPIC -o hook.so hook.c -ldl
```

Which creates a shared object file (hook.so), which can be used with **LD_PRELOAD** to override functions in dynamically linked programs.

- **gcc**: The GNU Compiler Collection (GCC), used to compile C code.

- **shared**: Generates a shared library (.so file instead of an executable). Shared libraries are dynamically loaded into memory when needed.

- **fPIC (Position-Independent Code)**: Ensures the generated machine code can be loaded at any memory address. Required for shared libraries to work correctly with LD_PRELOAD.

- **o hook.so**: Specifies the output file name (hook.so).

- **hook.c**: The source C file to compile.

- **ldl (Link with libdl)**: Links against the libdl library, which provides functions like dlsym(), dlopen(), and dlclose(). Needed for dynamic symbol resolution (e.g., dlsym(RTLD_NEXT, "open") to get the original function).

The result will be a *hook.so* shared library. The next step is to compile a normal C program indicating to the linker to use hour custom shared library with first prority:

```c
#include <stdio.h>
#include <unistd.h>
int main() {
        write(1,"Hello, World!\n",14);
        return 0;
}
```

Then we compile it and execute it:

```sh
gcc -o test test.c; ./test
```

We should get "Hello, World!". Now, we make use of LD_PRELOAD to intercept write function:

```sh
export LD_PRELOAD=./hook.so ./test
```

We should get: "Goodbye, cruel world!".

<br>

### Overwriting GOT entries technique.

Function hooking via GOT overwriting involves modifying the GOT entry of a target function to point to our own malicious or monitoring function instead of the original function.

<br>

#### Fundamentals.

##### Overview.

We remember that, if we have the following code:

```c
#include <stdio.h>

int main() {
     printf("Hello, World!\n")
     return 0;
}
```

And we compile and execute it, the processor does not know were the function *printf()* is. At the preprocessing stage, the preprocessor include the *stdio.h* lib, where the *printf()* declaration relies, within the source code of our file. This will indicate to the processor how the function is (signature, return value, etc) but no his actual implementation or where to find it, this is other process called **dynamic symbol resolution** and happens at runtime, it consists in resolve the symbol (a reference to a external function or piece of code) with the actual memory address where the implementation of the referenced data is.

This happens in several steps, mainly involves two structures: 

- **PLT (Procedure Linkage Table)**: A stub used to call external functions.

- **GOT (Global Offset Table)**: Stores addresses of dynamically linked functions resolved at run time. Initally, a GOT entry for a function in the first call points to a resolver that loads the actual address of the function.

<br>

##### Dynamic Symbol Resolution step by step.

- **Function call in main()**

     First, as we said before; The compiled binary does not contain a direct call to printf(). Instead, the compiler generates a call to the PLT stub for printf.

     Example disassembly of main():

     ```assembly
     call   printf@plt
     ```

     This means the function call is indirect, and execution jumps to the PLT entry for printf.

     <br>

- **Execution enters the PLT. First call.**
     
     Second, the PLT (Procedure Linkage Table) is a section of the ELF binary (which exists in the .plt section) generated during the linking process which contains *stub functions* (a small piece of code that acts as an intermediary between the program and an actual dynamically linked function) for dynamically linked functions. It acts as an intermediary before the function address is resolved.

     Each function defined in the source code has an entry in the PLT. For example, the PLT entry for *printf()* typically looks like this:

     ```assembly
     printf@plt:
          jmp    QWORD PTR [GOT+offset]   ; Jump to the GOT entry for printf
          push   index                    ; Push relocation index onto stack
          jmp    plt[0]                    ; Jump to PLT0 (common resolver)
     ```

     If the functions has been already resolved, then execution jumps to the function address, if not (first call) execution continues.     

     <br>

- **GOT lookup**

     As we say before, the first time *printf()* is called, its GOT entry does not contain the real function address but instead points back to the PLT.

     The GOT entry for printf before resolution points to PLT:

     ```assembly
     GOT[printf] → PLT stub (resolver)
     ```

     Since the function is not resolved yet, execution jumps to PLT0, which is a special entry in the PLT that calls the dynamic linker. Is convenient to say that if we manage to force the GOT to point to a custom address, we will actually execute our custom function instead of the intended one, performing efectively function hooking. We will see this later.

     <br>

- **Dynamic Linker**

     Since this is the first call, the PLT calls the GOT and the GOT points to the *resolver stub* in PLT, which calls the dynamic linker (ld-linker.so), which is responsible for resolving symbols at runtime. The PLT0 entry performs:

     ```assembly
     call _dl_runtime_resolve
     ```

     *_dl_runtime_resolve()* is a function part of the dynamic loader. It looks up in the *dynamic symbol table*. It finds the real address of *printf()* in libc (/lib/x86_64-linux-gnu/libc.so.6).

     <br>

- **Updating GOT Entry**

     Once the real function address is found, the dynamic linker overwrites the GOT entry for printf with the real address.

     The GOT entry for printf after resolution:

     ```assembly
     GOT[printf] → 0x7ffff7e4e0b0 (actual address of printf)
     ```

     In following main's pritnf() calls; The next time printf is called; the PLT stub jumps to the GOT entry. The GOT entry now holds the real function address. Execution goes directly to *printf()* without calling the dynamic linker again.

     <br>

In summary, in the dynamic symbol resolution process go over the followings paths in order to locate the function implementation memory address storage location:

- First call:

```assembly
main() → function@plt → GOT (points back to PLT) → PLT0 → Dynamic Linker → function location
```

<br>

- Following calls:

```assembly
main() → function@plt → GOT (now contains real printf address) → function location.
```

<br>

##### Visualizating PLT and GOT in real binary. objdump and readelf.

In Linux, we can check for the PLT and GOT of a binary with *objdump* and *readelf* commands respectively.

Suppose we have the following code in C:

```c
#include <stdio.h>

int main() {

     printf("Hello, World!\n");
     return 0;
}
```

Then, we compile our code (gcc test.c -o test) and execute the following command:

```sh
objdump -d test | grep "<printf@plt>"
```

We can check that the output is empty, then we instead do the following command:

```sh
$ objdump -d test | grep "@plt"        

0000000000001020 <puts@plt-0x10>:
0000000000001030 <puts@plt>:
0000000000001040 <__cxa_finalize@plt>:
    1112:       e8 29 ff ff ff          call   1040 <__cxa_finalize@plt>
    1147:       e8 e4 fe ff ff          call   1030 <puts@plt>
```

This means that, when we call printf() in our code, the linker instead of calling *printf()* calls *puts()*. The output shows the call.

<br>

Now, we can also confirm this with:

```sh
 readelf -r test | grep "puts"  

000000004000  000300000007 R_X86_64_JUMP_SLO 0000000000000000 puts@GLIBC_2.2.5 + 0
```

This command allow us to visualize the calls to the PLT and the GOT.

<br>

##### Memory protection.

In the other hand, the GOT is a part of the ELF file which get mapped on memory when its executed as we see in the Segments and Sections part. The image of the ELF binary gets mapped over several virtual memory regions. 

Our objective is to chagen an entry of the GOT, which is mapped in a memory region without write permissions as we see in previous sections. So first, we have to change memory permissions and then proceed to change the contents of the memory address of the GOT that contains the legacy function. This can be achieve with **mprotect()** syscall in unix systems.

<br>

###### Memory pagination.

Lets explain what pagination is before dive into this function. 

Pagin is how the virtual memory system breaks memory into fixed-size chunks called *pages*. We can think about the memory like a big excel book, then the pages would be fixed height rows of usually 4096 bytes. This page are the most basic units in memory management, each with his own permissions, that are used to translate virtual memory addresses into physical ones. Segments from a ELF bianry are later mapped into pages in virtual memory (Sections --> Segments --> Pages).

Thus, if we want to change permissions over a GOT entry, we have to use *mprotect* to change the permissions of the page/s in which this entry is stored.

<br>

###### Mprotect() syscall.

If we access the *man* documentation of *mprotect* function we can see the description: "mprotect() changes the access protections for the calling process's memory pages containing any part of the address range in the interval [addr, addr+len-1].  addr must be aligned to a page boundary". So, is a function we can use to change the permissions of a designated memory page.

It has the following signature:

```c
int mprotect(void *addr, size_t len, int prot);
```

Let's describe the arguments:

- **void \*addr**: This argument is the starting virtual adress of the memory region you want to change protections for. As we now, about pagination, this argument must containt **the initial address of the page that contains the segment you want to modify**. (If not EINVAL error is trigged). Let's se an example, if our region memory target is 0x601018, then the code to implement is:

     ```c
     long pagesize = sysconf(_SC_PAGESIZE);  // usually 4096
     void *page_start = (void *)((uintptr_t)target_addr & ~(pagesize - 1));
     ```

     This lines first get the page syze by performing *sysconf()* syscall (get configuration information at runtime; unistd.h). Then, the second line gets the starts address of the page that contains a target_address.

     1. First we cast *target_addr* to a *uintptr_t* to make it available to hold safely a memory address.
     2. With NOT operation "~(pagesize - 1), we flip all the bites and clear to 0 those lower bites getting a mask of a page chunk in hexadecimal's memory address terms.
     3. With AND operation between the target address and our custom mask ((uintptr_t)target_addr & ~(pagesize - 1)) we fit our target address to the mask getting the starting address of the chunk in which our target address lays.


- **size_t len**: This is the lenth of the memory region to be changed, this doesn't have to be page-aligned, but internally it's rounded up to page boundaries. So for example, we can get the size of a page and get a multiple of it:

     ```c
     mprotect(page_start, pagesize * 2, PROT_READ | PROT_WRITE);
     ```


- **int prot**: This is the new memory protection flag, we use the following bitwise OR constants:

     PROT_READ - Pages can be read
     PROT_WRITE - Pages can be written
     PROT_EXEC	- Pages can be executed
     PROT_NONE	- No access (useful for guard pages)

So this function, in summary, change the access permissions (prot) of a region of memory of a dessignated lenght (len), starting by an address (addr). The properly way to use this function is to fit the initial address and the length into a memory page. We will se an example later. 

<br>

##### PIE Binaries.

PIE is an acronym for Position Indepent Executable. Is a compiler and linker feature that makes the entire binary position independent, so the binary can be loaded in random address at runtime (ASLR, Address Space Layout Randomization) in order to prevent memory corruption attacks (BOF, ROP, etc).

This affects in the location of the GOT entry of the function we want to hook. A PIE compiler's file will randomize this address at runtime making it untargetable and our code (as much as we try to locate this entry with readelf -r) will fail with segmentation code.

We can detect if a binary is being compiled by a compiler which have the PIE feature using readelf:

```sh
$ readelf -h ./test             
ELF Header:
<...>
  Type:                              DYN (Position-Independent Executable file)
<...>
```

This means, again, that this file support ASLR. A non PIE compiler's file would have 'EXEC' instead.

We can bypass this hardening measure by adding in the compiling command the *-no-pie* flag to gcc binary:

```sh
$ gcc -no-pie -o test test.c                 
$ readelf -h ./test         
<...>
  Type:                              EXEC (Executable file)
```

Now we can reliably locate GOT write's function entry. This will be explained later.

<br>

#### GOT entry overwriting.

##### Overview the concept.

Function hooking via GOT overwriting involves modifying the GOT entry of a target function to point to our own malicious or monitoring function instead of the original. Let's see a practical example.

<br>

##### Steps overview.

1. Find GOT entry of write() in the target binary.

2. Make the GOT entry writable (using mprotect()).

3. Overwrite the GOT entry with the address of our custom function.

4. Verify the hook is working.

<br>

##### Process step by step.

Lets suppose we have the following code in C:

```c
#include <stdio.h>
#include <unistd.h>

int main() {
    write(1, "Hello, World!\n", 16);
    return 0;
}
```

Compiled in *test* binary (remember to use -no-pie flag):

```sh
gcc -no-pie -o test test.c
```

And we want to use other function instead write.

<br>

###### Locating write() in the GOT. Readelf -r & objdump -R.

For this task, we will use *readelf* command with *-r* flag. This will displays relocations in the ELF file. Relocations are modifications that must be applied to an executable or shared object at load time or link time for several reasons, in our case because shared libraries don’t have fixed addresses and need relocations at runtime.

```sh
$ readelf -r test | grep write -C3

Relocation section '.rela.plt' at offset 0x610 contains 1 entry:
  Offset          Info           Type           Sym. Value    Sym. Name + Addend
000000004000  000300000007 R_X86_64_JUMP_SLO 0000000000000000 write@GLIBC_2.2.5 + 0
```
We could achieve the same thing by:

```sh
$ objdump -R ./test | grep write
0000000000004000 R_X86_64_JUMP_SLOT  write@GLIBC_2.2.5
```

This means that, in our case, the GOT entry that points to real *write()* address is at offset *0x004000*.

<br>

##### Writing exploit. Static case.
###### Overviewing the code.

Our task is to modify our test.c code in order to add C code that modifies the GOT entry address to aim to our custom function. 

Lets overview the following code:

```c
#define _GNU_SOURCE
#include <stdio.h>
#include <stdlib.h>
#include <stdint.h>
#include <unistd.h>
#include <sys/mman.h> //Memory managment declarations.

void hook_function()
{
    printf("Hooked write!\n");
}

void overwrite_got_entry(uintptr_t *got_entry, void *new_function)
{
    // Change memory protection to allow writing to the GOT
    mprotect((void *)((uintptr_t)got_entry & ~0xFFF), 4096, PROT_READ | PROT_WRITE);
    
    // Overwrite GOT entry
    *got_entry = (uintptr_t)new_function;
}

int main() {
    uintptr_t *got_write = (uintptr_t *)0x404008; // Replace with actual GOT entry address
    
    // Overwrite the GOT entry for write()
    overwrite_got_entry(got_write, hook_function);
    
    // Call write() (now redirected)
    write(1, "Hello, World!\n", 15);
    
    return 0;
}
```

- First, we define the function we wanna use to hook. When the hook GOT works, this function will be executed instead of the real *write()*.

```c
void hook_function()
{
    write(1, "Hooked write!\n", 14);
}
```

- Next, we define the function wich will overwrite the GOT entry.

```c
void overwrite_got_entry(uintptr_t *got_entry, void *new_function){

     mprotect((void *)((uintptr_t)got_entry & ~0xFFF), 4096, PROT_READ | PROT_WRITE);
     *got_entry = (uintptr_t)new_function;
}
```

- Then, we have the main code, which calls the swicthing GOT entry's function and then calls write.

```c
int main() {
    uintptr_t *got_write = (uintptr_t *)<WRITE GOT ADDRESS>; // Replace with actual GOT entry address
    
    // Overwrite the GOT entry for write()
    overwrite_got_entry(got_write, hook_function);
    
    // Call write() (now redirected)
    write(1, "Hello, World!\n", 15);
    
    return 0;
}
```

###### Triggering exploit.

If we compile this code with a random value of \<WRITE GOT ADDRESS\> and then get the real address like we see before:

```sh
$ gcc -no-pie -o test test.c

$ readelf -r ./test | grep write
000000404008  000300000007 R_X86_64_JUMP_SLO 0000000000000000 write@GLIBC_2.2.5 + 0
```

This means our GOT entry is at offset 0x404008, then we replace this number in the code above and recompile and execute:

```sh
$ gcc -no-pie -o test test.c;./test
Hooked write!
```

Instad of write "Hello World", we manage to hook the write function and replace it with printf "Hooked write!"

<br>


##### Writing exploit. Dynamic case.

Before, we try to locate the GOT in an efficient manner and the code is in certain way static. Now our pretensión is to write a code that dynamically parse the ELF Headers and the relocation table. 

Let's examine the following code:

```c
#define _GNU_SOURCE
#include <stdio.h>
#include <stdlib.h>
#include <stdint.h>
#include <unistd.h>
#include <string.h>
#include <link.h>
#include <sys/mman.h>
#include <fcntl.h>
#include <elf.h>

void hook_function()
{
    printf("Goodbye cruel world!");
}

uintptr_t find_write_got_entry() {
    int fd = open("/proc/self/exe", O_RDONLY);

    Elf64_Ehdr ehdr;
    pread(fd, &ehdr, sizeof(ehdr), 0);

    Elf64_Shdr shdr;
    Elf64_Shdr sh_str;
    pread(fd, &sh_str, sizeof(sh_str), ehdr.e_shoff + ehdr.e_shstrndx * sizeof(shdr));

    char *shstrtab = malloc(sh_str.sh_size);
    pread(fd, shstrtab, sh_str.sh_size, sh_str.sh_offset);

    Elf64_Shdr relaplt = {0};
    Elf64_Shdr dynsym = {0};
    Elf64_Shdr strtab = {0};

    for (int i = 0; i < ehdr.e_shnum; i++) {
        pread(fd, &shdr, sizeof(shdr), ehdr.e_shoff + i * sizeof(shdr));
        const char *name = shstrtab + shdr.sh_name;

        if (strcmp(name, ".rela.plt") == 0 || strcmp(name, ".rela.plt.sec") == 0) {
            relaplt = shdr;
        } else if (strcmp(name, ".dynsym") == 0) {
            dynsym = shdr;
        } else if (strcmp(name, ".dynstr") == 0) {
            strtab = shdr;
        }
    }


    char *strtab_data = malloc(strtab.sh_size);
    pread(fd, strtab_data, strtab.sh_size, strtab.sh_offset);

    Elf64_Rela rela;
    Elf64_Sym sym;

    for (size_t off = 0; off < relaplt.sh_size; off += sizeof(Elf64_Rela)) {
        pread(fd, &rela, sizeof(rela), relaplt.sh_offset + off);
        size_t sym_idx = ELF64_R_SYM(rela.r_info);
        pread(fd, &sym, sizeof(sym), dynsym.sh_offset + sym_idx * sizeof(Elf64_Sym));

        const char *name = strtab_data + sym.st_name;
        if (strcmp(name, "write") == 0) {
            free(shstrtab);
            free(strtab_data);
            close(fd);
            return rela.r_offset;
        }
    }
}

void overwrite_got_entry(uintptr_t *got_entry, void *new_function)
{
    long pagesize = sysconf(_SC_PAGESIZE);
    void *page_start = (void *)((uintptr_t)got_entry & ~(pagesize - 1));
    mprotect(page_start, pagesize, PROT_READ | PROT_WRITE);

    *got_entry = (uintptr_t)new_function;
}

int main() {
    uintptr_t got_addr = find_write_got_entry();
    printf("[+] GOT entry for write(): %p\n", (void *)got_addr);

    overwrite_got_entry((uintptr_t *)got_addr, hook_function);

    // This call will be redirected
    write(1, "Hello, World!\n", 15);

    return 0;
}
```

This code add a code chunk that aims to locate the GOT entry of the write function at runtime. 

This code is pretty looklike the static version with the difference of the adding of the *find_write_got_entry()*. This function parses the ELF file of the current executable (/proc/self/exe) to find the GOT (Global Offset Table) entry corresponding to the write function.
It returns the address (offset) in memory where the GOT entry for write() is located.

Let's break down this function:

- First, we open the current excutable in READNONLY mode:

     ```c
     int fd = open("/proc/self/exe", O_RDONLY);
     ```

- Then, we read the ELF headers of the binary and store in a Elf64_Ehdr structure variable:

     ```c
     Elf64_Ehdr ehdr; //We declare a variable 
     pread(fd, &ehdr, sizeof(ehdr), 0); //preads() reads from fd an amount of bytes equal to the size of ehdr (Elf64_Ehdr structure) at offset 0 and stores it on ehdr variable.
     ```

     Now, ehdr contains basic info about the ELF file. With the information provided by the ELF Headers, we can read the section from the ELF in order to locate where the GOT get mapped in memory since is a non PIE binary.

- Now, we use the information retrieved in the efdr variable to get the sections of the ELF. Precisely, we read the *section names table* and store it on sh_str variable: 

     ```c
     Elf64_Shdr shdr;
     Elf64_Shdr sh_str; //We declare two variables of the Section Header structure type.
     pread(fd, &sh_str, sizeof(sh_str), ehdr.e_shoff + ehdr.e_shstrndx * sizeof(shdr)); //We use pread() function to read from the ELF binary a section as much bigger as a section header, sizeof(sh_str), starting at ehdr.e_shoff + ehdr.e_shstrndx, this is; the Section Header OFFset (shoff) plus an index to access the section header string table.
     ```

     This sections contains names for all other sections (.text, .data,...).  

- Then, we read the specific string table, in order to find specific sections with useful information:

     ```c
     char *shstrtab = malloc(sh_str.sh_size); //We define a "string" and initialize it allocating memory
     pread(fd, shstrtab, sh_str.sh_size, sh_str.sh_offset); //Read from the section table.
     
     Elf64_Shdr relaplt = {0};
     Elf64_Shdr dynsym = {0};
     Elf64_Shdr strtab = {0};
     
     
     for (int i = 0; i < ehdr.e_shnum; i++) { //Iterate every section header with a for loop
          
          pread(fd, &shdr, sizeof(shdr), ehdr.e_shoff + i * sizeof(shdr));
          const char *name = shstrtab + shdr.sh_name; //Then we get the name of the section header.
          
          
          if (strcmp(name, ".rela.plt") == 0 || strcmp(name, ".rela.plt.sec") == 0) {
               relaplt = shdr;
          } else if (strcmp(name, ".dynsym") == 0) {
               dynsym = shdr;
          } else if (strcmp(name, ".dynstr") == 0) {
               strtab = shdr;
          } //We compare if the name is one we desire. If it is, we store it.
     }
    ```

    This sections are critical:

    - **.rela.plt**: Holds relocations entries for PLT functions.
    - **.dynsym**: Holds dynamic symbols.
    - **.dynstr**: Holds names of symbols.

     Is in this sections where we gona find 'write' function GOT entry. 

- First, we resolve the name of the symbols by going to *.dynstr* and reading it and storing it in *strtab_data*:

     ```c
     char *strtab_data = malloc(strtab.sh_size);
     pread(fd, strtab_data, strtab.sh_size, strtab.sh_offset);
     ```

- Then we iterate over each relaction entry of the *.rela.plt* section and each index reading it and compare if the name of this symbol is 'write', if it is, we return the offset of the relocation entry and close all file descriptors:

     ```c
     Elf64_Rela rela;
     Elf64_Sym sym;

     for (size_t off = 0; off < relaplt.sh_size; off += sizeof(Elf64_Rela)) {
          pread(fd, &rela, sizeof(rela), relaplt.sh_offset + off);
          size_t sym_idx = ELF64_R_SYM(rela.r_info);
          pread(fd, &sym, sizeof(sym), dynsym.sh_offset + sym_idx * sizeof(Elf64_Sym));

          const char *name = strtab_data + sym.st_name;
          if (strcmp(name, "write") == 0) {
               free(shstrtab);
               free(strtab_data);
               close(fd);
               return rela.r_offset;
          }
     }
     ```

     At the end, we have gathered the relocation entry of the 'write' symbol at runtime without need to write directly in the code after search it manually.

So, in summary; we opned the binary to read the ELF headers and retrieve some sections with important information. Then, using this sections, we lookout for 'write' symbol and retrieve the entry of the relocation entry.

Then the other parts of the code ar pretty the same as before. We just define the function to replace and we change the permissions over the page where the GOT entry relies and then overwrite the entry with our function.

The result is, in our code, execute write is equal to execute 'hook_function'.

If we compile and execute our code as follows we would get:

```sh
$ gcc -no-pie -o test test.c;./test

[+] GOT entry for write(): 0x404008
Goodbye cruel world!
```

