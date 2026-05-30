T <sub>(turnaround)</sub> = T<sub> (completion)</sub> − T <sub>(arrival)</sub>   
Turnaround time is one of the performance metric of scheduling policies.
### **1. First-Come, First-Served (FCFS)**

- **Definition:** The simplest scheduling algorithm where the process that arrives first is executed first. It is **non-preemptive**, meaning once a process starts, it runs to completion.
    
- **Key Characteristic:** Implemented using a FIFO (First-In, First-Out) queue.
    
- **Drawback:** Suffers from the "Convoy Effect," where short processes get stuck waiting behind a long process.
    

### **2. Shortest Job First (SJF)**

- **Definition:** A **non-preemptive** algorithm that selects the waiting process with the smallest execution time (burst time) to run next.
    
- **Key Characteristic:** It drastically reduces the average waiting time compared to FCFS.
    
- **Drawback:** It requires knowing the burst time in advance (which is often impossible) and can lead to starvation for long processes.
    

### **3. Shortest Time-to-Completion First (STCF) / Preemptive SJF (PSJF)**

- **Definition:** The **preemptive** version of SJF. If a new process arrives with a shorter remaining burst time than the currently running process, the scheduler pauses (preempts) the current process to run the new one.
    
- **Key Characteristic:** It is optimal for minimizing average turnaround time.
    
- **Drawback:** Like SJF, it requires knowledge of future burst times and requires frequent context switching overhead.

# Another Metric: Response Time:

  <h3>T<sub>response</sub> = T<sub>firstrun</sub> − T<sub>arrival</sub></h3>
  This is also very important to consider response time as the turnaround time.
### **4. Round Robin (RR)**

- **Definition:** A **preemptive** scheduling algorithm designed specifically for time-sharing systems. Each process is assigned a fixed time slot called a **Time Quantum** (or Time Slice).1
    
- **Mechanism:** The scheduler cycles through the ready queue.2 If a process exceeds its quantum, it is interrupted and moved to the back of the queue.3 If it finishes before the quantum ends, it leaves the system voluntarily.
    
    +1
    
- **Key Characteristic:** It prioritizes **fairness** and **responsiveness**.4 No process waits too long for CPU attention.5
    
    +1
    
- **Drawback:** Performance depends heavily on the size of the Time Quantum.
    
    - **Too small:** Excessive context switching overhead.6
        
    - **Too large:** Behaves like FCFS.7
        

---

### **Comparison of Scheduling Algorithms**

Here is how Round Robin stacks up against the ones we discussed earlier:

|**Feature**|**FCFS (First-Come, First-Served)**|**SJF (Shortest Job First)**|**STCF / PSJF (Shortest Time-to-Completion)**|**RR (Round Robin)**|
|---|---|---|---|---|
|**Type**|Non-Preemptive|Non-Preemptive|Preemptive|Preemptive|
|**Criteria**|Arrival Time|Burst Time (Total)|Remaining Burst Time|Time Quantum|
|**Throughput**|Low (due to Convoy Effect)|High|High|Medium (depends on quantum)|
|**Response Time**|Poor (bad for interactive systems)|Medium|Medium|**Excellent** (best for interactive systems)|
|**Starvation**|No|Yes (possible for long jobs)|Yes (possible for long jobs)|No|
|**Best For**|Batch systems|Minimizing Avg. Waiting Time|Minimizing Avg. Turnaround Time|Time-sharing / Interactive systems|

### **Summary of Trade-offs**

- **For Speed (Turnaround Time):** **STCF** is mathematically optimal. It finishes jobs as fast as possible but requires knowing the future (length of jobs).
    
- **For Fairness (Response Time):** **Round Robin** is the winner. Everyone gets a turn, making it ideal for systems where users interact with the computer (like your laptop).
    
- **For Simplicity:** **FCFS** is the easiest to implement but creates the worst user experience if a large job hogs the CPU.



For, all of the above algorithms. We can't say them perfect. As, one algorithm optimizes the turnaround time while not optimizing the response time and vise - versa.

# MLFQ (Multi-Level feedback Queue)

This algorithm contains several rules and more optimal and better than any of the above algorithms.
-> The refined set of MLFQ rules, spread throughout the chapter, are re-produced here for your viewing pleasure: 
• <b>Rule 1</b>:  If Priority(A) > Priority(B), A runs (B doesn’t).
• <b>Rule 2</b>:  If Priority(A) = Priority(B), A & B run in RR.
• <b>Rule 3</b>:  When a job enters the system, it is placed at the highest
priority (the topmost queue).
• <b>Rule 4</b>:  Once a job uses up its time allotment at a given level (re-
gardless of how many times it has given up the CPU), its priority is
reduced (i.e., it moves down one queue).
• <b>Rule 5</b>:  After some time period S, move all the jobs in the system
to the topmost queue.


# Ticket Scheduling

**Ticket Scheduling**, formally known as **Lottery Scheduling**, is a proportional-share scheduling algorithm. Instead of optimizing for turnaround time or fairness via strict queues (like Round Robin), it relies on randomness to distribute CPU time according to the importance of each process.

### How It Works: The Core Mechanism

The central concept is simple: **tickets represent a share of the resource.**

1. **Ticket Allocation:** Every process is assigned a certain number of tickets. The number of tickets represents the process's relative share of the CPU.
    
    - _Example:_ Process A has 75 tickets. Process B has 25 tickets. Total tickets = 100.
        
2. **The Lottery:** When the scheduler needs to pick the next process to run, it holds a "lottery." It generates a random number between 0 and the total number of tickets.
    
3. **The Winner:** The scheduler determines which process holds the winning ticket. That process gets the CPU for a time slice.
    

**Example Scenario:** Imagine the scheduler picks a random number **80**.

- Process A holds tickets 0 through 74.
    
- Process B holds tickets 75 through 99.
    
- Since 80 falls in Process B's range, **Process B runs.**
    

### Key Features of Ticket Scheduling

To make this system practical, the algorithm supports several ticket manipulations:

- **Ticket Currency:** A user can allocate tickets among their own jobs in their own "currency," which the OS then converts to the global ticket currency. This allows modular resource management.
    
- **Ticket Transfer:** A process can temporarily hand over its tickets to another process. This is useful in client-server setups; a client can give its tickets to the server to ensure the server runs quickly to answer the client's request.
    
- **Ticket Inflation:** A process can boost its own ticket count. This is generally only allowed among trusted processes to avoid a single process monopolizing the CPU.
    

---

### Why It Works

The effectiveness of Lottery Scheduling relies on probability and the **Law of Large Numbers**.

**1. Probabilistic Fairness** Over a long period, the percentage of time a process holds the CPU will naturally converge with the percentage of tickets it holds. While it may not be exact over a _short_ period (e.g., 10 seconds), over a _long_ period, Process A (with 75% of tickets) will receive almost exactly 75% of the CPU time.

**2. Extreme Simplicity** Most "fair" schedulers (like Stride Scheduling or Linux's CFS) require complex bookkeeping to track exactly how much time every process has used (virtual runtime). Lottery scheduling is stateless; it only needs to know how many tickets exist and how to generate a random number. This makes it very lightweight to implement.

**3. Immediate Responsiveness** Because the selection is random, a newly created process with tickets doesn't have to wait at the back of a queue. It immediately participates in the next lottery and has a chance to run instantly.

### The Downside

The main drawback is **short-term unfairness**. Because it is random, it is possible (though unlikely) for a high-priority process to lose the lottery 10 times in a row. This makes it unsuitable for real-time systems where strict timing guarantees are required.

 