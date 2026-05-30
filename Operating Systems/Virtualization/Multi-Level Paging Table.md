In a modern operating system, the address space can be massive (e.g., $2^{64}$ bytes in a 64-bit system). If we used a single-level page table, the table itself would be too large to fit in contiguous physical memory.

A **Multi-level Page Table** solves this by breaking the page table into smaller chunks, effectively creating a "page table for page tables." This allows the OS to only keep the necessary parts of the table in memory, saving significant space.

---

## How it Works

Think of it like a library's filing system:

1. **Level 1 (Page Directory):** A master list that points to smaller "sub-directories."
    
2. **Level 2 (Page Table):** These sub-directories point to the actual physical frames in RAM.
    

If a large block of virtual memory isn't being used, the OS simply doesn't create the corresponding Level 2 tables, which is where the memory savings come from.

### Address Translation Breakdown

In a 2-level system, a virtual address is typically split into three parts:

- **Directory Index:** To find the entry in the Level 1 table.
    
- **Table Index:** To find the entry in the Level 2 table.
    
- **Offset:** To find the specific byte within the physical page.
    

---

## Practical Example: 32-bit x86 Architecture

Let’s look at how a classic 32-bit system (like older versions of Linux or Windows) handles this.

- **Virtual Address Space:** $2^{32}$ bytes (4 GB).
    
- **Page Size:** 4 KB ($2^{12}$ bytes).
    
- **The Problem:** A single-level table would need $2^{20}$ entries. At 4 bytes per entry, that’s **4 MB** of contiguous RAM per process. If you have 100 processes, that's 400 MB just for tables!
    

### The 2-Level Solution:

The 32-bit address is split as **10-10-12**:

1. **Outer Page Directory (10 bits):** Points to one of 1,024 inner page tables.
    
2. **Inner Page Table (10 bits):** Points to one of 1,024 physical pages.
    
3. **Offset (12 bits):** Locates the byte within the 4 KB page.
    

**Why this is efficient:** If a process only uses 1 MB of memory, the OS only needs the 1 Master Directory and _one_ Level 2 table. Total memory used for the table: **8 KB** instead of 4 MB.

---

## Real-World OS Implementations

| **OS / Architecture**     | **Levels**        | **Reasoning**                                                                             |
| ------------------------- | ----------------- | ----------------------------------------------------------------------------------------- |
| **Linux (x86_64)**        | **4 or 5 Levels** | 64-bit spaces are so vast that 2 levels aren't enough to keep tables small.               |
| **Windows 11**            | **4 Levels**      | Uses "PML4" (Page Map Level 4) to manage the massive virtual address range.               |
| **macOS (Apple Silicon)** | **3 or 4 Levels** | Often uses 16 KB page sizes, which changes the bit-split but keeps the multi-level logic. |

To understand how this works in a **32-bit system** with a **2-level page table**, let’s walk through a concrete translation.

### The Configuration

- **Virtual Address:** $32 \text{ bits}$
    
- **Page Size:** $4 \text{ KB } (2^{12} \text{ bytes})$
    
- **Split:** **10-10-12**
    
    - **P1 (10 bits):** Index for the **Page Directory** (Level 1).
        
    - **P2 (10 bits):** Index for the **Page Table** (Level 2).
        
    - **Offset (12 bits):** The specific byte within the $4 \text{ KB}$ page.
        

---

### Step-by-Step Translation Example

Imagine a process wants to access data at the virtual address:

**`0x00403005`**

#### 1. Convert to Binary and Split

First, we break the 32-bit hex address into its three component parts:

`0000000001 | 0000000011 | 000000000101`

- **P1 (Directory Index):** `0000000001` $\rightarrow$ **Decimal 1**
    
- **P2 (Table Index):** `0000000011` $\rightarrow$ **Decimal 3**
    
- **Offset:** `000000000101` $\rightarrow$ **Decimal 5**
    

#### 2. Access the Page Directory (Level 1)

The CPU looks at the **Page Directory Base Register (CR3 in x86)** to find the start of the Page Directory in RAM. It then goes to **Entry #1** (from P1).

- **Result:** Entry #1 contains the address of the specific **Page Table** needed for this memory range. Let's say it points to Page Table "B" located at physical address `0x7000`.
    

#### 3. Access the Page Table (Level 2)

The CPU now goes to address `0x7000` and looks at **Entry #3** (from P2).

- **Result:** Entry #3 contains the **Page Frame Number (PFN)**. Let's say it tells us the data is in physical frame **`0x9000`**.
    

#### 4. Apply the Offset

Finally, we take the Physical Frame address (`0x9000`) and add our **Offset (5)**.

- **Physical Address:** `0x9000` + `0x005` = **`0x9005`**
    

---

### Why this is "Smart"

If our process only uses a tiny bit of memory, we only need **one** Page Directory and **one** Page Table. The other 1,023 entries in the Page Directory will be marked as "Invalid," and the OS won't bother creating those 1,023 Page Tables in RAM at all.

Since you're building **Neuro-DSM**, you could use a similar multi-level logic to map "Global Virtual Addresses" to specific nodes in your cluster.


How, the OS decided the size of memory allocation for a program  

When you execute a program, the OS doesn't just guess; it follows a blueprint defined in the **Executable File** (like an **ELF** file on Linux or **PE** on Windows).

Here is the breakdown of how the OS makes those decisions:

---

### 1. The Code and Data (Fixed Size)

The size of your **Code (Text)** and **Initialized Data** is determined at **compile-time**.

- The compiler calculates exactly how many bytes the instructions and global variables take.
    
- This information is stored in the **Program Header Table** of the executable.
    
- **The OS Action:** When you run `./my_program`, the OS reads this header and maps exactly that much memory into the virtual address space.
    

---

### 2. The Heap (Dynamic & Lazy)

The OS doesn't actually know how much **Heap** memory you'll need.

- **Initial Allocation:** The OS typically allocates a small, default "start" for the heap (often zero or a single page).
    
- **The OS Action:** It sets a pointer called the **Program Break (`brk`)**.
    
- **Growth:** As your program calls `malloc()` or `new` in C++, the library asks the OS to move the `brk` pointer higher or creates new memory mappings via `mmap()`. It grows on demand.
    

---

### 3. The Stack (Fixed Limit, Lazy Growth)

The **Stack** size is usually governed by an OS-level limit.

- **The OS Action:** On Linux, you can see this limit by typing `ulimit -s` (usually **8 MB**).
    
- **Initial Allocation:** The OS doesn't actually give the program 8 MB of physical RAM immediately. It reserves the **Virtual Address Space** at the top of the memory map.
    
- **Growth:** As your functions call each other and push data onto the stack, "Page Faults" occur. The OS sees these faults, realizes they are within the 8 MB "legal" stack zone, and provides physical pages one by one.
    

---

### Summary Table

|**Segment**|**Determined By**|**When?**|**Allocation Strategy**|
|---|---|---|---|
|**Code/Text**|Compiler/Linker|Compile-time|Fixed (Read from File)|
|**Data**|Global variables|Compile-time|Fixed (Read from File)|
|**Heap**|Program logic|Runtime|Dynamic (Grows via `brk`/`mmap`)|
|**Stack**|OS Defaults|Startup|Reserved Virtual Space (Lazy RAM)|


The number of levels in a page table is determined by a simple goal: **ensuring that a single page table entry fits exactly within one physical page.**

If your page table is too big to fit into a single 4 KB page, you must split it into another level.

---

### The Formula

To calculate the levels, you need to look at:

1. **Address Space:** How many bits is the virtual address? (e.g., 32-bit or 64-bit).
    
2. **Page Size:** How large is one page? (Standard is 4 KB or $2^{12}$ bytes).
    
3. **Entry Size:** How many bytes does one Page Table Entry (PTE) take? (Usually 4 bytes for 32-bit, 8 bytes for 64-bit).
    

---

### Step-by-Step Example: Why 64-bit needs 4 Levels

Let’s calculate for a standard **64-bit Linux** system.

#### 1. How many entries fit in one page?

- **Page Size:** 4 KB ($4096$ bytes).
    
- **PTE Size:** 8 bytes (required to store a 64-bit physical address).
    
- **Entries per page:** $4096 / 8 = 512$ entries.
    
- To address 512 entries, we need **9 bits** ($2^9 = 512$).
    

#### 2. How many bits are for the offset?

- A 4 KB page size always uses **12 bits** for the offset ($2^{12} = 4096$).
    

#### 3. Doing the Math (The "Bit Split")

In a 64-bit system, we usually only use 48 bits for virtual addresses (the rest are sign-extended).

- **Total Bits to map:** 48 bits.
    
- **Minus Offset:** $48 - 12 = 36$ bits left for the page tables.
    
- **Divide by Bits per Level:** Since each level handles 9 bits (as calculated in Step 1):
    
    $$36 \text{ bits} / 9 \text{ bits per level} = \mathbf{4 \text{ Levels}}$$
    

---

### Summary Table: Levels vs. Architecture

| **Architecture**    | **Bits Used** | **Page Size** | **Bits per Level** | **Resulting Levels**                    |
| ------------------- | ------------- | ------------- | ------------------ | --------------------------------------- |
| **x86 (32-bit)**    | 32            | 4 KB          | 10                 | **2 Levels** ($10+10+12$)               |
| **x86_64 (48-bit)** | 48            | 4 KB          | 9                  | **4 Levels** ($9+9+9+9+12$)             |
| **x86_64 (57-bit)** | 57            | 4 KB          | 9                  | **5 Levels** (Used in high-end servers) |