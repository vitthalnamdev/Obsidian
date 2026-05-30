Since you are exploring low-level design and OS concepts, understanding `malloc` and `free` is critical. These are the manual gear-shifts of memory management in C (and the foundation of C++'s `new`/`delete`).

They reside in the `<stdlib.h>` library and interact directly with the **Heap**.

---

### 1. `malloc` (Memory Allocation)

`malloc` stands for **M**emory **ALLOC**ation. It asks the operating system (via the allocator) to reserve a block of contiguous memory on the Heap.

**Prototype:**

C

```
void* malloc(size_t size);
```

**Key Properties:**

1. **Void Pointer Return:** It returns a `void*` (generic pointer). You must cast it to the type you need (e.g., `int*`, `struct Node*`) so the compiler knows how to handle the data math.
    
2. **Uninitialized Data:** The memory it gives you contains "garbage" (whatever random bits were left there by the previous user). It does _not_ clear the memory (unlike `calloc`).
    
3. **Returns NULL on Failure:** If the Heap is full (or you ask for too much RAM), it returns `NULL`. You must always check for this.
    
4. **Byte-based:** It takes a size in **bytes**, not "number of items." You typically use the `sizeof()` operator.
    

**Example:**

C

```
// Request space for one integer (typically 4 bytes)
int* ptr = (int*)malloc(sizeof(int));

if (ptr == NULL) {
    // Handle error (heap out of memory)
}

*ptr = 100; // Use the memory
```

---

### 2. `free` (Memory Deallocation)

`free` tells the allocator that you are done with a specific block of memory, allowing it to be reused by future `malloc` calls.

**Prototype:**

C

```
void free(void* ptr);
```

**Key Properties:**

1. **Recycling, Not Erasing:** `free` does not necessarily "wipe" the data (set it to zero). It just marks that address in the allocator's bookkeeping list as "Available."
    
2. **Requires the Original Pointer:** You must pass the **exact** pointer address that `malloc` returned. Passing a pointer to the _middle_ of a malloc'd block causes undefined behavior.
    
3. **Under the Hood (Metadata):** How does `free(ptr)` know how many bytes to delete if you don't tell it the size?
    
    - _Secret:_ When `malloc` allocates 10 bytes, it actually allocates slightly more (e.g., 14 bytes). It stores the size (10) in a hidden header _just before_ the pointer it gives you. `free` looks at this hidden header to know how much memory to reclaim.
        

---

### 3. Common Errors (The Danger Zone)

Manual memory management is powerful but dangerous. Here are the specific errors that occur most frequently.

#### A. Memory Leak

This happens when you allocate memory but lose the pointer to it before calling `free`. The memory remains "occupied" until your program terminates, eating up RAM.

C

```
void memoryLeak() {
    int* ptr = (int*)malloc(sizeof(int));
    *ptr = 10;
    
    // ERROR: Function returns without calling free(ptr).
    // The 4 bytes are still reserved on the heap, but we lost the address.
    return; 
}
```

#### B. Dangling Pointer

This occurs when you `free` a pointer, but you keep using that pointer afterwards. The pointer still points to the same memory address, but that address now belongs to the OS (or has been re-assigned to a different variable).

C

```
void danglingPointer() {
    int* ptr = (int*)malloc(sizeof(int));
    *ptr = 50;

    free(ptr); // Memory is returned to system

    // ERROR: ptr is "dangling". It points to memory we don't own.
    // This might crash, or worse, overwrite someone else's new data.
    printf("%d", *ptr); 
}
```

- **Fix:** Always set `ptr = NULL` immediately after freeing it.
    

#### C. Double Free

Trying to free the same memory twice. This corrupts the allocator's internal list of "free blocks," often leading to immediate crashes.

C

```
void doubleFree() {
    int* ptr = (int*)malloc(sizeof(int));
    
    free(ptr); 
    free(ptr); // ERROR: The allocator gets confused and crashes.
}
```

#### D. Heap Fragmentation

This isn't a crash, but a performance degradation. If you frequently `malloc` and `free` blocks of different sizes, the Heap becomes like Swiss cheese—lots of total free memory, but no single contiguous hole big enough for a large allocation.

### Summary Table

|**Feature**|**malloc**|**free**|
|---|---|---|
|**Input**|Size in bytes (`size_t`)|Pointer (`void*`)|
|**Output**|Pointer to first byte (or NULL)|None (`void`)|
|**Memory State**|Contains garbage (random values)|Data persists until overwritten|
|**Major Risk**|Forgetting to check for NULL|Double free or Dangling pointer|

**A Note for C++:**

Since you code in C++, you likely use `new` and `delete`.

- `new` calls `malloc` internally but **also** calls the Constructor (initializes the object).
    
- `delete` calls the Destructor (cleans up resources) and **then** calls `free`.
    
    Using `malloc` in C++ creates the memory but **skips** the constructor, which is why `new` is preferred for objects.