This is a foundational technique in Operating Systems used to provide **dynamic relocation** and memory protection. It is one of the earliest and simplest hardware-based methods for virtualizing memory.

In this model, the OS assumes that each process is loaded into a **contiguous** chunk of physical memory. To manage this, the CPU utilizes two special hardware registers known as the **Base** register and the **Bound** (or Limit) register.

### **1. How It Works**

When a program is running, it thinks it is accessing memory starting from address `0` (its virtual address space). However, the operating system has loaded the program at a completely different location in physical memory.

The hardware (Memory Management Unit or MMU) performs the translation on every memory reference:

- **The Base Register:** Stores the starting physical address where the process is loaded in memory.
    
- **The Bound Register:** Stores the size (or the limit) of the process's address space.
    

#### **Address Translation Formula**

When a process tries to access a virtual address ($VA$), the hardware automatically calculates the physical address ($PA$) using:

$$PA = VA + Base$$

#### **The Protection Check**

Before accessing the memory, the hardware must verify that the address is legal. It checks if the virtual address is within the bounds:

$$0 \le VA < Bound$$

- **If valid:** The CPU accesses the memory at $PA$.
    
- **If invalid:** The hardware raises an exception (often called a **Trap**), and the OS terminates the process (this is what happens when you see a "Segmentation Fault").
    

---

### **2. Example Scenario**

Imagine a process of size **4KB** is loaded into physical memory at address **16KB**.

- **Base Register:** 16384 (16KB)
    
- **Bound Register:** 4096 (4KB)
    

If the program tries to load data from virtual address **128**:

1. **Check:** Is $128 < 4096$? **Yes.**
    
2. **Translate:** $128 + 16384 = 16512$.
    
3. **Action:** The CPU fetches data from physical address **16512**.
    

If the program tries to load data from virtual address **5000**:

1. **Check:** Is $5000 < 4096$? **No.**
    
2. **Action:** Trap! The OS kills the process.
    

---

### **3. Merits (Advantages)**

- **Simplicity:** The hardware requirements are minimal (just two registers and simple adder/comparator logic).
    
- **Speed:** Address translation and protection checks happen in hardware, making them extremely fast with negligible overhead.
    
- **Protection (Isolation):** It ensures that a process cannot access memory outside its allocated region, preventing it from corrupting the OS or other processes.
    
- **Dynamic Relocation:** The OS can easily move a process's address space. If the OS needs to move a process to a different physical location (e.g., during compaction), it only needs to copy the data and update the **Base** register. The process itself doesn't need to know it moved.
    

### **4. Demerits (Disadvantages)**

- **External Fragmentation:** This is the biggest drawback. Since the process must be stored **contiguously** (in one solid block), the physical memory eventually becomes "Swiss cheese" with small holes of free space. You might have enough total free memory to run a program, but not in one contiguous block.
    
- **No Sharing:** It is difficult to share memory between processes (e.g., shared code libraries) because the mapping is a simple 1-to-1 addition. You cannot map the same physical code page to two different virtual address spaces easily.
    
- **Inflexible Growth:** The stack and heap are confined within the fixed "Bound." If a process needs to grow its stack or heap dynamically beyond the allocated chunk, it may fail if the adjacent physical memory is already occupied by another process.
    

### **Summary Table**

|**Feature**|**Description**|
|---|---|
|**Hardware**|Two registers (Base & Bound) per CPU|
|**Translation**|$Physical = Virtual + Base$|
|**Constraint**|Address space must be placed **contiguously** in physical memory|
|**Primary Issue**|**External Fragmentation** (free memory exists but is chopped up)|

This method paved the way for more complex virtualization techniques like **Segmentation** (which uses multiple base/bound pairs) and **Paging** (which removes the contiguous requirement entirely).