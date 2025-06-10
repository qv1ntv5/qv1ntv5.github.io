---
layout: post
title: Building my own malloc in C.
subtitle: A malloc code written in C.
tags: [C]
---
### Defining the idea.

In this section we will built a custom malloc() function with also a free() and later a garbage collector.
First, the central idea is that we are implementing a custom memory allocator along with a free function which will allocate and deallocate memory from a static array wich will emulate a Heap.

<br>

#### Static heap array.

First of all, we will present our *heap*. For our matters, we can define a heap as a continous memory block dynamically accesed at runtime by the program to store volatile data. In our code we will use as a heap a static array to *char* with a fixed capacity initialized to zero. 

```c
#define HEAP_CAP 640000
char heap[HEAR_CAP] = {0};
```

Since char datatype's size is one byte, our array is in fact a continous block of free-bytes and this is pretty convinient because this structure respect pointer's arithmetic. 

<br>

<div style="text-align:center">
<img src="{{ 'assets/img/C/image-12.png' | relative_url }}" text-align="center"/>
</div>

<br>

<br>

#### Allocation and Deallocation. Chunk structure and lists.

As we was saying before, code implements a custom memory allocator that manages a fixed pool of memory using two functions.

```c
heap_alloc(size) //Finds a free chunk big enough, marks it as allocated, returns a pointer to the first byte of the allocated chunk.
heap_free(ptr)   //Finds the allocated chunk, moves it back to the freed list.
```

And two data structures:

```c
Chunk_List alloced_chunks; //Tracks allocated memory pieces  
Chunk_List freed_chunks; //Tracks available memory pieces
```

The idea is to track those chunks from the array which are allocated and freed in order to reuse those chunks which are available. In order to be able to do this, we design first a structure of a chunk with two fields that defines a chunk:

```c
typedef struct {
    char *start;
    size_t size;
} Chunk;
```

This structure have two fields, two main caracteristics of a chunk in the heap:

- **start**: A pointer to the first address (the first byte) in which the chunk starts.
- **size**: The size of the chunk.

<br>

<div style="text-align:center">
<img src="{{ 'assets/img/C/image-13.png' | relative_url }}" text-align="center"/>
</div>

<br>

So the idea is to store this information in a slot on a list, in order to be able to efficiently track a chunk. This list would be a 'metadata table' for allocated chunks.

In order to implement this list, we write the following structure:

```c
typedef struct {
    size_t count;
    Chunk chunks[CHUNK_LIST_CAP];
} Chunk_List;
```

This structure have a counter of how many chunks do we have allocated and an array which would be the actual list. 

So this list is built with slots which have the different instance of the chunk structure; a pointer (ptr) and a size (s1) of each chunk. 

We can see this structure like as we picture in the following image:

<br>

<div style="text-align:center">
<img src="{{ 'assets/img/C/image-10.png' | relative_url }}" text-align="center"/>
</div>

<br>

This list will be also applied over the freed chunks. **In summary, we will have two lists, allocated chunks and freed chunks. This will help us to track both allocated and freed chunks in order to be able to deallocated or allocate it again quickly**.

<br>

### Explaining the code.

#### Macros.

The macros of the code are mainly two, with one exception with no importance.:

```c
#define HEAP_CAP 640000 //The number of bytes of our heap.
#define CHUNK_LIST_CAP 1024 //The number of chunks available in our Chunk_list.
```

<br>

#### Global definitions.

At start time, we have the following global definitions in the code:

- Memory pool of 640000 initialize to zero bytes:

```c
char heap[HEAP_CAP] = {0};
```

- Chunk Tracking Lists

```c
Chunk_List alloced_chunks = {0}; //Tracking the alloced chunks. At start-time there isn't any alloced chunk.

Chunk_List freed_chunks = { //Tracking the available chunks, at start-time there is only one available chunk which is the heap itself. 
    
    .count = 1,
    .chunks = { [0]{.start = heap, .size = sizeof(heap)} }, //It consist of an array of one element [0] which have a pointer to the start of our array (heap) and the size of the array

}
```

Is worth to mention that in C, when we define an array, the name of the array defines a pointer to the top of that array. In our case we define an array called *heap*, thus, the *heap* term references a pointer to the starting byte of the array and later asign it to the *start* field of the chunk [0]. 

<br>

#### Operations.

##### Secondary Operations.

We present first the secondary operations such as Insert, Merge, or Remove since this functions are part of main functions such as heap_alloc and heap_free. Is a requirement to have a good understanding of the second one to understand properly the main one and the code it self.

<br>

###### Chunk_list_insert.

Let's consider the *chunk_list_insert*, this function adds a chunk to the chunk list adding to a slot the pointer to the start address of the chunk and the size of the chunk and then it sort the list.

In the following explanation, we might remember that in order to access a field of a structure referenced by a pointer we use arrows instead of dots. 


```c
//This function accepts as arguments the following items; the list of chunks and the pointer and size of the chunk to be insert.
void chunk_list_insert(Chunk_List *list, void *start, size_t size){ 
    
    //Evaluate if there is available space in the chunk list.
    assert(list->count < CHUNK_LIST_CAP);
    
    // Asign to the last slot's fields the pointer and size
    list->chunks[list->count].start = start; 
    list->chunks[list->count].size = size;

    //Sort numerically the array by the pointer.
    for (size_t i = list->count; i > 0 && list->chunks[i].start < list->chunks[i-1].start; --i){
        const Chunk t = list->chunks[i];
        list->chunks[i] = list->chunks[i-1];
        list->chunks[i-1] = t;
    }

    //Expand the occupied slots count of the list.
    list->count += 1;
}
```

Is worth to take an approach on how and why the array gets sorted.

First of all, we decide to sort the array in order to be able to search and find a required chunk in the minnor time possible. So we sort it for efficency purpouses in order to reduce the algorithm time-complexity grade. 

We make an appendix to explain this. 

The array gets sorted in the following way:

```c
//We implement a decreasing loop which goes trough the top of the array and only performs if the pointer is smaller than its subsequent.
for (size_t i = list->count; i > 0 && list->chunks[i].start < list->chunks[i-1].start; --i){

    //Then we interchange the position on the list of both of them.
    const Chunk t = list->chunks[i];
    list->chunks[i] = list->chunks[i-1];
    list->chunks[i-1] = t;
}
```

After the loop we get an array where from the top, the pointers are bigger than the next in list downwards.

<br>

###### Chunk_list_merge.

This function merges chunks from a source list into a destination list, attempting to combine adjacent chunks.

```c

//This function accepts two chunk lists
void chunk_list_merge(Chunk_List *dst, const Chunk_List *src){

    //Set destination list counter to 0.
    dst->count = 0;

    //Then we iterate over every item on the source list.
    for (size_t i = 0; i < src->count; ++i){

        const Chunk chunk = src->chunks[i];
        
        //If destination have elements:
        if (dst->count > 0){

            //Get a pointer of the last chunk in the destination list.
            Chunk *top_chunk = &dst->chunks[dst->count - 1]; 

            //Check if both chunks are adjacents. If are, extend the size of one of them and insert the extended element on the list.
            if (top_chunk->start + top_chunk->size == chunk.start){ 
                top_chunk->size += chunk.size;
            } else {
                chunk_list_insert(dst, chunk.start, chunk.size);
            }

        //If not, insert the element.
        } else {
            chunk_list_insert(dst, chunk.start, chunk.size);
        }
    }
}
```

Let's explain how the code checks if two chunks are adjacent. For this matter, we are using *pointher arithmetics*, in fact, we are performing the following operation:

```c
top_chunk->start + top_chunk->size == chunk.start
```

We known that start is a pointer and size is an integer, so we are adding an integer to a pointer, this is; we are advancing to the pointer *start* position so many offsets as the *size* times the pointer *start* size. 

Thus, since *start* is pointing to a char (remember Heap array) this is one byte size, so we are advancing from the top_chunk's start position, top_chunk's *size* bytes which is in fact the entire top_chunk and we are checking if the resulting offset is equivalent to the position of the start address of the chunk. If it is, is because the start of the chunk is automatically after the end of the top_chunk in the heap.

<br>

<div style="text-align:center">
<img src="{{ 'assets/img/C/image-11.png' | relative_url }}" text-align="center"/>
</div>

<br>

<br>

###### Chunk_list_find.

The next function we gonna see is the *find* function. This function tries to find a pointer in a list. If the serach operation success, the function return the position of the pointer in the array, if fails, return -1. 

```c
//This function accepts as arguments the list to lookup and the pointer to find.
int chunk_list_find(const Chunk_List *list, void *ptr){

    //A simple search algorith which searchs in all items if any of it is equal to the requested pointer.
    for (size_t i = 0; i < list->count; ++i) {
        if (list->chunks[i].start == ptr) {
            return (int) i;
        }
    }
    return -1;
}
```

This function attems to find a position in an array, we will leverage this position as an index to locate chunks and deallocate it.

<br>

###### Chunk_list_remove.

This function removes an item from a list.

```c
//The function acceps as arguments the list to remove the item and the index of the slot in the list to be removed.
void chunk_list_remove(Chunk_List *list, size_t index){
    
    //Evaluates if the index is whithin the range of the available slots.
    assert(index < list->count);

    //From the index position, we asign the current slot the value of the next one till the penultimate. This effectively removes the item from the list and also prevents to form an empty gap in the array.
    for (size_t i = index; i < list->count -1; ++i){
        list->chunks[i] = list->chunks[i + 1];
    }

    //Then we quit one to the ocuppied slots counter.
    list->count -= 1;
}
```

Is worth to note that is very important to keep the array from free-gaps in order to preserve the algorithm logic simple.

<br>

##### Main operations

###### Heap_alloc.

First, our alloc memory, this functiion gets a size in bytes and returns a pointer to the allocated memory or NULL if the allocation fails:

```c
void *heap_alloc(size_t size){

    //Prevent to allocate zero-size chunks.
    if (size > 0) {

        //Merge chunks from freed_chunks to avoid fragmentation. (Merge freed_chunks into tmp_chunks and replace).
        chunk_list_merge(&tmp_chunks, &freed_chunks); 
        freed_chunks = tmp_chunks; 

        //Search for a suitable chunks.
        for (size_t i = 0; i < freed_chunks.count; ++i){ 
            const Chunk chunk = freed_chunks.chunks[i];

            //We get a chunk of memory with just the requested size.
            if (chunk.size >= size){

                //Remove from available chunks.
                chunk_list_remove(&freed_chunks, i);

                //We incorporate a new allocated chunk with the requested size and, if there is more size available from the selected chunk we create a new chunk in the available list.
                const size_t tail_size = chunk.size - size;
                chunk_list_insert(&alloced_chunks, chunk.start, size);
                
                if (tail_size > 0){
                    chunk_list_insert(&freed_chunks, chunk.start + size, tail_size);
                }
                return chunk.start;
            }
        }
    }
    return NULL;
}
```

<br>

###### heap_free.

The following code free chunks, it finds the pointer in the alloced list, then insert the chunk in freed list and remove from the alloced one. 

```c
void heap_free(void *ptr){

    if (ptr != NULL) {
        const int index = chunk_list_find(&alloced_chunks, ptr);
        assert(index >= 0);
        chunk_list_insert(&freed_chunks, alloced_chunks.chunks[index].start, alloced_chunks.chunks[index].size);
        chunk_list_remove(&alloced_chunks, (size_t) index);
    }
}
```

<br>

### Complete Code.

Lets consider the following code.

```c
#include <assert.h>
#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>

#define HEAP_CAP 640000
#define CHUNK_LIST_CAP 1024

typedef struct {
    char *start;
    size_t size;
} Chunk;

typedef struct {
    size_t count;
    Chunk chunks[CHUNK_LIST_CAP];
} Chunk_List;

void chunk_list_insert(Chunk_List *list, void *start, size_t size){
    assert(list->count < CHUNK_LIST_CAP);
    list->chunks[list->count].start = start;
    list->chunks[list->count].size = size;
    for (size_t i = list->count; i > 0 && list->chunks[i].start < list->chunks[i-1].start; --i){
        const Chunk t = list->chunks[i];
        list->chunks[i] = list->chunks[i-1];
        list->chunks[i-1] = t;
    }
    list->count += 1;
}

void chunk_list_merge(Chunk_List *dst, const Chunk_List *src){

    dst->count = 0;

    for (size_t i = 0; i < src->count; ++i){

        const Chunk chunk = src->chunks[i];
        
        if (dst->count > 0){
            Chunk *top_chunk = &dst->chunks[dst->count - 1];

            if (top_chunk->start + top_chunk->size == chunk.start){
                top_chunk->size += chunk.size;
            } else {
                chunk_list_insert(dst, chunk.start, chunk.size);
            }
        } else {
            chunk_list_insert(dst, chunk.start, chunk.size);
        }
    }
}

void chunk_list_dump(const Chunk_List *list){

    printf("Chunks (%zu):\n", list->count);
    for (size_t i = 0; i < list->count; ++i){
        printf("    start: %p, size: %zu\n", list->chunks[i].start, list->chunks[i].size);
    }
}

int chunk_list_find(const Chunk_List *list, void *ptr){

    for (size_t i = 0; i < list->count; ++i) {
        if (list->chunks[i].start == ptr) {
            return (int) i;
        }
    }
    return -1;
}


void chunk_list_remove(Chunk_List *list, size_t index){

    assert(index < list->count);
    for (size_t i = index; i < list->count -1; ++i){
        list->chunks[i] = list->chunks[i + 1];
    }
    list->count -= 1;
}
  
char heap[HEAP_CAP] = {0};

Chunk_List alloced_chunks = {0};
Chunk_List freed_chunks = {
    .count = 1,
    .chunks = { [0]{.start = heap, .size = sizeof(heap)} },
};

Chunk_List tmp_chunks = {0};

void *heap_alloc(size_t size){

    if (size > 0) {
        chunk_list_merge(&tmp_chunks, &freed_chunks);
        freed_chunks = tmp_chunks;

        for (size_t i = 0; i < freed_chunks.count; ++i){
            const Chunk chunk = freed_chunks.chunks[i];
            if (chunk.size >= size){
                chunk_list_remove(&freed_chunks, i);

                const size_t tail_size = chunk.size - size;
                chunk_list_insert(&alloced_chunks, chunk.start, size);
                
                if (tail_size > 0){
                    chunk_list_insert(&freed_chunks, chunk.start + size, tail_size);
                }

                return chunk.start;
            }
        }
    }
    return NULL;
}

void heap_free(void *ptr){

    if (ptr != NULL) {
        const int index = chunk_list_find(&alloced_chunks, ptr);
        assert(index >= 0);
        chunk_list_insert(&freed_chunks, alloced_chunks.chunks[index].start, alloced_chunks.chunks[index].size);
        chunk_list_remove(&alloced_chunks, (size_t) index);
    }
}

#define N 10

void *ptrs[N] = {0};

int main(){

    for (int i = 0; i < N; ++i){
        ptrs[i] = heap_alloc(i);
    }

    for (int i = 0; i < N; ++i){
        if (i % 2 == 0){
            heap_free(ptrs[i]);
        }
    }

    heap_alloc(10);

    chunk_list_dump(&alloced_chunks);
    chunk_list_dump(&freed_chunks);
}
```

<br>