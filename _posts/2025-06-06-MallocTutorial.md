---
layout: post
title: Malloc Tutorial
subtitle: A brief tutorial about how malloc() and free() work internally.
tags: [C]
---
### Malloc's tutorial.

#### Introduction.

*Malloc* is the function to allocate memory blocks in a C program. The purpouse of this section is build a simple own-malloc version in order to understand the underlying concepts:

- How memory is managed in a process.
- Reallocation and freeing memory chunks.

<br>

#### Presenting Malloc. Principal features & Signature.

*Malloc* (referred as malloc(3) in Posix documentation), is a standard C library function (included in the standard C runtime library; libc, that any compliant C environment must provide) that *allocates* memory chunks. We will get into this later.

This functions usually complies the following rules:

- *malloc* function allocates at least the number of bytes passed to the function by argument.
- *malloc* function returns a pointer that points to a space where the program can read or write successfully, (allocated memory space).
- *malloc* will not reuse allocated space unless the pointer which references it gets freed.
- *malloc* also has to be fast, deterministic and thread-safe. In short terms, it has to be a reliable function when use it.

The signature of malloc is:

```c
void* malloc(size_t size);
```

This means that malloc accepts only one argument which is the number of bytes of memory to be allocated an returns a void pointer, which is a pointer that can points to any data type (most likely it ought to be cast after call).

<br>

#### The Process's Memory. Stack and Heap.

##### Process and Virtual Address.

A process is the instance of a program loaded in memory. Each process has its own virtual address space dynamically translated into physical address by the *Memory Management Unit* and the *kernel*.

This virtual space can be split in several parts. In general terms we have:

- Some space for the code.
- A stack where local and volatile data are stored.
- Some space for constants and global variables.
- Unorganized space for program's data; HEAP.

<br>

##### The heap.

###### Introducing the heap.

In C programming, from the allocator's point of view, the heap is a continuous space of memory, meaning that handles memory as a linear space, with three bounds:

- A starting point (start_brk); pointer which denotes the beginning of the heap segment.
- A maximum limit (managed through sys/ressource.h’s functions getrlimit(2) and setrlimit(2)).
- *brk/program break*, a pointer which marks the end of the mapped memory space within the reserved memory space for the process.

<br>

<div style="text-align:center">
<img src="{{ 'assets/img/C/image-1.png' | relative_url }}" text-align="center"/>
</div>


<br>

In the image above we can see the heap, is the space between the starting point and the break and it grows to upper memory addresses, this means; *memory address above the break point aren't part of the heap* 

Thus, when we call *malloc*, this functions gets a chunk from the unmapped heap region and allocate it, moving the *break* limit making the mapped region to grown and the unmmaped region to shrink.

<br>

<div style="text-align:center">
<img src="{{ 'assets/img/C/image-2.png' | relative_url }}" text-align="center"/>
</div>

<br>

###### brk and sbrk.

*brk(2)* and *sbrk(2)* are both system calls provided directly by the Unix kernel to manage a process's data segment (the heap). They are also both obsolote so in contrast with malloc, they aren't part of the standard C library or the Posix-standard. Current dynamic memory managment is handled by mmap().

- *brk()*:

    ```c
    int brk(const void *addr);
    ```

    Place the break at the given address *addr* and returns 0 if operation is succesfull (-1 otherwise).

    <br>

- *sbrk(2)* This function is a wrapper of brk, (this means that internally calls brk()).This function moves the program break by the given increment in bytes. It returns the previous or the new break address. If faild it returns (void *) -1. 

    Is worth to mention that, when the increment is null, then the return value is the current break address and it can be used to retrieve the initial position of the heap:

    ```c
    void* sbrk(intptr_t incr);
    int* brkptr = sbrk(0); 
    ```
    
    We will use sbrk as our main tool to implement *malloc*. 

    <br>

###### Unmapped Region and No-Man's Land.

We saw earlier that the break mark the end of the mapped virtual adress space: accessing adresses above the break should trigger a bus error. The remaining space between the break and the maximum limit of the heap is not associated to physical memory by the virtual memory manager of the system (the MMU and the dedicated part of the kernel.).

Physical memory and virtual memory is organize in *pages* (frames for the physical memory) of fixed size (on most actual system a page size is 4096 bytes). Thus, the break pointer may not be on pages boundary.

<br>

<div style="text-align:center">
<img src="{{ 'assets/img/C/image-3.png' | relative_url }}" text-align="center"/>
</div>

<br>

The image above presents the previous memory organisation with page boundaries. We can see the break may not correspond to a page boundary. The status of the memory betwee the break and the next page boundary is, in fact, available. 

The issue is that you don’t have any clue on the position of the next boundary, you can find it but it is system dependant and badly advise. **This no-man’s land is often a root of bugs**. Some wrong manipulations of pointer outside of the heap can success most of the time with small tests and fail only when manipulating larger amount of data.

<br>

#### Dummy malloc.

First, we will play with sbrk(2) to code a dummy malloc. This malloc is probabily the
worst one, even if it is the simplest and quiet the fastest one.

The idea is very simple, each time malloc is called we move the break by the amount of space required and return the previous address of the break. This is the main functionality of *malloc*, to move the break pointer in order to release an amount of memory in the mapped section of the Heap.

It is simple and fast, it takes only three lines, but we cannot do a real free and of course realloc is impossible.

This malloc waste a lot of space in obsolete memory chunks. It is only here for educationnal purpose and to try the sbrk(2) syscall. For educationnal purposes, we will also add some error management to our malloc.

```c
/* Dummy malloc code */

#include <sys/types.h>
#include <unistd.h>

void* malloc(size_t size) {

    void* p;
    p = sbrk(0); //The current break position pointer.
    
    if (sbrk(size) == (void*) -1) { //Then, we move the break pointer a size of bytes up and check the result of the operation and return NUll if fails, other wise, we return the position of break.
        return NULL;
    }

    return p;
}
```

As a result, when we use this malloc we retrieve a pointer that points to the upper memory address of a new release memory chunk available to be write in in the mapped section.

<br>

#### Organizing the Heap.

##### Defining the idea.

The previous section we craft a dumby malloc since we have a lack of information about the Heap. In this section we will try to find an organisation of the heap so that we can have an efficient malloc but also a free and a realloc.

We need extra-information at begining of each chunks indicating at least:

- The size of the chunk
- The address of the next one and whether its free or not.

This can be achieved by adding a small block at the begining of each chunk containing the metadata that will consist in:

- A pointer to the next chunk.
- A flag to mark free chunks
- The size.

This block would be before the pointer rlinked list in eturned by malloc that points now, not to the first address of the chunk (that englobes the metadata too), but of the data block:

<br>

<div style="text-align:center">
<img src="{{ 'assets/img/C/image-4.png' | relative_url }}" text-align="center"/>
</div>

<br>

So our memory chunk allocated by malloc now consist of the data chunk of the requested size and also a small chunk of metadata.

<br>

##### Implementing it in C.

###### Brief Annex: Structure Padding and Memory Alignament.

Lets start introducing some definitions:

- A *struct* in C is just a contigous block of memory that holds different members often of different types (different sizes). 

- A *word* is the natural unit of data used by a processor, typically corresponding to the size of its general-purpose registers and the amount of data it can process or transfer in a single operation. This means that what a processor understands for a word is architecture-specific. (4 bytes in 32-bits processors, 8 for 64-bits). 

    This also means that the processor accesses memory in multiples of the word size. That is, if the word size is 4 bytes and the processor wants to read memory at address 0x1002, it will read from 0x1000 to 0x1004 in order to access 0x1002.

With this clear, *Structure padding* is a mechanism implemented by operating systems in order to make more efficient processor's access to memory. 

If we consider the following structure in a 32 bits architecture:

```c
struct abc {
    char a; // 1 byte
    char b; // 1 byte
    int c; // 4 byte
}
```
We could think that the total size of the structure would be the amount of the size of his fields (1 + 1 + 4) but this is wrong.

Then, we remember that, our procesor can only read a word's size in a cycle, so it would read 'a', 'b' and the two first bytes of 'c', lefting two bytes for the second cycle:

<br>

<div style="text-align:center">
<img src="{{ 'assets/img/C/image-5.png' | relative_url }}" text-align="center"/>
</div>

<br>

Two cicles are required to access contents of variable 'c' and this is not efficient since is an unnecesary wastage of CPU cycles whenever we want to access to 'c' value. Is necesary to implement a way that makes the procesor to use the minimum number of CPU cycles to access a variable value. 

So the processor implements the *structure padding* mechanism, in which the processor adds some empty bytes of memory in order to ensure the alignament of data members with the word size. Ensuring efficency in the requiered CPU cycles to access data in memory.

Following the previous example, the efficent way to organize the data over the address we would be to create a empty space of two bytes between 'b' and 'c' variables. This would result to 'c' to be stored on a memory address location multiple of the word size (4 bytes).

<br>

<div style="text-align:center">
<img src="{{ 'assets/img/C/image-6.png' | relative_url }}" text-align="center"/>
</div>

<br>

This initially would solve the problem and in fact change the size of the structure since we add the 2 bytes size of empty space for the padding:

```
char + char + padding + int == 1 + 1 + 2 + 4 == 8
```

Note that the padding changes (and structure's size) changes if the order of the data types changes. Lets consider the following sort:

```c
struct abc {
    char a; // 1 byte
    int c; // 4 byte
    char b; // 1 byte
}
```

This would provoke to add an empty space of 3 bytes between 'a' and 'c' and between 'c' and 'b' obtaining:

```c
char + padding + int + padding + char == 1 + 3 + 4 + 3 + 1 == 8
```

<br>

###### Linked lists.

Now, we traslate the idea above in to C code through the *linked lists*. 

A *single linked list* is a linear data structure in C where each element (called node) is connected to the next one using pointers. 

- the data, 
- the pointer to the next node. 

The last node's (also known as tail) pointers point to NULL to indicate the end of the linked list. 

```c
struct Node {
  
  // Data field - can be of any type and count
  int data;
  
  // Pointer to the next node
  struct Node* next;
}
```

The C code above was an example, in our case, it would turn out like the following:

```c
typedef struct s_block *t_block; //Create a type allias 't_block' for a pointer to a structure.Is a reference to a type using another name.

struct s_block {
    size_t size; //Size of the chunk.
    t_block next; //Pointer for the next chunk.
    int free; //Flag for the state.
};
```

Now, we have to allocate enough space with *sbrk()* for the struct’s size: (assuming 64bit system - 8 bytes word)

- 8 bytes --> size_t size (unsigned long). This is 8 bytes in 64bc 
- 8 bytes --> t_block (pointer to structure s_block). A pointer type is 8 bytes in 64bit systems or 4 bytes in 32bit systems
- 4 bytes --> int free (signed integer)

Thus, when the compiler encounters an access to a struct field (like for example free as s.free) it just translates it to the base address of the struct instance plus the offset of the appropriate member/field desired to be accessed within the structure.

However, we have to be careful with structure padding and member alignment when calculating structure member offsets and the structure size itself, as one could initially think that sizeof(struct s_block) would be 20 (8 + 8 + 4), but in reality it will be 24 due to member alignment (8 + 8 + 8) just as we seen in the annex before.


Thus, when the compiler encounter an access to struct field (like for example free; s.free) it just translate it to the base address of the struct plus the sum of the length of the previous field without complex memory address operations.

So we allocate enough space with sbrk() for the chunck (including the size of the meta-data plus the size of the data block) and put the address of the old break in a variable of type t_block.

```c
typedef struct s_block *t_block; //Create a type allias 't_block' for a pointer to a structure.Is a reference to a type using another name.

struct s_block {
    size_t size;
    t_block next;
    int free;
};

t_block b = (t_block)sbrk(0); //We save the program break.
sbrk(sizeof(struct s_block) + size); // Expand the heap by adding the needed space for the struct (metadata) and the size of the chunk (the memory chunk it self).

b->size = size; //Store the value of the requested size in the size field of the struct pointed to by b.
```
We can notice that, we assign sbrk(0), a pointer to program break, to a t_block b data type; which is a pointer to a structure (s_block). This can be confusing at first time since the program break is not memory chunk, but when we then expand the heap, the previous program_break now points to the head of a totally valid new memory chunk:


```
       Heap Start
           |
     [ used memory ]
           |
           v
+------------------------+
|  struct s_block        |   <-- b points here, previous break.
+------------------------+
|  user data (size)      |
+------------------------+
           |
           v
  [ new program break ]
```

<br>

#### A First Fit Malloc.

Now that we know how to organize the heap, we will try to implement the classic *first fit* malloc.  

First Fit is one of the classic strategies for implementing malloc() — it's how your custom memory allocator decides where to place a new allocation in the heap. We will do this *by going through the chunks list and stop when we find a free block with enough space for the requested allocation.*

<br>

##### Memory alignament.

Remembering what we said about *structure padding*, the same thing and for the same reason applies to pointers; **memory addresses (pointers) must often be multiples of the size of the data type they point to.**

Since the pointer holds the first address in which data is stored, if this address is not aligned with a multiple of the size of the data type (for example an integer), the data may be fetched between several CPU cycles rather than just one, making more likely the need of more CPU cycles than required to access the data referenced by the pointer.

<br>

<div style="text-align:center">
<img src="{{ 'assets/img/C/image-7.png' | relative_url }}" text-align="center"/>
</div>

<br>


In the image above, the only valid address for a pointer to an integer are 0x996, 0x1000 or 0x1004. Every other memory in this segment would lead to the need of at least two CPU cycles to read our integer. 

This is what we call *memory alignment*, which enounces: **any data type must be stored in a memory address multiple of his own size value.** For example, an integer only can be stored in an address multiple of 4.

<br>

##### Aligned Chunks. Arithmetic trick.

Going back to our memory chunk, since our meta-data block is already aligned, the only thing we need is to align the size of the data block. This means that if we when we allocate a block of memory (e.g., 13 bytes), you can’t just hand out 13 bytes we have to round it up to the next multiple of 4 (the word size), 16 bytes in this case, so that the next block or data remains aligned.

Assuming, obviously, that a size is just an integer, we use the following arithmetic trick to get the next multiple of four since one given integer:

```
m = (n - 1)/4 · 4 + 4 for any integer, n.
```

We can extrapole this to C by defining the following macro:

```c
#define align4(x) (((((x) -1) > >2) < <2)+4)
```

We can applie this to transform the size of any chunk into a multiple of 4.

<br>

##### Finding a chunk: the First Fit Algorithm.

Now, we have to implement an algorithm in order to go to over a list and find sufficiently wide chunk for user requirements.

We begin at the base addresss of the heap (we will get in to this later), then we test the current chunk, if fit; return his address, if not; we continue to the next chunk. In this process, the hole key thing is to keep the last visited chunk, in order to be able to expand the heap if we don't find a suitable size.

Let's draw this in order to get a graphic vision of the algorithm:

```txt
heap_start --> [used | 32] --> [free | 64] --> [used | 16] --> NULL
                                                         ↑
                                                    last visited
                                                   (program break)
```

Since the heap starts in lower address and grows upwards, then, the pointer to the last chunk coincides with the *program break* since is the toppest part of the heap. Thus, if we keep in each iteration with the previous pointer we visited, when we get a NULL when we iterates over the list we then we knoe that the previous pointer is in fact the program break.

We apply this in C through the next function:

```c
t_block find_block(t_block *last, size_t size){ //This function needs two arguments, the requested size and a pointer to t_block (this is a pointer to a pointer to the metadata).

    //Also, this function starts at the base of the heap and then go over the heap's chunks in order to find is there is any chunk that fits.

    t_block b = (t_block)base; //We define a pointer to the s_block structure and assign it the base. This would serve us as the start point. Then, we define the following loop. 

    while(b && !(b->free && b->size >= size)) { //This means that, while b is not NULL pointer, and the chunk referenced is not free or chunk's size is less than the requested size. Then,

        *last = b; //We modify the the last pointer in order to retain the last chunk.
        b = b->next; //Then, we assign b the pointer to the next chunk and reevaluate.
    
    //Thus, if we find a chunk we return it nd last will point to the previous chunk to that one, and if not we return NULL and *last will point to the program break
    }
    return(b);
}
```

<br>

##### Extending the Heap.

In the previous block we implement a algorthim which goes over a linked list of chunks in order to find one free and whichi fit the requested size.

Now, we won’t always have a fitting chunk, and sometimes (especially at the begining of the
program using our malloc) we need to extends the heap. This is quite simple: *we move the break and initialize a new block, of course we need to update the next field of the last block on the heap.*

First, we define a macro to hold the size of meta-data block:

```c
#define BLOCK_SIZE sizeof(struct s_block)
```

Then implement the following C code:

```c
t_block extend_heap(t_block last , size_t s){

    t_block b;
    b = sbrk (0); //Save program break.

    if (sbrk(BLOCK_SIZE + s) == (void*) -1) //Expand the heap by adding space for metadata and size of the chunk as we see previously and return NULL if fails and returns -1.
        return (NULL );

    b->size = s;
    b->next = NULL; //Since this is the last memory chunk, the valur to the 'next pointer' is NULL.

    if (last)
        last ->next = b; //If the pointer of the previous chunk is not NULL, then last's 'next pointer' will point to the current chunk. Program break previous heap's expanding.

    b->free = 0; //Then we sett current chunk as not free.

    return (b);
}
```

We just expand the heap as we see in previous sections and return NULL if sbrk fails or the pointer to the new chunk if succeed.

(void*) -1 is a special value in C that typically represents an error return value for functions that return a void*, such as sbrk() or mmap().

<br>

##### Spliting blocks.

In the previous fit first algorithm, we just provide the first memory chunk that fit in the size, but this is not efficient since there is a wasted of a lot of space (request 2 bytes and find a block of 256 bytes):

<br>

<div style="text-align:center">
<img src="{{ 'assets/img/C/image-8.png' | relative_url }}" text-align="center"/>
</div>

<br>

A first solution is to split blocks: when a chunk is wide enough to held the asked size plus a new chunk (at least BLOCK SIZE + 4, remember that 4 is in 32-bits system, our example, the word's size bytes, so 4 bytes is the minimum extra space we need to ensure a new chunk is available), we insert a new chunk in the list:

<br>

<div style="text-align:center">
<img src="{{'assets/img/C/image-9.png' | relative_url }}" text-align="center"/>
</div>

<br>

We can implement this split following the next two steps:

- First, we just add a another field to struct s_block of *type characters array*:

    ```c
    char data[1];
    ```

    Sorted at the end of the s_block structure:

    ```c
    struct s_block {
        size_t size;
        t_block next;
        int free;
        char data[1];
    };
    ```

    This array is formaly translate as a pointer to a char to the first element of the list, at placing it at the end of the list, this pointer is effectively telling us where the metadata of the chunks end, and where the chunk itself begins.

    After this, of course we have to update the BLOCK_SIZE macro:

    ```c
    #define BLOCK_SIZE 12 
    ```

    In an 32 bits architecture, the size of structure would be size_t + t_block + int + char ptr == 4 bytes + 4 bytes + 4 bytes + 1 byte (4 if we consider the structure padding). But, we defined above that the block size will be 12, this takes out of the equation the size of the data pointer, making that this pointer now points to the first address of the chunk it self.

    If we instead get 

    ```c
    #define BLOCK_SIZE sizeof(struct s_block) 
    ```
    We would get 16 bytes (4+4+4+4 as we see before) including 4 bytes more between the metadata and the chunk.

    So, in summary, we set the size to 12 in order to make the data pointer behave as a separator between the metadata and the s_block.

- Ultimately, we define the following split_block function:

    ```c
    void split_block (t_block b, size_t s){ //Accepts as arguments the block which fits and the request space by the user.
        t_block new; // We define a new chunk.

        new = (t_block)(b->data+s); // The pointer to point to the new block, points to the ends of the metadata plus the size request by the user.
        new ->size = b->size - s - BLOCK_SIZE ; // The new size of the chunk is the subtraction of the size requested by the user plus the metadata minus the original size.
        new ->next = b->next; // The 'next pointer' of the new chunk obviously remains untouch.
        new ->free = 1; //The new chunks is a free chunk.


        b->size = s; 
        b->next = new; //Then we change the previous size of the original block and make its 'next pointer' to points to the new chunk.
    }
    ```

    So, in summary, this new function, gets as arguments the chunk and the request size and creates whithin a space for metadata just after the size request is meet. Then it changes the atributes value performing to separate blocks.

<br>

##### The malloc function.

Now, we can do our malloc function. This would be none but a wrapper around previous functions. 

Before implement this malloc function, we have to consider to:

- **Align the request size:** This means that when we request some memory allocation, this allocation must fit to the memory alignment requirements; be a multiple of the word's size. For example;

    ```c
    malloc(13); --> 13 should get alligned to 16.
    malloc(7); --> 7 should get alligned to 8.
    ```

    Without this regulation CPU can fall into penalties or crashes due to misaligned access.

    For this task we will use the align macro we define before:

    ```c
    #define align4(x) (((((x) -1) > >2) < <2)+4)
    ```

    Which finds the next multiple of 4 of the 'x' variable.

- **Test if is malloc's first call:** We have to test if a call to malloc is the first, in order to create the linked lists and expand the heap. In subsequent calls, we will search a fit chunk in the list and then, if we not find a suitable chunk, expand the heap. This can be achieve through the base of the Heap as we will see.


    First of all, we have to remember that the function 'find_block' use a global variable called 'base' which is a pointer to the start of the heap we haven't define yet:

    ```c
    void *base=NULL;
    ```

    It is a void pointer and it is NULL initialized. The first thing our malloc does is to test
    base, if it NULL then this the first time we’re called, otherwise we start the previously described algorithm.

<br>

So, having this considerations, our malloc performs the following steps:

- First align the requested size.

- If *base* is initialized.

    - Search for a free chunk wide enough;

        - If we found a chunk:

            - Try to split the block (the difference between the requested size and the size of the block is enough to store the meta-data and a minimal block, 4 bytes)
            
            - Mark the chunk as used (b->free=0;)

        - Otherwise: we extend the heap. Note the use of the last: find block put the pointer to the last visited chunk in last, so we can access it during the extension without traversing the whole list again.
    
    - Otherwise: we extended the heap (which is empty at that point.) Note that our function extend heap works here with last=NULL.

<br>

The complete malloc code would be as follows:

```c
#include <stddef.h>
#include <unistd.h>
#define BLOCK_SIZE 12 
#define align4(x) (((((x)-1)>>2)<<2)+4)

typedef struct s_block *t_block;
void *base=NULL;

struct s_block {
    size_t size;
    t_block next;
    int free;
    char data[1];
};

t_block extend_heap(t_block last , size_t s){
    t_block b = (t_block)sbrk(0);
    if(sbrk(BLOCK_SIZE + s) == (void*)-1)
        return (NULL);
    
    b->size = s;
    b->next = NULL;

    if (last)
        last->next = b;

    b->free = 0;
    
    return(b);
}

t_block find_block(t_block *last , size_t size ){
    t_block b = (t_block)base;

    while(b && !(b->free && b->size >= size)) {
        *last = b;
        b = b->next;
    }

    return (b);
}

void split_block(t_block b, size_t s){
    t_block new;
    new = (t_block)(b->data+s);

    new ->size = b->size - s - BLOCK_SIZE ;
    new ->next = b->next;
    new ->free = 1;

    b->size = s;
    b->next = new;
}

void* malloc(size_t size) { //It only requires one argument, the size of the allocated memory.

    size_t s = align4(size); //Although the requested size, s, we need to align it with the following next multiple of the word's size value. This is because the memory alignment principle.
    t_block b, last; //t_block is an alias to a pointer of a structure which holds the metadata of the chunk in order to store some valueable information: size,

    if (base) { //If base is not NULL, then we have to go over the heap in order to see if we find a valid free size-fit memory chunk.

        last = base; //We set the last pointer as the base of the heap.
        b = find_block(&last,s); //Then, we attemp to find a chunk.

        //Then, we evaluate what find_block has returns us, if is not a NULL pointer, then it points to a chunk and we then evaluate if we can split it in order to evade memory wasting.

        if (b) {
            last = base;
            if ((b->size - s) >= (BLOCK_SIZE + 4)) //We check if there is, without the size of the chunk it self (b->size - s), minimal space (at least the word's size: 4 bytes) to hold a new chunk (this is the metadata plus the chunk space: BLOCK_SIZE + 4).
                split_block(b,s); 
    
            b->free = 0; //We set the chunk as not free.
        } else { //If bis NULL, there is not suitable chunk and the extend the heap from the last chunk in the heap. Remember from find_block that if b is null, then last holds the program_break.

            last = base;
            b = find_block(&last,s);
            
            if(!b) //We check if we manage to extend the heap.
                return(NULL);
        }

    } else {
        b = extend_heap(last, s);

        if(!b) //We check if the pointer to the extended heap region is valid, if not return NULL.
            return(NULL);
        
        base = b; //Assign b pointer to base in order to not repeat this loop.
    }

    return(b->data); //At last we return the first memory address of the chunk (not a pointer to the metadata) Remember that data holds the first valid address of the allocated chunk it self.
}
```

<br>

### Calloc, Free, and Realloc.

Now that we have pretty clear how malloc works lets deep in other functions that provides auxiliary functionality that complements to the behaviour of malloc.

<br>

#### Calloc.

Calloc is pretty the same as Malloc with the difference that calloc initiates the memory to 0. So we can implement calloc in C understanding it like a wrapper for malloc with extra functionality:

```c
size_t *calloc_custom(size_t number, size_t size) { //In this case, we accept the number of chunks to be allocated and the size of each chunk.
    size_t *ptr = malloc(number * size); //We allocate that memory and get the pointer.
    if (ptr) { //If operation is succesfull and ptr is not NULL, we get the number of words (4bytes in32 bits) which fit in the chunk and initiate those chunks to 0
        size_t total_words = (number * size) / sizeof(size_t);
        for (size_t i = 0; i < total_words; i++) {
            ptr[i] = 0; //We zero every 4 bytes chunk.
        }
    }
    return ptr; //Then return pointer.
}
```

A brief incisión is that:

```c
int* ptr;
ptr[0] = 0 == *(int*)(ptr + 0) //Which means, write to '0' <sizeof(int)> bytes starting in the offset (0 + sizeof(int)*0) from the memory address holded in ptr.
ptr[1] = 0 == *(int*)(ptr + 1) //Which means, write to '0' <sizeof(int)> bytes starting in the offset (0 + sizeof(int)*1) from the memory address holded in ptr.
...
ptr[i] = 0 == *(int*)(ptr + i) //Which means, write to '0' <sizeof(int)> bytes starting in the offset (0 + sizeof(int)*i) from the memory address holded in ptr.
```

So in that loop we are writing to '0' chunks of 4 bytes performing steps of 4 bytes, so we are writing all bytes to 0.

<br>

#### Free.

In short terms, *free()* function is used to deallocated allocated memory previously allocated with malloc or similar.  

<br>

##### The space fragmentation problem. Fusion() function.

A major issue of malloc is *fragmentation*: after several use of malloc and free, we end with a heap divided in many chunks individually to small to satify big malloc while the whole free space would have been sufficient. 

This issue is known as the space fragmentation problem. For example; when we select a free chunk wider enough to hold the requested allocation and another chunk, we split the current chunk. While this offer a better usage of the memory (the new chunk is free for further reservation) it introduces more fragmentation.

We can implement some solutions to this problem:

A solution to eliminate some fragmentation is to fusion free chunks. When we free a chunk, and its neighbors are also free, we can fusion them in one bigger chunk. All we have to do is to test the next and previous chunks. This also raises other problem; who do we find the previous chunk? We can double link our list.

For this, we implement the following function. This function act with a modification of our s_block struct we will explain later.

```c
t_block fusion(t_block b){
    if (b->next && b->next->free){ //If b->next pointer is not NULL, and b->next->free is not 0, this means the next chunk is free; then we fusion both blocks by updating the fields of the first:
        b->size += BLOCK_SIZE + b->next ->size; //We add to the first's size the second's size along with the metadasize.
        b->next = b->next ->next; //Then we assign the get out of the linked list the second chunk making it in fact invisible and occuping his space in the heap.

        if (b->next)
            b->next->prev = b;//This is part of a later modification of our structure s_block entity adding a prev field (double link). Here we are setting the prev field of the next pointer as b, again getting out the second chunk out of the linked list.
    }
    return(b)
}
```

Fusion is straightforward: if the next chunk is free, we sum the sizes of the current chunk and the next one, plus the meta-data size. Then, the we make next field point to the successor of our successor and if this successor exists we update its predecessor.

<br>

##### Finding the right chunk.

The other free’s issue is to find, efficiently, the correct chunk with only the pointer returned by malloc. In fact, there are several issues here:

- Finding the metadata pointer.

    When malloc returns a pointer, it dont returns a pointer to the s_block structure, it returns a pointer to the first memory address of the chunk itself as we see before. In order to get the address to the block we can implement the following code:

    ```c
    t_block get_block (void *p) { //Giving a pointer,
        char *tmp = p; //Convert p to a char* pointer.
        return (p = tmp -= BLOCK_SIZE); //Then return the result of an arithmetic operation: subtract BLOCK_SIZE bytes (since tmp is char*, his size is 1 byte) to go back to the start of the metadata (struct s_block), and return it cast as t_block.
    }
    ```

- Validating the input pointer (is it realy a malloc’ed pointer ?)
    
    In order to solve this problem, we implement the following actions.

    First, in order to identify valid pointers, this is; pointers which aims to malloc'eds chunks, we implement a self-contained match. 
    
    We add a new field on the s_block struct: a pointer *ptr* which points to the data pointer. Then, if we call free(p) and we can find the relation 'p->ptr == p->data', then we can be sure that 'p' points to malloc'ed chunk that can be free.

    ```c
    struct s_block {
        size_t size;
        struct s_block *next;
        struct s_block *prev;
        int free;
        void *ptr; //Pointer which later will point to data.
        char data[1];
        };
    ```

    Then, the function that valids the rule described before.

    ```c
    int valid_addr(void *p){

        if (base){
            if (p>base && p<sbrk(0)){ //First we check if p is between the program break and the heap's base, so we check if is exists within the heap
                return (p == (get_block(p))->ptr); //Then, we get the metadata of the chunk and check the selfmatch describe above.
            }
        }
        return(0)
    }
    ```
<br>

##### The free function.

Now that we know how to check a chunk, this seems pretty easy. The free() function recieves a pointer, then validates it and mark it free.

We an also try to release memory from the heap if the chunk is at the end of the heap. 

```c
void free(void* p) {

    t_block b;
    if (valid_addr(p)) //If pointer is valid.

        b = get_block(p); //Get the metadata block.
        b->free = 1;

        if (b->prev && b->prev->free) //If previous block exists and is free, we fusion.
            b = fusion(b->prev);

        if (b->next){ //If next chunk is not NULL, we attempt to fusion.
            fusion(b);
    
        } else { //If b->next is NULL, then current chunk is the last chunk in the heap.

            if (b->prev)
                b->prev->next = NULL; //Then, prev 'next-pointer' points to NULL or
            else
                base = NULL; //b->prev is NULL thus, no more chunks in heap.

            brk(b) //We set the program break to the b pointer, reducing the heap by freeing the b chunk.
        }
}
```

<br>

#### Realloc.

In the other han, *realloc* function is straightforward. Basically, we only need a memory copy operation.

```c
void copy_block(t_block src, t_block dst){

    int *sdata, *ddata;
    size_t  i;
    sdata = src->ptr;
    ddata = dst->ptr;
    for (i=0;i*4 < src->size && i*4 < dst->size;i++) //This loops ensure we don't copy beyodnd source/destiny limits. The multiplication by 4 is since we are copying word's size.
        ddata[i] = sdata[i];

}
```

With the copy operation defined, we can now write the realloc function.

```c
t_block realloc(void *p, size_t newsize){

    t_block b = (t_block)malloc(newsize); //Allocate a new pointer with the new size.

    if(b){ //If malloc succed, then we copy the contents and free previous block.
        copy_block(p, b);
        free(p);
        return(b);
    } else {
        return(NULL);
    }
}
```

Although, we can write something more efficient, we could validate if we really need to realloc or we can just:

- Check if the size don't change. In that case we do nothing.
- If the new size is less than the original, we try to split the chunk if is posible.
- If the new size is bigger than original and the next chunk is free, we attempt to fusion.

```c
void *realloc(void *p, size_t size){

    size_t  s;
    t_block b, new;
    void    *newp;
    
    if (!p) //If p is null, realloc behaves exactly as malloc.
        return(malloc(size));
    
    if(valid_addr(p)){

        s = align4(size); //We align the size.
        b = get_block(p); //Get the metadata blok
        if(b->size >= s){ //Check if current pointer stores a bigger size that requested.

            if(b->size -s >= (BLOCK_SIZE + 4)) //If it is, and we can, we split the current bblock.
                split_block(b,s);

        } else { //If not, then we try to fusion with the net chunk if is possible.

            if(b->next && b->next->free && (b->size + BLOCK_SIZE + b->next->size) >= s) {//We check if the fusion of both chunks fulfill our requirements. Then, we fusion.

                fusion(b);
                if (b->size - s >= (BLOCK_SIZE +4)) //If there is still some space available, we split.
                    split_block(b,s);
            } else { //If not, realloc.

                newp = malloc(s);
                if(!newp)
                    return(NULL);

                new = get_block(newp);
                copy_block(b,new);
                free(p);
                return(newp);          
            }

        } 
    }
}
```

<br>