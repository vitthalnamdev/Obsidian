A **Translation Lookaside Buffer (TLB)** is a specialized hardware cache used by the memory management unit (MMU) to speed up virtual-to-physical address translation. Without it, every memory access would require at least two trips to physical memory: one for the page table and one for the data itself, significantly slowing down the system.

+1

---

## How TLBs Make Paging Faster

In a standard paging system, the CPU generates a virtual address consisting of a **Page Number** and an **Offset**. To find the physical address, the OS must look up the page number in a **Page Table** stored in RAM.

+1

The TLB acts as a "shortcut." Because it is built using **SRAM** (Static RAM) and integrated directly into the CPU, it is much faster than the main DRAM.

1. **The Check:** When the CPU needs an address, it first checks the TLB.
    
2. **TLB Hit:** If the page number is found, the physical frame number is retrieved instantly. The system avoids the slow memory-intensive page table walk.
    
    +1
    
3. **TLB Miss:** If it’s not there, the MMU must perform the traditional page table walk in RAM. The result is then cached in the TLB for future use.
    
    +1
    

---

## The Structure of a TLB

A TLB is typically **fully associative** or **set-associative**, meaning the hardware can search all entries simultaneously (in parallel) to find a match. Each entry consists of two main parts:

- **Tag:** Usually the Virtual Page Number (VPN).
    
- **Data:** The corresponding Physical Frame Number (PFN) and protection bits (Read/Write/Execute, Dirty bit, Valid bit).
    

---

## Handling Context Switching: ASID vs. Flushing

When the OS switches from Process A to Process B (a context switch), the virtual addresses in the TLB for Process A are no longer valid for Process B. There are two ways to handle this:

### 1. TLB Flushing

The simplest method is to **flush (invalidate)** the entire TLB during every context switch.

- **Pros:** Simple to implement.
    
- **Cons:** High performance penalty. After a switch, the new process starts with a "cold" TLB and suffers frequent misses until the cache is refilled.
    

### 2. Address Space Identifiers (ASID)

Modern CPUs use an **ASID** (sometimes called a PID tag in the TLB). This is a unique 8-bit or 12-bit number assigned to each process by the OS.

- **How it works:** Each TLB entry is tagged with an ASID. When the CPU searches the TLB, it only considers entries where the `ASID` matches the current process's ID.
    
- **The Benefit:** The TLB can simultaneously hold translations for multiple processes. When you switch back to Process A, its entries are often still in the TLB, allowing for a "warm" start and much better performance.
    

---

## Performance Math: Effective Access Time (EAT)

To see the impact, we calculate the EAT. If memory access takes $100ns$ and a TLB lookup takes $1ns$, with a $99\%$ hit rate:

$$EAT = 0.99 \times (1ns + 100ns) + 0.01 \times (1ns + 100ns + 100ns)$$

$$EAT \approx 102ns$$

Without the TLB, every access would take $200ns$ (one for the table, one for the data). The TLB essentially cuts memory latency in half.

---


To understand how the hardware handles a memory request, we can look at the logic used by the Memory Management Unit (MMU). In a modern system, this process is incredibly fast, but the logic follows a specific "check-first" hierarchy.

Here is the pseudo-code representing a virtual address translation using a TLB:

### TLB Translation Logic

C++

```
// Input: VirtualAddress (VPN + Offset)
// Output: PhysicalAddress (PFN + Offset)

Function TranslateAddress(VirtualAddress):
    VPN = ExtractPageNumber(VirtualAddress)
    Offset = ExtractOffset(VirtualAddress)
    CurrentASID = GetCurrentProcessASID()

    // 1. Parallel Search in TLB
    TLBEntry = SearchTLB(VPN, CurrentASID)

    if (TLBEntry != NULL): 
        // --- TLB HIT ---
        if (CanAccess(TLBEntry, CurrentPermissions)):
            PFN = TLBEntry.FrameNumber
            Return Combine(PFN, Offset)
        else:
            Raise AccessViolationException()

    else:
        // --- TLB MISS ---
        // The hardware (or OS) must now walk the Page Table in RAM
        PTE = WalkPageTable(VPN) 

        if (PTE.Valid == False):
            // The page is not in RAM (Page Fault)
            Raise PageFaultException()
        
        else:
            // 2. Update TLB with the new translation for next time
            // If TLB is full, a replacement policy (like LRU) is used
            UpdateTLB(VPN, PTE.FrameNumber, CurrentASID)
            
            // 3. Retry the instruction
            Return TranslateAddress(VirtualAddress)
```

---

### Key Components in the Code

- **`SearchTLB(VPN, CurrentASID)`**: This happens in hardware. The MMU looks at all TLB entries simultaneously. It matches both the **Virtual Page Number** and the **ASID** to ensure Process A doesn't accidentally use Process B's translated address.
    
- **`UpdateTLB()`**: When a miss occurs, the new translation is cached. If the TLB is full, the hardware must decide which old entry to kick out.
    
- **`WalkPageTable()`**: This is the "expensive" part. It involves accessing the **Page Table Base Register (PTBR)** and reading from main memory, which is roughly 100x slower than a TLB hit.
    

---

### Hardware vs. Software Management

It is worth noting that while most modern architectures (like x86) handle the **TLB Miss** entirely in hardware, some architectures (like MIPS) raise a "TLB Miss Exception" and let the Operating System handle the page table walk in software.

 

 