---
layout: post
title: Garbage Collector.
subtitle: Simple conservativer sweep-mark Garbage Collector.
tags: [C]
---

### Garbage Collector.

This is the second part, or the continuation from the previous report in which we build a custom memory allocator from a heap array. Now, we are going to pick up the code where we leave it and implement a garbage collector with several changes in the original code.

<br>

#### Explaning the idea.

First, a Garbage Collector (for now on: GC), is a program responsable for free unreachable memory in the program in order to optimize the use of the dinamic memory allocation space. 

We are referening as *unreachable memory* to those allocated chunks that doesn't have a reference (pointer) in the program making them unnaccesible. From the program perspective this kind of chunks no longer exists and represents a waste of memory if it is never used again. 

The dynamic memory is stored in the *heap*, and the references to the heap are mainly stored on the *stack*; a memory region that stores function-related data. Whenever we call malloc in main, a pointer to an new alloced chunk in the heap is stored on the main function's stack: 

*For educational purpouses, the following pictures do not represent a realistic stack/heap esqueme, they are just an ilustrative way to show the GC logic.*

<br>

<div style="text-align:center">
<img src="{{ 'assets/img/C/stack-heap.png' | relative_url }}" text-align="center"/>
</div>

<br>

This means that our job is to build a program that sweep the main's stack portion in order to find references to the heap, (this will be pointers to reachable chunks), the rest are marked as unreachable chunks and reclaimed to be freed.

<br>

<div style="text-align:center">
<img src="{{ 'assets/img/C/stack-heap2.png' | relative_url }}" text-align="center"/>
</div>


<br>

Thus, after the action of the GC, the unreachable chunk gets freed and available to be usable again for malloc:

<br>


<div style="text-align:center">
<img src="{{ 'assets/img/C/stack-heapGC.png' | relative_url }}" text-align="center"/>
</div>


<br>

### Explaining the code.

Now we have in some way clear the idea, let's see how implemented in code-terms. We depart from the heap allocator code written before.

At first, we divided the contents from the original file in three, following the standard modularization in C. 

Each file serves to different purpouses:

 - **heap.h** - Header File (declarations / API).
 - **heap.c** - Implementation.
 - **main.c** - Usage / Application.

Relative to the code of the first version of *heap.c* file, several changes are made:

- **Pointer Width Handling**; The *heap* and *malloc* now works with *word-size* slots instead of bytes in order to improve memory alignment in the code. This idea will be explained later.

- **Garbage Collection Added**; The restructured code introduces a simple GC on top of a manual heap allocator (malloc), using stack scanning to determine what memory is still in use.

- **Getting top and base of the stack**. Our implementation of the GC parses the stack memory portion relative to the *main* function in order to identify potential pointers to the heap. In order to be able to explore this portion we have first to provide the range of memory to be scanned (top and base of the stack). This is done by the following way:

    - First, we have modify *malloc* function to set the value of the variable *stack_base* with the base of the stack of the function caller (__builtin_frame_address(1), note the 1), since malloc is called inside main, the caller function is main function. Is worth to mention that the base of the stack not have necesarily to coincide with the base of the stack of the main function.
    
    - On the other hand, the GC is implemented on *heap_collect()* and in this function we set the value of *start_base* with the current frame of the stack (__builtin_frame_address(0), note the 0) which is the top of the stack at that very moment.

    With this, the GC is provided with a range of memory from the stack that corresponds with the first memory allocation to the moment in which a clean job is needed.

    <div style="text-align:center">
    <img src="{{ 'assets/img/C/memory_region.png' | relative_url }}" text-align="center"/>
    </div>


<br>

### Heap.h

The file **heap.h** is a header file which defines the public interface of a custom memory allocator system and a garbage collector. This is, in other words, the contents and definitions (not implementations) that can be share with other applications.

```c
//Check if HEAP_H_ is not defined.(If not, define it. This serves as a flag to prevent the inclusion of the file more than once. If the file has already been included, then the flag exist and then, then directive above prevent the current inclusion.)
#ifndef HEAP_H_
#define HEAP_H_

//Include standard libraries for assert, integers and several functions.
#include <assert.h>
#include <stdint.h>
#include <stdlib.h>

//Define UNIMPLEMENTED statement.
#define UNIMPLEMENTED \
    do { \
        fprintf(stderr, "%s:%d: %s is not implemented yet\n", \
                __FILE__, __LINE__, __func__); \
        abort(); \
    } while(0)

//Define the size of heap in bytes and check if it is aligned with the size of a word.
#define HEAP_CAP_BYTES 640000
static_assert(HEAP_CAP_BYTES % sizeof(uintptr_t) == 0,
              "The heap capacity is not divisible by "
              "the size of the pointer. Of the platform.");
#define HEAP_CAP_WORDS (HEAP_CAP_BYTES / sizeof(uintptr_t))

//Heap definition.
extern uintptr_t heap[HEAP_CAP_WORDS];
extern const uintptr_t *stack_base;

//Heap operations Functions.
void *malloc(size_t size_bytes);
void free(void *ptr);
void heap_collect();

#define CHUNK_LIST_CAP 1024

//Chunk structs.
typedef struct {
    uintptr_t *start;
    size_t size;
} Chunk;

typedef struct {
    size_t count;
    Chunk chunks[CHUNK_LIST_CAP];
} Chunk_List;

//Heap-Free tracklists chunks.
extern Chunk_List alloced_chunks;
extern Chunk_List freed_chunks;
extern Chunk_List tmp_chunks;

//List operations functions.
void chunk_list_insert(Chunk_List *list, void *start, size_t size);
void chunk_list_merge(Chunk_List *dst, const Chunk_List *src);
void chunk_list_dump(const Chunk_List *list, const char *name);
int chunk_list_find(const Chunk_List *list, uintptr_t *ptr);
void chunk_list_remove(Chunk_List *list, size_t index);

#endif // HEAP_H_
```

<br>

### Heap.c

Now, *heap.c* file implements *heap.h* definitions.

This file implements a custom heap memory allocator with a *mark-and-sweep garbage collector*. This code changes the contents of the original heap.c code:

```c
#include <stdio.h>
#include <stdbool.h>
#include <string.h>
#include "./heap.h"


uintptr_t heap[HEAP_CAP_WORDS] = {0};
const uintptr_t *stack_base = 0;

bool reachable_chunks[CHUNK_LIST_CAP] = {0};
void *to_free[CHUNK_LIST_CAP] = {0};
size_t to_free_count = 0;

Chunk_List alloced_chunks = {0};
Chunk_List freed_chunks = {
    .count = 1,
    .chunks = {
        [0] = {.start = heap, .size = sizeof(heap)}
    },
};
Chunk_List tmp_chunks = {0};

void chunk_list_insert(Chunk_List *list, void *start, size_t size)
{
    assert(list->count < CHUNK_LIST_CAP);
    list->chunks[list->count].start = start;
    list->chunks[list->count].size  = size;

    for (size_t i = list->count;
            i > 0 && list->chunks[i].start < list->chunks[i - 1].start;
            --i) {
        const Chunk t = list->chunks[i];
        list->chunks[i] = list->chunks[i - 1];
        list->chunks[i - 1] = t;
    }

    list->count += 1;
}

void chunk_list_merge(Chunk_List *dst, const Chunk_List *src)
{
    dst->count = 0;

    for (size_t i = 0; i < src->count; ++i) {
        const Chunk chunk = src->chunks[i];

        if (dst->count > 0) {
            Chunk *top_chunk = &dst->chunks[dst->count - 1];

            if (top_chunk->start + top_chunk->size == chunk.start) {
                top_chunk->size += chunk.size;
            } else {
                chunk_list_insert(dst, chunk.start, chunk.size);
            }
        } else {
            chunk_list_insert(dst, chunk.start, chunk.size);
        }
    }
}

void chunk_list_dump(const Chunk_List *list, const char *name)
{
    printf("%s Chunks (%zu):\n", name, list->count);
    for (size_t i = 0; i < list->count; ++i) {
        printf("  start: %p, size: %zu\n",
               (void*) list->chunks[i].start,
               list->chunks[i].size);
    }
}

int chunk_list_find(const Chunk_List *list, uintptr_t *ptr)
{
    for (size_t i = 0; i < list->count; ++i) {
        if (list->chunks[i].start == ptr) {
            return (int) i;
        }
    }

    return -1;
}

void chunk_list_remove(Chunk_List *list, size_t index)
{
    assert(index < list->count);
    for (size_t i = index; i < list->count - 1; ++i) {
        list->chunks[i] = list->chunks[i + 1];
    }
    list->count -= 1;
}

void *malloc(size_t size_bytes)
{
    stack_base = (const uintptr_t*)__builtin_frame_address(1);
    const size_t size_words = (size_bytes + sizeof(uintptr_t) - 1) / sizeof(uintptr_t);
    
    if (size_words > 0) {
        chunk_list_merge(&tmp_chunks, &freed_chunks);
        freed_chunks = tmp_chunks;

        for (size_t i = 0; i < freed_chunks.count; ++i) {
            const Chunk chunk = freed_chunks.chunks[i];
            if (chunk.size >= size_words) {
                chunk_list_remove(&freed_chunks, i);

                const size_t tail_size_words = chunk.size - size_words;
                chunk_list_insert(&alloced_chunks, chunk.start, size_words);

                if (tail_size_words > 0) {
                    chunk_list_insert(&freed_chunks, chunk.start + size_words, tail_size_words);
                }

                return chunk.start;
            }
        }
    }

    return NULL;
}

void free(void *ptr)
{
    if (ptr != NULL) {
        const int index = chunk_list_find(&alloced_chunks, ptr);
        assert(index >= 0);
        assert(ptr == alloced_chunks.chunks[index].start);
        chunk_list_insert(&freed_chunks,
                          alloced_chunks.chunks[index].start,
                          alloced_chunks.chunks[index].size);
        chunk_list_remove(&alloced_chunks, (size_t) index);
    }
}

static void mark_region(const uintptr_t *start, const uintptr_t *end)
{
    for (; start < end; start += 1) {
        const uintptr_t *p = (const uintptr_t *) *start;
        for (size_t i = 0; i < alloced_chunks.count; ++i) {
            Chunk chunk = alloced_chunks.chunks[i];
            if (chunk.start <= p && p < chunk.start + chunk.size) {
                if (!reachable_chunks[i]) {
                    reachable_chunks[i] = true;
                    mark_region(chunk.start, chunk.start + chunk.size);
                }
            }
        }
    }
}

void heap_collect()
{
    const uintptr_t *stack_start = (const uintptr_t*)__builtin_frame_address(0);
    memset(reachable_chunks, 0, sizeof(reachable_chunks));
    mark_region(stack_start, stack_base + 1);

    to_free_count = 0;
    for (size_t i = 0; i < alloced_chunks.count; ++i) {
        if (!reachable_chunks[i]) {
            assert(to_free_count < CHUNK_LIST_CAP);
            to_free[to_free_count++] = alloced_chunks.chunks[i].start;
        }
    }

    for (size_t i = 0; i < to_free_count; ++i) {
        free(to_free[i]);
    }
}
```

<br>

#### Memory Model.

First, we define a simulated heap:

```c
uintptr_t heap[HEAP_CAP_WORDS] = {0};
```

A statically allocated block of memory initialize to 0, each slot is *uintptr_t* type (unsigned integer of the size of a pointer), this ensure the heap is alligned to pointer-size boundaries. 

More specifically, this change is done in order to gain consistency with the *heap_collect()* function's logic. The GC will sweep chunks of memory both from the stack and the heap (recursive search to reachable chunks) in search of memory references to alloced chunks, thus, we force the slots of the heap to be word's size (changing the type from *char* to *uintptr_t*) in order to make the sweep go 'word to word' intead of 'byte to byte'.

Let's considet a heap array sorted byte to byte:
<br>

```less
       HEAP       
|----------------|
| 0x61cc15965060 | 
| 0x61cc15965061 | 
| 0x61cc15965062 |
| 0x61cc15965063 |
| 0x61cc15965064 |
| 0x61cc15965065 |
| 0x61cc15965066 |
| 0x61cc15965067 |
|----------------| WORD / 8-bytes
| 0x61cc15965068 |
| 0x61cc15965069 |
| 0x61cc1596506a |
| 0x61cc1596506b |
| 0x61cc1596506c |
| 0x61cc1596506d |
| 0x61cc1596506e |
| 0x61cc1596506f |
|----------------| WORD / 8-bytes
```

Sweep it getting 8 contiguous bytes at a time would consist to check the following regions after one another till the end of the heap:  

```less
[0x61cc15965060, 0x61cc15965068] --> [0x61cc15965061, 0x61cc15965069] --> [0x61cc15965062, 0x61cc1596506a] ...
```

Moving the region one byte size through the heap at each iteration.

When we change the type of the heap, we force the data to be stored in memory address multiples of the word size (this is related with how the processor accesses memory; word size at time), at the start of each slot, (which also are of the size of a pointer) so when we go word by word sweeping the heap we can safely assume there isn't pointers between slots.

```less
|-------------------------------------|
| WORD @ 0x61cc15965060 → 0x60 - 0x67 |
|-------------------------------------|
| WORD @ 0x61cc15965068 → 0x68 - 0x6f |
|-------------------------------------|
| WORD @ 0x61cc15965070 → 0x70 - 0x77 |
|-------------------------------------|
| WORD @ 0x61cc15965078 → 0x78 - 0x7f |
|-------------------------------------|
| WORD @ 0x61cc15965080 → 0x80 - 0x87 |
|-------------------------------------|
| WORD @ 0x61cc15965088 → 0x88 - 0x8f |
|-------------------------------------|
| WORD @ 0x61cc15965090 → 0x90 - 0x97 |
|-------------------------------------|
| WORD @ 0x61cc15965098 → 0x98 - 0x9f |
|-------------------------------------|
| WORD @ 0x61cc159650a0 → 0xa0 - 0xa7 |
|-------------------------------------|
| WORD @ 0x61cc159650a8 → 0xa8 - 0xaf |
|-------------------------------------|
| WORD @ 0x61cc159650b0 → 0xb0 - 0xb7 |
|-------------------------------------|
| WORD @ 0x61cc159650b8 → 0xb8 - 0xbf |
|-------------------------------------|
```

In this esqueme, we can safely check each slot in look for a pointer without check space between slots as we did in the first versión of the heap.

<br>

Also, we set an empty variable for the base of the stack, this will be important in the malloc() function and the implementation of the Garbage Collector:

```c
const uintptr_t *stack_base = 0;
```

<br>

#### Data Structures.

Then, we define two structures:

```c
typedef struct {
    uintptr_t *start;
    size_t size;
} Chunk;
```

- A representation a region of memory which can be allocated; pointer and size of the chunk.

```c
typedef struct {
    size_t count;
    Chunk chunks[CHUNK_LIST_CAP];
} Chunk_List;
```

- A list (array) of chunks used to track, allocated, freed o temporary memory chunks.

<br>

#### Heap allocation logic.

This function; *malloc()* is barely a replication from the previous heap.c heap_alloc() function, with several changes:


```c
void *malloc(size_t size_bytes)
{
    stack_base = (const uintptr_t*)__builtin_frame_address(1);
    const size_t size_words = (size_bytes + sizeof(uintptr_t) - 1) / sizeof(uintptr_t);

    if (size_words > 0) {
        chunk_list_merge(&tmp_chunks, &freed_chunks);
        freed_chunks = tmp_chunks;

        for (size_t i = 0; i < freed_chunks.count; ++i) {
            const Chunk chunk = freed_chunks.chunks[i];
            if (chunk.size >= size_words) {
                chunk_list_remove(&freed_chunks, i);

                const size_t tail_size_words = chunk.size - size_words;
                chunk_list_insert(&alloced_chunks, chunk.start, size_words);

                if (tail_size_words > 0) {
                    chunk_list_insert(&freed_chunks, chunk.start + size_words, tail_size_words);
                }

                return chunk.start;
            }
        }
    }

    return NULL;
}
```

As the previous function, this function allocates memory from a manually managed heap requesting only the size in bytes. It looks through the list of free chunks, merge contigous blocks, finds one that’s big enough, splits it if necessary, adds the new chunk to the allocated list and returns a pointer to the start position of the chunk.

However, it have suitable changes:

- Change type from bytes to word in order to fit with the pointer alignment strategy mentioned above.

    ```c
    const size_t size_words = (size_bytes + sizeof(uintptr_t) - 1) / sizeof(uintptr_t);
    ```
   

- Added a value for the variable stack_base. We set it to have the value of the stack base of the stack frame of the previous function. If we call malloc through main, the stack base will be setted to the base of the main() function.

    ```c
    stack_base = (const uintptr_t*)__builtin_frame_address(1);// The 1 value is calling the stack base of the caller function.
    ```

<br>

#### Heap free logic

The free() function doesn't have any modifications:

```c
void free(void *ptr){

    if (ptr != NULL) {
        const int index = chunk_list_find(&alloced_chunks, ptr);
        assert(index >= 0);
        chunk_list_insert(&freed_chunks, alloced_chunks.chunks[index].start, alloced_chunks.chunks[index].size);
        chunk_list_remove(&alloced_chunks, (size_t) index);
    }
}
```

If the passed pointer is not null, then we find the chunk referenced by the pointer in the alloced chunks list, we evaluate if the index is not negative and then we insert a new chunk in the freed list and remove the chunk associated to the index from the allocated list.

<br>

#### Chunk list utilities.

The list's operations like insert(), merge() and else doesn't have any changes and persist as in the first version of the code. 

<br>

#### Garbage Collector.

##### Explaining the idea.

As we said in the introduction, the Garbage Collector (GC) is a program that attepts to reclaim memory that was allocated by the program but is no longer referenced (garbage). 

Suppose we have the following code:

```c
int main() {

    int *arr = malloc(100 * sizeof(int));

    int num = 5;
    arr = &num;

    free(arr);

    return 0;
}
```

In the code above first we are defining a pointer and initializing it with a chunk of memory of 100 integers size. Then, we define an integer and store in the pointer *arr* the memory address of the *num* variable, thus, the reference to the 100 integer block is lost and when free is called over *arr*, only the chunk in which num's value is stored gets freed. We said that the 100 integer block is no longer referenced in the program and is unreachable.

In simple terms, the GC would locate all the referenced chunks in the heap, by sweeping the stack and the heap in search of pointers to alloced chunks, and reclaims the rest of the memory (marked as unreachable) as free making it available again for new allocations.

This GC has three major característics:

- Is conservative; guess what might be a pointer.
- Is non-moving; it doesn't relocate objects.
- Mark-and-sweep; marks reachable memory and reclaims the rest.

Our GC gets a memory region, start checking every word within and assume that is a pointer. Then examines if the supposed pointer points to any part of an alloced chunk (for each chunk) and if it is, it marks the chunk as reachable and recursively check the chunk for references to other chunks. At the term of the process, free all not marked as reachable.

<br>

##### Explaining the code.

###### Global variables.

The Garbage Collector is defined through the following globals:

```c
bool reachable_chunks[CHUNK_LIST_CAP] = {0};
void *to_free[CHUNK_LIST_CAP] = {0};
size_t to_free_count = 0;
const uintptr_t *stack_base = 0;
```

The globals function are:

- **reachable_chunks**; An array of boolean values which have the purpouse of mark if an allocated chunk is reachable.
- **to free**; Hold pointers to chunks what will be free.
- **to_free_count**; Counter to track the many chunks to free.
- **stack_base**; Marks the top of the stack.

<br>

###### Functions.

The GC gets triggered in *heap_collect*, this code:

```c
void heap_collect()
{
    const uintptr_t *stack_start = (const uintptr_t*)__builtin_frame_address(0);
    memset(reachable_chunks, 0, sizeof(reachable_chunks));
    mark_region(stack_start, stack_base + 1);

    to_free_count = 0;
    for (size_t i = 0; i < alloced_chunks.count; ++i) {
        if (!reachable_chunks[i]) {
            assert(to_free_count < CHUNK_LIST_CAP);
            to_free[to_free_count++] = alloced_chunks.chunks[i].start;
        }
    }

    for (size_t i = 0; i < to_free_count; ++i) {
        free(to_free[i]);
    }
}
```

First, store the pointer of the start of the stack:

```c
const uintptr_t *stack_start = (const uintptr_t*)__builtin_frame_address(0);
```

First, *__builtin_frame_address(0)* return a void pointer to the current stack frame, this is the stack frame from the current function, which is essentially the last frame introduced in the stack (remember FIFO). We are getting thus, the start of the stack, and, we cast to (const unintptr_t*) since unintptr_t is a variable of the size of a word. This is neccesary because we want to scan the stack word by word.

Then, it marks the memory of the reachable_chunks array to 0 (as unreachable):

```c
memset(reachable_chunks, 0, sizeof(reachable_chunks));
```

We do this even if in the definition of this array we defined all the blocks to 0. This is beacause the array will pass over multiple GC cycles and we need to clean it before every of it.

Then, we call mark_region() function:

```c
static void mark_region(const uintptr_t *start, const uintptr_t *end)
{
    for (; start < end; start += 1) {
        const uintptr_t *p = (const uintptr_t *) *start;
        for (size_t i = 0; i < alloced_chunks.count; ++i) {
            Chunk chunk = alloced_chunks.chunks[i];
            if (chunk.start <= p && p < chunk.start + chunk.size) {
                if (!reachable_chunks[i]) {
                    reachable_chunks[i] = true;
                    mark_region(chunk.start, chunk.start + chunk.size);
                }
            }
        }
    }
}
```

This function considers a memory region '\[start, end\)' and treats each word as a posible pointer (uintptr_t) and check if points to an alloced chunk, if it does, mark it as reachable and scans recursively to find other pointers. 

- First, implement a for loop that goes over *start* to *end* (this is from the current stack frame till the base of the stack as we know from the heap_collect() code) 

    ```c
    for (; start < end; start += 1) //Note that the stack grows downwards, from the base to the current frame (start) so the start have the lowest memory address and we are going upward in memory start += 1 to reache the highest memory address (the base of the stack).
    ```

    Is worth to remember that, applying pointer arithmetics, start += 1, goes iterating word after word, so, we are sweeping the stack word by word.

- Then we assume that the contents of the start pointer is a pointer to *uintptr_t* and check if this pointer is a valid reference to any of the allocated chunks by the program. In order to perform this task we perform the following comparison:

    ```c
    if (chunk.start <= p && p < chunk.start + chunk.size)
    ```

    This is in fact checking if *p* exists within the range of the start and the end of the chunk. Let's think about this carefully. The GC needs to know if a chunk is reachable, originally we can think that this condition is true if *p* is equal to *chunk.start* but this would lead to false negatives since a chunk of memory can have internals positions like fields or elements that can also be referenced in the program. 
    
    Consider for example a structure:

    ```c
    typedef struct {
        int a;
        int b;
    } Pair;
    ```

    We know that a structure is none but a contiguous region of memory that holds several fields, each field can be referenced through memory address, in this case; &Pair->a or &Pair->b. This two memory address are within the range of the start and the end of the chunk and a pointer pointing to it is a valid reference to the chunk. This does not means that in the chunk exists pointers, it means that within the chunk exists memory address that can be referenced by a pointer (from the perspective of the programmer) and must be checked.

    The key is that **we are getting each word's size portion of the stack, considering it as a pointer and then check if his contents (memory address) falls within the memory region of an allocated chunk, for every chunk allocated by the program**, if it is this way, then the pointer holds a memory address to the allocated chunk, or what is the same; the word is a pointer and the pointer contains a reference to the chunk so the chunk is reachable.

    Then, we scan it recursively in search of more pointers to other chunks.
 

- Then, once mark_region() has finished, we evaluate the boolean value of each chunk in reachable_chunk array, mark_region had changed to *true* those that are reachable, if it is false it adds the chunk to the *to_free* list and calls free() for every element of that array efectively freeing the unreachable memory.

To prove this program, we can find the complete code and a poc on the following [repository](https://github.com/qv1ntv5/HeapAlloc-GC)
<br>

