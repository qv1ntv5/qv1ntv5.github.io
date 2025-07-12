---
layout: post
title: Hashtables.
subtitle: Explanation of hashtables in C.
tags: [C]
---

### HashTables

#### Presentation of the problem.

Let's suppose we have the following exercise: Given a roman numeral, convert it to an integer.

The roman numbers are built through a finite set of simbols (Numerals) that have each equivalent decimal number as the following table shows:

| **Roman Numeral** | **Integer Value** | 
| - | - |
| I | 1 | 
| V | 5 |
| X | 10 |
| L | 50 |
| C | 100 |
| D | 500 |
| M | 1000 |

Thus, this table can be represented in C through a *hashtable*, like the following:

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#define TABLE_SIZE 7

typedef struct Entry {
    char key;
    int value;
    struct Entry* next;
} Entry;

typedef struct {
    Entry* buckets[TABLE_SIZE];
} HashTable;

unsigned int hash(char key) {
    return key % TABLE_SIZE;
}

HashTable* createHashTable() {
    HashTable* table = malloc(sizeof(HashTable));
    for (int i = 0; i < TABLE_SIZE; i++) {
        table->buckets[i] = NULL;
    }
    return table;
}

void insert(HashTable* table, char key, int value) {
    unsigned int index = hash(key);
    Entry* entry = malloc(sizeof(Entry));
    entry->key = key;
    entry->value = value;
    entry->next = table->buckets[index];
    table->buckets[index] = entry;
}

int get(HashTable* table, char key) {
    unsigned int index = hash(key);
    Entry* entry = table->buckets[index];
    
    while (entry != NULL) {
        if (entry->key == key) {
            return entry->value;
        }
        entry = entry->next;
    }
    return -1;
}

void initRomanTable(HashTable* table) {
    insert(table, 'I', 1);
    insert(table, 'V', 5);
    insert(table, 'X', 10);
    insert(table, 'L', 50);
    insert(table, 'C', 100);
    insert(table, 'D', 500);
    insert(table, 'M', 1000);
}

void freeHashTable(HashTable* table) {
    for (int i = 0; i < TABLE_SIZE; i++) {
        Entry* entry = table->buckets[i];
        while (entry != NULL) {
            Entry* temp = entry;
            entry = entry->next;
            free(temp);
        }
    }
    free(table);
}

int romanToInt(HashTable* romanTable, char *string){

    int total = 0;
    for(int i = 0; i < strlen(string); i++){
        int integer = get(romanTable, string[i]);
        total += integer;
    }

    return total;
}

int main() {
    HashTable* romanTable = createHashTable();
    initRomanTable(romanTable);
    
    //For simplicity, we assume that the roman number is at maximum 5 roman numerals.
    char* romannumptr = (char*) malloc(sizeof(char)*5);

    printf("Please, introduce a roman number, for example: LXVVI.\n");
    scanf("%s", romannumptr);
   
    int romaninteger =  romanToInt(romanTable, romannumptr);
    
    printf("The roman number %s is equivalent to %d integer.\n", romannumptr, romaninteger);
    freeHashTable(romanTable);
    return 0;
}
```

<br>

#### Hashtable definition.

A 'hashtable' in C is an object which stores data in an associative way allowing fast access over the data, average O(1). Let's deep how did we define our hash table to resolve this exercise. 

Essentially, **a hashtable in C consist in a large array of pointers to structures that holds a key, a value, and a pointer to other entry in order to deal with collisions**.

In simple terms, **a hashtable is an array of linked lists** (see [C's linked lists](https://www.learn-c.org/en/Linked_lists)); is a fast lookup structure that uses a hash function to convert keys into array indices, and uses linked lists at each index to store entries that happen to collide.

<br>

#### Structures.

First, we define the Entry structure:

```c
typedef struct Entry {
    char key; //This will store the roman char.
    int value; //This will be the integer.
    struct Entry* next; //A pointer to other Entry object. Necesary to handle collision as we will see.
} Entry;
```

Our structure defines all our components, the key, the value and a pointer to Entry to handle collisions. 

Now we define an array of Entry pointers. This is in fact our HashTable; an array of pointers to Entry structures:

```c
typedef struct {
    Entry* buckets[TABLE_SIZE];
} HashTable;
```

Later we will see that, in order to initilize our hashtable we will use a pointer to our hashtable.

<br>

#### Hash function. Chars and integer promotion.

Now we create a Hash function. This function associate a key to an array index, this works through the *module* function although it could be more deep:

```c
unsigned int hash(char key){
    printf(" Hash function: '%c' (ASCII %d) -> bucket %d\n", key, (int)key, key % TABLE_SIZE);
    return key % TABLE_SIZE;
}
```

Let's take a deeper look in this function. The function bassically performs a mathematical operation with a *char* data type and an *int*.

This is posible because two mainly C considerations:

- First, precisely *char* data type, for C processor, is virtually a small integer that gets associated to a character through the ASCII table. This means that *char* is first a number and when gets printed it transforms through the ASCII table to a char, so there is no problem to treat it as an integer.

- Second, since C performs something called *Integer Promotion*, this is; when a mathematical operation is performed between two variables of different data typers, the smaller data type gets promoted to the data type of the bigger variable, in this case, a *char* gets promoted to *int* and now operation is allowed.

<br>

#### Creating the hashtable.

With this elements we now are able to create our hashtable.

```c
HashTable* createHashTable() {
    printf("Creating hash table with %d buckets...\n", TABLE_SIZE);
    //Declaration of the hashtable
    HashTable* table = malloc(sizeof(HashTable));
    
    // Initialize all buckets to NULL (empty)
    for (int i = 0; i < TABLE_SIZE; i++) {
        table->buckets[i] = NULL;
    }
    
    printf("Hash table created! All buckets initialized to NULL.\n\n");
    return table;
}
```

It's important to note that our Hashtable has been initialice as a pointer instead of a straight initialization, is like initiate an *int* referencing a pointer:

```c
int* varptr = (int*) malloc(sizeof(int));
varptr* = 65;
```

This have several benefits:

In this way, the hashtable gets allocated in dynamic memory allocation, which survives until the termination of the *createHashTable()* function. If we return directly the array instead, we would copy the structure out of the function which is less eficient, returning the pointer omly manipulates the address and remains intact the structure.

Also, this allow us to pass the structure by reference which allows modification of the structure it self and is faster than passing, again, a copy of the structure.

*In summary, when you create a large structure that will be accessed or modified in multiple places, it's more efficient and practical to work with a pointer to its memory address (i.e., pass around the pointer) rather than copying the entire structure each time.*

In fact, always you want to pass a value you want to modify to a function in C, you pass this value as a reference, this is, pass the address memory where the value is stored. This is in order to allow access to the actual value, otherway; a copy of this value is transfered and then the function does not modify the real value.

<br>

#### Inserting Entries and handling collisions.

The *hash function* however have a subtle gap: **collision**. This means, different keys gets, through the hash function, the same index^, for example:

```
15 % 5 = 0
25 % 5 = 0
```

Let's see how handle this situation. Lets consider the code of the following function:

```c
//This function associate a pair key-value on a hashtable. (Check that, since we are modifying a structure, we are passing it by reference; pointer)
void insert(HashTable* table, char key, int value) {
    printf("Inserting: '%c' -> %d\n", key, value);
    
    //Calculate association with hash key to obtain an index.
    unsigned int index = hash(key);
    
    //Create a new slot in the array of the hashtable.
    Entry* newEntry = malloc(sizeof(Entry));
    newEntry->key = key;
    newEntry->value = value;
    
    //And then insert the entry pointer in the array.
    newEntry->next = table->buckets[index];  
    table->buckets[index] = newEntry;        

    //Check if there is a collision.
    printf("  Inserted at bucket[%d]\n", index);
    if (newEntry->next != NULL) {
        printf("  COLLISION! Chained with existing entry.\n");
    }
    printf("\n");
}
```

The function attemps to include a new entry in the array of the hashtable. 

- At first calculates the *index* based on the key and the hash function.

    ```c
    unsigned int index = hash(key);
    ```

<br>

- Gets the *key* and the *value* data passed by argument and define a new Entry with those values. 

    ```c
    Entry* newEntry = malloc(sizeof(Entry));
    newEntry->key = key;
    newEntry->value = value;
    ```

<br>

- Then insert this Entry in the array by making the *next* pointer to point to the bucket to be inserted (we will explain this in a moment) and then assign the array with the Entry pointer.

    ```c
    newEntry->next = table->buckets[index];  
    table->buckets[index] = newEntry;
    ```

    This last step efectively avoids confusion about entries with the same index since we nest the pairs key-value buckets making use of the *next* pointer. 
    
    What we do essentially is to make *next* pointer to point to the array at the *index* slot's content:
    
    ```c
    newEntry->next = table->buckets[index];
    ```

    And then introduce *newEntry* to the bucket effectively displacing the previous entry.
    
    ```c
    table->buckets[index] = newEntry;
    ```
    
    Thanks to the first step, the *next* pointer of *newEntry* now points whatever was in the entry previously and we chained two pairs effectively avoiding collision.

    In order to see this grafically, suppose we have the following key-value pairs: **A-1**, **C-10**, **D-15**. Then, when we call the function to insert each entry on the table, first; the hash function resolves the index (0,1,2), then assign the address of the empty bucket to the *next* pointer of the Entry structure resulting in *next* pointing to NULL pointer, and then asign each Entry to each slot considering each index:

    ```less
    table->buckets:
    [0] → [ A | 1  | NULL ]
    [1] → [ C | 10 | NULL ]
    [2] → [ D | 15 | NULL ]
    ```

   Now, suppose we want to one pair more to the array: B-7. 
   
   We call the function and resolves the new key-pair's index: 1, we have a collision with C-10 pair. 
   
   So, the function follow the steps: asign to *next* the address within bucket \[1\] (newEntry->next = table->buckets\[index\]) and then introduce the address of the B-7 pair's Entry on the same (table->buckets\[index\] = newEntry). 
   
   Now, *next* pointer of B-7 points to the previous address existing in bucket \[1\] (C-10) and we succesfully difference between the two different entries with the same index:

    ```less
    table->buckets:
    [0] → [ A | 1  | NULL ]
    [1] → [ B | 7  | ->(C-10) ] → [ C | 10 | NULL ]
    [2] → [ D | 15 | NULL ] 
    ```

<br>

- As a last step. we check where *next* points to. If *next* is pointing to a non null object, it means that a collision have been avoided.

    ```c
    if (newEntry->next != NULL) {
        printf("  COLLISION! Chained with existing entry.\n");
    }
    ```

<br>

<div style="text-align:center">
<img src="{{ 'assets/img/C/hashtable.png' | relative_url }}" text-align="center"/>
</div>

<br>

#### Getting entries from the hash table.

Now that we the have the elements identified and sorted, lets define a function to search and get an element in the table. Lets see the following function:

```c
//This functions search a 'key' on a 'hashtable'.
int get(HashTable* table, char key) {
    printf("Searching for: '%c'\n", key);
    
    //Calculate index bucket to look in
    unsigned int index = hash(key);
    
    //Store the contents of the index's bucket in an entry
    Entry* current = table->buckets[index];
    
    //Go through the nested list of the bucket till find the value of the request key.
    int steps = 0;
    while (current != NULL) {
        steps++;
        printf("  Step %d: Checking entry with key '%c'\n", steps, current->key);
        
        if (current->key == key) {
            printf("  FOUND! Value: %d\n\n", current->value);
            return current->value;
        }
        
        current = current->next;  //Move to next entry in chain
    }
    
    //If the loop exit, we didn't find the key.
    printf("  NOT FOUND!\n\n");
    return -1;  //Key not found
}
```

We get the index associated with the requested key and explore the nested list of the index's bucket untill find the key and return its value.

<br>

#### Freeing Hashtable.

As a last step, is important to free all the pointers involved with the hashtable.

```c
//Free al the pointers involved with 'table'.
void freeHashTable(HashTable* table) {

    //For every entry, we free the pointer of the slot and the linked list.
    for (int i = 0; i < TABLE_SIZE; i++) {
        Entry* entry = table->buckets[i];
        while (entry != NULL) {
            Entry* temp = entry;
            entry = entry->next;
            free(temp);
        }
    }
    free(table);
}
```

### Appliying Hashtables to the exercise.

Until here we almost see the basics of hashtables in general terms. Now, lets see how can we apply the concepts presented above to deal with the roman numeral's problem.

<br>

#### Inserting roman numerals in the hashtable.

First, having the different structures presented above defined, then we start to insert our items (the roman numerals):

```c
void initRomanTable(HashTable* table) {
    insert(table, 'I', 1);
    insert(table, 'V', 5);
    insert(table, 'X', 10);
    insert(table, 'L', 50);
    insert(table, 'C', 100);
    insert(table, 'D', 500);
    insert(table, 'M', 1000);
}
```

With this, we have a totally functional code, the main function can look like follows:

```c
int main() {
    HashTable* romanTable = createHashTable();
    initRomanTable(romanTable);
    
    char* romannumptr = (char*) malloc(sizeof(char)); 
    printf("Please, introduce a roman numeral: I, V, X, L, C, D, M.\n");
    scanf("%c", romannumptr);

    int romaninteger = get(romanTable, *romannumptr);
    
    printf("The roman numeral %c is equivalent to %d integer.\n", *romannumptr, romaninteger);
    freeHashTable(romanTable);
    return 0;
}
```

We initiate the hashtable, insert the pair key-values and we reclaim a value of a romannumeral request to the user. 

But, to be fair, this is not practical, since to use the code like this we could implemented a case scenario. The reason why we implemented this code is because we want to prevent efficiency in code in front of heavy requests of composite roman numerals (LXXVI, for example).

<br>

#### Reading roman numbers.

We know that roman numbers are combinations of roman numberals which are read by adding the integer associated to the roman numeral from left to right.

```less
LXVVI = 50 + 10 + 5 + 5 + 1 
```

So, we can build a more useful function that, with a given roman number, it gathers as a string, pass each char as a key to the function *get* to retrieve the integer and add the integer to the amount in every iteration.

```c
int romanToInt(HashTable* romanTable, char *string){

    int total = 0;
    for(int i = 0; i < strlen(string); i++){
        int integer = get(romanTable, string[i]);
        total += integer;
    }

    return total;
}
```

And we implement in our main as follows:

```c
int main() {
    HashTable* romanTable = createHashTable();
    initRomanTable(romanTable);
    
    //For simplicity, we assume that the roman number is at maximum 5 roman numerals.
    char* romannumptr = (char*) malloc(sizeof(char)*5);

    printf("Please, introduce a roman number, for example: LXVVI.\n");
    scanf("%s", romannumptr);
   
    int romaninteger =  romanToInt(romanTable, romannumptr);
    
    printf("The roman number %s is equivalent to %d integer.\n", romannumptr, romaninteger);
    freeHashTable(romanTable);
    return 0;
}
```