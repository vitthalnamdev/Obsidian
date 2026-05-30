## 1. How `malloc()` and `free()` Work

These are standard C library functions used for **dynamic memory allocation**.

- **`malloc(size_t size)`**: When you call this, the allocator looks at the heap to find a contiguous block of free memory that is at least `size` bytes. If found, it marks that space as "allocated" and returns a pointer to it.
    
- **`free(void *ptr)`**: This tells the allocator that the memory at `ptr` is no longer needed. The allocator puts this chunk back into the "free pool" so it can be reused later.
    

> **Note:** The allocator usually stores a small "header" just before the returned pointer. This header contains the size of the block, so when you call `free()`, the system knows exactly how much space it's taking back.

---

## 2. The Free List

The **Free List** is the data structure used to track available memory. It is often a **linked list** where each node represents a free chunk of memory.

When memory is freed, it's added back to this list. If two adjacent blocks are free, the allocator performs **coalescing**—merging them into one larger block to prevent fragmentation.

---

## 3. Allocation Strategies

When `malloc()` is called, and there are multiple free blocks available, the system must decide which one to pick. Here are the common approaches:

### **First Fit**

- **How it works:** Scans the free list from the beginning and picks the **first** block that is big enough.
    
- **Pros:** Very fast.
    
- **Cons:** Can clutter the beginning of the list with small, useless fragments.
    

### **Best Fit**

- **How it works:** Scans the entire list and picks the **smallest** block that is still large enough for the request.
    
- **Pros:** Minimizes "wasted" space in the chosen block.
    
- **Cons:** Slower (must check everything) and leaves behind tiny, unusable "slivers" of memory.
    

### **Worst Fit**

- **How it works:** Scans the entire list and picks the **largest** available block.
    
- **Pros:** The remaining portion after allocation is usually large enough to be useful for another request.
    
- **Cons:** Slower and quickly uses up the largest blocks needed for big requests.
    

### **Next Fit**

- **How it works:** Similar to First Fit, but it starts searching from where the **last** allocation ended rather than the beginning.
    
- **Pros:** Spreads the allocation more evenly across the heap.
    

---

## 4. Fragmentation: The Main Enemy

- **Internal Fragmentation:** Memory wasted inside an allocated block (e.g., asking for 10 bytes but getting a 16-byte block because that's the minimum size).
    
- **External Fragmentation:** Total free memory is enough to satisfy a request, but it is split into small, non-contiguous pieces, so the request fails.
    

---


To understand how `malloc()` and `free()` manage memory behind the scenes, you have to look at the **invisible metadata** attached to every block of memory.

---

## 1. The Hidden Header

When you call `ptr = malloc(20)`, the allocator actually reserves more than 20 bytes. It carves out extra space (usually 8 or 16 bytes) immediately **before** the pointer it gives you. This is the **Header**.

### What’s inside a Header?

- **Size:** The total size of the block.
    
- **Magic Number/Status:** A flag indicating if the block is "Free" or "Allocated."
    
- **Pointers:** (Only when free) Links to the next/previous free block.
    

When you call `free(ptr)`, the allocator looks at `ptr - sizeof(header)` to find the metadata. This is how `free()` knows exactly how many bytes to reclaim without you telling it the size!

---

## 2. Managing the Free List (Pointers)

The "Free List" isn't a separate area of memory; it’s embedded within the free holes themselves. This is a clever trick called an **embedded linked list**.

1. **When a block is Allocated:** The space is yours to use. The only thing the system keeps is the header.
    
2. **When a block is Freed:** The system transforms that empty space into a node for the linked list. It writes a `next` pointer directly into the space where your data used to be.
    

### The Pointer "Handshake"

- **The Head Pointer:** The allocator maintains a global variable (often called `head`) that points to the first free block in the heap.
    
- **Traversing:** When you call `malloc()`, the allocator follows the `next` pointers: `head -> block1 -> block2` until it finds a block that fits your request.
    
- **Unlinking:** Once a block is chosen, the allocator "unplugs" it from the list by updating the pointers of the surrounding blocks.
    

---

## 3. The Lifecycle: Allocation & Deallocation

### Allocation Step-by-Step

- **Search:** Traverses the free list (using First Fit or Best Fit).
    
- **Splitting:** If a 100-byte free block is found but you only need 20 bytes, the allocator **splits** the block. It creates a new header for the remaining 80 bytes and keeps it in the free list.
    
- **Return:** It returns the address of the memory immediately following the header.
    

### Deallocation Step-by-Step

- **Identity Check:** `free(ptr)` calculates the header position.
    
- **Insertion:** The block is added back to the free list (usually at the front for speed).
    
- **Coalescing (Merging):** The allocator checks if the neighbors of the newly freed block are also free. If the block to the left is free, it merges them into one giant block to prevent **External Fragmentation**.
    

---

## Summary Table

| **Feature**          | **Allocated Block**                    | **Free Block**                      |
| -------------------- | -------------------------------------- | ----------------------------------- |
| **Header**           | Contains Size + "Allocated" flag       | Contains Size + "Free" flag         |
| **Payload**          | Your actual data (strings, ints, etc.) | **Pointers** to the next free block |
| **Pointer returned** | Address of the Payload                 | Not applicable (internal to OS)     |