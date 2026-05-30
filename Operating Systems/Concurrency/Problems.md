## 📌 What is a Thread?

> A thread is the **smallest unit of execution** scheduled by the OS.

It consists of:

- **Program Counter (PC)** → next instruction
- **Registers** → current computation state
- **Stack** → local variables & function calls
- **Thread metadata** → ID, state, scheduling info

---

## ⚙️ How `pthread_create` Works

- `pthread_create()` is a **user-level wrapper**
- Internally calls Linux syscall: **`clone()`**

### 🔧 `clone()` flags (for threads)

- `CLONE_VM` → shared memory
- `CLONE_FILES` → shared file descriptors
- `CLONE_SIGHAND` → shared signal handlers
- `CLONE_THREAD` → same process

### 🧠 Flow

1. Allocate stack
2. Create thread control block
3. Call `clone()`
4. Start execution of new thread

👉 Threads in Linux = **processes sharing resources**

---

## 🧱 Stack Allocation

- Each thread has its **own stack**
- Allocated using `mmap()`
- Default size ≈ **8 MB**

### 📌 Memory Layout

Stack (per thread) ↓  
Heap (shared)      ↑  
Data (shared)  
Code (shared)

---

## 🔀 Kernel vs User Threads

### 🧵 Kernel Threads (KLT)

- Managed by OS
- True parallelism
- Higher overhead

### 🧵 User Threads (ULT)

- Managed in user space
- Faster switching
- No real parallelism

### 🧠 Models

|Model|Description|
|---|---|
|1:1|One user thread → one kernel thread (Linux)|
|M:1|Many user threads → one kernel thread|
|M:N|Many user threads → many kernel threads|

👉 Linux uses **1:1 model**

---

## 🖥️ Mapping to CPU & Hyperthreading

### 🔹 CPU Basics

- Multiple **cores**
- Each core runs 1–2 threads (with hyperthreading)

### 🔹 Scheduling

- OS maps threads → CPU cores
- Extra threads wait in queue

### 🔹 Hyperthreading (SMT)

- One core runs **2 threads**
- Improves utilization (not true doubling)

---

## 🔄 Context Switching

When switching threads:

- Save:
    - PC
    - Registers
    - Stack pointer
- Load next thread state

👉 Enables multitasking

---

## ⚠️ Concurrency Problem (VERY IMPORTANT)

When threads share data → **race conditions**

From your notes:

- Multiple threads updating shared variable → incorrect result
- Output becomes **indeterminate** (changes every run)

### 🔑 Key Terms

- **Critical Section** → code accessing shared data
- **Race Condition** → timing-dependent bug
- **Mutual Exclusion** → only one thread executes critical section

---

## 🧠 Final Mental Model

> Threads = **independent execution contexts**  
> sharing memory but having **separate stacks and registers**,  
> created via `clone()`, scheduled by OS,  
> and mapped dynamically onto CPU cores.

---

## 🚀 One-line takeaway

> A thread is **PC + registers + stack**, scheduled by the OS, sharing process memory, and executing concurrently with other threads.


# ⚠️ Race Condition Example (C with pthreads)  
  
## 🧪 Code  Example.....

```c
#include <stdio.h>  
#include <pthread.h>  
  
#define NUM_ITER 1000000  
  
int counter = 0; // shared variable  
  
void* worker(void* arg) {  
    for (int i = 0; i < NUM_ITER; i++) {  
       counter++; // NOT thread-safe  
    }  
    return NULL;  
}  
  
int main() {  
pthread_t t1, t2;  
  
pthread_create(&t1, NULL, worker, NULL);  
pthread_create(&t2, NULL, worker, NULL);  
  
pthread_join(t1, NULL);  
pthread_join(t2, NULL);  
  
printf("Final counter value: %d\n", counter);  
return 0;  
}
```
  
---

## 🎯 Expected Output

Since:

- Thread 1 increments → 1,000,000 times
- Thread 2 increments → 1,000,000 times

👉 Expected:

Final counter value: 2000000

---

## ❌ Actual Output (Race Condition)

Different runs may produce different results:

Final counter value: 1783452

Final counter value: 1932104

Final counter value: 2000000   (sometimes correct by luck)

---

## 🧠 Why This Happens

The operation:

counter++;

is NOT atomic. It expands to:

1. Load counter from memory → register  
2. Increment register  
3. Store register back to memory

This will create problem , because if there is a context switch after step 2 , but before step-3. Then, think about it there will be problems.... Because each threads has its own set of registers..

---

### 🔄 Interleaving Problem

Example:

- Thread 1 reads `counter = 50`
- Thread 2 reads `counter = 50`
	- Thread 1 writes `51`
- Thread 2 writes `51`

👉 Final value = **51 instead of 52**

---

## 🔑 Key Insight

> Multiple threads entering a **critical section** without synchronization → **race condition**





