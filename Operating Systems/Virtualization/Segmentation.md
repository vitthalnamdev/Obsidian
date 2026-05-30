	**Segmentation** is a generalization of the Base and Bound method. Instead of having just _one_ base and bound pair for the entire process (which forces the whole address space to be contiguous), we have a base and bound pair for **each logical segment** of the address space: the **Code**, the **Heap**, and the **Stack**.
	
	This allows the OS to place each segment independently in physical memory, solving the "sparse address space" problem. You don't need to waste physical memory for the empty space between the Heap and the Stack.
	
	### **1. The Virtual Address Structure**
	
	In segmentation, the virtual address is split into two parts:
	
	1. **Segment ID (Selector):** The top few bits tell the hardware _which_ segment we are accessing (e.g., 00=Code, 01=Heap, 10=Stack).
	    
	2. **Offset:** The remaining lower bits tell the hardware _where_ inside that segment we are looking.
	    
	
	Example: Imagine a 14-bit virtual address where the top 2 bits are the Segment ID.
	
	$$\underbrace{10}_{\text{Seg ID}} \underbrace{001011010110}_{\text{Offset}}$$
	
	---
	
	### **2. Calculating the Offset**
	
	The offset is simply the virtual address without the segment bits. You extract it using bitwise operations.
	
	If the top $k$ bits are the Segment ID, the offset is the lower $N-k$ bits.
	
	- **Segment ID** = `(VirtualAddress & SEG_MASK) >> SHIFT_AMOUNT`
	    
	- **Offset** = `VirtualAddress & OFFSET_MASK`
	    
	
	**Example:**
	
	- **Virtual Address:** 4200 (Decimal) -> `01 0000 0110 1000` (Binary)
	    
	- **Top 2 bits:** `01` -> Segment 1 (Heap)
	    
	- **Lower 12 bits:** `0000 0110 1000` -> Offset 104
	    
	
	---
	
	### **3. Pseudo-Code Implementation**
	
	Since you are comfortable with C++, here is how the hardware logic would look in a simulation. Note the special handling for the **Stack**, which often grows _backwards_ (towards lower addresses).
	
	C++
	
	```
	// Hardware structures (The Segment Table)
	struct Segment {
	    uint32_t base;        // Physical starting address
	    uint32_t size;        // Limit (Bound)
	    bool grows_negative;  // 1 if segment grows down (like Stack), 0 otherwise
	    bool can_read;        // Protection bit
	    bool can_write;       // Protection bit
	};
	
	// Assuming 14-bit VA, top 2 bits for Seg ID
	const uint32_t SEG_MASK    = 0x3000; // Top 2 bits (1100...)
	const uint32_t OFFSET_MASK = 0x0FFF; // Bottom 12 bits
	const int SHIFT_AMOUNT     = 12;
	
	uint32_t translate_address(uint32_t virtual_addr, Segment seg_table[]) {
	    // 1. Extract Segment ID and Offset
	    uint32_t seg_id = (virtual_addr & SEG_MASK) >> SHIFT_AMOUNT;
	    uint32_t offset = virtual_addr & OFFSET_MASK;
	
	    // 2. Look up the specific segment hardware register
	    Segment current_seg = seg_table[seg_id];
	
	    // 3. Bounds Check (Protection)
	    if (current_seg.grows_negative) {
	        // For stack: Hardware checks if offset is within the valid "negative" range
	        // In some architectures, the offset is relative to the TOP of the segment
	        // If offset is too "low" (conceptually), it's a fault.
	        if (offset < current_seg.size) { 
	            raise_trap("Segmentation Fault (Stack Overflow)");
	        }
	    } else {
	        // For Code/Heap: Standard check
	        if (offset >= current_seg.size) {
	            raise_trap("Segmentation Fault (Out of Bounds)");
	        }
	    }
	
	    // 4. Permission Check (Optional but standard)
	    // if (writing && !current_seg.can_write) raise_trap("Protection Fault");
	
	    // 5. Calculate Physical Address
	    // Note: hardware simply adds the Base to the Offset
	    uint32_t phys_addr = current_seg.base + offset;
	
	    return phys_addr;
	}
	```
	
	### **4. Detailed Example: The "Negative Growth" Stack**
	
	The Stack is tricky because it starts at high addresses and grows down.
	
	- **Virtual World:** The stack might start at virtual address `16KB` and grow down to `12KB`.
	    
	- **Physical World:** The OS allocates physical memory for it at address `28KB`.
	    
	
	If the hardware supports "Negative Growth":
	
	1. **Base Register:** Set to the physical address of the **highest** virtual address (e.g., `28KB`).
	    
	2. **Offset Calculation:** The hardware might treat the offset as a negative number (using 2's complement) or simply use the logic:
	    
	    $$Physical = Base + Offset$$
	    
	    _Wait, if the stack grows down, the offset must be negative relative to the base?_
	    
	    **Correct Approach:** In many implementations (like x86), the Base points to the _bottom_ (lowest address) of the segment in physical memory. The limit specifies the size. The hardware checks that the address hits valid bytes.
	    
	    - **Alternative Approach (OSTEP):** If `GrowsNegative == 1`, the hardware calculates the offset from the _top_ of the address space.
	        
	    - $Offset = VA - (Max Segment Size)$
	        
	    - $Physical = Base + Offset$ (Where Offset is a negative number).
	        
	
	### **Merits vs. Demerits of Segmentation**
	
	| **Merits**                                                                                                                      | **Demerits**                                                                                                                 |     |
	| ------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------- | --- |
	| **No Internal Fragmentation:** You allocate exactly what the code/stack needs.                                                  | **External Fragmentation:** Variable-sized segments leave "holes" in physical memory that are too small to use.              |     |
	| **Sharing:** You can easily share the Code segment between processes by pointing their "Code Base" to the same physical memory. | **Compaction is Slow:** To fix the holes (fragmentation), the OS has to copy segments around memory, which is CPU intensive. |     |
	| **Dynamic Stack:** The stack can grow independently.                                                                            |                                                                                                                              |     |
	
		Take a look at internet, at how to calculate the offset from the virtual          address.