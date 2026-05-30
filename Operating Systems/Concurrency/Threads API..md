# 🧵 1. Thread Creation — What _really_ happens

When you call:

pthread_create(...)

👉 You are asking OS:

> “Create another execution flow inside the SAME process”

So:

- Same heap (shared)
- Same global variables
- Different stack

---

## 🔥 Detailed Example (with multiple threads)

```c
#include <stdio.h>  
#include <pthread.h>  
#include <unistd.h>  
  
void* worker(void* arg) {  
    int id = *(int*)arg;  
  
    for(int i = 0; i < 3; i++) {  
        printf("Thread %d running iteration %d\n", id, i);  
        sleep(1);  
    }  
  
    return NULL;  
}  
  
int main() {  
    pthread_t t1, t2;  
  
    int id1 = 1, id2 = 2;  
  
    pthread_create(&t1, NULL, worker, &id1);  
    pthread_create(&t2, NULL, worker, &id2);  
  
    pthread_join(t1, NULL);  
    pthread_join(t2, NULL);  
  
    printf("Main finished\n");  
}
```

---

### 🧠 What you'll observe:

Output will be **interleaved**:

Thread 1 running iteration 0  
Thread 2 running iteration 0  
Thread 1 running iteration 1  
Thread 2 running iteration 1  
...

👉 This is **true concurrency**

---

## ⚠️ Classic Bug (VERY IMPORTANT)

```c
for(int i = 0; i < 2; i++) {  
    pthread_create(&threads[i], NULL, worker, &i);  
}
```

❌ Problem:

- All threads get SAME address `&i`
- By the time thread runs → `i` changed

👉 Output becomes weird/unpredictable

---

## ✅ Fix

```c
int ids[2] = {1, 2};  
pthread_create(&t1, NULL, worker, &ids[0]);  
pthread_create(&t2, NULL, worker, &ids[1]);
```

---

# 🧵 2. pthread_join — Why it's critical

## ❌ Without join

```c
int main() {  
    pthread_t t;  
  
    pthread_create(&t, NULL, worker, NULL);  
  
    printf("Main exits\n");  
}
```

### 🚨 Possible output:

Main exits

👉 Thread never runs (process exits)

---

## ✅ With join

pthread_join(t, NULL);

👉 Guarantees:

- Thread completes
- Resources cleaned up

---

## 🧠 Deep Insight

There are **2 types of threads**:

- joinable (default)
- detached

---

## 🔥 Detached Thread Example

pthread_t t;  
pthread_create(&t, NULL, worker, NULL);  
pthread_detach(t);

👉 Now:

- Cannot `join`
- OS cleans automatically

---

# 🧵 3. Race Condition — The REAL Problem

---

## ❌ Broken Code

```c
int counter = 0;  
  
void* work(void* arg) {  
    for(int i = 0; i < 100000; i++) {  
        counter++;  
    }  
}
```

---

## 🧠 What actually happens internally

`counter++` is NOT atomic:

LOAD counter  
ADD 1  
STORE counter

👉 Two threads interleave:

Thread A: load 5  
Thread B: load 5  
Thread A: store 6  
Thread B: store 6   ❌ lost update

---

## 🔥 Demonstration Code

```c
#include <stdio.h>  
#include <pthread.h>  
  
int counter = 0;  
  
void* inc(void* arg) {  
    for(int i = 0; i < 1000000; i++) {  
        counter++;  
    }  
    return NULL;  
}  
  
int main() {  
    pthread_t t1, t2;  
  
    pthread_create(&t1, NULL, inc, NULL);  
    pthread_create(&t2, NULL, inc, NULL);  
  
    pthread_join(t1, NULL);  
    pthread_join(t2, NULL);  
  
    printf("Counter = %d\n", counter);  
}
```

👉 Expected: `2000000`  
👉 Actual: ❗ random (like 1329387)

---

# 🧵 4. Mutex — The Correct Fix

---

## ✅ Safe Version

```c
#include <stdio.h>  
#include <pthread.h>  
  
int counter = 0;  
pthread_mutex_t lock;  
  
void* inc(void* arg) {  
    for(int i = 0; i < 1000000; i++) {  
        pthread_mutex_lock(&lock);  
        counter++;  
        pthread_mutex_unlock(&lock);  
    }  
    return NULL;  
}  
  
int main() {  
    pthread_t t1, t2;  
  
    pthread_mutex_init(&lock, NULL);  
  
    pthread_create(&t1, NULL, inc, NULL);  
    pthread_create(&t2, NULL, inc, NULL);  
  
    pthread_join(t1, NULL);  
    pthread_join(t2, NULL);  
  
    printf("Counter = %d\n", counter);  
  
    pthread_mutex_destroy(&lock);  
}

```
---

## 🧠 Important Concepts

### 🔒 Critical Section

Code that must not run concurrently:

counter++;

---

## ⚠️ Deadlock Example (VERY IMPORTANT)

pthread_mutex_lock(&lock);  
pthread_mutex_lock(&lock); // ❌ deadlock

👉 Same thread trying to acquire lock twice

---

## ⚠️ Another Deadlock (2 locks)

Thread 1:  
lock(A)  
lock(B)  
  
Thread 2:  
lock(B)  
lock(A)

👉 Both stuck forever

---

# 🧵 5. Condition Variables — Deep Understanding

---

## 🔥 Problem: Busy Waiting (BAD)

while(ready == 0); // spinning

❌ wastes CPU

---

## ✅ Correct Approach

---

### Complete Example

```c
#include <stdio.h>  
#include <pthread.h>  
  
pthread_mutex_t lock = PTHREAD_MUTEX_INITIALIZER;  
pthread_cond_t cond = PTHREAD_COND_INITIALIZER;  
  
int ready = 0;  
  
void* consumer(void* arg) {  
    pthread_mutex_lock(&lock);  
  
    while (ready == 0) {  
        printf("Waiting...\n");  
        pthread_cond_wait(&cond, &lock);  
    }  
  
    printf("Consumed!\n");  
  
    pthread_mutex_unlock(&lock);  
    return NULL;  
}  
  
void* producer(void* arg) {  
    pthread_mutex_lock(&lock);  
  
    ready = 1;  
    printf("Produced!\n");  
  
    pthread_cond_signal(&cond);  
  
    pthread_mutex_unlock(&lock);  
    return NULL;  
}  
  
int main() {  
    pthread_t t1, t2;  
  
    pthread_create(&t1, NULL, consumer, NULL);  
    sleep(1);  
    pthread_create(&t2, NULL, producer, NULL);  
  
    pthread_join(t1, NULL);  
    pthread_join(t2, NULL);  
}

```
---

## 🧠 What happens internally

### Consumer:

1. locks mutex
2. sees `ready == 0`
3. sleeps → releases lock automatically

### Producer:

1. locks mutex
2. sets `ready = 1`
3. signals
4. unlocks

### Consumer:

1. wakes up
2. reacquires lock
3. continues

---

## ⚠️ Why `while`, not `if`

while (ready == 0)

👉 Because:

- spurious wakeups can happen
- signal is not guarantee of condition

