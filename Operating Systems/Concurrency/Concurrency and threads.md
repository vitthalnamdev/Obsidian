## Introduction

Thus far, we have seen the development of the basic abstractions that the OS performs.  
We have seen how to take a single physical CPU and turn it into multiple virtual CPUs,  
thus enabling the illusion of multiple programs running at the same time.

We have also seen how to create the illusion of a large, private virtual memory for each process.  
This abstraction of the address space enables each program to behave as if it has its own memory,  
while the OS multiplexes address spaces across physical memory (and sometimes disk).

In this note, we introduce a new abstraction for a single running process: **threads**.

---

## What is a Thread?

Instead of a single point of execution within a program (a single program counter),  
a multi-threaded program has multiple points of execution.

Each thread:
- Has its own program counter (PC)
- Has its own register set
- Shares the same address space with other threads

Threads are similar to processes, except:
- They share memory
- Context switching is lighter (no page table switch)

```c
struct Thread {
    Registers regs;
    void* stack;
    State state; // running, waiting, etc.
};
```

---

## Thread State

Each thread has:
- Program Counter (PC)
- Registers
- Stack

Thread state is stored in a **Thread Control Block (TCB)**.

---

## Stack Behavior

### Single-threaded process:
- One stack exists

### Multi-threaded process:
- Each thread has its own stack
- Stack contains:
  - Local variables
  - Function arguments
  - Return values

---

## Thread Creation Example

### Code Example

```c
#include <stdio.h>
#include <assert.h>
#include <pthread.h>

void *mythread(void *arg) {
    printf("%s\n", (char *) arg);
    return NULL;
}

int main(int argc, char *argv[]) {
    pthread_t p1, p2;
    int rc;

    printf("main: begin\n");

    rc = pthread_create(&p1, NULL, mythread, "A");
    assert(rc == 0);

    rc = pthread_create(&p2, NULL, mythread, "B");
    assert(rc == 0);

    rc = pthread_join(p1, NULL);
    assert(rc == 0);

    rc = pthread_join(p2, NULL);
    assert(rc == 0);

    printf("main: end\n");
    return 0;
}