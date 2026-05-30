**Paging** is a memory management scheme that eliminates the need for contiguous allocation of physical memory. This scheme permits the physical address space of a process to be non-contiguous.

## The Core Concept

In paging, the logical (virtual) address space is split into fixed-size blocks called **pages**. Physical memory is split into fixed-size blocks of the same size called **frames**. When a process is to be executed, its pages are loaded into any available memory frames from the backing store (hard disk).

---

## Dividing the Virtual Address

When the CPU generates a logical address, it doesn't "see" physical RAM directly. Instead, the address is treated as a binary number divided into two distinct parts:

1. **Page Number ($p$):** Used as an index into a **page table**. The page table contains the base address of each page in physical memory (the frame number).
    
2. **Page Offset ($d$):** Combined with the base address to define the physical memory address that is sent to the memory unit.
    

If the size of the logical address space is $2^m$ and the page size is $2^n$ addressing units (like bytes), then the high-order $m-n$ bits of a logical address designate the page number, and the $n$ low-order bits designate the page offset.

---

## Address Translation: Virtual to Physical

The **Memory Management Unit (MMU)** is the hardware device that handles this translation. Here is the step-by-step process of how the CPU identifies the physical address:

- **Step 1:** The CPU generates a virtual address consisting of $p$ (page number) and $d$ (offset).
    
- **Step 2:** The MMU uses the page number ($p$) to index into the **Page Table**.
    
- **Step 3:** The Page Table entry provides the corresponding **Frame Number** ($f$) in physical memory.
    
- **Step 4:** The Physical Address is constructed by appending the offset ($d$) to the Frame Number ($f$).
    
- **Step 5:** The actual hardware memory location is accessed using this calculated physical address.
    

### Why use an Offset?

The offset remains identical in both the virtual and physical addresses. Think of the **Page Number** as the "Building Number" and the **Offset** as the "Room Number." Even if you move the entire office to a different building (a different frame), the room number (offset) within that building stays the same.

---

## Summary Table

|**Component**|**Description**|
|---|---|
|**Page**|A fixed-size block of logical memory.|
|**Frame**|A fixed-size block of physical memory.|
|**Page Table**|A data structure used by the MMU to store the mapping between pages and frames.|
|**MMU**|Hardware that converts virtual addresses to physical addresses at runtime.|



Since the page table can be quite large, it isn't stored in CPU registers. Instead, it resides in **main memory (RAM)**, and the CPU uses a special register called the **Page Table Base Register (PTBR)** to point to its starting location.

---

## How the Page Table is Stored

Storing the table in RAM means every memory access theoretically requires _two_ hits: one for the page table entry and one for the actual data. To solve this, CPUs use a **TLB (Translation Lookaside Buffer)**—a high-speed cache that stores recently used mappings.

### Page Table Entry (PTE) Structure

A Page Table Entry doesn't just store the frame number; it contains several "control bits" that help the Operating System manage memory efficiently:

- **Frame Number:** The actual address of the physical memory block.
    
- **Present/Absent Bit (V):** Indicates if the page is currently in RAM or on the disk (Virtual Memory).
    
- **Protection Bit:** Defines access rights—**Read-only**, **Read-Write**, or **Execute**. This prevents a process from writing to its code segment.
    
- **Dirty Bit (Modified Bit):** Set by hardware whenever the page is written to. If the OS needs to swap this page out to disk, it checks this bit. If it's "0," the page hasn't changed, and the OS can just discard it. If it's "1," the OS must write the changes back to the disk.
    
- **Referenced Bit:** Set whenever the page is accessed (read or write). This helps the OS decide which pages to evict during page replacement (e.g., in the LRU algorithm).
    

---

## Pseudo Code: Virtual to Physical Translation

This logic represents how the **Memory Management Unit (MMU)** handles a memory request.

Python

```
def get_physical_address(virtual_address):
    # 1. Split the virtual address into Page Number and Offset
    # Assume 4KB page size (12 bits for offset)
    page_number = virtual_address >> 12
    offset = virtual_address & 0xFFF 
    
    # 2. Access the Page Table Entry (PTE) using PTBR
    pte = PageTable[page_number]
    
    # 3. Check for Page Fault
    if not pte.present_bit:
        raise PageFaultException("Page not in RAM")
        
    # 4. Check Permissions (Protection)
    if current_operation == "WRITE" and not pte.read_write_bit:
        raise SegmentationFault("Access Denied: Read-only page")
        
    # 5. Handle Dirty Bit
    if current_operation == "WRITE":
        pte.dirty_bit = 1
        
    # 6. Mark as referenced
    pte.referenced_bit = 1
    
    # 7. Combine Frame Number with Offset
    physical_address = (pte.frame_number << 12) | offset
    
    return physical_address
```

---

## Why the Dirty Bit Matters

The dirty bit is a massive performance optimizer. Imagine a process that has 100 pages. If the OS needs to free up space and picks a page that hasn't been modified (Dirty Bit = 0), it simply overwrites that frame. It doesn't waste time writing "nothing" to the hard drive.


## Inverted Page Table

While multi-level page tables grow based on the size of the **Virtual Address Space**, an **Inverted Page Table** is designed to scale with the size of **Physical Memory (RAM)**.

In a standard system, every process has its own page table. If you have 100 processes, you have 100 tables. In an inverted system, there is only **one single page table** for the entire operating system.

---

### The Concept: One Table to Rule Them All

Instead of having an entry for every virtual page, an Inverted Page Table has one entry for every **physical frame** in RAM.

- **Entry 0** represents Physical Frame 0.
    
- **Entry 1** represents Physical Frame 1.
    
- **And so on...**
    

Each entry stores which **Process ID (PID)** and which **Virtual Page Number (VPN)** currently occupy that frame.

---

### How Address Translation Works

Since the table is sorted by physical frames, but the CPU provides a virtual address, the search is "inverted":

1. **Input:** The CPU provides a (PID, Virtual Page Number, Offset).
    
2. **The Search:** The OS must search the entire table to find the entry that matches both the PID and the VPN.
    
3. **The Result:** The **index (row number)** where the match is found is the **Physical Frame Number**.
    
4. **Final Address:** Physical Frame Number + Offset.
    

---

### The Problem: Searching is Slow

If you have 16 GB of RAM, you have millions of frames. Searching the table line-by-line for every memory access would be incredibly slow. To fix this, OSs use a **Hash Table**.

1. The (PID + VPN) is fed into a hash function.
    
2. The hash points to a specific slot in a **Hash Anchor Table**.
    
3. This speeds up the search from $O(n)$ to $O(1)$ on average.
    

---

### Pros and Cons

|**Feature**|**Inverted Page Table**|**Multi-Level Page Table**|
|---|---|---|
|**Memory Usage**|**Very Low.** Fixed size based on RAM.|**High.** Grows with the number of processes.|
|**Search Speed**|Slower (requires hashing/searching).|Faster (direct indexing).|
|**Scalability**|Great for 64-bit systems.|Can become complex (4-5 levels).|




